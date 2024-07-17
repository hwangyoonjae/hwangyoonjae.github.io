---
layout: post
title: "[Docker] - Docker Tomcat + MariaDB 연동하기"
date: 2023-08-02
categories: Docker
tags: [Docker, Tomat, MariaDB]
image: /assets/post/docker_wallpaper.jpg
---

## Docker Tomcat 구성하기:
- Docker Tomcat 구성을 아래와 같이 파일 형태로 사용한다.
```bash
# 파일명은 docker-tomcat.yml
version: "3.3"
# 서비스 구성
services:
  tomcat1:
    image: tomcat
    container_name: tomcat1
    restart: always
    ports:
      - 10001:8080
    volumes:
      - ./tomcat1/webapps/:/usr/local/tomcat/webapps
    environment:
      TZ: "Asia/Seoul"
```

* * *

## Docker MariaDB 구성하기:
- Docker MariaDB 구성을 아래와 같이 파일 형태로 사용한다.
```bash
# 파일명은 docker-mariadb.yml
version: "3.3"
# 서비스 구성
services:
  master_mariadb:
    image: mariadb
    container_name: master_mariadb
    restart: always
    ports:
      - 3306:3306
    volumes:
      - ./mariadb/master/conf.d:/etc/mysql/conf.d
      - ./mariadb/master/data:/var/lib/mysql
      - ./mariadb/master/mysql-init-files/:/docker-entrypoint-initdb.d/
    environment:
      MARIADB_DATABASE: dockerdb
      MARIADB_USER: test
      MARIADB_PASSWORD: 12345
      MARIADB_ROOT_PASSWORD: 12345
      TZ: "Asia/Seoul"
```

* * *

## Tomcat index.jsp 생성하기:
- DB에 있는 데이터를 웹페이지의 보여주기 위해 아래와 같이 index.jsp를 생성한다.
```bash
$ cd /tomcat1/webapps/ROOT
$ vi index.jsp
```

```html
<%@ page contentType = "text/html; charset=utf-8" %>
<%@ page import = "java.sql.DriverManager" %>
<%@ page import = "java.sql.Connection" %>
<%@ page import = "java.sql.Statement" %>
<%@ page import = "java.sql.ResultSet" %>
<%@ page import = "java.sql.SQLException" %>
<html>
    <head>
        <title>사용자 테이블</title>
    </head>
    <body>
        <h1>사용자 테이블</h1>
        <table width="100%" border="1">
            <tr>
                <td>이름</td>
                <td>주소</td>
                <td>나이</td>
            </tr>
            <% // MySQL JDBC Driver Loading
                Class.forName("org.mariadb.jdbc.Driver");
                Connection conn = null;
                Statement stmt = null;
                ResultSet rs = null;
                try {
                    String jdbcDriver = "jdbc:mysql://DB서버IP:3306/DB명";
                    String dbUser = "사용자계정";
                    String dbPass = "사용자패스워드";
                    String query = "select * from 테이블명";

                    // Create DB Connection
                    conn = DriverManager.getConnection(jdbcDriver, dbUser, dbPass);

                    // Create Statement
                    stmt = conn.createStatement();

                    // Run Qeury
                    rs = stmt.executeQuery(query);

                    // Print Result (Run by Query)
                    while(rs.next()) {
                        %>
            <tr>
                <td><%= rs.getString("name") %></td>
                <td><%= rs.getString("address") %></td>
                <td><%= rs.getString("age") %></td>
            </tr>
            <%
                }
            } catch(SQLException ex) {
                out.println(ex.getMessage());
                ex.printStackTrace();
            } finally {
                // Close Statement
                if (rs != null) try { rs.close(); } catch(SQLException ex) {}
                if (stmt != null) try { stmt.close(); } catch(SQLException ex) {}

                // Close Connection
                if (conn != null) try { conn.close(); } catch(SQLException ex) {}
            }
            %>
        </table>
    </body>
</html>
```

* * *

## MariaDB JDBC 드라이버 이용하여 연동하기:
- 아래 URL 접속하여 MariaDB JDBC 드라이버 파일을 다운받는다. 
> * [MariaDB JDBC 다운로드](https://downloads.mariadb.com/Connectors/java/ "MariaDB JDBC 다운로드")

- 다운받은 jdbc 드라이버를 WEB-INF/lib 폴더 안에 넣어준다.
[![docker tomcat jdbc 파일 위치](/assets/images/docker/docker%20tomcat%20jdbc%20파일%20위치.PNG)](/assets/images/docker/docker%20tomcat%20jdbc%20파일%20위치.PNG)

* * *

## Docker Container 생성하기:
- 위에서 작성한 compose 파일을 통하여 Docker Container를 생성한다.
```bash
$ docker stack deploy -c docker-tomcat.yml web
$ docker stack deploy -c docker-mariadb.yml db
```

* * *

## 웹페이지에서 확인하기:
- Tomcat에서 DB값을 제대로 불러오는지 웹페이지로 접속하여 확인한다.
[![docker tomcat, mariadb 연동 성공 화면](/assets/images/docker/docker%20tomcat,%20mariadb%20연동%20성공%20화면.PNG)](/assets/images/docker/docker%20tomcat,%20mariadb%20연동%20성공%20화면.PNG)

* * *