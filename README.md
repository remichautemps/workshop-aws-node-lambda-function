# workshop-aws-node-lambda-function
A step by step guide to write a node lambda function.

In this project you'll learn:
- How to set up your AWS credentials to use your AWS account
- How to write a Lambda function using Node.js 18.x
- How to write a Makefile to build and deploy your Lambda function

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1 Create a simple Node.js program](#step-1-create-a-simple-nodejs-program)
- [Step 2 Build your Lambda function in the AWS console](#step-2-build-your-lambda-function-in-the-aws-console)
- [Step 3 Deploy your Lambda function with the AWS cli](#step-3-deploy-your-lambda-function-with-the-aws-cli)
- [Step 4 Deploy your Lambda function with a Makefile](#step-4-deploy-your-lambda-function-with-a-makefile)
- [Step 5 Use API Gateway to expose your Lambda function](#step-5-use-api-gateway-to-expose-your-lambda-function)
- [Step 6 Update our Lambda function to use your API Gateway URL](#step-6-update-our-lambda-function-to-use-your-api-gateway-url)

## Before you start

- [Node.js](https://nodejs.org/en/download/) installed on your machine
- [Make](https://www.gnu.org/software/make/) installed on your machine

## Prerequisites

To deploy your lambda function in AWS, you will need the AWS CLI installed on your machine.

- Get the [AWS cli](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- Get your AWS Access keys. (Skip to [STEP 2](#step-2-build-your-lambda-function-in-the-aws-console)- if you already have an AWS profile configured)
- Log into your AWS account. Click on your name in the top right corner of the screen.
  Your name > security credentials > Access Keys (Access Key ID and Secret Access Key) > Create New Access Key
  Save your Access Key ID and Secret Access Key in a safe place.
- Create your AWS profile. [AWS Configure profiles documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html)

#### Set up your AWS profile for the workshop
```bash
# Create a folder to store your AWS credentials
mkdir -p ~/.aws

# Add the following lines to the file ~/.aws/credentials 
vi ~/.aws/credentials
[YOUR_AWS_PROFILE_NAME]
aws_access_key_id=AKXXXXXd
aws_secret_access_key=XXXX

# Add the following lines to the file ~/.aws/config
vi ~/.aws/config
[profile YOUR_AWS_PROFILE_NAME]
region=ap-southeast-2
output=json
```

## Step 1 _Create a simple Node.js program_

NOTE: In this step, we want to create a very simple node program on your local machine.
This will help us to understand the concept of execution context and how to transform our program to be executed in a Lambda function.

- On your local machine, create a folder called `node-lambda-function`
- Inside this folder create a file called index.js with the following content:
    
```js
console.log('The console is printing a message on the terminal!')
```
- Run your program by executing the following command in a terminal: `cd node-lambda-function && node index.js`
- The string "The console is printing a message on the terminal!" is printed in your terminal.


## Step 2 _Build your Lambda function in the AWS console_

- Log into your AWS account. Search for Lambda in the search bar. Click on the Create function button.
- Select Author from scratch
- Name your function
- Select Node 18.x as the runtime
- Select Architecture: x86_64
- Click on the Create function button
- COPY the ARN of your Lambda function. You will need it in the next step.

NOTE: We created a lambda function in AWS. By default, Node lambda function have a built-in handler called index.mjs.

You should see the following code in the code editor, index.mjs:

```js
export const handler = async(event) => {
    // TODO implement
    const response = {
        statusCode: 200,
        body: JSON.stringify('Hello from Lambda!'),
    };
    return response;
};
````
- To understand the execution context, replace the content '//TODO implement' with our handler from earlier

```js
console.log('The console is printing a message on the terminal!')
````
NOTE: Now we have a Lambda function that is printing a message in the terminal and returns a json response. But where?
Let's Execute our Lambda function to find out.

- Click on Deploy button
- Click on Test button
- Click on the Configure test event button
- Select Create new event
- Name your test event
- Select Hello World as the event template
- Click on the Save button
- Click on the Test button

This executes your lambda function. It should print the following message in the terminal:

```bash
 Test Event Name
Test

Response
{
  "statusCode": 200,
  "body": "\"Here 
  
  
  
  es the json body response\""
}

Function Logs
START RequestId: 44635fc6-bde7-4148-90da-de89d704bf81 Version: $LATEST
2023-02-22T11:15:17.797Z	44635fc6-bde7-4148-90da-de89d704bf81	INFO	The console is printing a message... but where?
END RequestId: 44635fc6-bde7-4148-90da-de89d704bf81
REPORT RequestId: 44635fc6-bde7-4148-90da-de89d704bf81	Duration: 11.58 ms	Billed Duration: 12 ms	Memory Size: 128 MB	Max Memory Used: 65 MB	Init Duration: 211.68 ms

Request ID
44635fc6-bde7-4148-90da-de89d704bf81
````
NOTE: take a look at the Function Logs.

## Step 3 _Deploy your Lambda function with the AWS cli_

NOTE: So far we used the inline editor in the AWS console to write our code. This is fine for small projects. 
Now we want to upload code from our local machine to AWS.

- On your local machine, Replace the content of the index.js file with the following:
```js
exports.handler =  async function(event, context) {
  console.log('>>>>>>> I uploaded my code from my local machine! <<<<<<<<')
  const response = {
    statusCode: 200,
    body: JSON.stringify('Here goes the json body response'),
  };
  return response;
};
```
- On your local machine, open a terminal and go to the root of your project: `cd node-lambda-function`
- Zip your code with `zip function.zip index.js`
- Deploy your code with `aws lambda update-function-code --function-name THE_ARN_OF_YOUR_FUNCTION_YOU_COPIED_ON_STEP_2 --zip-file fileb://function.zip --profile YOUR_AWS_PROFILE_NAME`
- In the AWS console, click on the Test button. You should see the updated message in the terminal.

## Step 4 _Deploy your Lambda function with a Makefile_

NOTE: every time we want to deploy our code, we need to zip it and upload it to AWS. Wouldn't it be easier to have a command that does it all.
Well we can do that with a Makefile.

- Create a file called Makefile at the root of your project with the following content:
```Makefile
# Set the AWS region and profile
AWS_REGION ?= ap-southeast-2
AWS_PROFILE ?= myprofile

# Name of your Lambda function ARN and its Node.js entry file
LAMBDA_FUNCTION_NAME ?= REPLACE_WITH_YOUR_ARN_arn:aws:lambda:ap-southeast-2:XXXX:function:YOUR_FUNCTION_NAME
ENTRY_FILE ?= index.js

# Name of the ZIP file that contains the function code
ZIP_FILE_NAME ?= function.zip

# Create the ZIP file that will be uploaded to AWS
zip:
	zip $(ZIP_FILE_NAME) $(ENTRY_FILE)

# Package the ZIP file and upload it to AWS
deploy: zip
	aws lambda update-function-code --region $(AWS_REGION) --function-name $(LAMBDA_FUNCTION_NAME) --zip-file fileb://$(ZIP_FILE_NAME) --profile $(AWS_PROFILE)

# All the steps to build, zip and deploy the function
all: zip deploy

.PHONY: zip deploy

```
- On your local machine, open a terminal and go to the root of your project and run: `make` command. It will zip your code and upload it to AWS.

## Step 5 _Use API Gateway to expose your Lambda function_

- In AWS console, Select your Lambda function. Click on the Add trigger button.
- Select API Gateway
- Select Create a new API
- Select HTTP API
- Select Open
- Click on the Add button
- Click on the API Gateway button
- Copy the API endpoint URL
  e.g. https://xxxxxxxx.execute-api.ap-southeast-2.amazonaws.com/default/node-lambda-function

NOTE: Now we have an endpoint that we can call to execute our Lambda function.

## Step 6 _Update our Lambda function to use your API Gateway URL_

- Edit the index.js file and replace the content with the following:
```js
exports.handler = async (event, context) => {
  const body = JSON.parse(event.body);
  const response = {
    statusCode: 200,
    body: JSON.stringify(body),
    headers: {
      'Content-Type': 'application/json',
    },
  };
  return response;
};
```

NOTE: Now our function is expecting to be called by the API Gateway url created earlier. It will return a response with the same body as the request.

- Try to call the API Gateway endpoint with a POST request and a json body.

