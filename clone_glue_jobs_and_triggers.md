# AWS Glue Job & Trigger Cloning Script

This is a Python script that automates the cloning of AWS Glue jobs and their associated triggers. It is useful when you need to duplicate jobs under new names while retaining their configurations, parameters, and tags.

---

## 📌 Purpose

AWS Glue does not provide a native "rename" function for jobs. If you want to change job names or replicate jobs across environments, you must recreate them. Doing this manually for many jobs is error-prone and time-consuming.  

This script solves that problem by:
- Fetching the configuration of existing Glue jobs.
- Creating new jobs with updated names.
- Retaining all job properties (script path, DPUs/worker settings, parameters, connections, etc.).
- Fetching and applying **tags** from the original job.
- Cloning associated **triggers** (schedules), pointing them to the new job.

---

## ⚙️ How It Works

### 1. Job Cloning
- Uses `glue.get_job()` to fetch the job definition.
- Builds a new job configuration dynamically.
- Ensures **mutually exclusive capacity settings**:
  - If the job uses `WorkerType` + `NumberOfWorkers`, those are preserved.
  - Otherwise, `MaxCapacity` is used.
- Fetches **tags** separately using `glue.get_tags()` and applies them to the new job.
- Calls `glue.create_job()` to create the cloned job.

### 2. Trigger Cloning
- Uses `glue.get_triggers()` to list all triggers.
- Identifies triggers that reference the old job.
- Creates new triggers pointing to the new job.
- Ensures **unique trigger names** by appending the new job name.
- Preserves trigger state (`ENABLED` or `DISABLED`) in logs.  
  *(Note: In the AWS console, cloned triggers may initially appear in "Created" state — this can be adjusted later.)*

### 3. Job Mapping
- Job name mappings are stored in a text file (`glue_jobs.txt`).
- Each line maps an old job name to a new one:
  ```
  old-job-1:new-job-1
  old-job-2:new-job-2
  ```
- The script reads this file and clones jobs accordingly.

---

## 📄 Script

`clone_glue_jobs.py`

```python
import boto3

glue = boto3.client('glue')
sts = boto3.client('sts')

# Get account ID once
account_id = sts.get_caller_identity()["Account"]

def clone_glue_job(old_job_name, new_job_name):
    job_def = glue.get_job(JobName=old_job_name)['Job']

    # Build ARN manually to fetch tags
    region = glue.meta.region_name
    arn = f"arn:aws:glue:{region}:{account_id}:job/{old_job_name}"
    tags = glue.get_tags(ResourceArn=arn).get('Tags', {})

    # Remove fields that cannot be reused directly
    for field in ['Name', 'CreatedOn', 'LastModifiedOn']:
        job_def.pop(field, None)

    # Base required fields
    params = {
        "Name": new_job_name,
        "Role": job_def['Role'],
        "Command": job_def['Command'],
        "Tags": tags,  # ensure tags are retained
    }

    # Add optional fields only if present and non-empty
    optional_fields = [
        "DefaultArguments", "Connections", "MaxRetries", "Timeout",
        "GlueVersion", "Description", "ExecutionProperty",
        "NotificationProperty"
    ]
    for field in optional_fields:
        if field in job_def and job_def[field] not in (None, {}, [], ''):
            params[field] = job_def[field]

    # Handle capacity settings safely
    if job_def.get("WorkerType") and job_def.get("NumberOfWorkers"):
        params["WorkerType"] = job_def["WorkerType"]
        params["NumberOfWorkers"] = job_def["NumberOfWorkers"]
    elif job_def.get("MaxCapacity"):
        params["MaxCapacity"] = job_def["MaxCapacity"]

    glue.create_job(**params)
    print(f"Cloned job {old_job_name} → {new_job_name} (tags retained)")


def clone_triggers(old_job_name, new_job_name):
    triggers = glue.get_triggers()['Triggers']
    for trigger in triggers:
        actions = trigger.get('Actions', [])
        # Check if this trigger references the old job
        if any(action.get('JobName') == old_job_name for action in actions):
            # Use the new job name as the trigger name
            new_trigger_name = new_job_name

            # Update actions to point to new job
            new_actions = []
            for action in actions:
                new_action = action.copy()
                if new_action.get('JobName') == old_job_name:
                    new_action['JobName'] = new_job_name
                new_actions.append(new_action)

            # Base required fields
            params = {
                "Name": new_trigger_name,
                "Type": trigger['Type'],
                "Actions": new_actions,
            }

            # Add optional fields only if present and non-empty
            optional_fields = [
                "Description", "Schedule", "WorkflowName",
                "EventBatchingCondition"
            ]
            for field in optional_fields:
                if field in trigger and trigger[field] not in (None, {}, [], ''):
                    params[field] = trigger[field]

            glue.create_trigger(**params)

            # Preserve activation state (will show in logs, but Glue console may still show "Created")
            if trigger.get("State") == "ENABLED":
                glue.start_trigger(Name=new_trigger_name)
            elif trigger.get("State") == "DISABLED":
                glue.stop_trigger(Name=new_trigger_name)

            print(f"Cloned trigger {trigger['Name']} → {new_trigger_name} (state: {trigger.get('State')})")


def load_job_mapping(file_path):
    mapping = {}
    with open(file_path, 'r') as f:
        for line in f:
            line = line.strip()
            if not line or ':' not in line:
                continue
            old, new = line.split(':', 1)
            mapping[old.strip()] = new.strip()
    return mapping


def main():
    job_mapping = load_job_mapping("glue_jobs.txt")
    for old_name, new_name in job_mapping.items():
        clone_glue_job(old_name, new_name)
        clone_triggers(old_name, new_name)


if __name__ == "__main__":
    main()
```

---

## 🚀 Usage

1. **Install dependencies**  
   Ensure you have `boto3` installed:
   ```bash
   pip install boto3
   ```

2. **Configure AWS credentials**  
   Make sure your AWS CLI or environment variables are set up with permissions to:
   - Read Glue jobs and triggers.
   - Create Glue jobs and triggers.
   - Fetch tags.

3. **Prepare job mappings**  
   Create a `glue_jobs.txt` file with mappings:
   ```
   old-job-name1:new-job-name1
   old-job-name2:new-job-name2
   ```

4. **Run the script**  
   ```bash
   python clone_glue_jobs.py
   ```

5. **Verify in AWS Console**  
   - Check that new jobs are created with the correct configuration and tags.
   - Check that triggers are cloned and point to the new jobs.

---

## 📝 Notes & Caveats
- **Tags**: Retained via `get_tags` using the job ARN.
- **Triggers**: Created with unique names to avoid idempotency errors.
- **Trigger state**: Logged as preserved, but may show as "Created" in console. You can manually activate/deactivate if needed.
- **Workflows**: If jobs are part of Glue workflows, you’ll need to extend the script to update workflows as well.

---

## ✅ Example Output

When you run the script, you’ll see logs like:

```
Cloned job old-job-1 → new-job-1 (tags retained)
Cloned trigger old-trigger → old-trigger_new-job-1 (state preserved: ENABLED)
```
