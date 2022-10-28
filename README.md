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



