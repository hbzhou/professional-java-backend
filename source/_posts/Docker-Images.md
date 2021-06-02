---
title: Docker Images abbrlink: 42416 date: 2020-06-18 22:29:19 tags:
---

- nacos

    ```jsx
    docker run --env MODE=standalone --name nacos -d -p 8848:8848 nacos/nacos-server
    ```

- mysql

    ```jsx
    docker run -p 3306:3306 --name mysql -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6
    ```

-
