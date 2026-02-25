# AWS Services Cost Extractor

This Python script automates the process of retrieving **AWS service costs for the last full month** using the AWS Cost Explorer API (via AWS CLI). It outputs the results into a CSV file named with the month and year, making it easy to track monthly spending across services.

---

## 📋 Prerequisites

1. **AWS CLI installed and configured**  
   ```bash
   aws configure
   ```
   Ensure your credentials have access to AWS Cost Explorer.

2. **Enable Cost Explorer** in your AWS account (via the Billing Console).

3. **Python 3.7+** installed.

---

## Script

`extract_services.py`

```python
import json
import subprocess
import datetime
import csv

def get_last_month_period():
    today = datetime.date.today()
    first_day_this_month = today.replace(day=1)
    last_day_last_month = first_day_this_month - datetime.timedelta(days=1)
    start_date = last_day_last_month.replace(day=1).strftime("%Y-%m-%d")
    end_date = last_day_last_month.strftime("%Y-%m-%d")
    # Month name and year for file naming
    month_name = last_day_last_month.strftime("%B")
    year = last_day_last_month.strftime("%Y")
    return start_date, end_date, month_name, year

def get_services_with_cost():
    start_date, end_date, month_name, year = get_last_month_period()
    output_file = f"aws_services_{month_name}_{year}.csv"

    # Run AWS CLI command to fetch cost explorer data
    cmd = [
        "aws", "ce", "get-cost-and-usage",
        "--time-period", f"Start={start_date},End={end_date}",
        "--granularity", "MONTHLY",
        "--metrics", "UnblendedCost",
        "--group-by", "Type=DIMENSION,Key=SERVICE"
    ]
    result = subprocess.run(cmd, capture_output=True, text=True)

    if result.returncode != 0:
        print("Error running AWS CLI:", result.stderr)
        return

    data = json.loads(result.stdout)
    services = []
    for time_block in data.get("ResultsByTime", []):
        for group in time_block.get("Groups", []):
            keys = group.get("Keys", [])
            if keys:
                service_name = keys[0]
                amount = group["Metrics"]["UnblendedCost"]["Amount"]
                services.append((service_name, float(amount)))

    # Deduplicate by service (sum costs if multiple entries)
    service_costs = {}
    for svc, amt in services:
        service_costs[svc] = service_costs.get(svc, 0.0) + amt

    # Save to CSV
    with open(output_file, "w", newline="") as f:
        writer = csv.writer(f)
        writer.writerow(["Service", "Cost (USD)"])
        total_cost = 0.0
        for svc, amt in sorted(service_costs.items()):
            writer.writerow([svc, f"{amt:.2f}"])
            total_cost += amt
        # Add total row at the bottom
        writer.writerow([])
        writer.writerow(["Total", f"{total_cost:.2f}"])

    print(f"Saved {len(service_costs)} services with costs to {output_file}")
    print(f"Period: {start_date} → {end_date}")

if __name__ == "__main__":
    get_services_with_cost()
```

---

## 📝 Script Explanation

### 1. Date Calculation
```python
def get_last_month_period():
    ...
```
- Finds the first day of the current month.
- Subtracts one day → last day of the previous month.
- Returns start date, end date, month name, and year.

---

### 2. Fetching Costs
```python
cmd = [
    "aws", "ce", "get-cost-and-usage",
    "--time-period", f"Start={start_date},End={end_date}",
    "--granularity", "MONTHLY",
    "--metrics", "UnblendedCost",
    "--group-by", "Type=DIMENSION,Key=SERVICE"
]
```
- Runs AWS CLI Cost Explorer command.
- Groups results by **SERVICE**.
- Retrieves **UnblendedCost** (raw usage cost before credits/discounts).

---

### 3. Parsing Results
```python
data = json.loads(result.stdout)
...
services.append((service_name, float(amount)))
```
- Parses JSON output.
- Extracts service names and costs.
- Stores them in a list.

---

### 4. Deduplication
```python
service_costs = {}
for svc, amt in services:
    service_costs[svc] = service_costs.get(svc, 0.0) + amt
```
- If a service appears multiple times, sums up its costs.

---

### 5. Writing to CSV
```python
with open(output_file, "w", newline="") as f:
    writer.writerow(["Service", "Cost (USD)"])
    ...
    writer.writerow(["Total", f"{total_cost:.2f}"])
```
- Writes service costs into a CSV file.
- Adds a **Total row** at the bottom.

---

### 6. Output
```python
print(f"Saved {len(service_costs)} services with costs to {output_file}")
print(f"Period: {start_date} → {end_date}")
```
- Confirms how many services were saved.
- Displays the reporting period.

---
## 🚀 Usage

Run the script:

```bash
python3 extract_services.py
```

The script will:
- Automatically calculate the **last full month**.
- Fetch costs grouped by **AWS service**.
- Save results into a CSV file named:
  ```
  aws_services_<Month>_<Year>.csv
  ```

Example:
```
aws_services_January_2026.csv
```

---

## 📊 Example Output

```
Service,Cost (USD)
Amazon EC2,123.45
Amazon S3,45.67
AWS Lambda,12.34
AWS Backup,5.00

Total,186.46
```

---

## ✅ Summary
- **Automates monthly AWS cost reporting.**
- **Generates CSV files named by month/year.**
- **Provides per-service breakdown and total cost.**

---
