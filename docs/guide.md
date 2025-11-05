# 📸 AWS Lambda Image Resizer — Beginner Guide

This guide explains how to build an automatic image-resizing Project using:
- AWS S3
- AWS Lambda
- Lambda Layers (Pillow library)
- Python

## ✅ Architecture

1. Upload image → S3 bucket (`original-images`)
2. S3 event triggers Lambda
3. Lambda resizes image to 128x128
4. Resized image saved to another bucket (`resized-images`)

---

## ✅ Prerequisites

- AWS account
- Python installed locally
- AWS CLI configured
- Basic knowledge of S3 & Lambda (not required but helpful)

---

## 🚀 Step-by-Step Setup

### **1️⃣ Create S3 Buckets**
- `original-images-lambda`
- `resized-images-lambda`

### **2️⃣ Create Lambda Function**
- Runtime: Python 3.12
- Timeout: 1 Minute
- Environment Variable  
  `DEST_BUCKET = resized-images-lambda`

### **3️⃣ Create Lambda Layer**
Includes **Pillow** library.

### **4️⃣ Add Code**
Python script to resize images.
import json
import os
import boto3
from PIL import Image # Pillow library
s3_client = boto3.client('s3')
# Define your destination bucket name here
DESTINATION_BUCKET_NAME = os.environ.get('DESTINATION_BUCKET_NAME') # Get from environment variable
def lambda_handler(event, context):
    print(f"Received event: {json.dumps(event)}")
    # Get the bucket name and file key from the S3 event
    try:
        source_bucket_name = event['Records'][0]['s3']['bucket']['name']
        file_key = event['Records'][0]['s3']['object']['key']
        print(f"Image '{file_key}' uploaded to bucket '{source_bucket_name}'.")
    except KeyError as e:
        print(f"Error extracting S3 event details: {e}")
        return {
            'statusCode': 400,
            'body': json.dumps('Error: Invalid S3 event structure.')
        }

    # Define thumbnail parameters
    thumbnail_size = (128, 128) # 128x128 pixels

    # --- Download the image from S3 to Lambda's /tmp directory ---
    download_path = f'/tmp/{file_key}'
    upload_path = f'/tmp/resized-{file_key}'

    try:
        s3_client.download_file(source_bucket_name, file_key, download_path)
        print(f"Downloaded '{file_key}' to '{download_path}'.")
    except Exception as e:
        print(f"Error downloading file: {e}")
        return {
            'statusCode': 500,
            'body': json.dumps(f"Error downloading {file_key}: {str(e)}")
        }

    # --- Resize the image using Pillow ---
    try:
        with Image.open(download_path) as image:
            image.thumbnail(thumbnail_size) # Resizes in place, maintaining aspect ratio
            image.save(upload_path)
        print(f"Resized image saved to '{upload_path}'.")
    except Exception as e:
        print(f"Error resizing image: {e}")
        return {
            'statusCode': 500,
            'body': json.dumps(f"Error resizing {file_key}: {str(e)}")
        }

    # --- Upload the resized image to the destination S3 bucket ---
    try:
        s3_client.upload_file(upload_path, DESTINATION_BUCKET_NAME, f'resized-{file_key}')
        print(f"Uploaded 'resized-{file_key}' to '{DESTINATION_BUCKET_NAME}'.")
    except Exception as e:
        print(f"Error uploading resized image: {e}")
        return {
            'statusCode': 500,
            'body': json.dumps(f"Error uploading resized {file_key}: {str(e)}")
        }

    return {
        'statusCode': 200,
        'body': json.dumps(f'Successfully resized {file_key} and saved to {DESTINATION_BUCKET_NAME}')
    }


### **5️⃣ Add S3 Trigger**
Trigger on **Object Created (PUT)** for `.jpg`

### **6️⃣ Test**
Upload image → verify resized output.

---

## ✅ Troubleshooting

| Problem | Fix |
|---|---|
Lambda not triggered | Check S3 event notification  
Image not resizing | Check Pillow layer added  
Permission error | Add S3 & CloudWatch permissions to role  

---

## 🎉 Result

✅ Fully automated serverless image resizing  
✅ Great beginner AWS project  
✅ Perfect for resume & portfolio  

---

## 📂 Repository Structure

