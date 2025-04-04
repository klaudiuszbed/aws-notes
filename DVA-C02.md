# DVA-C02 Notes
> Notes created during preparation for the DVA-C02 exam. Mostly based on Adrian Cantrill's [course](https://learn.cantrill.io).

#### Table of contents
- [1.0 IAM, ACCOUNTS AND AWS ORGANISATIONS](#10-iam-accounts-and-aws-organisations)
- [1.1 Security](#11-security)


# 1.0 IAM, ACCOUNTS AND AWS ORGANISATIONS 

### 1.0.1 Security Token Service (STS)
- Generates temporary credentials (sts:AssumeRole)
    - sts:AssumeRole must be used in order to get new credentials after current expires
- Used by IAM Roles for example
- Similar to access keys
    - AccessKeyId - unique id of credentials,
    - Expiration - when temporary credentials expire, 
    - Secret Access Key - used to sign requests,
    - SessionToken - unique token which must be passed with all requests
- Requested by an identity (AWS or External - role Trust Policy must be configured)

# 1.1 Security 

## 1.1.2 Policy Interpretation

- Learn and be aware of **not** clauses in policies.
- Each deny policy has to work with allow policy (by default, eveyrything is denied)

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyNonApprovedRegions",
            "Effect": "Deny",
            "NotAction": [
                "cloudfront:*",
                "iam:*",
                "route53:*",
                "support:*"
            ],
            "Resource": "*",
            "Condition": {
                "StringNotEquals": {
                    "aws:RequestedRegion": [
                        "ap-southeast-2",
                        "eu-west-1"
                    ]
                }
            }
        }
    ]
}
```

## 1.1.3 AWS Permissions Evaluation 
### Evaluation Logic
0. Explicit Deny
1. Organization SCPs
2. Resource Policies
3. IAM Identity Boundaries
4. Session Policies (when Role is used and it defines boundaries for session)
5. Identity Policies

# 1.2 Monitoring and Logging

## 1.2.1 CloudWatch Architecture

**A percentile** - it is a statistical measure that indicates the value below which a certain percentage of observations fall in a dataset. For example, the 90th percentile (P90) is the value below which 90% of the data points lie. Percentiles help analyze data distribution and are useful in identifying thresholds or performance benchmarks.

**Example**:

```
[40ms, 50ms, 60ms, 70ms, 80ms, 90ms, 100ms]
P50 = 70ms,
meaning that 50% of the data points are less than or equal to 70ms.
```

### 1.2.1.3 Data Structure
- **Namespace**: Container for metrics (AWS/EC2). AWS ones starts with AWS/
- **Metric**: Time ordered sets of data points. Each metric has a namespace. Each data point has
    - timestamp, 
    - value,
    - unit of measure (e.g. count)
- **Dimension**: Used for glanural representation of data. It's a key value pair. For example type=xlarge for grouping data points by specicic EC2 instance type.

### 1.2.1.2 Resolution and Retention
1) Standard Resolution (60s)
    - Data is being gathered every 60s
2) High Resolution (1s - 60s)
    - Data is being gathered from 1s to 60s

- Data with resolution **< 60s** is stored for **3h**.
- Data with resolution in **minutes** is stored for **15 days**.
- Data with resolution in **hours** is stored for **63 days**.
- Data with resolution in **days** is stored for **15 months**.

### 1.2.1.3 Alarms
- Resolution settings affect Alarm settings, meaning alarm can work in high or low resolution.
    - Might be important for example for billing alarms or critical workloads

### 1.2.1.4 CloudWatch Logs - Subscriptions Filters
Subscription filters can be used to send filtered log groups to a given destination. Valid destinations are:
- Kinesis Data Stream
- Lambda
- Amazon Data Firehose
- Amazon OpenSearch

Example of putting subscription filter on CloudTrail log group:
```
aws logs put-subscription-filter \
    --log-group-name "CloudTrail/logs" \
    --filter-name "RootAccess" \
    --filter-pattern "{$.userIdentity.type = Root}" \
    --destination-arn "arn:aws:kinesis:region:123456789012:stream/RootAccess" \
    --role-arn "arn:aws:iam::123456789012:role/CWLtoKinesisRole"
```

## 1.2.2 AWS X-Ray
It's designed for tracking the session of the application. Very good for distributed systems where you can track the flow between services.

- **Tracing Header**: it's unique ID generated on the first service which has X-Ray configured. Used to track the request across the distributed application.
- **Segments**: data blocks, request, response, detailed info, issues
    - **Sub-segments**: more granular version of regular segments
- **Service Graph**: JSON document detailing services and resources which make up the application
- **Service Map**: Visual version of service graph. Display traces.

### 1.2.3 **Integration with AWS services**:
Like any other service in the AWS, x-ray is no different and requires IAM Permissions to be configured.
- EC2 - requires x-ray agent
- ECS - installed on tasks
- Lambda - enable option
- Beanstalk - preinstalled
- API Gateway - per stage option
- SNS & SQS

## 1.2.3 VPC Flow Logs

- **Important**: Only capture packet metadata (source/dest IP and ports, packet size, id etc.), not packet 
contents!
- Can be attached: 
    - to a VPC - then they're attached to all ENIs inside VPC
    - to a subnet - then they're attached to all ENIs inside that Subnet 
    - directly to ENIs
- Flow logs are **not real-time!** 
- Can be sent to CloudWatch logs or S3 (or SAA-C03.md for querying)


# 1.3 AWS CLI, DEVELOPER TOOLS & CI/CD 

## 1.3.1 CI/CD using AWSCode(Commit/Build/Deploy)

- Two important files:
    - **buildspec.yml**: Used for AWS CodeBuild to define build and test commands.
    - **appspec.yml**: Used for AWS CodeDeploy to specify deployment instructions.
- CodeDeploy destinations:
    - EC2 instances, or more using deployment group,
    - ElasticBeanstalk or OpsWorks,
    - AWS CloudFormation (used to update or create a new stack),
    - Amazon ECS or ECS with blue/green deployment model,
    - AWS Service Catalog, Alexa Skills Kit,
    - S3

## 1.3.2 AWS CodeBuild
- CodeBuild does both building and testing. Test is essentialy a type of build. So when it says CodeBuild it also means test.
- It's Alternative to Jenkins

### 1.3.2.1 buildspec.yml
**REMEMBER:** The file HAS to be at the root of the source

- Has **four main stages**
    1. `install` (only for major env installations, not dependencies)
    2. `pre_build` (sign in to things or install dependencies)
    3. `build` (commands run during the building process)
    4. `post_build` (package things up, push the docker image)
- Can define **environment variables**: 
    - shell variables, 
    - regular env variables,
    - parameter-store, 
    - secrets-manager
- Artifacts - what stuff to put where. Where artifacts are stored and where

## 1.3.3 AWS CodeDeploy
- Alternative to CloudFormation (for code), Jenkins, Ansible, Chef
- Doesn't deploy resources. It deploys code and many more:
    - pre-built web apps,
    - exe files,
    - packages,
    - scripts,
    - configuration,
    - media
- Deploys to:
    - EC2 (agent required to communicate with the service and perform deployment),
    - Lambda,
    - ECS,
    - on-prem (agent required to communicate with the service and perform deployment)

### 1.3.3.1 appspec.yml/json
- It manages deployments + lifecycle hooks

#### 1.3.3.1.1 Lifecycle event hooks

ApplicationStop

    Runs before the existing application is stopped.
    Use: Clean up resources, back up files, or gracefully shut down services.

DownloadBundle

    Runs after the application is stopped and before the new files are downloaded.
    Use: Prepare the environment or remove old files.

BeforeInstall

    Runs before the new application is installed.
    Use: Set permissions, create directories, or configure prerequisites.

Install

    Runs during the application installation phase.
    Use: Copy files, decompress bundles, or set up the application.

AfterInstall

    Runs after the application is installed but before starting it.
    Use: Configure installed files, restart services, or initialize databases.

ApplicationStart

    Runs after the new application version starts.
    Use: Start services, verify launch, or initialize processes.

ValidateService

    Runs as the final step to verify the application is working correctly.
    Use: Perform health checks, run tests, or ensure service availability.

# 1.4 Application Services, Event-Driven & Serverless 

## 1.4.1 SQS Extended Client Library 

- Used for handling messages over SQS max (256KB) - cannot be increased
- When large message is being sent, it's saved to S3 and SQS stores a link to it
- When message is received, the large message is loaded from S3
- When SQS message is deleted, the object in S3 is also deleted
- It's basically an interface to workaround the hard limit of 256KB messages in SQS

## 1.4.2 SQS Delay Queues

- Used to add delay to the process
- Maximum delay is 15 minut
- Once the message is added, it will be hidden for given period and all **receive message** operations will see **NoMessages** response

## 1.4.3 SQS (DLQ) Dead Letter Queues

- This is where undeleted messages are sent after X failed processing attempts,
    - when ReceiveCount on message exceeds the maxReceiveCount configured for the queue and the message is not deleted, it is moved to DLQ
- DLQ can be used to provide alternative way to handle some problematic cases,
    - DQL can have setup alarms and provide isolated diagnostics,
- Enqueue timestamp remains unchanged when the message is moved to DLQ. After retention period is exceeded, the message is dropped.
    - DLQ retention period should take this into account and be longer than the original regular queue
- One DLQ can work for multiple source queues

## 1.4.4 Step Functions

Step Functions are addressing issues that come from Lambda design:
- Lambda environment is stateless,
- Maximum execution time is 15 minutes, 
- It's FAAS, so it shouldn't be used for whole applications,
- Can be chained but gets messy at scale

### 1.4.4.1 State Machines
- Step Functions are **workflows**
    - Start -> States -> End
    - States are THINGS which occur inside workflows
- Maximum duration of one state machine is **1 year**
    - Standard Workflow (default) - 1 year
    - Express Workflow (for example for IoT) - up to 5 minutes
- State Machines can be started by API GW, EventBridge, IOT Rules, Lambda...
- They use IAM Roles for permissions
- State Machine can be "exported" using Amazon States Language which is a JSON template

### 1.4.4.2 States
- **SUCCEED** & **FAILED**
- **WAIT** - wait given period of time
- **CHOICE** - make decision based on provided options like stocks levels
- **PARALLEL** - parallel branches within state machines
- **MAP** - accepts list of things, for each item map state performs action
- **TASK** - tasks coordinate with external services to perform actual work (lambda, dynamodb, ECS, SNS, SQS, Glue)


## 1.4.5 Cognito

### Cognito User Pools vs. Identity Pools

| Feature | **User Pools** | **Identity Pools** |
|---------|--------------|----------------|
| **Main Purpose** | Authenticate users for your **frontend application** | Grant users **temporary AWS credentials** to access AWS services |
| **How it Works?** | Manages users with sign-up/sign-in, MFA, and user attributes | Maps authenticated users (from Cognito or external providers) to **IAM roles** |
| **Access Control** | Provides a **JWT token** (ID & Access token) for authentication | Provides **temporary AWS IAM credentials** (Access Key & Secret) |
| **Use Case** | When users need to log in to your **app** (Web/Mobile) | When users need **direct access** to AWS services (S3, DynamoDB, etc.) |
| **Identity Providers** | Cognito itself (built-in user directory) | Cognito User Pools, Google, Facebook, Apple, SAML, OpenID Connect |
| **Where is it Used?** | Frontend (authentication) | Backend & AWS services (authorization) |
| **Token Type** | JWT (ID token, Access token) | Temporary AWS IAM credentials (Access Key, Secret Key, Session Token) |

---

### Scenarion and usage

| Scenario | Use **User Pool** | Use **Identity Pool** |
|----------|----------------|----------------|
| Sign up/sign in users for your app | ✅ | ❌ |
| Authenticate with Google, Facebook, etc. | ✅ | ✅ (federated identity) |
| Protect backend API (e.g., API Gateway, Lambda) | ✅ | ❌ |
| Allow frontend to access AWS services directly (e.g., S3, DynamoDB) | ❌ | ✅ |
| Custom authentication (multi-tenancy, custom logic) | ✅ | ❌ |
---

### Sample route
Client -> FE Application -> Identity Provider (Google) -> FE Application -> Cognito -> Cognito grants temporary access to AWS resources

# 1.5 AWS Lambda In-Depth

## 1.5.1 Architecture

### Summary
Lambda has lifecycles. The code runs in the execution environment.
1. **INIT** - creates or unfreezes the execution environment
2. **INVOKE** - runs the function handler (cold start)
3. **NEXT INVOKE** - WARM START - lambda attempts to reuse the same environment for the next invocation (must be done quick enough)
4. **SHUTDOWN** - shuts down the environment if was no invocation for some time

### Lambda optimization
- **Lambda handler part** will run every time the code function is being invoked
- Everything above handler is being invoked only during the cold start
    - If it's warm start, it will run only code from the handler part

## 1.5.2 Monitoring & Logging & Tracing Lambda Based Applications 

### Summary
1. Monitoring - CloudWatch metrics
2. Logging - CloudWatch logs
3. Tracing - Lambda X-ray

## 1.5.3 Lambda Resource Policy

### Summary
- Lambda has two types of security policies
    1. Execution role - the role that Lambda assumes. Determines **WHAT** Lambda function **CAN DO**.
    2. Resource Policy - **WHO** can do **WHAT** with the Lambda
- Resource Policy is required when a service needs to invoke a lambda but can't assume a role. Example: S3 event
- Required for cross-account
- NOT required in the same AWS account when Identity has Identity Policy

# 1.6  APIs & API Gateway In-Depth 

## 1.6.2 Integrations

Each API Gateway implementation consist of 3 stages:
1. Request
2. Integration
3. Response

Request and Response stages are divided into 2 parts:
```
Method Request -> Integration Request
Method Response <- Integration Response
```
- Integration Request and Response can pass the data as it is (passthrough/proxy) or transform it 
- Integration means interacting with backend endpoints
- Method means interacting with client

### 1.6.2.1 Types of Integration
>**You chooose integration when adding a new Method to the resource.**

- **MOCK** - no backend, API GW just responds
- **HTTP** - backend HTTP custom integration, both backend and integration must be configured
- **HTTP Proxy** - pass through from client to integration unmodified, return from intergration to client unmodified
- **AWS** - lets an API expose AWS service actions
- **AWS_PROXY** (Lambda) - Low admin overhead Lambda endpoint. Default used in the console.

### 1.6.2.2 Mapping Templates
- Used for AWS and HTTP **non proxy** integrations
- It can: 
    - modify or rename request parameteres 
    - modify the body or the headers
    - filter/remove anything which isn't needed
- It uses Velocity Template Language (VTL)

#### **Common exam scenario**
Legacy system that uses SOAP API and needs translation so Mapping Templates are used to transform the request.

```
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:web="http://example.com/">
   <soapenv:Header/>
   <soapenv:Body>
      <web:GetUser>
         <web:UserId>123</web:UserId>
      </web:GetUser>
   </soapenv:Body>
</soapenv:Envelope>
```

**VTL:**
```
## Extract UserId from SOAP XML

#set($inputRoot = $input.path('$'))
{
  "userId": "$inputRoot.Envelope.Body.GetUser.UserId"
}
```

Translated to JSON for REST API:

```
{
  "userId": "123"
}
```

## 1.6.3 API Gateway Stages and Deployments 

- A stage is a named reference to a deployment, which is a snapshot of the API.
- API has to be deployed to have current configuration changes (just like with Lambda versions)
- Stages can be used for environments: Staging, Production, Test and each stage has its own configuration
- Unlike Lambda versions, they are mutable - they can be overwritten and rolled back

### 1.6.3.1 Stage Variables
Each stage can have its own Stage Variables like `ENV=PROD`, `ENV=DEV` etc.

Each stage variable can be used in the **Integrations** settings to refer specific resources like Lambda with **PROD/DEV alias**

## 1.6.4 Open API & Swagger 

#### Versions and quick info
- Swagger = OpenAPI v2
- OpenAPI = OpenAPI v3

These are technical specifications of an API. Can be used like CloudFormation but for API. Specification can be exported from the API GW and a new one can be created using it.

# 1.7  Elastic Beanstalk In-Depth 

## 1.7.1 Elastic Beanstalk Architecture

### 1.7.1.1 Key Points
- Beanstalk is a platform (PaaS) that manage all other AWS services to let you focus on the code and application
- Under the hood it uses SQS, EC2, ECS etc. and its fully customizable

### 1.7.1.2 Platforms
- Beanstalk uses a concept of platforms
- Currently supported platforms are:
    - Python, Ruby, Go, .Net, .Net Core, Java SE, Tomcat, 
    - Docker (single container and multi container)
    - Custom via packer (you can create your custom platform)

### 1.7.1.3 Application
- **APPLICATION** - In Beanstalk terms, this refers to the entire infrastructure and setup for running an app, rather than just the actual application code. It’s a collection of resources (like folders, containers, etc.) related to the application.
    - **ENVIRONMENT** - Inside an application, you have environments that store the services and settings required to run the app and they use platforms to run. Environments represent different stages (e.g., dev, staging, prod) or instances (e.g. processing env) of the application.
        - **APPLICATION VERSIONS** - They are specific labeled versions of **deployable code** for an application used in environments. The source bundle is stored in S3.
        - **DON'T PUT DB IN ENV:** Environments can use blue green deployments, AB testing and CNAME swap. In this case, database would be lost when one env is discarded. Create DB outside the Beanstalk instead.

### 1.7.1.4 Deployment Policies
Application versions can use different deployment type:
1. **ALL AT ONCE** - Deploy to all instances at once. Brief outage. After deployment, either everything is good, or everything is bad. Not really good for production.
2. **ROLLING** - Step by step. Only moves to next when previous is successfull. Goes instance by instance. For this you can setup batch size to control the amount of instances updated at the same time.
3. **ROLLING WITH BATCH** - In Rolling with batch deployment, the system deploys new application versions without taking down any instances. This strategy is designed to ensure that there is no downtime during the deployment process by maintaining the capacity throughout.
    - The key idea is to deploy a new batch of instances (e.g., 2 new instances) while maintaining the required capacity.
    - Batch size: When you set the batch size to 2, Elastic Beanstalk will first launch 2 new instances with the new version of the application, ensuring that the capacity is maintained.
    - After these 2 new instances are running with the new version, the system will update 2 instances from the existing pool (the old version), replacing them with the new version.

    This results in a rolling update where:

    ```
    2 instances are running with the new version,
    2 instances are still running with the old version,
    2 instances are redundantly running with the new version (since the new ones were launched in the first batch).
    ```
    At the end, additional instances are removed. This option is slower and more expensive but it's safer thus good for production usage.

4. **IMMUTABLE** - New instances are created, and if they're healthy, the old instances are then removed. Most expensive option since it doubles the amount of instances during deployment. 
5. **TRAFFIC SPLITTING** - Fresh instances with traffic split. Adds an A/B testing aspect to the immutable strategy before removing old instances.
6. **BLUE GREEN** (not actual deployment policy) - Manual deployment strategy where you prepare two separate environments with two different application versions. Thanks to this you have full control over the two environments. CNAME swap can be used to point the DNS name at desired environment.

### 1.7.1.5 Decoupling RDS instance existing in application environment
1. **Create an RDS snapshot** – Backup the database to ensure data safety.

2. **Enable Delete Protection** – Prevent accidental deletion of the RDS instance.

3. **Create a new environment** – Deploy a new Elastic Beanstalk environment with the same application version, but without an attached RDS.

4. **Ensure connectivity** – Configure the new environment to connect to the existing standalone RDS instance (adjust security groups if needed).

5. **Swap environments** – Perform a CNAME swap (DNS update) to redirect traffic to the new environment.

6. **Delete the old environment** – Once everything is confirmed working, remove the old Beanstalk environment (without deleting the standalone RDS).

### 1.7.1.6 Customizing with .ebextensions
Extension is a folder inside the ZIP/WAR application version package. 
- Folder should be called .ebextensions
- Add YAML or JSON files to it ending with .config 
- Use CFN format to add resources in the envrionment

### 1.7.1.7 HTTPS
Two ways to enable HTTPS in Elastic Beanstalk:
1. **Apply the certificate directly in the Load Balancer**

    Go to EB Console → Environment → Load Balancer Configuration and attach an SSL certificate.

2. **Configure it via .ebextensions**

    Use configuration files (.ebextensions) to automate SSL setup during deployment.

✅ Note: Ensure security groups allow HTTPS traffic and that the Load Balancer can communicate securely with EC2 instances.

### 1.7.1.8 EB and Docker
#### Single Docker
- Uses single EC2 instance with Docker instead ECS
- It can create containers in the environment based only on the provided Dockerfile

#### Multi-container Docker
- Uses ECS + Load Balancer

