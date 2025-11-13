# ITCS-6190 Assignment 3: AWS Data Processing Pipeline
Instructor: Marco Vieira  
Course: Cloud Computing for Data Analysis (ITCS 6190/8190), Fall 2025
Name : Atluri Sasi Vadana
This README documents the end-to-end build, test, and verification steps for the assignment pipeline:
S3 → Lambda → Glue Crawler → Athena → EC2 (Flask dashboard).  
Each major step includes exact console clicks or CLI commands, expected outputs, troubleshooting hints, and a placeholder for the required screenshot.

---

## Summary of deliverables to include with this README
- S3 bucket structure screenshot (showing `raw/`, `processed/`, `enriched/`)
- IAM roles list screenshot (showing all three roles)
- Lambda function overview screenshot (code + configuration)
- Lambda S3 trigger screenshot (configured)
- Processed CSV file in S3 screenshot (`processed/filtered_<originalname>.csv`)
- Glue crawler CloudWatch or crawler run success screenshot
- Athena query CSV files in S3 `enriched/` screenshot
- Final web page screenshot (EC2 dashboard)

---

## Prerequisites & assumptions
- You have an AWS account with appropriate permissions.
- You created IAM roles: `Lambda-S3-Processing-Role`, `Glue-S3-Crawler-Role`, `EC2-Athena-Dashboard-Role`.
- You uploaded `orders.csv` into your repo or local machine for S3 upload.
- Glue database name used in this README: `orders_db`
- Glue table created by crawler: `processed` (if different, replace `processed` in SQL queries below)

---

## 1. Create S3 bucket and folders
### Purpose
Store raw input, processed output, and Athena query results.

### Console steps
1. AWS Console → S3 → **Create bucket**.
2. Enter a unique bucket name (example: `itcs6190-assignment3-yourinitials`).
3. Leave defaults or adjust region; complete creation.
4. Inside the bucket, click **Create folder** three times and make folders:
   - `raw/`
   - `processed/`
   - `enriched/`

### Commands (AWS CLI alternative)
```bash
aws s3 mb s3://itcs6190-assignment3-yourinitials --region us-east-1
aws s3api put-object --bucket itcs6190-assignment3-yourinitials --key raw/
aws s3api put-object --bucket itcs6190-assignment3-yourinitials --key processed/
aws s3api put-object --bucket itcs6190-assignment3-yourinitials --key enriched/
```

### Expected result
Bucket with three folders visible in the S3 console.

### Screenshot placeholder


<img width="975" height="543" alt="image" src="https://github.com/user-attachments/assets/211f02c0-e780-44b1-ae3c-8a92b3a0abef" />
> Caption: S3 console showing `raw/`, `processed/`, `enriched/` prefixes.
---

## 2. Create IAM roles and attach policies
### Purpose
Give Lambda, Glue, and EC2 the permissions they need.

### Roles to create (Console steps)
**Lambda Execution Role**
1. AWS Console → IAM → Roles → Create role.
2. Select **AWS service → Lambda**.
3. Attach policies: `AWSLambdaBasicExecutionRole`, `AmazonS3FullAccess`.
4. Name: `Lambda-S3-Processing-Role`.

**Glue Service Role**
1. Create role for **AWS service → Glue**.
2. Attach: `AmazonS3FullAccess`, `AWSGlueConsoleFullAccess`, `AWSGlueServiceRole`.
3. Name: `Glue-S3-Crawler-Role`.

**EC2 Instance Profile**
1. Create role for **AWS service → EC2**.
2. Attach: `AmazonS3FullAccess`, `AmazonAthenaFullAccess`.
3. Name: `EC2-Athena-Dashboard-Role`.
4. When launching EC2, select this role as **IAM instance profile**.

### Expected result
Three roles visible under IAM → Roles.

### Screenshot placeholder
<img width="975" height="550" alt="image" src="https://github.com/user-attachments/assets/405910e6-eb72-45fd-91a0-d49e619c1566" />
> Caption: IAM roles page showing the three roles created.

---

## 3. Create the Lambda function
### Purpose
Automatically process raw CSV uploads and write cleaned CSV to `processed/`.

### Console steps
1. AWS Console → Lambda → Create function → Author from scratch.
2. Function name: `FilterAndProcessOrders`
3. Runtime: Python 3.9 (or 3.10/3.11)
4. Permissions: Use existing role → `Lambda-S3-Processing-Role`
5. Create.

### Replace code
1. In the function code editor, paste the content of `LambdaFunction.py` provided with your repo.
2. Ensure handler matches filename/entry point. If the file's default handler is `lambda_function.lambda_handler`, confirm in Configuration → General configuration → Handler.

### Save and publish
Click **Deploy** (or Save).

### Screenshot placeholder
<img width="975" height="630" alt="image" src="https://github.com/user-attachments/assets/fd7e8604-a071-4181-a1e1-dfa7efaad9db" />
> Caption: Lambda function page showing code and role configuration.

---

## 4. Configure S3 trigger on Lambda
### Purpose
Ensure Lambda runs when a `.csv` is uploaded to `raw/`.

### Steps
1. In Lambda → Configuration → Triggers → Add trigger.
2. Source: S3
3. Bucket: select your bucket (e.g., `itcs6190-assignment3-yourinitials`)
4. Event type: **All object create events**
5. Prefix: `raw/`
6. Suffix: `.csv`
7. Acknowledge and Add.

### Screenshot placeholder
<img width="975" height="470" alt="image" src="https://github.com/user-attachments/assets/8395dbdc-4714-450d-8621-748c2da3d620" />
> Caption: Lambda triggers list with S3 trigger configured.

---

## 5. Upload test raw CSV and verify Lambda processed it
### Purpose
Trigger Lambda by uploading `orders.csv` and verify `processed/filtered_<name>.csv` appears.

### Steps
1. AWS Console → S3 → open your bucket → open `raw/` → **Upload**.
2. Choose the local `orders.csv` file and Upload.
3. Wait 15–30 seconds.

### Verify in S3 (processed/)
1. Open `processed/` and click the refresh icon.
2. Confirm a file named `filtered_<originalname>.csv` exists (example `filtered_orders.csv`).

### If file is missing — check CloudWatch logs
1. AWS Console → CloudWatch → Log groups → `/aws/lambda/FilterAndProcessOrders`.
2. Open the latest log stream and confirm lines:
   - `Lambda triggered by S3 event.`
   - `Successfully read file from S3: orders.csv`
   - `Filtered file successfully written to S3: processed/filtered_orders.csv`

### Troubleshooting
- If CloudWatch shows error about missing `Records`, you likely tested Lambda using "Test" without S3 payload. Re-upload to `raw/`.
- If permission errors appear, confirm Lambda role has `AmazonS3FullAccess`.

### Screenshot placeholders
<img width="975" height="459" alt="image" src="https://github.com/user-attachments/assets/f762cc7e-ed95-48b3-adb1-4001ef6d7b9e" />

> Caption: S3 raw/ folder showing uploaded orders.csv.

<img width="975" height="432" alt="image" src="https://github.com/user-attachments/assets/8cd6b105-5d83-4dbe-8357-998dce7517f0" />
> Caption: S3 processed/ folder showing the filtered CSV file.

<img width="975" height="463" alt="image" src="https://github.com/user-attachments/assets/7bebf99d-db31-4903-a297-5f47336ed9e4" />
> Caption: CloudWatch log showing successful processing and S3 write.

---

## 6. Create and run AWS Glue Crawler
### Purpose
Crawl the CSV in `processed/` to create a table in the Glue Data Catalog.

### Create crawler (Console)
1. AWS Console → Glue → Crawlers → Create crawler.
2. Name: `orders_processed_crawler`.
3. Data source: S3 → choose `s3://your-bucket-name/processed/`
4. IAM role: `Glue-S3-Crawler-Role`
5. Output: Create/select database `orders_db`
6. Finish.

### Run crawler
1. Select crawler `orders_processed_crawler` → **Run crawler**.
2. Wait for status to change to **Succeeded**.

### Confirm table
1. Glue → Data Catalog → Tables (or Databases → orders_db → Tables).
2. Locate table created; common name patterns: `processed`, `processed_filtered_orders_csv`, etc. Record table name — this README uses `processed`.

### If table created in wrong DB
- Edit crawler → Output → set `orders_db` explicitly → run crawler again.

### Screenshot placeholders
<img width="975" height="467" alt="image" src="https://github.com/user-attachments/assets/397c992b-6710-471d-94f2-1a8bd74ff84a" />

> Caption: Glue crawler run status showing Succeeded.

<img width="975" height="493" alt="image" src="https://github.com/user-attachments/assets/867b503b-a0b4-467c-8481-f6a7613dceb1" />
> Caption: Glue table view showing schema and S3 location.

---

## 7. Query the processed data with Athena
### Purpose
Use SQL to generate the reports required by the assignment. Set Athena query results S3 location to `s3://your-bucket-name/enriched/athena-results/`.

### Athena setup
1. AWS Console → Athena → Settings → set Query result location to `s3://your-bucket-name/enriched/athena-results/`.
2. Select Catalog: `AwsDataCatalog`, Database: `orders_db`.
3. Use the table name created by the crawler (this README uses table `processed`).

### Correct SQL queries (copy & paste)

**1) Total Sales by Customer**
```sql
SELECT customer,
       SUM(CAST(amount AS DOUBLE)) AS TotalAmountSpent
FROM processed
GROUP BY customer
ORDER BY TotalAmountSpent DESC;
```

**2) Monthly Order Volume and Revenue**
```sql
SELECT date_trunc('month', CAST(orderdate AS DATE)) AS OrderMonth,
       COUNT(orderid) AS NumberOfOrders,
       ROUND(SUM(CAST(amount AS DOUBLE)), 2) AS MonthlyRevenue
FROM processed
GROUP BY 1
ORDER BY OrderMonth;
```

**3) Order Status Dashboard**
```sql
SELECT status,
       COUNT(orderid) AS OrderCount,
       ROUND(SUM(CAST(amount AS DOUBLE)), 2) AS TotalAmount
FROM processed
GROUP BY status;
```

**4) Average Order Value (AOV) per Customer**
```sql
SELECT customer,
       ROUND(AVG(CAST(amount AS DOUBLE)), 2) AS AverageOrderValue
FROM processed
GROUP BY customer
ORDER BY AverageOrderValue DESC;
```

**5) Top 10 Largest Orders in February 2025**
```sql
SELECT orderdate, orderid, customer, amount
FROM processed
WHERE CAST(orderdate AS DATE) BETWEEN DATE '2025-02-01' AND DATE '2025-02-28'
ORDER BY CAST(amount AS DOUBLE) DESC
LIMIT 10;
```

### After each query
- Athena writes results to the configured S3 result folder. Confirm CSV files appear in `s3://your-bucket-name/enriched/athena-results/`.
- Download sample CSV to inspect.

### Screenshot placeholders
<img width="975" height="539" alt="image" src="https://github.com/user-attachments/assets/f54977e1-fd5e-4ee5-9e4d-47978638fc3f" />
<img width="975" height="481" alt="image" src="https://github.com/user-attachments/assets/9d41876d-a4b2-44e3-a672-e78d8ff4ec97" />
<img width="975" height="513" alt="image" src="https://github.com/user-attachments/assets/5ab3f53e-6037-4542-9387-5ef27d078d1f" />
<img width="975" height="502" alt="image" src="https://github.com/user-attachments/assets/30b968f9-0233-4d0f-8ec6-6312f90d52cc" />
<img width="975" height="539" alt="image" src="https://github.com/user-attachments/assets/fc8b4382-d534-4019-a4c9-bb8cf3c2a973" />
> Caption: Athena UI showing query results and execution history.

<img width="975" height="516" alt="image" src="https://github.com/user-attachments/assets/f1949ac1-2961-4de3-837c-f333e89c98da" />
> Caption: S3 enriched folder showing Athena query CSV files.

---

## 8. Launch EC2 instance for dashboard
### Purpose
Host a Flask app that runs Athena queries and displays the results.

### Launch instance (Console)
1. AWS Console → EC2 → Launch instance.
2. Name: `Athena-Dashboard-Server`
3. AMI: Amazon Linux 2023
4. Instance type: `t2.micro`
5. Key pair: create or select and download the `.pem` file (save it securely)
6. Security group:
   - SSH (22) → Source: Your IP (recommended)
   - Custom TCP (5000) → Source: `0.0.0.0/0` (for public web testing)
7. IAM role (IAM instance profile): `EC2-Athena-Dashboard-Role`
8. Launch.

### Screenshot placeholder
<img width="975" height="460" alt="image" src="https://github.com/user-attachments/assets/0e925dfc-6754-4eae-a798-49a04aefac69" />
> Caption: EC2 console listing the running instance `Athena-Dashboard-Server`.

---

## 9. SSH into EC2 and install environment
### Local terminal (Windows PowerShell / macOS / Linux)
```bash
chmod 400 /path/to/keypairassignment3.pem
ssh -i /path/to/keypairassignment3.pem ec2-user@<YOUR_PUBLIC_IP>
```

### On EC2 (instance CLI)
```bash
# update OS and install Python/pip
sudo yum update -y
sudo yum install python3-pip -y

# install Flask and boto3
pip3 install Flask boto3
```

### Screenshot placeholders
<img width="975" height="702" alt="image" src="https://github.com/user-attachments/assets/9252f112-4702-4fca-91a5-a3874d20a1cd" />
> Caption: Terminal showing successful SSH connection to ec2-user@<PUBLIC_IP>.

<img width="975" height="650" alt="image" src="https://github.com/user-attachments/assets/74c4c4c1-4166-4151-ace2-475036c3992d" />
<img width="975" height="627" alt="image" src="https://github.com/user-attachments/assets/7c471161-4180-4f4e-abfd-bff137ce9383" />
> Caption: Terminal output for package installs: python3-pip and pip3 installs.

---

## 10. Create app.py and configure it
### On EC2
1. At the EC2 prompt:
```bash
nano app.py
```
2. Paste the provided `EC2InstanceNANOapp..py` script content into `app.py`.

3. Edit the top configuration variables (replace placeholders):
```python
AWS_REGION = "us-east-1"
ATHENA_DATABASE = "orders_db"
S3_OUTPUT_LOCATION = "s3://itcs6190-assignment3-yourinitials/enriched/athena-results/"
```

4. Important: Verify the queries in `queries_to_run` reference your actual table name `processed` (or the table name created by the crawler).

5. Save with Ctrl+X → Y → Enter.

### Screenshot placeholder
<img width="975" height="734" alt="image" src="https://github.com/user-attachments/assets/46236239-c621-4c4b-9861-d68e896f1555" />
> Caption: Nano editor showing the app.py content with AWS_REGION and S3_OUTPUT_LOCATION configured.

---

## 11. Start the Flask app and test the dashboard
### Run on EC2
```bash
python3 app.py
```

### Expected output on EC2 CLI
```
 * Serving Flask app 'app'
 * Debug mode: off
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```

### Open the dashboard in a browser
- Navigate to `http://<YOUR_PUBLIC_IP>:5000` from your local browser (replace `<YOUR_PUBLIC_IP>` with the instance public IP).

### Troubleshooting
- If page shows errors: check EC2 role permissions (Athena + S3), and inspect EC2 terminal logs for stacktrace (the app prints errors).
- Common error: invalid table name in queries — ensure `processed` (or your table name) exists in `orders_db`.
- If Flask fails to start with `SyntaxError`, open nano and check for accidental duplicate paste or missing newline at end — ensure `if __name__ == '__main__':` exists once, and nothing follows `app.run(...)`.

### Screenshot placeholder
<img width="975" height="441" alt="image" src="https://github.com/user-attachments/assets/6bac8dde-a110-4d32-9ac0-2c30b6294a16" />
<img width="975" height="444" alt="image" src="https://github.com/user-attachments/assets/7145be89-9209-4506-bf99-d9e666230d9d" />
<img width="975" height="467" alt="image" src="https://github.com/user-attachments/assets/65200dea-f8a9-44dc-bdc9-ab4390db8f22" />
> Caption: Browser view of the Athena Orders Dashboard showing all query tables.

---


## 13. Cleanup steps (to avoid charges)
1. Terminate EC2 instance (EC2 → Instances → Actions → Instance State → Terminate).
2. Delete Lambda function (Lambda → select function → Actions → Delete function).
3. Delete Glue crawler and database (Glue → Crawlers → delete; Glue → Databases → delete).
4. Empty and delete S3 bucket (S3 → bucket → Empty → Delete).
5. Delete IAM roles you created (IAM → Roles → select → Delete role).

---

## 14. Troubleshooting reference (common problems)
- **Lambda triggered but processed folder empty**: Confirm S3 trigger prefix `raw/` and suffix `.csv`. Inspect CloudWatch logs and correct IAM policy.
- **Glue crawler finds 1 table change but DB shows 0 tables**: The crawler may have created the table in a different database (often `default`). Check Glue → Tables list and move or re-run crawler with correct target DB.
- **Athena 'table not found'**: Verify `orders_db` is selected and table name matches crawler output. Update SQL to use the actual table name.
- **EC2 Flask app errors**: If `SyntaxError`, open `app.py` and remove duplicate code blocks. If `botocore`/`boto3` missing, re-run `pip3 install boto3`.

---

End of README.
