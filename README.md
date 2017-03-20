# Getting data from Kinesis Streams backed up to S3
The project is for using AWS Lambda with Amazon Kinesis Firehose.

The project is set up with a generic mvn archetype and the build occurs with Maven.

Java 8 is the prescribed JDK to compile to.

## Build and install

Things to look for:
Package com.amazonaws.proserv.lambda houses the lambda functions
- at the top of each class in the is package are instance variables.  YOU MUST EDIT these instance variables.  At the time of this writing, there was not a way to pass parameters into Java-based
  Lambda functions - and since a jar is used there is no need to extract them to propery files (but you very well can).

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

Helper classes are provided:
com.amazonaws.proserv.PopulateKinesisData
  - this class is just a helper to put data on to Kinesis.  It uses the SampleAWSCredentialProvider class to inspect for credentials.
com.amazonaws.proserv.SampleAWSCredentialProvider
  - just used to highlight the ease of writing your own...


Configs if running locally:
 - resources/AwsCredentials.properties - put your creds here if you are running on your local  NOTE: make sure this file IS NOT EXCLUDED in your Pom.xml if running locally.  It is excluded so as not to be packaged and distributed (a non-no).
