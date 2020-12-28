# Smith

Smith is a service that allows to run integrational long running tests in parallel in a serverless cloud.

Cloudformation templete will create AWS serverless resources - SQS, S3 buckets, ECS sevice, Api Gateway and VPC. 

API allows to pass tests you want to execute. Smith will spawn ECS instances, execute tests, store logs in S3, send results to SQS and stop ECS instances.

This allows to run as many tests as you want in parallel without the need of keeping long-running agent instances. Smith will manage resources for you and spawn servers for the execution time only.


E.g. - template will create endpoint like

https://n7bovw913i.execute-api.us-east-1.amazonaws.com/prod/tasks

Which can take POST requests with data of the following format:

{"launchId": "ID_OF_TEST_LAUNCH", "executables": [
         { "testcaseUuid": "1", "metadata": { "projectId":"com.testquack",     "artifactId":"smith-maven-s3-tests",     "version":"1.0-SNAPSHOT",     "package":"com.testquack.smith.maven",     "class":"TestScopeTest", "method":"resourceDataTest"}},
         {"testcaseUuid": "2",  "metadata": { "projectId":"com.testquack",     "artifactId":"smith-maven-s3-tests",     "version":"1.0-SNAPSHOT",     "package":"com.testquack.smith.maven",     "class":"TestScopeTest", "method":"simpleTest"}},
         {"testcaseUuid": "3", "metadata": { "projectId":"com.testquack",     "artifactId":"smith-maven-s3-tests",     "version":"1.0-SNAPSHOT",     "package":"com.testquack.smith.maven",     "class":"TestScopeTest", "method":"simpleTest"}},
     ]
 }
 
Eventually execution logs will be stored in 
smith-reports-[REGION]-[ACCOUNT_ID] S3 bucket
Corresponding messages will be sent to smith-results SQS so you could consume results
