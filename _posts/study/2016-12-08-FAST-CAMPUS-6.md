---
layout:     post
title:      Fast-Campus Java Web Programming Camp 6일차
author:     kingbbode
tags:       spring
subtitle:   강의 정리
category:  study
outlink: 0
---

여섯번째 강의
=============

배운 것
-------

### 자바스크립트

#### Diynamic Event Binding

```
$(".answer-write input[type='submit']").on("click", addAnswer);
$(".qna-comment-slipp-articles").on("click", ".delete-answer-form button[type='submit']", deleteAnswer);
```

#### Javascript를 사용함에 알아두면 좋은 것들

-	event loop + async
-	event delegation(event bubble, event propagation)
-	this
-	closure

### 외장 Tomcat Server 배포

#### Step 1. 톰캣 설치

```
wget {톰캣 tar.gz 링크주소}

tar -xvf {다운로드받은 파일}

심볼릭 링크 설정(선택)
ln -s {압축 풀린 경로} tomcat

`/bin`에 실행파일 있다.

`jps` - 떠있는 자바 프로세스를 확인할 수 있다.
```

#### Step 2. Spring Boot 설정

-	패키징 타입을 `WAR`로 변경

```JAVA
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-tomcat</artifactId>
  <scope>provided</scope>
</dependency>
```

`scope`를 `provided`로 하면 패키징시 제외

Tomcat은 패키징된 War의 Servlet을 실행

Embed Tomcat에서는 `@SpringBootApplication`로 실행할 메인 클레스를 명시를 해주지만, <br> 외장 톰켓에서는 SpringBootServletInitializer의 상속을 통해 실행될 Class를 지정할 수 있다.

#### Step 3. 배포

외장 톰켓의 기본 설정 APP은 `webapp/ROOT`

**빌드**

```
./mvnw clean package
```

빌드된 폴더를 해당 경로로 설정하여 톰켓 실행

#### Step 4. 배포 자동화

bash shell script 사용

```bash
#!/bin/bash

REPOSITORIES_DIR=~/my-jwp2
TOMCAT_HOME=~/tomcat

cd $REPOSITORIES_DIR
pwd
git pull
./mvnw clean package

$TOMCAT_HOME/bin/shutdown.sh

rm -rf $TOMCAT_HOME/webapps/ROOT
mv $REPOSITORIES_DIR/target/my-jwp-1.0 $TOMCAT_HOME/webapps/ROOT

$TOMCAT_HOME/bin/startup.sh

tail -500f $TOMCAT_HOME/logs/catalina.out
```
