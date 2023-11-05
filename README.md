# Convert-document-to-PDF-using-LibreOffice-with-AWS-Lambda

This is a simple serverless project using AWS Lambda to convert Word (doc, docx), PowerPoint (ppt, pptx), Excel (xlsx, xls, csv) and text files (txt) to PDF file.

Other libraries on the internet uses Microsoft Office to converts to PDF, but comes with a license cost.

Thankfully LibreOffice is an open-source tool, but unfortunately, LibreOffice isn't at Lambda execution ambient, so we need to create a own LibreOffice layer. 

When the file is uploaded to a S3 bucket, the Lambda will convert this file to PDF using LibreOffice and sending to a destination S3 Bucket, as you can see in the schematic below.

![alt text](https://github.com/kontrolinho/Convert-document-to-PDF-using-LibreOffice-with-AWS-Lambda/blob/main/Images/Lambda1.png?raw=true)


## Create 2 S3 Bucket

First of all, we need to create 2 S3 Buckets, one for a source file (docx, csv, ppt...) and another one for the PDF file, a destination bucket.

**The name you choose for a bucket must be globally unique and follow the [Amazon S3 Bucket naming rules](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html).**

-- **Creating a bucket using AWS CLI**

Run the following CLI command to create your source bucket (for uploaded files), and repeat the same command to create a destination bucket (remember to change the bucket name and keep the same region).

```
aws s3api create-bucket --bucket BUCKETNAME --region REGION \
--create-bucket-configuration LocationConstraint=REGION

```

-- **Creating a bucket using AWS Console**

1. Open the [S3 Buckets page](https://console.aws.amazon.com/s3/buckets)
2. Choose create bucket
3. Under General configuration, do the following.
   
    Bucket name **(remember to enter a globally unique name and follow S3 Bucket naming rules)**
  
    For AWS Region, choose the region closest to your geographical location (or the most cost effective).

4. Leave all other options as default.
5. Create Bucket.

![alt text](https://github.com/kontrolinho/Convert-document-to-PDF-using-LibreOffice-with-AWS-Lambda/blob/main/Images/Lambda18.png?raw=true)


## LibreOffice at AWS Lambda

 Packages that must be installed using yum, such as LibreOffice, isn't originally included at Lambda.
 
 So, LibreOffice uses a [Brotli compression algorithm](https://github.com/google/brotli), and it may not work for the first time or after an interval since the first run, it appears not working because Brotli is being unzipped.

 **YOUR LAMBDA MUST BE AT THE SAME REGION AS YOUR S3 BUCKETS**

-- **Brotlipy**

This library contains Python binding for the reference Brotli encoder/decoder, available at this [Github Repo](https://github.com/kuharan/Lambda-Layers).
At this project will be used Python 3.8

1. Open the [AWS Lambda Page](https://us-east-2.console.aws.amazon.com/lambda/home?region=us-east-2#/begin).

2. Open Layers page at AWS Lambda.

![alt text](https://github.com/kontrolinho/Convert-document-to-PDF-using-LibreOffice-with-AWS-Lambda/blob/main/Images/Lambda2.png?raw=true)

3. Create Layer.

4. Choose a name to your Layer, and upload your brotlipy lambda layer.

![alt text](https://github.com/kontrolinho/Convert-document-to-PDF-using-LibreOffice-with-AWS-Lambda/blob/main/Images/Lambda3.png?raw=true)

6. Create.
   
 -- **Creating a Function to LibreOffice**
1. Open the AWS Lambda Functions page.

![alt text](https://github.com/kontrolinho/Convert-document-to-PDF-using-LibreOffice-with-AWS-Lambda/blob/main/Images/Lambda4.png?raw=true)

2. Create Function
3. Select Author from Scratch
4. Under **Basic Information** do the following

a. At Function name, enter your function name.
  
  b. Runtime, **select Python 3.8**.

6. Leave all others options as default.

![alt text](https://github.com/kontrolinho/Convert-document-to-PDF-using-LibreOffice-with-AWS-Lambda/blob/main/Images/Lambda5.png?raw=true)

6. Open your recent created **Brotlipy-Layer** at **Layers Page**, and copy the **ARN**.

![alt text](https://github.com/kontrolinho/Convert-document-to-PDF-using-LibreOffice-with-AWS-Lambda/blob/main/Images/lambda13.png?raw=true)

7. Back to Functions Page, scrolldown to **Layers** and click at **Add a Layer**.
8. Under **Choose a Layer**, select **Specify an ARN**
9. Paste your Brotlipy-Layer ARN.
10. Click at **Verify**
11. Add.

![alt text](https://github.com/kontrolinho/Convert-document-to-PDF-using-LibreOffice-with-AWS-Lambda/blob/main/Images/lambda14.png?raw=true)


-- **PreBuilt AWS Region Layer ARN**

Use the same Version ARN as you Python version, and the same region. Open this [Github Repo](https://github.com/shelfio/libreoffice-lambda-layer#version-arns).
For example, i'm using us-east-2 (Ohio).
```
arn:aws:lambda:us-east-2:764866452798:layer:libreoffice-brotli:1
```

1. Open your Function
2. Scroll down to Layers, and select Add a Layer.
3. Under **Choose a Layer** do the following

   a. Select **Specify an ARN**

   b. Paste your selected ARN

   c. Click at Verify

5. Add.

![alt text](https://github.com/kontrolinho/Convert-document-to-PDF-using-LibreOffice-with-AWS-Lambda/blob/main/Images/Lambda7.png?raw=true)

-- **Security Credentials**

You'll see at the code, is required the Access Key, and Secret Access Key from an IAM User with S3 Permissions, read more at [
Where’s My Secret Access Key?](https://aws.amazon.com/pt/blogs/security/wheres-my-secret-access-key/).

Let's create an Access Key.
1. Open [Identity and Access Management (IAM) Page](https://us-east-1.console.aws.amazon.com/iamv2/home#/home).
2. At Quick Links in right side, click at [My Security Credentials](https://us-east-1.console.aws.amazon.com/iamv2/home#/security_credentials).
3. Scroll down to Create Access Key, and select **Create Access Key**.
4. Copy your Access Key, and Secret Access Key.

![alt text](https://github.com/kontrolinho/Convert-document-to-PDF-using-LibreOffice-with-AWS-Lambda/blob/main/Images/Lambda8.png?raw=true)

5. Done.


-- **Function Code**
 1. At AWS Lambda Page, click at Functions page, and select your function.
 2. At **Code**, under **Code Source**, paste the [code available at this Github Repo](https://github.com/kontrolinho/Convert-document-to-PDF-using-LibreOffice-with-AWS-Lambda/blob/main/converter.py). 
 3. **Please, change only the following variables:**
  
  
  **ACCESS_KEY : Access key from an IAM User with the S3 Permission.**
  
  **SECRET_ACCESS_KEY : Secret Access Key from an IAM User with S3 Permission.**
  
  **output_bucket : NAME of DESTINATION S3 Bucket.**

  ![alt text](https://github.com/kontrolinho/Convert-document-to-PDF-using-LibreOffice-with-AWS-Lambda/blob/main/Images/Lambda9.png?raw=true)

 
 4. Click at Deploy.


-- **Triggering the S3 Source Bucket**


Specify the bucket to trigger when a document is uploaded, do the following.
1. Open Lambda Function Page.
2. Under **Function Overview** select **Add trigger.**
3. At **Add Trigger**, under **Trigger configuration**, click at **Select a source** and select **S3**.
4. At **Bucket** select the **SOURCE BUCKET**.
5. Mark the **I Acknowledge... box.**
6. Add.

![alt text](https://github.com/kontrolinho/Convert-document-to-PDF-using-LibreOffice-with-AWS-Lambda/blob/main/Images/lambda10.png?raw=true)

-- **Configure Lambda**


Now it's time to modify the Lambda Function Memory size and Timeout.
1. Open Lambda Function Page.
2. Scrolldown to **Configuration** Label, and select **General Configuration.**
3. Click at Edit.

![alt text](https://github.com/kontrolinho/Convert-document-to-PDF-using-LibreOffice-with-AWS-Lambda/blob/main/Images/lambda11.png?raw=true)

Now, under **Basic Settings**, let's make some changes.

**Memory: 1920MB
Timeout: 10 minutes**

![alt text](https://github.com/kontrolinho/Convert-document-to-PDF-using-LibreOffice-with-AWS-Lambda/blob/main/Images/lambda12.png?raw=true)

Sometimes the process of unzipping of Brotli is heavy, and uses a lot of memory, and takes a long time to start up.

Don't worry, you can read more at [Configuring Lambda Function Options](https://docs.aws.amazon.com/lambda/latest/dg/configuration-function-common.html).


# Conclusion

That's all folks.


![alt text](https://github.com/kontrolinho/Convert-document-to-PDF-using-LibreOffice-with-AWS-Lambda/blob/main/Images/lambda15.png?raw=true)
![alt text](https://github.com/kontrolinho/Convert-document-to-PDF-using-LibreOffice-with-AWS-Lambda/blob/main/Images/lambda16.png?raw=true)
![alt text](https://github.com/kontrolinho/Convert-document-to-PDF-using-LibreOffice-with-AWS-Lambda/blob/main/Images/Lambda17.png?raw=true)
