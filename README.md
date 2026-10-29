# AWS Glue ETL Pipeline with Amazon S3

## Project Overview

This project demonstrates how to build an ETL (Extract, Transform, Load) pipeline using **AWS Glue and Amazon S3**.

Employee data is stored in an S3 bucket in CSV format. AWS Glue Crawlers discover the dataset structure and create metadata in the AWS Glue Data Catalog. AWS Glue Studio Visual ETL is then used to transform the employee data and write the output to Amazon S3 in Parquet format.

The project demonstrates cloud-based data integration, schema discovery, data transformation, and preparation of structured data for analytics.

## Objectives

- Store raw employee data in Amazon S3.
- Discover the dataset schema using an AWS Glue Crawler.
- Create a database and table in the AWS Glue Data Catalog.
- Build an ETL workflow using AWS Glue Studio Visual ETL.
- Transform employee records using the Drop Fields transformation.
- Remove the `city` column from the transformed dataset.
- Store curated output in Amazon S3 using Parquet format.

## Technologies Used

- **Amazon S3** – Stores raw and processed datasets.
- **AWS Glue** – Performs ETL operations.
- **AWS Glue Crawler** – Discovers dataset schemas.
- **AWS Glue Data Catalog** – Stores database and table metadata.
- **AWS Glue Studio Visual ETL** – Builds the visual transformation workflow.
- **AWS IAM** – Manages permissions for Glue resources.
- **CSV** – Raw input data format.
- **Apache Parquet** – Curated output format.

## Architecture

```text
             Employee CSV Dataset
                       |
                       v
                Amazon S3 Bucket
                       |
                       v
                    raw/
                 employees.csv
                       |
                       v
                 AWS Glue Crawler
                       |
                       v
             AWS Glue Data Catalog
                       |
                       v
              glue_training_db
                       |
                       v
                employees Table
                       |
                       v
              AWS Glue Studio ETL
                       |
                       v
              Drop Fields Transform
                  Remove city
                       |
                       v
                Amazon S3 Target
                       |
                       v
                    curated/
                Parquet Output
```

## S3 Bucket Structure

The project uses a single Amazon S3 bucket with separate prefixes for raw and curated data.

```text
cvv-task/
├── raw/
│   └── employees.csv
└── curated/
    └── Generated Parquet output files
```

The `raw/` prefix stores the original CSV dataset, while the `curated/` prefix stores the transformed output.

## Implementation Steps

### 1. Create an Amazon S3 Bucket

1. Open the AWS Management Console.
2. Navigate to Amazon S3.
3. Create a general-purpose S3 bucket.
4. Keep Block Public Access enabled.
5. Create the `raw/` and `curated/` prefixes.

### 2. Upload the Employee Dataset

Upload `employees.csv` to the `raw/` prefix.

Example S3 location:

```text
s3://YOUR_BUCKET_NAME/raw/employees.csv
```

Replace `YOUR_BUCKET_NAME` with the actual bucket name.

### 3. Configure IAM Permissions

Create or select an IAM role for AWS Glue.

The role must have the permissions required to:
- Read the raw dataset from S3.
- Write transformed data to the curated S3 location.
- Access AWS Glue Data Catalog resources.
- Write execution logs where configured.

Grant only the permissions required for the pipeline.

### 4. Create the Glue Data Catalog Database

1. Open the AWS Glue Console.
2. Navigate to Data Catalog databases.
3. Create a database named `glue_training_db`.
4. Save the database configuration.

This database stores metadata for the employee dataset.

### 5. Create and Run a Glue Crawler

1. Navigate to **AWS Glue → Crawlers**.
2. Create a crawler for the employee dataset.
3. Configure the S3 source location as the `raw/` prefix.
4. Select the appropriate IAM role.
5. Set the target database to `glue_training_db`.
6. Create the crawler and run it.

The crawler discovers the CSV structure and creates a table in the Data Catalog.

### 6. Verify the Data Catalog Table

After the crawler completes, open the `glue_training_db` database.

Verify the generated `employees` table and its schema.

The documented schema contains these columns:

| Column | Data type |
|---|---|
| `employee_id` | Integer |
| `name` | String |
| `department` | String |
| `salary` | Integer |
| `city` | String |

The exact schema depends on the input file and the types inferred by the crawler.

### 7. Build the Visual ETL Workflow

Open AWS Glue Studio and create a Visual ETL job.

Configure the following workflow:

```text
Data Catalog → employees → Transformation → S3 Target
```

**Source:** Select the `employees` table from `glue_training_db`.

**Transformation:** Add a Drop Fields transformation and select the `city` field for removal.

**Target:** Configure Amazon S3 as the output destination and select the `curated/` prefix.

### 8. Configure the S3 Target

Configure the target node with:

- **Target:** Amazon S3
- **Output location:** `s3://YOUR_BUCKET_NAME/curated/`
- **Output format:** Apache Parquet

Parquet is a columnar storage format suitable for analytical workloads.

### 9. Run the ETL Job

1. Save the Glue job.
2. Review the job configuration and IAM role.
3. Run the job.
4. Monitor its status in AWS Glue Studio.
5. Verify that the execution succeeds.

### 10. Verify the Curated Output

Open the S3 bucket and navigate to the `curated/` prefix.

Verify that the generated output files are present in Parquet format and that the transformed dataset no longer contains the `city` field.

## Transformation Logic

The pipeline applies the following transformation:

**Input:** Employee records containing employee ID, name, department, salary, and city.

**Transformation:** Drop the `city` field using AWS Glue Studio's Drop Fields node.

**Output:** Employee records containing employee ID, name, department, and salary.

This demonstrates how an ETL pipeline can remove unnecessary columns before storing data for analysis.

## Key Learnings

- Creating and organizing datasets in Amazon S3.
- Configuring IAM roles for AWS Glue.
- Using Glue Crawlers for automatic schema discovery.
- Creating databases and tables in the Glue Data Catalog.
- Building ETL pipelines using AWS Glue Studio Visual ETL.
- Applying column-removal transformations.
- Exporting transformed data in Parquet format.
- Monitoring ETL execution and troubleshooting permission errors.

## Troubleshooting

### Crawler fails

- Verify that the S3 source location is correct.
- Check that the Glue IAM role can access the raw dataset.
- Confirm that the target database exists.
- Review the crawler's execution details.

### Glue job cannot access S3

- Verify the Glue job's IAM role.
- Check the required S3 read and write permissions.
- Confirm the raw and curated S3 locations.

### Glue job creation fails

- Review the error message and account or service restrictions.
- Verify that the required IAM permissions are available.
- Check the selected AWS Region and Glue job configuration.

### No output in curated/

- Confirm that the job completed successfully.
- Verify the configured S3 target path.
- Check the job logs for transformation or write errors.

## Security Best Practices

- Keep S3 Block Public Access enabled.
- Avoid uploading confidential employee data to public repositories.
- Use IAM roles instead of hardcoded AWS credentials.
- Apply least-privilege access to S3 and Glue resources.
- Review AWS usage and clean up unused resources to manage costs.

## Future Enhancements

- Schedule the ETL job to run automatically.
- Add data-quality checks and invalid-record handling.
- Integrate Amazon Athena to query curated Parquet data.
- Add monitoring and failure notifications.
- Automate job deployment using infrastructure as code.

## Author

**CVV ANAND**

B.Tech Student – Artificial Intelligence and Machine Learning

GitHub: [Anandcvv05](https://github.com/Anandcvv05)

---

*This project demonstrates a cloud-based ETL workflow using AWS Glue and Amazon S3 for data preparation and transformation.*
