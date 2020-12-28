# Smith

Smith is a service that allows to run integrational long running tests in parallel in a serverless cloud.

Cloudformation templete will create AWS serverless resources - SQS, S3 buckets, ECS sevice, Api Gateway and VPC. 

API allows to pass tests you want to execute. Smith will spawn ECS instances, execute tests, store logs in S3, send results to SQS and stop ECS instances.

This allows to run as many tests as you want in parallel without the need of keeping long-running agent instances. Smith will manage resources for you and spawn servers for the execution time only.


E.g. - template will create endpoint like

https://n7bovw913i.execute-api.us-east-1.amazonaws.com/prod/tasks

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

##Supported tests
At this point Smith supports Maven Junit tests only. We work hard to bring in Gradle, Nunit, Pytest (and may othere) support.

##Maven Junit
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
It will use standard AWS authentication 
https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html
and upload test jars to your S3 bucket as if it was a repository
* Another way is to put your maven settings.xml file into "smith-private-settings-[REGION]-[ACCOUNT_ID]" bucket created by the template. If Smith will find settings.xml there - it will use it. 
In settings.xml you can specify alternative path to your maven repository



