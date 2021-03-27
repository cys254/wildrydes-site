# Server-less Web Application with AWS
## 1 References
* [Server-less Web Application with AWS](https://aws.amazon.com/serverless/build-a-web-app/)
* [AWS Free Tier](https://aws.amazon.com/free/)
* [AWS Management Console](https://aws.amazon.com/console/)
* [AWS CLI](https://aws.amazon.com/cli/)
* [GITHUB](https://github.com/)
* [AWS Amplify](https://aws.amazon.com/amplify/)
* [AWS Cognito](https://aws.amazon.com/cognito/)
* [AWS DynamoDB](https://aws.amazon.com/dynamodb/)
* [AWS Lambda](https://aws.amazon.com/lambda/)
* [AWS API Gateway](https://aws.amazon.com/api-gateway/)
* [AWS Amplify Pricing](https://aws.amazon.com/amplify/pricing/)
* [AWS Cognito Pricing](https://aws.amazon.com/cognito/pricing/)
* [AWS DynamoDB Pricing](https://aws.amazon.com/dynamodb/pricing/)
* [AWS Lambda Pricing](https://aws.amazon.com/lambda/pricing/)
* [AWS API Gateway Pricing](https://aws.amazon.com/api-gateway/pricing/)
## 2 Architecture
![AWS Server-less Web Application Sample Architecture](https://d1.awsstatic.com/diagrams/Serverless_Architecture.5434f715486a0bdd5786cd1c084cd96efa82438f.png)
## 3 Procedures
### 3.1 Setup AWS Account
1. Create AWS free tier account
1. Download and install AWS CLI
1. Create IAM user
    * add permissions(AdministratorAccess)
    * create access key
4. Configure AWS CLI
    * `aws configure`
    *  set access key ID and secret access key from last step
    *  set default region to "us-east-1"
    *  set default output format to "json"
### 3.2 Setup GITHUB Account
1. Create GITHUB account
2. Create SSH key: `ssh-keygen`
3. Add SSH key to GITHUB account under "Setting -> SSH and PGP keys"
### 3.3 Setup GITHUB Repository
1. Create GITHUB repositiry "wildrydes-site"
2. Clone GITHUB repository
3. Populate GITHUB repository
    ```sh
    aws s3 cp s3://wildrydes-us-east-1/WebApplication/1_StaticWebHosting/website ./ --recursive
    git add .
    git commit -m "new"
    git push
    ```
### 3.4 Enable Web Hosting with AWS Amplify Console
1. Use AWS amplify console
2. Select repository provider: GITHUB
3. Authorize AWS amplify to GITHUB account
4. Choose repository "wildrydes-site" and branch "main"
5. Save and deploy
### 3.5 Amazon Congnito Setup
1. Create Amazon Cognioto User Pool "WildRydes", note "pool Id"
2. Add app client "WildRydesWebApp", uncheck "Generate client secret" option, note "App client id"
3. Update `wild-ryde-site/js/config.js`, change `congnito` configuration
    * `userPoolId: '...'`
    * `userPoolClientId: '...'`
    * `region: 'user-east-1'`
4. Commit and push to GITHUB
5. Test register "/register.html", verify email address and login.
### 3.6 Amazon DynamoDB Table Setup
1. From "AWS Management Console -> Services -> Database -> DynamoDB"
2. Create table "Rides", use "RideId" for `Partition key`.
3. Note table `ARN`
### 3.7 IAM Role for Lambda Setup
1. From IAM console create new role "WildRydesLambda", select "AWS Lambda" as role type
2. Attached managed policy `AWSLambdaBasicExecutionRole` to this role
3. Add inline policy `DynamoDBWriteAccess`
    * service: `DynamoDB`
    * action: `PutItem`
    * Resources->Add ARN link: paste table ARN in last step
### 3.8 Create Lambda Function for Handling Requests
1. Create Lambda function `RequestUnicorn`
2. Choose `Node.js` as Runtie
3. Choose an existing role `WildRydesLambda` as created in last step
4. Replace function codes with [requestUnicorn.js](https://webapp.serverlessworkshops.io/serverlessbackend/lambda/requestUnicorn.js)
5. Save
### 3.9 Test Lambda Function
1. Create test event `TestRequestEvent` with following data
    ```json
    {
        "path": "/ride",
        "httpMethod": "POST",
        "headers": {
            "Accept": "*/*",
            "Authorization": "eyJraWQiOiJLTzRVMWZs",
            "content-type": "application/json; charset=UTF-8"
        },
        "queryStringParameters": null,
        "pathParameters": null,
        "requestContext": {
            "authorizer": {
                "claims": {
                    "cognito:username": "the_username"
                }
            }
        },
        "body": "{\"PickupLocation\":{\"Latitude\":47.6174755835663,\"Longitude\":-122.28837066650185}}"
    }
    ```
2. Test with event `TestRequestEvent`, verify test result:
    ```
    {
        "statusCode": 201,
        "body": "{...}",
        "headers": {
            "Access-Control-Allow-Origin": "*"
        }
    }
    ```
### 3.10 Setup Amazon API Gateway
1. Create new API `WildRydes`, keep `Edge optimized` as endpoint type.
2. Create a Cognito user pools authorizer `WildRydes`
    * Select `Cognito` for the type
    * Select `region`
    * Enter `Authorization` for `Token Source`
3. Verify authorizer configuraton
    * Visit `ride.html`, copy auth token from notification
    * Test authorizer
    * Paste auth token into `Authorization Token` field and test
    * Verify response is 200 and user info is displayed.
4. Create API resource `ride`
    * `Resource Path`: `ride`
    * Enable API Gateway CORS
    * Create `POST` method
    * Integration type: `Lambda Function` 
    * Check `Use Lambda Proxy in integration`
    * Choose `Lambda Region`
    * `Lambda Function`: `RequestUnicorn`
    * `Authorization`: `WildRydes` Cognito user pool authorizer
5. Deploy API
    * New Stage: `prod`
    * Note `invoke URL`
6. Update `js/config.js`
    * set `api.invokeUrl` to `invoke URL` from last step
    * commit and push

    

