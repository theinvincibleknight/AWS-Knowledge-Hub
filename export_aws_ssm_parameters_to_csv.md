## Automated Export of AWS SSM Parameters to CSV

```bash
#!/bin/bash
REGION="ap-south-1"
OUTPUT_FILE="ssm_parameters.csv"

echo "Name,Value" > $OUTPUT_FILE

aws ssm get-parameters-by-path \
  --path "/" \
  --recursive \
  --with-decryption \
  --region $REGION \
  --query "Parameters[*].[Name,Value]" \
  --output text | while read NAME VALUE; do
    echo "\"$NAME\",\"$VALUE\"" >> $OUTPUT_FILE
done

echo "Export complete. Saved to $OUTPUT_FILE"
```

---

### 📝 Script Explanation

1. **Define variables**
   ```bash
   REGION="ap-south-1"
   OUTPUT_FILE="ssm_parameters.csv"
   ```
   - `REGION` sets the AWS region where your SSM parameters are stored.
   - `OUTPUT_FILE` is the name of the CSV file where results will be saved.

2. **Write CSV header**
   ```bash
   echo "Name,Value" > $OUTPUT_FILE
   ```
   - Creates the output file and writes the header row (`Name,Value`).

3. **Fetch parameters in bulk**
   ```bash
   aws ssm get-parameters-by-path \
     --path "/" \
     --recursive \
     --with-decryption \
     --region $REGION \
     --query "Parameters[*].[Name,Value]" \
     --output text
   ```
   - `get-parameters-by-path` retrieves all parameters under the root path `/`.
   - `--recursive` ensures it fetches parameters from all nested paths.
   - `--with-decryption` decrypts values if they are stored as `SecureString`.
   - `--query "Parameters[*].[Name,Value]"` extracts only the name and value fields.
   - `--output text` formats the result as plain text (tab-separated).

4. **Process each parameter**
   ```bash
   | while read NAME VALUE; do
       echo "\"$NAME\",\"$VALUE\"" >> $OUTPUT_FILE
     done
   ```
   - The output from AWS CLI is piped into a `while read` loop.
   - Each line contains a parameter name and its value.
   - The script writes them into the CSV file, wrapping each field in quotes for safety.

5. **Completion message**
   ```bash
   echo "Export complete. Saved to $OUTPUT_FILE"
   ```
   - Prints a confirmation message once all parameters are exported.

---


- Instead of making **2500+ individual API calls** (`get-parameter` per name), this script uses **bulk retrieval** (`get-parameters-by-path`), which drastically reduces API calls and avoids throttling.
- The loop simply formats the bulk output into CSV.

---

### 📊 Example Output
Your CSV will look like this:

```
Name,Value
"/app/db/username","admin"
"/app/db/password","secret123"
"/service/api/key","abcd-efgh-ijkl"
```

---
