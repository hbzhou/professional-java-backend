---
title: ECS-FARGATE-DEPLOYMENT
date: 2020-05-12 11:08:39
tags: AWS
---
## create ecs cluster
   ```shell script 
      ecs-cli configure -c ${clustername} -r us-east-1  --default-launch-type FARGATE
      ecs-cli up -e
   ```
## tag the resource
   ```shell script 
      aws ecs tag-resource --resource-arn resource_ARN --tags key=stack,value=dev
   ```
## start or up ecs
   ```shell script
      ecs-cli compose -f %dockerComposeFile% --ecs-params %ecsParamsFile% -p %ProjectName% service start 
      ecs-cli compose -f %dockerComposeFile% --ecs-params %ecsParamsFile% -p %ProjectName% service start --force-deployment
      ecs-cli compose -f %dockerComposeFile% --ecs-params %ecsParamsFile% -p %ProjectName% service up --launch-type FARGATE --create-log-groups
   ```
## check status
   ```shell script
     ecs-cli compose -f %dockerComposeFile% --ecs-params %ecsParamsFile% -p %ProjectName% service ps
   ```
## stop or down esc
   ```shell script
    ecs-cli compose -f %dockerComposeFile% --ecs-params %ecsParamsFile% -p %ProjectName% service stop  
    ecs-cli compose -f %dockerComposeFile% --ecs-params %ecsParamsFile% -p %ProjectName% service down   
   ```
## check logs 
   * list cluster task info 
     ```shell script
       aws ecs list-tasks --cluster  ${clustername}
     ```
   * check container logs
     ```shell script
        ecs-cli logs  -c #{clustername} --container-name ${containerName} --task-id  %taskId% --since %minutes% -t
     ```
## config files
   * docker-compose.yml
        ```yaml
           version: '3'
           services:
             a20617-poc-demo-flowable-ee-service:
               image: 608014515287.dkr.ecr.us-east-1.amazonaws.com/a206171-ds2-poc-flowable-ee:3.6.0.32
               labels:
                 - 'flowable'
               logging:
                 driver: awslogs
                 options:
                   awslogs-group: /ecs/a20617-poc-demo-flowable-ee-service
                   awslogs-region: us-east-1
                   awslogs-stream-prefix: ecs
               ports:
                 - '8080:8080'
                 - '9200:9200'
               volumes:
                 - flowablelib:/opt/flowable/tomcat/webapps/flowable-engage/WEB-INF/lib
               environment:
                 - DB_URL=jdbc:h2:tcp://a20617-flowable-ee-demo-db.fr-nonprod.aws.thomsonreuters.com:1521/test;TRACE_LEVEL_FILE=0;DB_CLOSE_ON_EXIT=FALSE
                 - DB_USERNAME=sa
                 - DB_PASSWORD=sa
                 - ES_URL=http://a20617-flowable-ee-demo-db.fr-nonprod.aws.thomsonreuters.com:9200
             a20617-poc-demo-flowable-jars-service:
               image: 608014515287.dkr.ecr.us-east-1.amazonaws.com/a206171-ds2-poc-flowable-jars:latest
               labels:
                 - 'flowablejars'
               volumes:
                 - flowablelib:/opt/flowable/tomcat/webapps/flowable-engage/WEB-INF/lib
               logging:
                 driver: awslogs
                 options:
                   awslogs-group: /ecs/a20617-poc-demo-flowable-jars-service
                   awslogs-region: us-east-1
                   awslogs-stream-prefix: ecs
           volumes:
             flowablelib:
        ```
   * ecs-params.yml
       ```yaml
          version: 1
          task_definition:
            ecs_network_mode: awsvpc
            task_role_arn: arn:aws:iam::608014515287:role/206171-ecat-poc-camunda-ecsTaskExecuteRole
            task_execution_role: arn:aws:iam::608014515287:role/ecsTaskExecutionRole
            task_size:
              cpu_limit: 2048
              mem_limit: 4GB
            services:
              a20617-poc-demo-flowable-ee-service:
                essential: true
                cpu_shares: 1536
                mem_limit: 3GB
              a20617-poc-demo-flowable-jars-service:
                essential: false
                cpu_shares: 512
                mem_limit: 1GB
            docker_volumes:
              - name: flowablelib
                scope: task
          run_params:
            network_configuration:
              awsvpc_configuration:
                subnets:
                  - subnet-23b5866a
                security_groups:
                  - sg-010f859f1a7b1506c
                assign_public_ip: ENABLED
       ```
   * deploy.bat
     ```javascript
        @echo off
        
        set lanuchType=FARGATE
        set currentEnv=dev
        
        if ""%1"" == ""demo"" (
          set currentEnv=demo
        )
        
        set defaultProjectName=a206171-flowable-ee-%currentEnv%-taskdef
        set defaultDBProjectName=a206171-flowable-ee-%currentEnv%-h2db
        set defaultClusterName=a206171-flowable-ee-%currentEnv%-cluster
        set dockerComposeFile=docker-compose-%currentEnv%.yml
        set ecsParamsFile=ecs-params-%currentEnv%.yml
        set dbDockerComposeFile=db-docker-compose-%currentEnv%.yml
        set dbEcsParamsFile=db-ecs-params-%currentEnv%.yml
        
        echo defaultProjectName: %defaultProjectName%
        echo defaultDBProjectName: %defaultDBProjectName%
        echo defaultClusterName: %defaultClusterName%
        echo dockerComposeFile: %dockerComposeFile%
        
        set bpmComposeSericeCommand=ecs-cli compose -f %dockerComposeFile% --ecs-params %ecsParamsFile% -p %defaultProjectName% service
        set dbComposeServiceCommad=ecs-cli compose -f %dbDockerComposeFile% --ecs-params %dbEcsParamsFile% -p %defaultDBProjectName%  service
        
        rem current do not support EC2
        if ""%2"" == ""EC2"" (
          set lanuchType=EC2
        )
        if ""%3"" == ""EC2"" (
          set lanuchType=EC2
        )
        if ""%4"" == ""EC2"" (
          set lanuchType=EC2
        )
        
        echo ecs-cli configure -c %defaultClusterName% -r us-east-1 --default-launch-type %lanuchType%
        ecs-cli configure -c %defaultClusterName% -r us-east-1 --default-launch-type %lanuchType%
        
        if ""%2"" == ""up"" goto doUp
        if ""%2"" == ""down"" goto doDown
        if ""%2"" == ""status"" goto doStatus
        if ""%2"" == ""ps"" goto doStatus
        if ""%2"" == ""stop"" goto doStop
        if ""%2"" == ""start"" goto doStart
        
        if ""%2"" == ""db"" (
            if ""%3"" == ""up"" (
              goto doH2Up
            )
            if ""%3"" == ""down"" (
              goto doH2Down
            )
            if ""%3"" == ""status"" (
              goto doH2Status
            )
            if ""%3"" == ""ps"" (
              goto doH2Status
            )
        )
        goto doUsage
        
        :doUsage
        echo Usage:  deploy [env] ( commands ... )
        echo commands:
        echo   up                Service up in the AWS cloud
        echo   down              Down the running service in the AWS cloud, and also delete the service
        echo   status, ps        Check the task status for flowable in AWS
        echo   start             Starts one copy of each of the containers on an existing ECS service by setting the desired count to 1 (only if the current desired count is 0)
        echo   stop              Stops the running tasks that belong to the service created with the compose project. This command updates the desired count of the service to 0.
        echo   db up             Up run a h2 db, notice that after run a h2 db, need change the ip address in docker-compose.yml
        echo   db down           Down a h2 db instance
        echo   db status, ps     Check h2 db status
        goto end
        
        :doStart
        if ""%1"" == ""--force"" goto doForceStart
        goto doNormalStart
        
        :doNormalStart
        echo %bpmComposeSericeCommand% start
        %bpmComposeSericeCommand% start
        goto doStatus
        
        :doForceStart
        echo %bpmComposeSericeCommand% start --force-deployment
        %bpmComposeSericeCommand% start --force-deployment
        goto doStatus
        
        :doStop
        echo %bpmComposeSericeCommand% stop
        %bpmComposeSericeCommand% stop
        goto doStatus
        
        :doUp
        echo up command is danger, it will recreate the service and all data will lost
        set upContinue=n
        set /p upContinue=n continue to recreate ECS service [y/n] (default - %upContinue%)?:
        if ""%upContinue%"" == ""n"" goto end
        echo %bpmComposeSericeCommand% up --launch-type %lanuchType% --create-log-groups
        %bpmComposeSericeCommand% up --launch-type %lanuchType% --create-log-groups
        goto doStatus
        
        :doDown
        echo %bpmComposeSericeCommand% down
        %bpmComposeSericeCommand% down
        goto doStatus
        
        :doStatus
        echo %bpmComposeSericeCommand% ps
        %bpmComposeSericeCommand% ps
        goto end
        
        :doH2Up
        echo up command is danger, it will recreate the service and all data will lost
        set upContinue=n
        set /p upContinue=n continue to recreate ECS service [y/n] (default - %upContinue%)?:
        if ""%upContinue%"" == ""n"" goto end
        echo %dbComposeServiceCommad% up --launch-type %lanuchType% --create-log-groups
        %dbComposeServiceCommad% up --launch-type %lanuchType% --create-log-groups
        goto doH2Status
        
        :doH2Down
        echo down command is danger, it will recreate the service and all data will be lost
        set downContinue=n
        set /p downContinue=n continue to recreate ECS service [y/n] (default - %downContinue%)?:
        if ""%downContinue%"" == ""n"" goto end
        %dbComposeServiceCommad% down
        %dbComposeServiceCommad% down
        goto doH2Status
        
        :doH2Status
        echo %dbComposeServiceCommad% ps
        %dbComposeServiceCommad% ps
        
        :end
     ```
   * log.bat
     ```shell script
        @echo off
        set currentEnv=dev
        set minutes=2
        
        if ""%1"" == ""demo"" (
          set currentEnv=demo
        )
        
        if NOT ""%2"" == """" (
          set taskId=%2
          echo taskId=%taskId%
          goto doLog
        )
        
        if ""%2"" == """" (
          echo aws ecs list-tasks --cluster a206171-flowable-ee-%currentEnv%-cluster
          aws ecs list-tasks --cluster a206171-flowable-ee-%currentEnv%-cluster
        )
        
        :doUsage
        echo Usage:  log [env] [taskid] [minutes]
        goto end
        
        
        :doLog
        if NOT ""%3"" == """" (
          set minutes=%3
        )
        
        if ""%4"" == """" (
          echo ecs-cli logs -c a206171-flowable-ee-%currentEnv%-cluster --container-name a20617-poc-%currentEnv%-flowable-ee-service  --task-id  %taskId% --since %minutes% -t
          ecs-cli logs  -c a206171-flowable-ee-%currentEnv%-cluster --container-name a20617-poc-%currentEnv%-flowable-ee-service  --task-id  %taskId% --since %minutes% -t
        )
        
        if ""%4"" == ""jars"" (
          echo ecs-cli logs -c a206171-flowable-ee-%currentEnv%-cluster --container-name a20617-poc-%currentEnv%-flowable-jars-service  --task-id  %taskId% --since %minutes% -t
          ecs-cli logs  -c a206171-flowable-ee-%currentEnv%-cluster --container-name a20617-poc-%currentEnv%-flowable-jars-service  --task-id  %taskId% --since %minutes% -t
        )
        goto end
        
        :end

     ```
## link for reference
  * [Amazon ECS Command Line Reference](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
  

  