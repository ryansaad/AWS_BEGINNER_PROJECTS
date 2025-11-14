# Receipt-Processing-Tool
This project demonstrates a fully serverless and automated pipeline on AWS for digitizing and organizing physical or image-based receipts. Users can simply upload a receipt image to an S3 bucket, which triggers a workflow to extract key information, store it in a database, and send a summary email, all without manual intervention.

This solution showcases the practical application of several AWS serverless services working together to solve a common administrative problem, highlighting real-time data processing and automation capabilities.


**Services Used:**
* Amazon S3 (Simple Storage Service): Serves as the primary storage for raw receipt images and acts as the trigger point for the entire processing pipeline.
* AWS Lambda: The serverless compute service that executes the core logic. It's triggered by new object uploads to S3, orchestrates the data extraction, storage, and notification.
* Amazon Textract: An intelligent document processing service that automatically extracts text and data (like key-value pairs and table data) from scanned documents and images. It's
  crucial for converting receipt images into machine-readable text.
* Amazon DynamoDB: A fast, flexible NoSQL database service that stores the structured data extracted from the receipts (e.g., receipt ID, file name, extracted total, date, full text).
* Amazon SES (Simple Email Service): A scalable and cost-effective email sending service used to dispatch email notifications containing a summary of the processed receipt to a
  designated recipient.
* AWS IAM (Identity and Access Management): Manages access and permissions for AWS services, ensuring secure and controlled interactions between Lambda, S3, Textract, DynamoDB, and SES.
*******************************************************************************************
**Features**
* Automated Workflow: Initiates processing automatically upon receipt image upload to S3.
* Intelligent Data Extraction: Leverages Amazon Textract's machine learning capabilities for accurate text and data extraction from diverse receipt layouts.
* Structured Data Storage: Stores extracted details in a queryable NoSQL database (DynamoDB) for easy access and integration with other systems.
* Real-time Notifications: Sends immediate email summaries of processed receipts using Amazon SES.
* Serverless & Scalable: Built entirely on serverless components, providing inherent scalability, high availability, and a pay-as-you-go cost model.
*******************************************************************************************

**----Setup Guide-----**
Follow these steps to deploy and run the Automated Receipt Processing Tool in your AWS account.

# Prerequisites
An active AWS Account.
Basic familiarity with AWS Management Console.

Step-by-Step Deployment

# 1. Create an Amazon S3 Bucket
This bucket will store your receipt images and trigger the Lambda function.
Navigate to the S3 Console.
Click "Create bucket".
Bucket Name: Enter a globally unique name, e.g., your-name-receipt-processing-bucket.
AWS Region: Choose a region close to you.
Keep "Block all public access" enabled for security.
Click "Create bucket".
Once the bucket is created, click on its name.
Click "Create folder" and name it incoming. This is where you will upload your receipt images.

# 2. Create a DynamoDB Table
This table will store the structured data extracted from your receipts.
Navigate to the DynamoDB Console.
Click "Create table".
Table Name: Enter Receipts.
Partition Key: Enter ReceiptID (String).
Sort Key (Optional but Recommended): Enter Date (String).
Leave other settings as default.
Click "Create table".

# 3. Configure Amazon SES (Simple Email Service)
SES will be used to send email summaries of processed receipts.
Navigate to the SES Console.
In the navigation pane, under "Identities", choose "Email addresses".
Click "Create identity".
Select "Email identity".
Enter your "Email address" (e.g., your-email@example.com). This address will be used as both the sender and recipient for your receipt summaries.
Click "Create identity".
Crucially: Check your email inbox for a verification email from Amazon Web Services and click the verification link. The identity's status in the SES console must become "Verified" before proceeding.

# 4. Create an IAM Role for Lambda
This IAM role will grant your Lambda function the necessary permissions to interact with other AWS services.
Navigate to the IAM Console.
In the navigation pane, choose "Roles", then click "Create role".
Trusted Entity: Select "AWS service".
Use Case: Choose "Lambda".
Click "Next".
*Add Permissions: Attach the following policies by searching for each and checking the box:
*AmazonS3ReadOnlyAccess (for reading from S3)
*AmazonTextractFullAccess (for using Textract)
*AmazonDynamoDBFullAccess (for writing to DynamoDB)
*AmazonSESFullAccess (for sending emails)
*AWSLambdaBasicExecutionRole (for CloudWatch logs)
Click "Next".

Role Name: Enter: receipt-processing-lambda-role.
Click "Create role".

# 5. Create the AWS Lambda Function
This is the core compute service that orchestrates the receipt processing.

Navigate to the Lambda Console.Click "Create function".
Author from scratch: Select this option.
Function Name: Enter receipt-processing-function.
Runtime: Choose "Python 3.9" (or a later compatible version like Python 3.10).
Architecture: Leave as default (x86_64).
Change default execution role: Under "Execution role", select "Use an existing role".
Existing role: Choose receipt-processing-lambda-role from the dropdown.
Click "Create function".

# 6. Configure Lambda Function Settings
Adjust the timeout and add environment variables for your Lambda function.

General Configuration:
------------------------
Click "Edit" under "General configuration".
Set "Timeout" to 3 min 0 sec. Textract processing can sometimes take a bit longer.
Click "Save".

Environment Variables:
-----------------------
Click "Edit" under "Environment variables".
Click "Add environment variable" and add the following three variables:
Key: DYNAMODB_TABLE_NAME
Value: Receipts (the exact name of your DynamoDB table)
Key: SES_SENDER_EMAIL
Value: your-verified-email@example.com (the email you verified in SES)
Key: SES_RECIPIENT_EMAIL
Value: your-verified-email@example.com (the email you verified in SES)
Click "Save".

# 7. Add Lambda Function Code

Copy the content from the lambda_function.py file provided in this repository and paste it into the Lambda editor.
Click the "Deploy" button to save and deploy your changes.

# 8. Configure S3 Event Notification
This step connects your S3 bucket to your Lambda function, so Lambda is triggered when a new object is uploaded.
Go back to your S3 bucket (your-name-receipt-processing-bucket).
Click on the "Properties" tab.
Scroll down to the "Event notifications" section and click "Create event notification".
Event Name: Enter ReceiptUploadNotification.
Prefix: Enter incoming/ (this ensures only uploads to the incoming folder trigger the Lambda).
Event Types: Select "All object create events" (e.g., s3:ObjectCreated:Put).

Destination:
Select "Lambda function".
Choose your Lambda function receipt-processing-function from the dropdown.
Click "Create event notification".

## Testing Your Pipeline
* Upload a Receipt Image: Go to your S3 bucket (your-name-receipt-processing-bucket), navigate into the incoming/ folder, and upload a clear image of a receipt (e.g., a .jpg or .png file).

# Check Lambda Logs:
Go to your Lambda function (receipt-processing-function).
Click on the "Monitor" tab.
Click "View CloudWatch logs".
You should see recent log streams. Click on the latest one to see the execution details, including Textract output, DynamoDB storage details, and SES email status. Look for "Successfully stored item in DynamoDB" and "Email sent successfully!".

# Check DynamoDB:
Go to your DynamoDB table (Receipts).
Click on the "Explore items" tab. You should see a new item added with the extracted receipt details.
Check Your Email:
Check the inbox of the email address you verified with SES. You should receive an email summary of the processed receipt.

**Clean Up**
To avoid incurring ongoing AWS charges, it is crucial to delete all resources created for this project after you are done.
Empty S3 Bucket: Delete all objects within your your-name-receipt-processing-bucket (including those in the incoming/ folder).
Delete S3 Bucket: Delete the your-name-receipt-processing-bucket S3 bucket.
Delete DynamoDB Table: Delete the Receipts DynamoDB table.
Delete Lambda Function: Delete the receipt-processing-function Lambda function.
Delete IAM Role: Delete the receipt-processing-lambda-role IAM role.
Delete SES Identity: Delete the email address identity you verified in SES from the "Email addresses" section.
