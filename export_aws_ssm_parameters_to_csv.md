## Automated Export of AWS SSM Parameters to CSV

```bash
#!/bin/bash

# Set your AWS region (dummy value for public repo)
REGION="ap-south-1"

# Output file
OUTPUT_FILE="ssm_parameters.csv"

# Write CSV header
echo "Name,Value" > $OUTPUT_FILE

# Get all parameter names
aws ssm describe-parameters --region $REGION \
  --query "Parameters[].Name" --output text | tr '\t' '\n' | while read NAME; do
    # Fetch the value (with decryption for SecureString)
    VALUE=$(aws ssm get-parameter --name "$NAME" --with-decryption --region $REGION \
             --query "Parameter.Value" --output text)
    # Append to CSV
    echo "\"$NAME\",\"$VALUE\"" >> $OUTPUT_FILE
done

echo "Export complete. Saved to $OUTPUT_FILE"
```

---

## Code Explanation

1. **Region setup**  
   - Defines the AWS region (`ap-south-1` in this example).  
   - Ensures all AWS CLI commands run against the correct region.

2. **Output file initialization**  
   - Creates a CSV file (`ssm_parameters.csv`).  
   - Writes the header row (`Name,Value`) to make the file structured.

3. **List parameter names**  
   - Uses `aws ssm describe-parameters` to fetch all parameter names in the region.  
   - Formats the output into a line‑by‑line list using `tr`.

4. **Loop through parameters**  
   - Iterates over each parameter name.  
   - For each name, retrieves its value using `aws ssm get-parameter`.  
   - Includes `--with-decryption` so that `SecureString` parameters are decrypted before export.

5. **Write to CSV**  
   - Appends each parameter name and value pair to the CSV file in `"Name","Value"` format.  
   - Ensures the file can be easily opened in Excel or other tools.

6. **Completion message**  
   - Prints a final message confirming the export and the location of the CSV file.

---

## Usage

### Prerequisites
- AWS CLI installed and configured (`aws configure`).  
- IAM permissions to access SSM parameters (`ssm:DescribeParameters` and `ssm:GetParameter`).  

### Run Command
```bash
sh export_ssm.sh
```

### Output
- A file named `ssm_parameters.csv` will be created in the current directory.  
- Example contents:
  ```
  Name,Value
  /demo/app/db_password,mySecret123
  /demo/app/api_key,abcd-efgh-ijkl
  ```

---
