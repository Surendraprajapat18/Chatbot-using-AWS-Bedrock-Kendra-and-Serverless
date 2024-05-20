# Chatbot Using AWS Bedrock, Kendra, and Serverless

This repository showcases how to build a generative AI chatbot using AWS services. The primary services used are:

* **Amazon Bedrock**: Simplifies building and scaling generative AI applications with foundation models.
* **Amazon Kendra**: An intelligent enterprise search service for searching across content repositories.
* **AWS Lambda**: A compute service that runs code in response to events and automatically manages the compute resources.

To build the chatbot with Amazon Bedrock and Amazon Kendra, the following architecture is used:

![Chatbot Architecture](https://github.com/Surendraprajapat18/Chatbot-using-AWS-Bedrock-Kendra-and-Serverless/assets/97840357/be6b6ef3-14a8-474e-8298-5b32c9d47543)

## 1. Environment Setup

### Launch Workshop in AWS Account

#### Launch a CloudFormation Stack
An AWS CloudFormation template is used to set up lab resources in your chosen AWS Region. The CloudFormation template creates the following AWS resources:

* **AWS Cloud9**: A cloud-based IDE for writing, running, and debugging code.
* **Amazon S3**: Object storage service used for storing documents indexed by Amazon Kendra.

To launch the stack:

1. Download the CloudFormation template: [![Download](https://img.shields.io/badge/Download-%230077B5.svg?logo=arduino&logoColor=white)](https://github.com/Surendraprajapat18/Chatbot-using-AWS-Bedrock-Kendra-and-Serverless/blob/main/ws-startup-stack.yaml)
2. Store the YAML template file on your local machine.
3. Navigate to CloudFormation in the AWS Management Console.
4. Choose "Upload a template file" and select the downloaded template.
   ![Upload Template](https://github.com/Surendraprajapat18/Chatbot-using-AWS-Bedrock-Kendra-and-Serverless/assets/97840357/2382cad5-3648-4e9c-a079-158d41ab8950)
5. Name the stack, e.g., `bedrock-workshop-environment`.
   ![Stack Name](https://github.com/Surendraprajapat18/Chatbot-using-AWS-Bedrock-Kendra-and-Serverless/assets/97840357/627d813b-b073-4fd7-84ae-b7a6771649c6)
6. Keep the default values for stack options and choose "Next."
7. Acknowledge the IAM resource notice and submit the template.

### Launch AWS Cloud9 and Set Up the Environment

1. In the Environments section, for `bedrock-workshop-environment`, choose "Open" under Cloud9 IDE.
   ![Open Cloud9](https://github.com/Surendraprajapat18/Chatbot-using-AWS-Bedrock-Kendra-and-Serverless/assets/97840357/c26eb47c-41d6-41e5-a3db-739ced76b3dc)
2. Confirm that the AWS Cloud9 environment loaded properly.
   ![Cloud9 Loaded](https://github.com/Surendraprajapat18/Chatbot-using-AWS-Bedrock-Kendra-and-Serverless/assets/97840357/49a09063-ec5f-4c00-8679-7dd8b5dbadc4)

#### Set Up Environment

1. Change the stack name `<your-startup-stack-name>` and set the environment variable `S3BucketName`:
    ```sh
    export CFNStackName=<your-startup-stack-name>
    export S3BucketName=$(aws cloudformation describe-stacks --stack-name ${CFNStackName} --query "Stacks[0].Outputs[?OutputKey=='S3BucketName'].OutputValue" --output text)
    echo "S3 bucket name: $S3BucketName"
    ```

## 2. Chatbot Application and Amazon Kendra Setup

### Build and Deploy the Serverless Application

1. Open the AWS Cloud9 environment.
2. Clone the source code:
    ```sh
    cd ~/environment
    git clone https://github.com/aws-samples/bedrock-serverless-workshop.git
    ```
3. Build and deploy the backend and frontend:
    ```sh
    cd ~/environment/bedrock-serverless-workshop
    chmod +x startup.sh
    ./startup.sh
    ```
    The script performs the following tasks:
    - Upgrades the OS and installs `jq` software.
    - Installs Node.js version 16.
    - Builds the backend using `sam build`.
    - Deploys the backend using `sam deploy`.
    - Installs Amplify and builds the frontend.
    - Publishes the frontend application using Amplify.
4. If prompted with a git alert, choose OK.
5. During Amplify setup, keep the default selections for hosting and deployment options.

After completion, you will see a screen similar to this:
![Amplify Setup](https://github.com/Surendraprajapat18/Chatbot-using-AWS-Bedrock-Kendra-and-Serverless/assets/97840357/f7da9f43-0817-4db6-b8df-d8c9030d04c0)

Copy and store the values for `amplifyapp` URL, `user_id`, and `password` in a text editor.
![Credentials](https://github.com/Surendraprajapat18/Chatbot-using-AWS-Bedrock-Kendra-and-Serverless/assets/97840357/b2a1c917-a2ff-4506-a012-7aa1d617e346)

### Index Sample Data with Amazon Kendra

1. Download sample documents:
    ```sh
    cd ~/environment/bedrock-serverless-workshop
    mkdir sample-documents
    curl https://www.federalreserve.gov/monetarypolicy/files/fomcminutes20230920.pdf --output sample-documents/fomcminutes20230920.pdf
    curl https://sustainability.aboutamazon.com/2022-sustainability-report.pdf --output sample-documents/2022-sustainability-report-amazon.pdf
    curl https://investor.henryschein.com/static-files/fcf569ec-fdbb-4591-b73d-0d1d849efd78 --output sample-documents/2022-hs1-10k.pdf
    ```
2. Upload documents to S3:
    ```sh
    aws s3 cp prompt-engineering s3://$S3BucketName/prompt-engineering/ --recursive
    aws s3 cp sample-documents s3://$S3BucketName/sample-documents/ --recursive
    ```
3. In the Amazon Kendra console, sync the `S3DocsDataSource` to index documents.
    ![Kendra Index](https://github.com/Surendraprajapat18/Chatbot-using-AWS-Bedrock-Kendra-and-Serverless/assets/97840357/4eec32e8-a860-4667-92cd-c554d032f1f9)
    
4. Query the Amazon Kendra index with sample questions.
    ![Sample Query](https://github.com/Surendraprajapat18/Chatbot-using-AWS-Bedrock-Kendra-and-Serverless/assets/97840357/f8562464-b65e-40e3-aa2f-0756f946a1e8)

## 3. Chatbot User Experience with Amazon Bedrock

### Activate Model Access

1. In the Amazon Bedrock console, enable access for Anthropic Claude and AI21 Labs Jurassic-2 Ultra models.
2. Use the Bedrock playground to interact with the models.
    ![Activate Models](https://github.com/Surendraprajapat18/Chatbot-using-AWS-Bedrock-Kendra-and-Serverless/assets/97840357/a74037e7-d87b-46a8-ab52-2f2d05ef8e91)
3. On the Selection Provider dropdown menu, select the Anthropic and Claude 2 v2 model.
    ![Select Model](https://github.com/Surendraprajapat18/Chatbot-using-AWS-Bedrock-Kendra-and-Serverless/assets/97840357/0f526fb3-dc8f-4cfc-ae56-72fa1cf90b8c)
4. Type a question and choose "Run." For example: "What is the distance to the moon from Earth?"
    ![Model Response](https://github.com/Surendraprajapat18/Chatbot-using-AWS-Bedrock-Kendra-and-Serverless/assets/97840357/b334b9fa-f507-4387-ac22-f87d888ef33a)

This repository provides a detailed guide to building a robust AI chatbot using AWS services. For more information, refer to the official AWS documentation linked in this repository
[![AWS official Reposetory](https://img.shields.io/badge/AWSReposetory-%230077B5.svg?logo=arduino&logoColor=white)](https://catalog.us-east-1.prod.workshops.aws/workshops/27eb3134-4f33-4689-bb73-269e4273947a/en-US)
