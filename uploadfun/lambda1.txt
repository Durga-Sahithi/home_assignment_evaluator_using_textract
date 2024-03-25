import boto3
import base64
from io import BytesIO
from datetime import datetime

s3 = boto3.client('s3')
dynamodb = boto3.client('dynamodb')

def lambda_handler(event, context):
    try:
        student_id = event['studentID']  # Fetch student ID
        file_name = event['fileName']
        file_data = event['fileData'].split(',')[1]
        
         # Define the folder path
        folder_path = 'responses/'  # Change this according to your requirement
        
        # Upload file to S3
        s3.upload_fileobj(
            Fileobj=BytesIO(base64.b64decode(file_data)),
            Bucket='simassignmenteval1',
            Key=folder_path+file_name
        )
        
        # Store file details in DynamoDB
        timestamp = datetime.now().isoformat()
        dynamodb.put_item(
            TableName='datastore',
            Item={
                'fileId': {'S': file_name},
                'studentID': {'S': student_id},  # Partition key
                's3Location': {'S': f'https://simassignmenteval1.s3.us-east-1.amazonaws.com/{folder_path}{file_name}'},
                'uploadedAt': {'S': timestamp}
            }
        )
        
        return {'message': 'Data uploaded successfully'}  # Updated response message
    
    except Exception as e:
        print("Error:", e)
        raise Exception('Failed to upload data')  # Updated error message
