# Project 2: AWS CLI S3 Manipulation

This project demonstrates proficiency with the AWS Command Line Interface (CLI) by managing Amazon S3 resources programmatically.

## Tasks
* Create an IAM user with programmatic access to S3.
* Configure the AWS CLI on a local machine.
* Create two S3 buckets.
* Upload a local file to the first bucket.
* Copy the file from the first bucket to the second bucket.

## Step-by-Step Instructions

### 1. Create an IAM User
1.  In the IAM Dashboard, create a new user named `cli-user`.
2.  Attach the `AmazonS3FullAccess` policy directly to the user.
3.  Create an access key for the user and save the **Access Key ID** and **Secret Access Key**.

### 2. Configure the AWS CLI
1.  Install the AWS CLI on your local machine.
2.  Run the configure command:
    ```bash
    aws configure
    ```
3.  Enter your saved Access Key ID, Secret Access Key, a default region (e.g., `us-east-1`), and default output format (`json`).

### 3. Create S3 Buckets
* Run the `aws s3 mb` (make bucket) command to create two globally unique buckets.
    ```bash
    # Replace with your own unique names
    aws s3 mb s3://your-unique-name-bucket-1
    aws s3 mb s3://your-unique-name-bucket-2
    ```

### 4. Manipulate Files
1.  Create a sample text file on your local machine named `test.txt`.
2.  Upload the file to the first bucket using the `aws s3 cp` command:
    ```bash
    aws s3 cp test.txt s3://your-unique-name-bucket-1/
    ```
3.  Copy the file from the first bucket to the second:
    ```bash
    aws s3 cp s3://your-unique-name-bucket-1/test.txt s3://your-unique-name-bucket-2/
    ```

### 5. Final Verification
* List the contents of both buckets to confirm the file exists in both locations.
    ```bash
    aws s3 ls s3://your-unique-name-bucket-1/
    aws s3 ls s3://your-unique-name-bucket-2/
    ```
