## Automated Export of AWS ECS clusters and services

```bash
#!/bin/bash
REGION="ap-south-1"
OUTPUT_FILE="ecs_clusters_services.csv"

# Write CSV header
echo "ClusterName,ServiceName" > $OUTPUT_FILE

# Get all ECS cluster ARNs
aws ecs list-clusters --region $REGION --query "clusterArns[]" --output text | tr '\t' '\n' | while read CLUSTER_ARN; do
  # Use the full ARN when calling list-services (ECS accepts ARN or name)
  CLUSTER_NAME=$(basename "$CLUSTER_ARN")

  # Get all service ARNs in the cluster
  aws ecs list-services --cluster "$CLUSTER_ARN" --region $REGION --query "serviceArns[]" --output text | tr '\t' '\n' | while read SERVICE_ARN; do
    SERVICE_NAME=$(basename "$SERVICE_ARN")
    echo "\"$CLUSTER_NAME\",\"$SERVICE_NAME\"" >> $OUTPUT_FILE
  done
done

echo "Export complete. Saved to $OUTPUT_FILE"

```

### 📝 Script Explanation

1. **Set variables**
   ```bash
   REGION="ap-south-1"
   OUTPUT_FILE="ecs_clusters_services.csv"
   ```
   - Defines the AWS region where ECS is running.
   - Sets the output CSV file name.

2. **Write CSV header**
   ```bash
   echo "ClusterName,ServiceName" > $OUTPUT_FILE
   ```
   - Creates the CSV file and writes the header row.

3. **List ECS clusters**
   ```bash
   aws ecs list-clusters --region $REGION --query "clusterArns[]" --output text
   ```
   - Retrieves all ECS cluster ARNs in the region.
   - `--query "clusterArns[]"` ensures only the cluster ARNs are returned.
   - `--output text` formats them as plain text (tab-separated).

4. **Loop through clusters**
   ```bash
   | tr '\t' '\n' | while read CLUSTER_ARN; do
       CLUSTER_NAME=$(basename "$CLUSTER_ARN")
   ```
   - Converts tab-separated ARNs into line-separated values.
   - For each cluster ARN, extracts the cluster name (last part of the ARN).

5. **List services in each cluster**
   ```bash
   aws ecs list-services --cluster "$CLUSTER_ARN" --region $REGION --query "serviceArns[]" --output text
   ```
   - Retrieves all service ARNs inside the given cluster.
   - Again, outputs them as plain text.

6. **Loop through services**
   ```bash
   | tr '\t' '\n' | while read SERVICE_ARN; do
       SERVICE_NAME=$(basename "$SERVICE_ARN")
       echo "\"$CLUSTER_NAME\",\"$SERVICE_NAME\"" >> $OUTPUT_FILE
   done
   ```
   - Converts service ARNs into line-separated values.
   - Extracts the service name (last part of the ARN).
   - Appends a row to the CSV with cluster name and service name.

7. **Completion message**
   ```bash
   echo "Export complete. Saved to $OUTPUT_FILE"
   ```
   - Prints a confirmation once all clusters and services are exported.

---

### 📊 Example Output
Your CSV will look like:

```
ClusterName,ServiceName
my-cluster,web-service
my-cluster,api-service
another-cluster,worker-service
```
