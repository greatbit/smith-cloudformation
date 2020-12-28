# Smith

Smith is a service that allows to run integrational long running tests in parallel in a serverless cloud.

Cloudformation templete will create AWS serverless resources - SQS, S3 buckets, ECS sevice, Api Gateway and VPC. 

API allows to pass tests you want to execute. Smith will spawn ECS instances, execute tests, store logs in S3, send results to SQS and stop ECS instances.

This allows to run as many tests as you want in parallel without the need of keeping long-running agent instances. Smith will manage resources for you and spawn servers for the execution time only.


E.g. - template will create endpoint like

https://[SOME_RANDOM_HASH].execute-api.us-east-1.amazonaws.com/prod/tasks

Which can take POST requests with data of the following format:

```
{"launchId": "ID_OF_TEST_LAUNCH", "executables": [
         { "testcaseUuid": "1", "metadata": { "projectId":"com.testquack",     "artifactId":"smith-maven-s3-tests",     "version":"1.0-SNAPSHOT",     "package":"com.testquack.smith.maven",     "class":"TestScopeTest", "method":"resourceDataTest"}},
         {"testcaseUuid": "2",  "metadata": { "projectId":"com.testquack",     "artifactId":"smith-maven-s3-tests",     "version":"1.0-SNAPSHOT",     "package":"com.testquack.smith.maven",     "class":"TestScopeTest", "method":"simpleTest"}},
         {"testcaseUuid": "3", "metadata": { "projectId":"com.testquack",     "artifactId":"smith-maven-s3-tests",     "version":"1.0-SNAPSHOT",     "package":"com.testquack.smith.maven",     "class":"TestScopeTest", "method":"simpleTest"}},
     ]
 }
 ```
 
Eventually execution logs will be stored in 
smith-reports-[REGION]-[ACCOUNT_ID] S3 bucket
Corresponding messages will be sent to smith-results SQS so you could consume results

## Supported tests
At this point Smith supports Maven Junit tests only. We work hard to bring in Gradle, Nunit, Pytest (and may othere) support.

## Maven Junit
To make Smuth execute your tests it should have access to them. 

1. First you'll have to pack and deploy your tests as jars.
In order to do that - add the following plugin to your tests:

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.2.0</version>
    <executions>
        <execution>
            <goals>
                <goal>test-jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

and add distributeion management pointing to S3 buckets created by Cloud Formation Template

```
<distributionManagement>
    <snapshotRepository>
        <id>my-repo-bucket-snapshot</id>
        <url>s3://smith-artifacts-snapshot-[REGION]-[ACCOUNT_ID]</url>
    </snapshotRepository>
    <repository>
        <id>my-repo-bucket-release</id>
        <url>s3://smith-artifacts-release-[REGION]-[ACCOUNT_ID]</url>
    </repository>
</distributionManagement>
```

2. Your tests should be available to Smith. Here you have 3 optioms:

* Your tests are available in Maven Central. Doubtfully it will be the case but if so - you don't need to do anything more

* You can upload your test to the S3 bucket created by Smith Cloud Formation Template. To do that you have to add the following extension to your project in a Build section:

```
<build>
    <extensions>
        <extension>
            <groupId>com.testquack.smith</groupId>
            <artifactId>s3-storage-wagon</artifactId>
            <version>1.0</version>
        </extension>
    </extensions>
...
</build>
```

Add AWS credentials to your ~/.m2/settings.xml file:
```
<server>
    <id>my-repo-bucket-snapshot</id>
    <username>AWS_ACCESS_KEY</username>
    <password>AWS_SECRET_KEY</password>
    <configuration>
        <region>us-east-1</region>
    </configuration>
</server>
<server>
    <id>my-repo-bucket-release</id>
    <username>AWS_ACCESS_KEY</username>
    <password>AWS_SECRET_KEY</password>
    <configuration>
        <region>us-east-1</region>
    </configuration>
</server>
```

* Another way is to put your maven settings.xml file into "smith-private-settings-[REGION]-[ACCOUNT_ID]" bucket created by the template. If Smith will find settings.xml there - it will use it. 
In settings.xml you can specify alternative path to your maven repository

## Deploy
Deploy your tests using 
mvn clean deploy

## Run
Thats it. Run your tests using API with the data format above.

But the easiest way is to run tests through QuAck (testquack.com).
QuAck allows you to import tests into the system and features Smith Runner. Smith Runner is a plugin that allows to run tests througb QuAck UI in Smith and get and store results.

## Example
Here you'll find tests examples that are being uploaded to QuAck and Smith. This allows using Smith Launher plugin in QuAck to run tests and store results in QuAck.
https://github.com/greatbit/smith-junit-maven-tests-example


