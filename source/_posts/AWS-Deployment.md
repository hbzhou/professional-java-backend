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
     docker build -t #{imageName} .
     docker tag #{imageName}:#{version} #{imageName}:#{version}
   ```
* aws login to generate token with cloud-tool-fr
    ```javascript
       cloud-tool-fr --region us-east-1 --profile default login --account-id "accountId" --role "aws role" --username "username" --password "password"
    ```
*  login in  ECR and push image
   ```javascript
      aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin #{registryAddr}/#{imageName}
      docker push #{registryAddr}/#{imageName}:#{version}
   ```
* restart ECS
  ```javascript  
   aws ecs update-service --cluster #{imageName}-cluster --service #{imageName}-service --force-new-deployment
  ```