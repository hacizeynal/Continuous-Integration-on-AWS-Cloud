# Continuous Integration on AWS Cloud

In the previous [project](https://github.com/hacizeynal/Continuous-Integration-Using-Jenkins-Nexus-Sonarqube-Slack)
, we built a Continuous Integration pipeline on Jenkins and integrated it with the following tools Maven, Checkstyle, Slack, SonaType Nexus, and SonarQube. Unfortunately building and managing all those tools are requiring huge effort and overhead. We will build the same pipeline with AWS services which will give us a lot of advantages,such as less overhead for managing, less human input, and getting rid of managing standalone servers.

### Current scenario

* Agile Software Development Lifecycle
* Developers make regular code changes (commits)
* Those commits needs to be Tested & Built regularly

### Problem with current scenario

* In Agile SDLC there will be frequent
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

We will create private Git repository in AWS Cloud. Please follow [link](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-ssh-unixes.html#setting-up-ssh-unixes-keys) to configure SSH settings in order to clone/push code to Code Commit.

[![Screenshot-2022-10-29-at-09-58-28.png](https://i.postimg.cc/CKHBptMr/Screenshot-2022-10-29-at-09-58-28.png)](https://postimg.cc/WDzbM5k0)





