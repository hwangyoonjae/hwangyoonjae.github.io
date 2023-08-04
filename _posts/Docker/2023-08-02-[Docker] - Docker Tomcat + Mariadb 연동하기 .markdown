---
title: "[Docker] - Docker Tomcat + MariaDB 연동하기"
categories:
  - Docker
tags:
  - [Docker, Tomat, MariaDB]

toc: true
toc_sticky: true

date: 2023-08-02
last_modified_at: 2023-08-02
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
      MARIADB_DATABASE: testdb
      MARIADB_USER: test
      MARIADB_PASSWORD: 12345
      MARIADB_ROOT_PASSWORD: 12345
      TZ: "Asia/Seoul"
```

* * *

## Tomcat index.jsp 생성하기:
```java
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