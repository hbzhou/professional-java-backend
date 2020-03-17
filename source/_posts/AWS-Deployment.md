---
title: AWS-Deployment
date: 2020-03-17 11:19:14
tags: docker  AWS
---

* prepare docker file,package and build image
  * Dockerfile
    ```javascript 
      FROM openjdk:8-jdk-alpine
      ARG JAR_FILE=target/${build-final-name}-${build-version}-SNAPSHOT.jar
      COPY ${JAR_FILE} app.jar
      ENTRYPOINT ["sh","-c"," java -jar app.jar "]
    ```
  * build package
    ```javascript
        mvn clean package
    ```
  * build image
   ```javascript
     docker build -t a206171-ecat-workflow-entitytransformer .
     docker tag a206171-ecat-workflow-entitytransformer:latest 608014515287.dkr.ecr.us-east-1.amazonaws.com/a206171-ecat-workflow-entitytransformer:latest
   ```
* aws login to generate token with cloud-tool-fr
    ```javascript
    cloud-tool-fr --region us-east-1 --profile default login --account-id "608014515287" --role "human-role/200016-PowerUser" --username "MGMT\MC267998" --password "rU3X96CwgSyqs6Yr"
* login in  ECR and push image
   ```javascript
      aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 608014515287.dkr.ecr.us-east-1.amazonaws.com/a206171-ecat-workflow-entitytransformer
    
      docker push 608014515287.dkr.ecr.us-east-1.amazonaws.com/a206171-ecat-workflow-entitytransformer:latest
   ```
* restart ECS
  ```javascript  
   aws ecs update-service --cluster a206171-ecat-workflow-entitytransformer-cluster --service a206171-ecat-workflow-entitytransformer-service --force-new-deployment
  ```