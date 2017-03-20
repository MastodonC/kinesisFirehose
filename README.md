# Getting data from Kinesis Streams backed up to S3
The code in this project has largely been borrowed from <https://github.com/awslabs/aws-big-data-blog/tree/master/aws-blog-firehose-lambda/kinesisFirehose>.  It has been adapted for our purposes, but is quite configurable as most of the variables can be changed by using environment variables in the lambda function.
The project is for using AWS Lambda with Amazon Kinesis Firehose.

The project is set up with a generic mvn archetype and the build occurs with Maven.

Java 8 is the prescribed JDK to compile to.

## Build and install
To build the code into a jar:
```
mvn clean install
```
To create the Lambda jar distribution (Java 8!!!), which will be in target/firehoseLambda-VERSION.jar

```
aws lambda create-function \
--region eu-central-1 --role arn:aws:iam::165664414043:role/lambda-kinesis-role \
--function-name kixi-event-backup  \
--zip-file fileb://$(pwd)/target/firehoseLambda-1.0.jar \
--handler com.amazonaws.proserv.lambda.KinesisToFirehose::kinesisHandler \
--runtime java8
```

To update the function with a new jar:
```
aws lambda update-function-code --function-name kixi-event-backup --zip-file fileb://$(pwd)/target/firehoseLambda-1.0.jar
```

Necessary before a run: specify the following environment variables in your lambda function Advanced settings:
* firehoseEndpointURL: the AWS endpoint for firehose for your region of choice (<http://docs.aws.amazon.com/general/latest/gr/rande.html#fh_region>)                                                                                                                                                               
* deliveryStreamName: the stream we want to back up the content from                                                                                                                                                               
* deliveryStreamRoleARN: role for the firehose, mostly allowing access to both the lambda function and S3                                                                                                                                                           
* targetBucketARN: the S3 bucket we're targetting                                                                                                                                                                   
* targetPrefix: the 'directory' in the S3 bucket

Helper classes are provided:
com.amazonaws.proserv.PopulateKinesisData
  - this class is just a helper to put data on to Kinesis.  It uses the SampleAWSCredentialProvider class to inspect for credentials.
com.amazonaws.proserv.SampleAWSCredentialProvider
  - just used to highlight the ease of writing your own...


Configs if running locally:
 - resources/AwsCredentials.properties - put your creds here if you are running on your local  NOTE: make sure this file IS NOT EXCLUDED in your Pom.xml if running locally.  It is excluded so as not to be packaged and distributed (a non-no).
