# AWS Developer Associate Study Notes 📚 ☁️

![AWS Developer Associate](https://img.shields.io/badge/AWS-Developer%20Associate-orange?style=for-the-badge&logo=amazon-aws)

> Last updated: April 2025  
> These notes cover key topics for the AWS Certified Developer - Associate exam (DVA-C02)

## Table of Contents
- [AWS Cognito](#aws-cognito)
  - [User Pools](#user-pools)
  - [Identity Pools](#identity-pools)
  - [Cognito vs IAM](#cognito-vs-iam)
- [AWS Lambda](#aws-lambda)
  - [Key Features](#key-features)
  - [Execution Model](#execution-model)
  - [Concurrency](#concurrency)
  - [Code Examples](#lambda-code-examples)
- [Amazon DynamoDB](#amazon-dynamodb)
  - [Key Concepts](#key-concepts)
  - [Capacity Modes](#capacity-modes)
  - [Indexes](#indexes)
  - [Code Examples](#dynamodb-code-examples)
- [Amazon S3](#amazon-s3)
  - [Storage Classes](#storage-classes)
  - [Security](#security)
  - [Performance](#performance)
  - [Code Examples](#s3-code-examples)
- [Amazon SQS](#amazon-sqs)
  - [Queue Types](#queue-types)
  - [Message Processing](#message-processing)
  - [Visibility Timeout](#visibility-timeout)
  - [Dead-Letter Queues](#dead-letter-queues)
  - [Code Examples](#sqs-code-examples)
- [AWS API Gateway](#aws-api-gateway)
  - [Endpoint Types](#endpoint-types)
  - [Integration Types](#integration-types)
  - [Security](#api-gateway-security)
- [Elastic Beanstalk](#elastic-beanstalk)
  - [Deployment Options](#deployment-options)
  - [Environment Types](#environment-types)
- [AWS CloudFormation](#aws-cloudformation)
  - [Template Structure](#template-structure)
  - [Intrinsic Functions](#intrinsic-functions)
- [AWS X-Ray](#aws-x-ray)
  - [Key Concepts](#x-ray-key-concepts)
  - [Instrumentation](#instrumentation)
- [Exam Preparation Tips](#exam-preparation-tips)
- [Additional Resources](#additional-resources)

---

## AWS Cognito

AWS Cognito is a fully managed service that provides authentication, authorization, and user management for web and mobile applications.

### User Pools

User Pools are user directories that provide:
- **Sign-in functionality** for app users
- **Integration with API Gateway & Application Load Balancer**
- Support for social identity providers like Facebook, Google, and Amazon
- Support for enterprise identity providers via SAML and OpenID Connect

### Identity Pools

Identity Pools (Federated Identities) provide:
- **AWS credentials to access AWS resources directly** from client applications. Situable AWS services such as **Amazon S3** and **DynamoDB**.
- Temporary AWS credentials with configurable permissions
- Support for anonymous guest access
- Seamless integration with User Pools

### Cognito vs IAM

| Feature | Cognito | IAM |
|---------|---------|-----|
| Primary Use | External users (B2C, B2B) | Internal users & services |
| Authentication | Supports SAML, social identity providers | AWS-specific authentication |
| Scale | Designed for millions of users | Better for smaller number of users |
| Management | Self-service user management | Admin-controlled |

---

## AWS Lambda

AWS Lambda is a serverless compute service that runs your code in response to events and automatically manages the underlying compute resources.

### Key Features

- **Serverless**: No server management required
- **Event-driven**: Functions are triggered by events
- **Pay-per-use**: Only pay for the compute time consumed
- **Automatic scaling**: Scales automatically with the size of the workload
- **Integrated security**: Integrated with IAM for secure access control
- **AWS SDK**: This provides the ability to programmatically work with AWS Services, like CodeCommit

### Execution Model

- **Cold Start**: First invocation or after a period of inactivity
- **Warm Start**: Subsequent invocations reuse the container
- **Execution Context**: Environment where your function runs. The context object in a Lambda function provides metadata about the function and the current invocation, included context.awsRequestId.
- **Handler Function**: Entry point for Lambda execution. Initialize SDK clients and database connections outside of the function handler,  take advantage of execution environment reuse.

### Concurrency

- **Reserved Concurrency**: Guarantees a set number of concurrent executions
- **Provisioned Concurrency**: Pre-initialized execution environments
- **Account Concurrency Limit**: Default 1,000 concurrent executions per region

### Lambda@Edge

- **Lambda@Edge**: functions can only be created in the us-east-1.

### <a name="lambda-code-examples"></a>Code Examples

**Node.js Lambda Function:**
```javascript
exports.handler = async (event) => {
    console.log('Event: ', JSON.stringify(event, null, 2));
    
    // Process the event
    const records = event.Records || [];
    const processedRecords = records.map(record => {
        // Process each record
        return {
            id: record.dynamodb?.NewImage?.Id?.S || 'unknown',
            processedAt: new Date().toISOString()
        };
    });
    
    return {
        statusCode: 200,
        body: JSON.stringify({
            message: 'Processing complete',
            processedItems: processedRecords.length,
            items: processedRecords
        })
    };
};
```

**Python Lambda Function:**
```python
import json
import logging
from datetime import datetime

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    logger.info(f"Event: {json.dumps(event)}")
    
    # Process the event
    records = event.get('Records', [])
    processed_records = []
    
    for record in records:
        # Process each record
        record_id = record.get('dynamodb', {}).get('NewImage', {}).get('Id', {}).get('S', 'unknown')
        processed_records.append({
            'id': record_id,
            'processedAt': datetime.now().isoformat()
        })
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': 'Processing complete',
            'processedItems': len(processed_records),
            'items': processed_records
        })
    }
```

---

## Amazon DynamoDB

Amazon DynamoDB is a fully managed NoSQL database service that provides fast and predictable performance with seamless scalability.

### Key Concepts

- **Tables**: Collection of items (similar to rows in relational databases)
- **Items**: Collection of attributes (similar to columns)
- **Primary Key**: Uniquely identifies each item in a table
  - **Partition Key**: Simple primary key
  - **Partition Key + Sort Key**: Composite primary key

### Capacity Modes

- **Provisioned Capacity**: Specify read and write capacity units
- **On-Demand Capacity**: Pay-per-request pricing
- **Auto Scaling**: Automatically adjust provisioned capacity

### Indexes

- **Local Secondary Index (LSI)**: Same partition key as table but different sort key
- **Global Secondary Index (GSI)**: Different partition key and optional sort key
- **Sparse Indexes**: Only contain items that have the indexed attribute

### <a name="dynamodb-code-examples"></a>Code Examples

**DynamoDB Operations with AWS SDK for JavaScript:**
```javascript
const AWS = require('aws-sdk');
const dynamoDB = new AWS.DynamoDB.DocumentClient();

// Put item
async function createItem(tableName, item) {
    const params = {
        TableName: tableName,
        Item: item
    };
    
    try {
        await dynamoDB.put(params).promise();
        return { success: true };
    } catch (error) {
        console.error('Error creating item:', error);
        return { success: false, error };
    }
}

// Get item
async function getItem(tableName, key) {
    const params = {
        TableName: tableName,
        Key: key
    };
    
    try {
        const result = await dynamoDB.get(params).promise();
        return result.Item;
    } catch (error) {
        console.error('Error getting item:', error);
        return null;
    }
}

// Query items
async function queryItems(tableName, keyConditionExpression, expressionAttributeValues) {
    const params = {
        TableName: tableName,
        KeyConditionExpression: keyConditionExpression,
        ExpressionAttributeValues: expressionAttributeValues
    };
    
    try {
        const result = await dynamoDB.query(params).promise();
        return result.Items;
    } catch (error) {
        console.error('Error querying items:', error);
        return [];
    }
}
```

---

## Amazon S3

Amazon Simple Storage Service (S3) is an object storage service offering industry-leading scalability, data availability, security, and performance.

### Storage Classes

- **S3 Standard**: General-purpose storage for frequently accessed data
- **S3 Intelligent-Tiering**: Automatic cost optimization
- **S3 Standard-IA**: Infrequent access, rapid retrieval
- **S3 One Zone-IA**: Lower cost for infrequently accessed data
- **S3 Glacier**: Low-cost archival storage
- **S3 Glacier Deep Archive**: Lowest cost for long-term retention

### Security

- **Bucket Policies**: Resource-based policies for S3 buckets
- **IAM Policies**: Identity-based policies for AWS users/roles
- **ACLs**: Legacy access control lists
- **Encryption**: Server-side (SSE-S3, SSE-KMS, SSE-C) and client-side encryption

### Performance

- **Multipart Upload**: Parallel uploads for large objects
- **Transfer Acceleration**: Fast, secure file transfers
- **S3 Select**: Retrieve only needed data from objects
- **Partitioning Strategy**: Optimize key names for performance

### <a name="s3-code-examples"></a>Code Examples

**S3 Operations with AWS SDK for JavaScript:**
```javascript
const AWS = require('aws-sdk');
const s3 = new AWS.S3();

// Upload file to S3
async function uploadFile(bucketName, key, body, contentType) {
    const params = {
        Bucket: bucketName,
        Key: key,
        Body: body,
        ContentType: contentType
    };
    
    try {
        const result = await s3.upload(params).promise();
        return result.Location;
    } catch (error) {
        console.error('Error uploading file:', error);
        return null;
    }
}

// Get file from S3
async function getFile(bucketName, key) {
    const params = {
        Bucket: bucketName,
        Key: key
    };
    
    try {
        const result = await s3.getObject(params).promise();
        return result.Body;
    } catch (error) {
        console.error('Error getting file:', error);
        return null;
    }
}

// Generate pre-signed URL
function generatePresignedUrl(bucketName, key, expirationSeconds = 3600) {
    const params = {
        Bucket: bucketName,
        Key: key,
        Expires: expirationSeconds
    };
    
    try {
        return s3.getSignedUrl('getObject', params);
    } catch (error) {
        console.error('Error generating pre-signed URL:', error);
        return null;
    }
}
```
---

## Amazon SQS

Amazon Simple Queue Service (SQS) is a fully managed message queuing service that enables you to decouple and scale microservices, distributed systems, and serverless applications.

### Queue Types

- **Standard Queues**: 
  - Nearly unlimited throughput
  - At-least-once delivery
  - Best-effort ordering
  - Suitable for most use cases where high throughput is required
  - ReceiveMessage API retrieves one or more messages

- **FIFO Queues**: 
  - First-In-First-Out delivery guarantee
  - Exactly-once processing
  - Limited to 300 transactions per second (TPS) per API method
  - Suitable for applications where order of operations is critical

### Message Processing

- **Message Lifecycle**:
  1. Producer sends message to queue
  2. Message is distributed across SQS servers
  3. Consumer polls queue and processes message
  4. Consumer deletes message from queue

- **Long Polling vs Short Polling**:
  - **Short Polling**: Returns immediately, even if queue is empty
  - **Long Polling**: Waits for messages to arrive (up to 20 seconds), reducing empty responses and API calls

### Visibility Timeout

- Period during which a message is invisible to other consumers after being retrieved
- Default: 30 seconds, Maximum: 12 hours
- Can be extended for messages that need longer processing time
- If not deleted before timeout expires, message becomes visible again

### Dead-Letter Queues

- Destination for messages that fail processing multiple times
- Useful for:
  - Isolating problematic messages for debugging
  - Analyzing failed message patterns
  - Implementing custom retry logic
- Configure with `RedrivePolicy` that specifies `maxReceiveCount`

### <a name="sqs-code-examples"></a>Code Examples

**SQS Operations with AWS SDK for JavaScript:**
```javascript
const AWS = require('aws-sdk');
const sqs = new AWS.SQS({ region: 'us-east-1' });

// Send message to SQS queue
async function sendMessage(queueUrl, messageBody, messageGroupId = null) {
    const params = {
        QueueUrl: queueUrl,
        MessageBody: JSON.stringify(messageBody),
        // Required for FIFO queues
        ...(messageGroupId && { MessageGroupId: messageGroupId }),
        // Optional message deduplication ID for FIFO queues
        ...(messageGroupId && { MessageDeduplicationId: `${Date.now()}-${Math.random()}` })
    };
    
    try {
        const result = await sqs.sendMessage(params).promise();
        console.log('Message sent successfully:', result.MessageId);
        return result.MessageId;
    } catch (error) {
        console.error('Error sending message:', error);
        throw error;
    }
}

// Receive messages from SQS queue
async function receiveMessages(queueUrl, maxMessages = 10, waitTimeSeconds = 20) {
    const params = {
        QueueUrl: queueUrl,
        MaxNumberOfMessages: maxMessages,
        WaitTimeSeconds: waitTimeSeconds, // Long polling
        VisibilityTimeout: 30
    };
    
    try {
        const result = await sqs.receiveMessage(params).promise();
        return result.Messages || [];
    } catch (error) {
        console.error('Error receiving messages:', error);
        return [];
    }
}

// Delete message from SQS queue
async function deleteMessage(queueUrl, receiptHandle) {
    const params = {
        QueueUrl: queueUrl,
        ReceiptHandle: receiptHandle
    };
    
    try {
        await sqs.deleteMessage(params).promise();
        console.log('Message deleted successfully');
        return true;
    } catch (error) {
        console.error('Error deleting message:', error);
        return false;
    }
}

// Change message visibility timeout
async function changeMessageVisibility(queueUrl, receiptHandle, visibilityTimeout) {
    const params = {
        QueueUrl: queueUrl,
        ReceiptHandle: receiptHandle,
        VisibilityTimeout: visibilityTimeout
    };
    
    try {
        await sqs.changeMessageVisibility(params).promise();
        console.log(`Visibility timeout changed to ${visibilityTimeout} seconds`);
        return true;
    } catch (error) {
        console.error('Error changing message visibility:', error);
        return false;
    }
}
```

**SQS Operations with AWS SDK for Python:**
```python
import json
import boto3
import uuid
from datetime import datetime

# Initialize SQS client
sqs = boto3.client('sqs', region_name='us-east-1')

# Send message to SQS queue
def send_message(queue_url, message_body, message_group_id=None):
    params = {
        'QueueUrl': queue_url,
        'MessageBody': json.dumps(message_body)
    }
    
    # Add FIFO queue specific parameters if needed
    if message_group_id:
        params['MessageGroupId'] = message_group_id
        params['MessageDeduplicationId'] = f"{datetime.now().timestamp()}-{uuid.uuid4()}"
    
    try:
        response = sqs.send_message(**params)
        print(f"Message sent successfully: {response['MessageId']}")
        return response['MessageId']
    except Exception as e:
        print(f"Error sending message: {e}")
        raise e

# Receive messages from SQS queue
def receive_messages(queue_url, max_messages=10, wait_time_seconds=20):
    try:
        response = sqs.receive_message(
            QueueUrl=queue_url,
            MaxNumberOfMessages=max_messages,
            WaitTimeSeconds=wait_time_seconds,  # Long polling
            VisibilityTimeout=30
        )
        return response.get('Messages', [])
    except Exception as e:
        print(f"Error receiving messages: {e}")
        return []

# Delete message from SQS queue
def delete_message(queue_url, receipt_handle):
    try:
        sqs.delete_message(
            QueueUrl=queue_url,
            ReceiptHandle=receipt_handle
        )
        print("Message deleted successfully")
        return True
    except Exception as e:
        print(f"Error deleting message: {e}")
        return False

# Process messages from SQS queue
def process_messages(queue_url):
    messages = receive_messages(queue_url)
    
    for message in messages:
        try:
            # Parse message body
            body = json.loads(message['Body'])
            print(f"Processing message: {body}")
            
            # Process the message (replace with your business logic)
            # ...
            
            # Delete the message after successful processing
            delete_message(queue_url, message['ReceiptHandle'])
        except Exception as e:
            print(f"Error processing message: {e}")
            # Consider extending visibility timeout if processing failed
            # but might succeed with more time
```

---

## EBS

Amazon EBS is permanent, mountable storage solution for Amazon EC2 instances.

### Encryption

Enable encription on the beggining. EBS volumes are not encrypted by default without additional configuration.


---

## AWS API Gateway

Amazon API Gateway is a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs.

### Endpoint Types

- **Edge-Optimized**: For global clients, routed through CloudFront
- **Regional**: For clients in the same region
- **Private**: Accessible only within a VPC

### Integration Types

- **Lambda Function**: Direct integration with AWS Lambda
- **HTTP**: Integration with HTTP endpoints
- **AWS Service**: Integration with AWS services
- **Mock**: Return static responses without backend integration
- **VPC Link**: Access resources in a VPC

### <a name="api-gateway-security"></a>Security

- **CORS**: Enable CORS for the method in API GW if you request from other domain like S3 bucket.
- **IAM Roles and Policies**: Control access using IAM
- **Lambda Authorizers**: Custom authorization with Lambda functions. It is an API Gateway feature that uses a Lambda function to control access to your API.
- **Cognito User Pools**: User authentication with Cognito
- **API Keys**: Identify and authorize API clients
- **Resource Policies**: Control who can invoke the API

---

## Elastic Beanstalk

AWS Elastic Beanstalk is an easy-to-use service for deploying and scaling web applications and services.

### Deployment Options

- **All at once**: Deploy to all instances simultaneously
- **Rolling**: Deploy in batches
- **Rolling with additional batch**: Launch new instances in batches
- **Immutable**: Launch a full set of new instances
- **Blue/Green**: Create a new environment and switch traffic

### Environment Types

- **Web Server Environment**: For traditional web applications
- **Worker Environment**: For background processing tasks

---

## AWS CloudFormation

AWS CloudFormation provides a common language to describe and provision all the infrastructure resources in your cloud environment.

### Template Structure

- **Format Version**: Template format version
- **Description**: Template description
- **Metadata**: Additional information about the template
- **Parameters**: Values to pass to your template at runtime
- **Mappings**: Key-value mappings for conditional parameter values
- **Conditions**: Conditions that control resource creation
- **Transform**: Specifies macros that transform the template
- **Resources**: Stack resources and their properties
- **Outputs**: Values that are returned when viewing stack properties

### Intrinsic Functions

- **Fn::GetAtt**: Returns the value of an attribute from a resource
- **Fn::Join**: Joins values with a delimiter
- **Fn::Sub**: Substitutes variables in an input string
- **Ref**: Returns the value of the specified parameter or resource
- **Fn::ImportValue**: Returns the value of an output exported by another stack

---

## AWS X-Ray

AWS X-Ray helps developers analyze and debug distributed applications, providing insights into how your application and its underlying services are performing.

### <a name="x-ray-key-concepts"></a>Key Concepts

- **Segments**: Data about the work done by your application
- **Subsegments**: More granular timing information
- **Traces**: Collection of segments generated by a single request
- **Service Graph**: JSON representation of services and connections
- **Sampling**: Control the amount of data recorded

### Instrumentation

- **X-Ray SDK**: Instrument your application code
- **X-Ray Daemon**: Collects and buffers segments before sending to X-Ray
- **X-Ray API**: Send trace data directly to X-Ray
- **Integration with AWS services**: Automatic instrumentation

---

## Exam Preparation Tips

1. **Focus on hands-on experience**:
   - Set up a free tier AWS account
   - Practice with key services (Lambda, DynamoDB, S3, API Gateway)
   - Build a simple serverless application

2. **Understand service integrations**:
   - How services work together
   - Common architectures and patterns
   - Service limits and constraints

3. **Master IAM concepts**:
   - Roles, policies, and permissions
   - Security best practices
   - Least privilege principle

4. **Review AWS documentation**:
   - Developer guides
   - FAQs for key services
   - Whitepapers

5. **Take practice exams**:
   - Identify knowledge gaps
   - Practice time management
   - Get comfortable with question formats

---

## Additional Resources

- [AWS Documentation](https://docs.aws.amazon.com/)
- [AWS Developer Center](https://aws.amazon.com/developers/)
- [AWS Skill Builder](https://skillbuilder.aws/)
- [AWS Certification](https://aws.amazon.com/certification/)
- [AWS Whitepapers](https://aws.amazon.com/whitepapers/)
- [AWS Samples on GitHub](https://github.com/aws-samples)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)

---

> 💡 **Pro Tip**: Focus on understanding the "why" behind AWS services and best practices, not just memorizing facts. The exam tests your ability to apply knowledge to real-world scenarios.
