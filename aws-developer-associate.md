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
- [AWS KMS](#aws-kms)
  - [Key Concepts](#kms-key-concepts)
  - [Key Types](#key-types)
  - [Key Policies](#key-policies)
  - [Integration with AWS Services](#kms-integration)
  - [Code Examples](#kms-code-examples)
- [Amazon EBS](#ebs)
  - [Encryption](#encryption)
- [AWS API Gateway](#aws-api-gateway)
  - [Endpoint Types](#endpoint-types)
  - [Integration Types](#integration-types)
  - [Security](#api-gateway-security)
- [Elastic Load Balancing (ELB)](#elastic-load-balancing-elb)
  - [ELB Types](#elb-types)
  - [Key Features](#elb-key-features)
- [Elastic Beanstalk](#elastic-beanstalk)
  - [Deployment Options](#deployment-options)
  - [Environment Types](#environment-types)
- [AWS CloudFormation](#aws-cloudformation)
  - [Template Structure](#template-structure)
  - [Intrinsic Functions](#intrinsic-functions)
- [AWS CodeDeploy](#aws-codedeploy)
  - [Deployment Types](#deployment-types)
  - [EC2 Deployment Hooks](#ec2-deployment-hooks)
  - [Lambda Deployment Hooks](#lambda-deployment-hooks)
  - [ECS Deployment Hooks](#ecs-deployment-hooks)
  - [EKS Deployment Hooks](#eks-deployment-hooks)
  - [Code Examples](#codedeploy-code-examples)
- [AWS X-Ray](#aws-x-ray)
  - [Key Concepts](#x-ray-key-concepts)
  - [Instrumentation](#instrumentation)
- [AWS CloudWatch](#aws-cloudwatch)
  - [Metrics](#metrics)
  - [Alarms](#alarms)
  - [Logs](#logs)
  - [Events](#events)
  - [Code Examples](#cloudwatch-code-examples)
- [Exam Preparation Tips](#exam-preparation-tips)
- [AWS Step Functions](#aws-step-functions)
  - [State Types](#state-types)
  - [Execution Models](#execution-models)
  - [Error Handling](#step-functions-error-handling)
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
- Provides hosted web ui

### Identity Pools

Identity Pools (Federated Identities) provide:
- **AWS credentials to access AWS resources directly** from client applications. Situable AWS services such as **Amazon S3** and **DynamoDB**.
- Temporary AWS credentials with configurable permissions
- Support for anonymous guest access
- Seamless integration with User Pools
- With developer authenticated identities, you can register and authenticate users via your own existing authentication process

### Cognito vs IAM

| Feature | Cognito | IAM |
|---------|---------|-----|
| Primary Use | External users (B2C, B2B) | Internal users & services |
| Authentication | Supports SAML, social identity providers | AWS-specific authentication |
| Scale | Designed for millions of users | Better for smaller number of users |
| Management | Self-service user management | Admin-controlled |

### Operations

- **Cognito Sync**: Synchronize user profile data across mobile devices and the web without requiring your own backend

---

## AWS Lambda

AWS Lambda is a serverless compute service that runs your code in response to events and automatically manages the underlying compute resources.

### Key Features

- **Serverless**: No server management required
- **Event-driven**: Functions are triggered by events
- **Pay-per-use**: Only pay for the compute time consumed
- **Automatic scaling**: Scales automatically with the size of the workload
- **Integrated security**: Integrated with IAM for secure access control
- **Integrated loging**: Lambda automatically integrates with CloudWatch Logs (Needs proper role)
- **AWS SDK**: This provides the ability to programmatically work with AWS Services, like CodeCommit
- **Alias**: Permits traffic shifting to assign a percentage of traffic to the new version

### Execution Model

- **Cold Start**: First invocation or after a period of inactivity
- **Warm Start**: Subsequent invocations reuse the container
- **Execution Context**: Environment where your function runs. The context object in a Lambda function provides metadata about the function and the current invocation, included context.awsRequestId.
- **Handler Function**: Entry point for Lambda execution. Initialize SDK clients and database connections outside of the function handler, take advantage of execution environment reuse.

### Concurrency

- **Reserved Concurrency**: Guarantees a set number of concurrent executions
- **Provisioned Concurrency**: Pre-initialized execution environments
- **Account Concurrency Limit**: Default 1,000 concurrent executions per region

### Integration

- **Encryption helpers**: Environment variables to store secrets securely for use with Lambda functions. Lambda always encrypts environment variables at rest.
- **Event source mapping**: AWS Lambda resource that reads from an event source and invokes a Lambda function. Services supported Amazon Kinesis, Amazon DynamoDB and Amazon Simple Queue Service.
- **Lambda@Edge**: functions can only be created in the us-east-1.

### Troubleshooting

- **Maximum execution**: 900 seconds

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
  - **Partition Key + Sort Key**: Composite primary key. Sort key ideally to group related items using range queries.

### Capacity Modes

- **Provisioned Capacity**: Specify read and write capacity units
- **On-Demand Capacity**: Pay-per-request pricing
- **Auto Scaling**: Automatically adjust provisioned capacity

### Indexes

- **Local Secondary Index (LSI)**: Same partition key as table but different sort key. Consume capacity units from the base table since they share the same partition key. 
- **Global Secondary Index (GSI)**: Different partition key and optional sort key. Contains a set of attributes from the base table, but organized by a different primary key than the table. Need provision read and write capacity units.
- **Sparse Indexes**: Only contain items that have the indexed attribute

### Operations

- **BatchGetItem**: Permits the retrieval of multiple items from one or more tables in a single operation
- **Parallel Scan API**: Scan is distributed across your table’s partitions
- **Limit parameter**: To control the amount of data returned per request, prevent throttling

### Security

- **IAM policy**: can restrict access to individual items in a table, access to the attributes in those items, or both at the same time.
- **IAM Condition**: can allow or deny access to items and attributes in DynamoDB tables, indexes and key values.
- **Read-only access**: To certain items and attributes in a table or a secondary index
- **write-only access**: To certain attributes in a table, based upon the identity of that user

---

### Provisioned Capacity

- **RCU**:
    - 1 RCU per 2 eventually consistent read up to 4KB.
    - 1 RCU per strongly consistent read up to 4KB.
    - 2 RCU per transactional read request up to 4KB.

- **WCU**: 
    - 1 WCU per standard write request up to 1KB/second.
    - 2 WCU per transactional write request up to 1KB/second.

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
- **throttling**: 3,500 PUT/COPY/POST/DELETE or 5,500 GET/HEAD requests per second per prefix

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
  - **Long Polling**: Waits for messages to arrive (up to 20 seconds), reducing empty responses and API calls. ReceiveMessageWaitTimeSeconds attribute to 20 seconds.

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

### Operations

- **VPC endpoint**: Communicate without traversing the public Internet
- **Max message size**: Only supports messages up to 256KB in size
- **Deduplication**: FIFO queue and configure the Lambda function to add a message deduplication token to the message body

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

## AWS KMS

AWS Key Management Service (KMS) is a managed service that makes it easy for you to create and control the encryption keys used to encrypt your data.

### <a name="kms-key-concepts"></a>Key Concepts

- **Customer Master Keys (CMKs)**: Primary resources in AWS KMS
  - **AWS Managed CMKs**: Created, managed, and used on your behalf by AWS services
  - **Customer Managed CMKs**: Created, managed, and used by you
  - **AWS Owned CMKs**: Collection of CMKs that AWS services own and manage for use in multiple AWS accounts
- **Key Material**: The cryptographic material used for encryption and decryption
  - Can be generated by AWS KMS or imported by you
- **Key Rotation**: Automatic or manual rotation of key material
  - AWS Managed CMKs: Automatically rotated every year
  - Customer Managed CMKs: Optional automatic rotation every year
- **Aliases**: Friendly names for your CMKs
- **Key Identifiers**: Used to identify CMKs (Key ID, Key ARN, Alias name, Alias ARN)

### Key Types

- **Symmetric CMKs**: 256-bit AES-GCM keys for encryption and decryption
  - Most common and default type
  - Key material never leaves AWS KMS unencrypted
- **Asymmetric CMKs**: RSA or ECC key pairs for encryption/decryption or signing/verification
  - Public key can be downloaded and used outside AWS
  - Private key remains in AWS KMS
- **HMAC CMKs**: Keys for generating and verifying hash-based message authentication codes

### Key Policies

- **Resource-based policies**: Control access to the CMK
  - Every CMK has exactly one key policy
  - Default key policy gives the AWS account full access to the CMK
- **IAM policies**: Can be used in combination with key policies
- **Grants**: Delegate permissions to AWS services or users
  - More granular and temporary access
  - Used for programmatic access to CMKs

### <a name="kms-integration"></a>Integration with AWS Services

- **S3**: Server-side encryption with KMS keys (SSE-KMS)
- **EBS**: Encryption of volumes
- **RDS/Aurora**: Encryption of database instances
- **DynamoDB**: Encryption at rest
- **Lambda**: Environment variable encryption
- **CloudTrail**: Log file encryption
- **Secrets Manager**: Encryption of secrets
- **Systems Manager Parameter Store**: Encryption of secure string parameters

### Troubleshooting

- **ThrottlingException**: Data key caching can improve performance, reduce cost, and help you stay within service limits as your application scales ( LocalCryptoMaterialsCache). Additionally, the developer can request an increase in the quota for AWS KMS.



---

## <a name="ebs"></a>EBS

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

### Troubleshooting

- **Cache**: The client must send a request that contains the Cache-Control: max-age=0 header.
- **$connect and $disconnect**: these routes capture events when a user connects and disconnects from the WebSocket API.
---

## <a name="elastic-load-balancing"></a>Elastic Load Balancing (ELB)

Elastic Load Balancing automatically distributes incoming application traffic across multiple targets, such as EC2 instances, containers, and IP addresses.

### <a name="elb-types"></a>ELB Types

- **Application Load Balancer (ALB)**:
  - **Layer 7** (HTTP/HTTPS) load balancer
  - **Path-based routing**: Routes based on URL path
  - **Host-based routing**: Routes based on host header
  - **Support for WebSockets** and HTTP/2
  - **Target groups**: EC2 instances, ECS tasks, Lambda functions, IP addresses
  - **Health checks**: Application-level health monitoring
  - **Security**: Integrates with AWS WAF and AWS Shield

- **Network Load Balancer (NLB)**:
  - **Layer 4** (TCP/UDP/TLS) load balancer
  - **Ultra-high performance**: Millions of requests per second
  - **Low latency**: ~100ms (vs ~400ms for ALB)
  - **Static IP addresses**: One per AZ, supports Elastic IPs
  - **Preserve client IP**: Client IP is preserved
  - **Target groups**: EC2 instances, IP addresses, ALB
  - **Health checks**: TCP, HTTP/HTTPS health checks

- **Gateway Load Balancer (GWLB)**:
  - **Layer 3/4** (IP packets) load balancer
  - **For third-party virtual appliances**: Firewalls, IDS/IPS, deep packet inspection
  - **Uses GENEVE protocol** (port 6081)
  - **Transparent network gateway**: Single entry/exit for traffic
  - **Load balancer**: Distributes traffic to virtual appliances
  - **Target groups**: EC2 instances, IP addresses

- **Classic Load Balancer (CLB)** - *Previous generation*:
  - **Layer 4/7** load balancer
  - **Basic load balancing** for HTTP/HTTPS/TCP
  - **Limited features**: No advanced routing, no target groups, host or path-based routing
  - **Legacy support**: For EC2-Classic network

### <a name="elb-key-features"></a>Key Features

- **Cross-Zone Load Balancing**: Distributes traffic evenly across all registered targets in all AZs
- **Sticky Sessions**: Routes requests from the same client to the same target
- **Connection Draining**: Allows in-flight requests to complete before deregistering targets
- **Health Checks**: Monitors the health of registered targets
- **SSL/TLS Termination**: Decrypts HTTPS traffic before sending to targets
- **SNI (Server Name Indication)**: Supports multiple TLS certificates on a single listener
- **Access Logs**: Detailed information about requests (stored in S3)
- **Integration**: Works with Auto Scaling, CloudWatch, AWS Certificate Manager

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

### Troubleshooting

- **.ebextensions**: Place yaml and json files with .config extension for the EB configuration.

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

## AWS CodeDeploy

AWS CodeDeploy is a fully managed deployment service that automates software deployments to various compute services such as Amazon EC2, AWS Lambda, Amazon ECS, and Kubernetes.

### Deployment Types

- **In-place Deployment**: The application on each instance in the deployment group is stopped, the latest application revision is installed, and the new version of the application is started and validated.
- **Blue/Green Deployment**: Instances are provisioned for the replacement environment, and the latest application revision is installed. Traffic is then rerouted to the replacement instances, and the original instances can be terminated.

### EC2 Deployment Hooks

EC2 deployments with CodeDeploy use the following lifecycle hooks in this order:

1. **ApplicationStop**: Stops any running application to prepare for deployment.
2. **DownloadBundle**: CodeDeploy agent copies the application revision files to a temporary location.
3. **BeforeInstall**: Pre-installation scripts or tasks (e.g., backing up current version, decrypting files).
4. **Install**: Copies application revision files to final location.
5. **AfterInstall**: Post-installation tasks (e.g., configuration, permissions changes).
6. **ApplicationStart**: Starts the deployed application.
7. **ValidateService**: Tests and validates that the application is working correctly.

### Lambda Deployment Hooks

Lambda deployments use a specific set of lifecycle hooks that execute in a defined order:

1. **BeforeAllowTraffic**: Runs before traffic is shifted to the deployed Lambda function version.
2. **AllowTraffic**: CodeDeploy shifts traffic to the new Lambda function version.
3. **AfterAllowTraffic**: Runs after all traffic is shifted to the deployed Lambda function version.


**Traffic Shifting Configurations for Lambda:**

- **Canary**: Traffic is shifted in two increments. You can specify the percentage of traffic shifted to the updated Lambda function version in the first increment and the interval, in minutes, before the remaining traffic is shifted.
- **Linear**: Traffic is shifted in equal increments with an equal number of minutes between each increment.
- **All-at-once**: All traffic is shifted from the original Lambda function to the updated Lambda function version at once.

### ECS Deployment Hooks

ECS deployments with CodeDeploy use the following lifecycle hooks in this order:

1. **BeforeInstall**: Runs before the replacement task set is created.
2. **Install**: CodeDeploy creates the replacement task set.
3. **AfterInstall**: Runs after the replacement task set is created and one of the target groups is associated with it.
4. **BeforeAllowTraffic**: Runs before traffic is rerouted to the replacement task set.
5. **AllowTraffic**: CodeDeploy reroutes traffic to the replacement task set.
6. **AfterAllowTraffic**: Runs after traffic is rerouted to the replacement task set.


**Traffic Shifting Configurations for ECS:**

- **Canary**: Traffic is shifted in two increments.
- **Linear**: Traffic is shifted in equal increments with equal time intervals.
- **All-at-once**: All traffic is shifted at once.

### EKS Deployment Hooks

For EKS deployments, CodeDeploy uses the following lifecycle hooks in this order:

1. **BeforeInstall**: Runs before Kubernetes applies the deployment YAML.
2. **Install**: CodeDeploy applies the Kubernetes deployment YAML.
3. **AfterInstall**: Runs after the Kubernetes deployment is created but before any traffic shifting.
4. **BeforeAllowTraffic**: Runs before traffic is shifted to the new Kubernetes deployment.
5. **AllowTraffic**: CodeDeploy begins shifting traffic according to the deployment configuration.
6. **AfterAllowTraffic**: Runs after all traffic has been shifted to the new Kubernetes deployment.


### <a name="codedeploy-code-examples"></a>Code Examples

**AppSpec File for EC2 Deployment:**
```yaml
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html/
hooks:
  ApplicationStop:
    - location: scripts/stop_application.sh
      timeout: 300
      runas: root
  BeforeInstall:
    - location: scripts/before_install.sh
      timeout: 300
      runas: root
  AfterInstall:
    - location: scripts/after_install.sh
      timeout: 300
      runas: root
  ApplicationStart:
    - location: scripts/start_application.sh
      timeout: 300
      runas: root
  ValidateService:
    - location: scripts/validate_service.sh
      timeout: 300
      runas: root
```

**AppSpec File for Lambda Deployment:**
```yaml
version: 0.0
Resources:
  - myLambdaFunction:
      Type: AWS::Lambda::Function
      Properties:
        Name: myLambdaFunction
        Alias: myLambdaFunctionAlias
        CurrentVersion: 1
        TargetVersion: 2
Hooks:
  BeforeAllowTraffic: !Ref BeforeAllowTrafficHookFunctionArn
  AfterAllowTraffic: !Ref AfterAllowTrafficHookFunctionArn
```

**AppSpec File for ECS Deployment:**
```yaml
version: 0.0
Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: <TASK_DEFINITION>
        LoadBalancerInfo:
          ContainerName: "sample-app"
          ContainerPort: 80
        PlatformVersion: "LATEST"
Hooks:
  - BeforeInstall: "LambdaFunctionToValidateBeforeInstall"
  - AfterInstall: "LambdaFunctionToValidateAfterInstall"
  - AfterAllowTraffic: "LambdaFunctionToValidateAfterAllowTraffic"
```

**AppSpec File for EKS Deployment:**
```yaml
version: 0.0
Resources:
  - TargetService:
      Type: AWS::EKS::Service
      Properties:
        Name: my-deployment
        Namespace: default
Hooks:
  - BeforeInstall: "LambdaFunctionToValidateBeforeInstall"
  - AfterInstall: "LambdaFunctionToValidateAfterInstall"
  - BeforeAllowTraffic: "LambdaFunctionToValidateBeforeAllowTraffic"
  - AfterAllowTraffic: "LambdaFunctionToValidateAfterAllowTraffic"
```

---

## AWS X-Ray

AWS X-Ray helps developers analyze and debug distributed applications, providing insights into how your application and its underlying services are performing.

### <a name="x-ray-key-concepts"></a>Key Concepts

- **Segments**: Data about the work done by your application. Use PutTraceSegments API to send segments. GetTraceSummaries API to search associated segments.
- **Subsegments**: More granular timing information
- **Traces**: Collection of segments generated by a single request
- **Service Graph**: JSON representation of services and connections
- **Sampling**: Control the amount of data recorded
- **Annotations and metadata**: You can add annotations and metadata to the segments that the X-Ray SDK creates, or to custom subsegments that you create (Metadata not indexed)

### Instrumentation

- **X-Ray SDK**: Instrument your application code
- **X-Ray Daemon**: Collects and buffers segments before sending to X-Ray
- **X-Ray API**: Send trace data directly to X-Ray
- **Integration with AWS services**: Automatic instrumentation

---

## AWS CloudWatch

Amazon CloudWatch is a monitoring and observability service that provides data and actionable insights for AWS resources and applications.

### Metrics

- **Standard Metrics**: Automatically collected for AWS services
- **Custom Metrics**: User-defined metrics for applications or business data
- **Metric Resolution**: Standard (1-minute) or High Resolution (1-second)
- **Metric Math**: Perform calculations across multiple metrics
- **Metric Retention**: 15 months by default, with different aggregation levels

### Alarms

- **Metric Alarms**: Monitor a single CloudWatch metric or expression
- **Composite Alarms**: Monitor multiple alarms using logical operators (AND, OR)
- **Alarm States**: OK, ALARM, INSUFFICIENT_DATA
- **Alarm Actions**:
  - Send notifications via SNS
  - Perform Auto Scaling actions
  - Execute EC2 actions (stop, terminate, reboot, recover)

### Logs

- **Log Groups**: Collections of log streams sharing the same retention and permissions
- **Log Streams**: Sequences of log events from the same source
- **Log Insights**: Query and analyze log data
- **Subscription Filters**: Route log data to other services (Lambda, Kinesis)
- **Metric Filters**: Extract metrics from log events
- **On-Premises Usage**: Need to install CW Agent and set IAM credentials

### Events

- **CloudWatch Events/EventBridge**: Near real-time stream of system events
- **Event Patterns**: Match events based on their content
- **Scheduled Events**: Trigger actions on a schedule
- **Targets**: Services that can receive events (Lambda, SNS, SQS, etc.)

### <a name="cloudwatch-code-examples"></a>Code Examples

**CloudWatch Operations with AWS SDK for JavaScript:**
```javascript
const AWS = require('aws-sdk');
const cloudWatch = new AWS.CloudWatch({ region: 'us-east-1' });

// Put custom metric data
async function putMetricData(namespace, metricName, value, dimensions) {
    const params = {
        Namespace: namespace,
        MetricData: [
            {
                MetricName: metricName,
                Value: value,
                Dimensions: dimensions.map(d => ({ Name: d.name, Value: d.value })),
                Timestamp: new Date(),
                Unit: 'Count' // Can be Count, Seconds, Microseconds, Milliseconds, Bytes, Kilobytes, etc.
            }
        ]
    };
    
    try {
        await cloudWatch.putMetricData(params).promise();
        console.log(`Successfully published metric ${metricName} to ${namespace}`);
        return true;
    } catch (error) {
        console.error('Error publishing metric data:', error);
        return false;
    }
}

// Create or update an alarm
async function createAlarm(alarmName, metricName, namespace, threshold, comparisonOperator, dimensions) {
    const params = {
        AlarmName: alarmName,
        MetricName: metricName,
        Namespace: namespace,
        Statistic: 'Average', // Can be SampleCount, Average, Sum, Minimum, Maximum
        Dimensions: dimensions.map(d => ({ Name: d.name, Value: d.value })),
        Period: 60, // Seconds
        EvaluationPeriods: 1,
        Threshold: threshold,
        ComparisonOperator: comparisonOperator, // GreaterThanThreshold, GreaterThanOrEqualToThreshold, etc.
        AlarmActions: [
            'arn:aws:sns:us-east-1:123456789012:my-alarm-topic' // Replace with your SNS topic ARN
        ]
    };
    
    try {
        await cloudWatch.putMetricAlarm(params).promise();
        console.log(`Successfully created/updated alarm ${alarmName}`);
        return true;
    } catch (error) {
        console.error('Error creating/updating alarm:', error);
        return false;
    }
}

// Get metric statistics
async function getMetricStatistics(namespace, metricName, dimensions, startTime, endTime) {
    const params = {
        Namespace: namespace,
        MetricName: metricName,
        Dimensions: dimensions.map(d => ({ Name: d.name, Value: d.value })),
        StartTime: startTime,
        EndTime: endTime,
        Period: 60, // Seconds
        Statistics: ['Average', 'Maximum', 'Minimum', 'Sum', 'SampleCount']
    };
    
    try {
        const result = await cloudWatch.getMetricStatistics(params).promise();
        return result.Datapoints;
    } catch (error) {
        console.error('Error getting metric statistics:', error);
        return [];
    }
}
```

**CloudWatch Logs Operations with AWS SDK for JavaScript:**
```javascript
const AWS = require('aws-sdk');
const cloudWatchLogs = new AWS.CloudWatchLogs({ region: 'us-east-1' });

// Create a log group
async function createLogGroup(logGroupName) {
    const params = {
        logGroupName: logGroupName
    };
    
    try {
        await cloudWatchLogs.createLogGroup(params).promise();
        console.log(`Successfully created log group ${logGroupName}`);
        return true;
    } catch (error) {
        if (error.code === 'ResourceAlreadyExistsException') {
            console.log(`Log group ${logGroupName} already exists`);
            return true;
        }
        console.error('Error creating log group:', error);
        return false;
    }
}

// Create a log stream
async function createLogStream(logGroupName, logStreamName) {
    const params = {
        logGroupName: logGroupName,
        logStreamName: logStreamName
    };
    
    try {
        await cloudWatchLogs.createLogStream(params).promise();
        console.log(`Successfully created log stream ${logStreamName} in group ${logGroupName}`);
        return true;
    } catch (error) {
        if (error.code === 'ResourceAlreadyExistsException') {
            console.log(`Log stream ${logStreamName} already exists in group ${logGroupName}`);
            return true;
        }
        console.error('Error creating log stream:', error);
        return false;
    }
}

// Put log events
async function putLogEvents(logGroupName, logStreamName, logEvents) {
    // Get the sequence token if the stream already has events
    let sequenceToken;
    try {
        const streams = await cloudWatchLogs.describeLogStreams({
            logGroupName: logGroupName,
            logStreamNamePrefix: logStreamName
        }).promise();
        
        if (streams.logStreams.length > 0 && streams.logStreams[0].uploadSequenceToken) {
            sequenceToken = streams.logStreams[0].uploadSequenceToken;
        }
    } catch (error) {
        console.error('Error getting sequence token:', error);
    }
    
    // Format log events
    const formattedEvents = logEvents.map(event => ({
        message: typeof event === 'string' ? event : JSON.stringify(event),
        timestamp: new Date().getTime()
    }));
    
    // Put log events
    const params = {
        logGroupName: logGroupName,
        logStreamName: logStreamName,
        logEvents: formattedEvents,
        ...(sequenceToken && { sequenceToken: sequenceToken })
    };
    
    try {
        const result = await cloudWatchLogs.putLogEvents(params).promise();
        console.log(`Successfully put ${formattedEvents.length} log events`);
        return result.nextSequenceToken;
    } catch (error) {
        console.error('Error putting log events:', error);
        return null;
    }
}
```

**CloudWatch Operations with AWS SDK for Python:**
```python
import boto3
import time
from datetime import datetime, timedelta

# Initialize CloudWatch client
cloudwatch = boto3.client('cloudwatch', region_name='us-east-1')
logs = boto3.client('logs', region_name='us-east-1')

# Put custom metric data
def put_metric_data(namespace, metric_name, value, dimensions):
    try:
        response = cloudwatch.put_metric_data(
            Namespace=namespace,
            MetricData=[
                {
                    'MetricName': metric_name,
                    'Value': value,
                    'Dimensions': [{'Name': d['name'], 'Value': d['value']} for d in dimensions],
                    'Timestamp': datetime.now(),
                    'Unit': 'Count'  # Can be Count, Seconds, Microseconds, Milliseconds, Bytes, Kilobytes, etc.
                }
            ]
        )
        print(f"Successfully published metric {metric_name} to {namespace}")
        return True
    except Exception as e:
        print(f"Error publishing metric data: {e}")
        return False

# Create or update an alarm
def create_alarm(alarm_name, metric_name, namespace, threshold, comparison_operator, dimensions):
    try:
        response = cloudwatch.put_metric_alarm(
            AlarmName=alarm_name,
            MetricName=metric_name,
            Namespace=namespace,
            Statistic='Average',  # Can be SampleCount, Average, Sum, Minimum, Maximum
            Dimensions=[{'Name': d['name'], 'Value': d['value']} for d in dimensions],
            Period=60,  # Seconds
            EvaluationPeriods=1,
            Threshold=threshold,
            ComparisonOperator=comparison_operator,  # GreaterThanThreshold, GreaterThanOrEqualToThreshold, etc.
            AlarmActions=[
                'arn:aws:sns:us-east-1:123456789012:my-alarm-topic'  # Replace with your SNS topic ARN
            ]
        )
        print(f"Successfully created/updated alarm {alarm_name}")
        return True
    except Exception as e:
        print(f"Error creating/updating alarm: {e}")
        return False

# Get metric statistics
def get_metric_statistics(namespace, metric_name, dimensions, start_time, end_time):
    try:
        response = cloudwatch.get_metric_statistics(
            Namespace=namespace,
            MetricName=metric_name,
            Dimensions=[{'Name': d['name'], 'Value': d['value']} for d in dimensions],
            StartTime=start_time,
            EndTime=end_time,
            Period=60,  # Seconds
            Statistics=['Average', 'Maximum', 'Minimum', 'Sum', 'SampleCount']
        )
        return response['Datapoints']
    except Exception as e:
        print(f"Error getting metric statistics: {e}")
        return []

# Create a log group
def create_log_group(log_group_name):
    try:
        logs.create_log_group(logGroupName=log_group_name)
        print(f"Successfully created log group {log_group_name}")
        return True
    except logs.exceptions.ResourceAlreadyExistsException:
        print(f"Log group {log_group_name} already exists")
        return True
    except Exception as e:
        print(f"Error creating log group: {e}")
        return False

# Create a log stream
def create_log_stream(log_group_name, log_stream_name):
    try:
        logs.create_log_stream(
            logGroupName=log_group_name,
            logStreamName=log_stream_name
        )
        print(f"Successfully created log stream {log_stream_name} in group {log_group_name}")
        return True
    except logs.exceptions.ResourceAlreadyExistsException:
        print(f"Log stream {log_stream_name} already exists in group {log_group_name}")
        return True
    except Exception as e:
        print(f"Error creating log stream: {e}")
        return False

# Put log events
def put_log_events(log_group_name, log_stream_name, log_events):
    # Get the sequence token if the stream already has events
    sequence_token = None
    try:
        streams = logs.describe_log_streams(
            logGroupName=log_group_name,
            logStreamNamePrefix=log_stream_name
        )
        
        if streams['logStreams'] and 'uploadSequenceToken' in streams['logStreams'][0]:
            sequence_token = streams['logStreams'][0]['uploadSequenceToken']
    except Exception as e:
        print(f"Error getting sequence token: {e}")
    
    # Format log events
    formatted_events = []
    for event in log_events:
        if isinstance(event, str):
            message = event
        else:
            import json
            message = json.dumps(event)
        
        formatted_events.append({
            'message': message,
            'timestamp': int(time.time() * 1000)
        })
    
    # Put log events
    params = {
        'logGroupName': log_group_name,
        'logStreamName': log_stream_name,
        'logEvents': formatted_events
    }
    
    if sequence_token:
        params['sequenceToken'] = sequence_token
    
    try:
        response = logs.put_log_events(**params)
        print(f"Successfully put {len(formatted_events)} log events")
        return response.get('nextSequenceToken')
    except Exception as e:
        print(f"Error putting log events: {e}")
        return None
```

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
## AWS Step Functions

AWS Step Functions is a serverless orchestration service that lets you combine AWS Lambda functions and other AWS services to build business-critical applications.

### State Types

- **Task**: Perform work using Lambda functions or AWS services
- **Choice**: Add branching logic based on input
- **Parallel**: Execute branches simultaneously
- **Map**: Process items in an array in parallel
- **Wait**: Pause execution for a specified time
- **Succeed/Fail**: Stop execution with success or failure
- **Pass**: Pass input to output or inject fixed data

### Execution Models

- **Standard Workflow**: Long-running, exactly-once execution, up to 1 year
- **Express Workflow**: Short-lived, at-least-once execution, up to 5 minutes
  - **Synchronous**: Wait for workflow to complete
  - **Asynchronous**: Don't wait for completion

### Step Functions Error Handling

- **Retry**: Automatic retry with exponential backoff
- **Catch**: Define fallback states when errors occur
- **States.ALL**: Catch all errors
- **States.Timeout**: Catch execution timeouts
- **Service Integration Patterns**: AWS SDK, optimized, and HTTP task integrations

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
