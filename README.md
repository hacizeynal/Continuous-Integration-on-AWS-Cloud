# Continuous Integration on AWS Cloud

In the previous [project](https://github.com/hacizeynal/Continuous-Integration-Using-Jenkins-Nexus-Sonarqube-Slack)
, we built a Continuous Integration pipeline on Jenkins and integrated it with the following tools Maven, Checkstyle, Slack, SonaType Nexus, and SonarQube. Unfortunately building and managing all those tools are requiring huge effort and overhead. We will build the same pipeline with AWS services which will give us a lot of advantages,such as less overhead for managing, less human input, and getting rid of managing standalone servers.

### Current scenario

* Agile Software Development Lifecycle
* Developers make regular code changes (commits)
* Those commits needs to be Tested & Built regularly

### Problem with current scenario

* In Agile SDLC there will be frequent code changes
* Not so frequently code will be tested
* Bugs will be accumulated and it will be hard to fix them for Developers

### Solution for the problem

* Build and Test for every commit 
* Automated build process 
* Developer should be notified for every Build & Test (If code fails to pass any QualityGate)

### AWS Services for project

* Code Commit > Version Control Remote Repository
* Code Artifact > Maven Repository for Dependencies
* Code Build > Fully managed continuous integration service that compiles source code, runs tests, and produces software packages that are ready to deploy
* Code Deploy > Artifact Deployment Service
* SonarCloud > Cloud Based SonarQube service
* CheckStyle > Code Analysis from build job
* CodePipeline > Service to integrate all jobs together
* S3 Bucket > Storing Artifacts
* SNS > Notifications 
* IAM 

High Level Overview for pipeline 

[![Screenshot-2022-10-28-at-22-04-44.png](https://i.postimg.cc/Kz3kmVxV/Screenshot-2022-10-28-at-22-04-44.png)](https://postimg.cc/k69XcTVF)

## Set up CodeCommit on AWS

### Create Private Repository

We will create private Git repository in AWS Cloud. Please follow [link](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-ssh-unixes.html#setting-up-ssh-unixes-keys) to configure SSH settings in order to clone/push code to Code Commit.

[![Screenshot-2022-10-29-at-09-58-28.png](https://i.postimg.cc/CKHBptMr/Screenshot-2022-10-29-at-09-58-28.png)](https://postimg.cc/WDzbM5k0)

### Create Artifact Repository

We will create Artifact Repository which will download Maven dependencies from Maven Central repo and save it on our AWS Artifact Repo.

We will update settings.xml and pom.xml with code artifact details such as new token ,repository name and list of servers.

Snippet from settings.xml with new details.

```
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <servers>
        <server>
            <id>zhajili-devops-maven-central-store</id>
            <username>aws</username>
            <password>${env.CODEARTIFACT_AUTH_TOKEN}</password>
        </server>
    </servers>
<profiles>
  <profile>
    <id>zhajili-devops-maven-central-store</id>
    <repositories>
      <repository>
        <id>zhajili-devops-maven-central-store</id>
    <url>https://zhajili-devops-866308211434.d.codeartifact.us-east-1.amazonaws.com/maven/maven-central-store/</url>
      </repository>
    </repositories>
  </profile>
</profiles>
<activeProfiles>
        <activeProfile>default</activeProfile>
    </activeProfiles>
<mirrors>
  <mirror>
    <id>zhajili-devops-maven-central-store</id>
    <name>zhajili-devops-maven-central-store</name>
    <url>https://zhajili-devops-866308211434.d.codeartifact.us-east-1.amazonaws.com/maven/maven-central-store/</url>
    <mirrorOf>*</mirrorOf>
  </mirror>
</mirrors>
</settings>

```
### Configure Cloud SonarQube 

We will need login account from Cloud SonarQube for successfull authentication.After succesfull login to **sonarcloud.io** ,we will need to create security token to integrate Cloud SonarQube to our AWS CodeCommit.

Since we will login via GitHub account ,all GitHub repositories will be imported by default to SonarQube dashboard ,but we will create project manually for code analysis. We will need to remember following parameters from SonarQube for successfull integration

* Security Token
* SonarQube Cloud URL
* Organization name
* Project name

After saving above parameters ,we will need to navigate AWS and open **Systems Manager>Parameter Store** service.
For all of them will need to create parameter on SSM

[![Screenshot-2022-10-30-at-00-20-47.png](https://i.postimg.cc/j5tx9C0n/Screenshot-2022-10-30-at-00-20-47.png)](https://postimg.cc/8FXgfpCT)

### Create Build Project

It is like creating Jenkins project on premise ,this time we will do it via CodeBuild on AWS.

Following configuration parameters will set

* Source Provider > AWS CodeCommit (It will be our private repo ,like our GitHub repo) 
* Repository > zhajili-DevOps 
* Branch > main 
* Operating System > Ubuntu (since we will run maven commands) 
* Environment type > Linux
* Role name > codebuild-first_build_project_-service-role (this role should have access to parameters which we created via SSM ,so we will need to give correct access via IAM)
* Buildspec > We can think this file as Jenkinsfile ,this file will have commands/instucations that needs to be executed.
We need to update env with the same parameter name which we added to the SSM.

```
version: 0.2
env:
  parameter-store:
    LOGIN: sonartoken
    HOST: HOST
    Organization: Organization
    Project: Project
    CODEARTIFACT_AUTH_TOKEN: codeartifact-token
phases:
  install:
    runtime-versions:
      java: corretto8
    commands:
    - cp ./settings.xml /root/.m2/settings.xml
  pre_build:
    commands:
      - apt-get update
      - apt-get install -y jq checkstyle
      - wget http://www-eu.apache.org/dist/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
      - tar xzf apache-maven-3.5.4-bin.tar.gz
      - ln -s apache-maven-3.5.4 maven
      - wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-3.3.0.1492-linux.zip
      - unzip ./sonar-scanner-cli-3.3.0.1492-linux.zip
      - export PATH=$PATH:/sonar-scanner-3.3.0.1492-linux/bin/
  build:
    commands:
      - mvn test
      - mvn checkstyle:checkstyle
      - echo "Installing JDK11 as its a dependency for sonarqube code analysis"
      - apt-get install -y openjdk-11-jdk
      - export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
      - mvn sonar:sonar -Dsonar.login=$LOGIN -Dsonar.host.url=$HOST -Dsonar.projectKey=$Project -Dsonar.organization=$Organization -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ -Dsonar.junit.reportsPath=target/surefire-reports/ -Dsonar.jacoco.reportsPath=target/jacoco.exec -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
      - sleep 5
      - curl https://sonarcloud.io/api/qualitygates/project_status?projectKey=$Project >result.json
      - cat result.json
      - if [ $(jq -r '.projectStatus.status' result.json) = ERROR ] ; then $CODEBUILD_BUILD_SUCCEEDING -eq 0 ;fi
```
Let's start the Build ,after some time we see that Build has failed. After checking logs we can following logs

```
[Container] 2022/10/30 13:23:41 Phase context status code: Decrypted Variables Error Message: AccessDeniedException: User: arn:aws:sts::866308211434:assumed-role/codebuild-first_AWS_build_project-service-role/AWSCodeBuild-e4bc6a2e-ca48-4d73-9514-43b59ad8a665 is not authorized to perform: ssm:GetParameters on resource: arn:aws:ssm:us-east-1:866308211434:parameter/Project because no identity-based policy allows the ssm:GetParameters action

```
Remember that above we have mentioned the **codebuild-first_build_project_-service-role** should have proper permission to access SSM Parameter store.

We can run Policy Simulator and can see that this particular role does not have access.

[![Screenshot-2022-10-30-at-14-30-32.png](https://i.postimg.cc/DZ642Ny8/Screenshot-2022-10-30-at-14-30-32.png)](https://postimg.cc/34yx1nGT)
















