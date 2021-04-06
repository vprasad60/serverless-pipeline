# Project Overview
The goal of this project is to reconstruct the serverless data pipeline in AWS described in the figure below. In summary, a producer Lambda function will read from a DyanmoDB table every minute and send the contents to a queue in SQS. A consumer Lambda function will be triggered by messages filling the queue, and will look up the contents of message on the relevant Wikipedia article. AWS Comprehend will be used to perform sentiment analysis and entity extraction, and the results will be uploaded to an S3 bucket. 

**Primary reference**: [awslambda tutorial by Noah Gift](https://github.com/noahgift/awslambda)

![](https://camo.githubusercontent.com/bb29cd924f9eb66730bbf7b0ed069a6ae03d2f1a/68747470733a2f2f757365722d696d616765732e67697468756275736572636f6e74656e742e636f6d2f35383739322f35353335343438332d62616537616638302d353437612d313165392d393930392d6135363231323531303635622e706e67)

## Potential Disk Space Issue
After you create the producer function and begin creating the consumer function, it is possible that you will not have enough space if you are using a t.2 micro instance. To free space, it is recommended to delete some default Docker images in Cloud9. Run the following commands:

```
docker image ls
docker image rm IMAGE_ID 
```
Replace IMAGE_ID above with the proper docker image id. Recommended images for removal are Python 2.7, Python 3.6, and all nodejs images. 

## Procedure
### DyanmoDB
1. Create a table in DynamoDB. If the name of the table is not "fang" be sure to change the table name in the producer.py function
2. Make sure to set the unique identifier to "name" to be consistent with the variable in the producer.py function
3. Add items to the list, such as company or food names

### SQS
1. Create a queue with the default settings in SQS. Make sure that its name corresponds to the main one in the producer.py function

### S3
1. Create an S3 bucket and make sure the name is the same as in the consumer.py function

### Lambda Functions
#### Producer
1. On the AWS Cloud9 IDE, navigate to the left side and click the AWS Explorer button
2. Create an AWS Lambda function and choose Python 3.7 and the Hello World SAM application
3. If you create the function in the main environment folder, a new folder with the function name will be generated. 
4. In the hello-world subfolder, replace the app.py code with the code from producer.py
5. Change region and table variables as necessary in the producer.py code
6. Enter the following commands to deploy the lambda application
```
sam build --use-container
sam deploy --guided
```
7. After filling out the fields in step 6, the Lambda application will be deployed and you can check it in the console
8. Adjust the role in the Lambda application to have adminstrator access. Go to IAM, create a role for Lambda use cases, and attach the AdminstratorAccess policy
9. For the Lambda function, check the Permissions tab under Configuration and change the execution role to the one you just created.
10. Apply a EventBridge CloudWatch trigger to the function and in the *Schedule Expression* field, input 'rate(1 minute)'.

#### Consumer
1. Repeat steps 1-9 above. If you do not have space when at step 6, refer to the **Potential Disk Space Issue** section above
2. Apply an SQS trigger to the consumer Lambda function and select your specified SQS queue

### Running the Pipeline
1. Enable the trigger for both the producer and consumer lambda functions
2. Within a minute you should see messages populating the queue in SQS (they may also disappear quickly once the consumer runs). These messages will contain items from your table
3. Disable the trigger of the producer lambda function
4. Disable the trigger of the consumer lambda function
5. Check the CloudWatch logs and the S3 bucket to see if appropriate results were generated and/or if there were any errors
