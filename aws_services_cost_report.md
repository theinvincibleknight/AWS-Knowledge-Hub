# AWS Services Cost Report Script

This Python script automatically generates a **CSV report of all AWS services used in your account for the last full month**, along with the associated costs. The output file is named with the month and year for clarity, e.g., `aws_services_January_2026.csv`.

---

## 📌 Purpose

Managing AWS costs can be challenging when multiple services are in use. While the AWS Billing Console and Cost Explorer provide visibility, this script automates the process of:

- Fetching all AWS services that incurred costs in the last full month.
- Summarizing the **UnblendedCost** (raw cost before discounts/credits).
- Saving results into a **CSV file** with service names and costs.
- Naming the file with the month and year for easy identification.

This makes it simple to track monthly service usage and costs, and integrate the data into other reporting or automation workflows.

---

## ⚙️ How It Works

1. **Date Calculation**
   - The script automatically determines the **last full month**.
   - Example: If today is February 24, 2026 → the report will cover **January 1–31, 2026**.

2. **Cost Explorer Query**
   - Uses the AWS CLI `ce get-cost-and-usage` command.
   - Groups results by `SERVICE`.
   - Retrieves the `UnblendedCost` metric for each service.

3. **Data Processing**
   - Deduplicates services (summing costs if multiple entries exist).
   - Sorts services alphabetically.
   - Adds a **Total row** at the bottom of the CSV.

4. **Output**
   - Saves results into a CSV file named:
     ```
     aws_services_<Month>_<Year>.csv
     ```
   - Example:
     ```
     aws_services_January_2026.csv
     ```

---

## 📄 Script

`extract_services_last_month.py`

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

## 🚀 Usage

1. **Install dependencies**
   ```bash
   pip install boto3
   ```

2. **Configure AWS CLI**
   Ensure your AWS CLI is configured with credentials that have access to **Cost Explorer**:
   ```bash
   aws configure
   ```

3. **Enable Cost Explorer**
   Cost Explorer must be enabled in your AWS account (via the Billing Console).

4. **Run the script**
   ```bash
   python extract_services_last_month.py
   ```

5. **Check the output**
   A CSV file will be generated in the same directory, e.g.:
   ```
   aws_services_January_2026.csv
   ```

---

## 📊 Example Output

`aws_services_January_2026.csv`:

```
Service,Cost (USD)
Amazon EC2,123.45
Amazon S3,45.67
AWS Lambda,12.34
AWS Backup,5.00

Total,186.46
```

---

## 📝 Notes
- **Costs shown are UnblendedCost** (raw usage cost before credits/discounts).
- **Free services** (like IAM) may not appear since they don’t incur costs.
- You can modify the script to use other metrics (e.g., `BlendedCost` or `UsageQuantity`) if needed.
- The script currently reports **last full month only**. It can be extended to generate reports for multiple months in one run.

---
