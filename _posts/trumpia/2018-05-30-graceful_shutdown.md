---
layout: post
title: graceful shutdown
---

### graceful shutdown 이란
  * 현재 까지 요청 받은 처리에 대해서 처리를 마무리 한 후에 종료하는 것.
  * 종료 시에도 서비스에 문제가 없다.
  * 우리의 경우 이 처리가 안되어 async 처리 등을 못하는 경우가 많았다.

### 종류
  * 일반적인 Agent 에서 종료 하는 방법
  * Tomcat 에서 종료하는 방법
  * Jetty 등을 사용하는 internal Api 에서 종료하는 방법을 구현해야 한다.  
    
### 구현
  * Agent 에서 구현
     * Agent 재시작 할 때, -9 (강제) 옵션을 주면 안됨 - killall java
     * Runtime.getRuntime().addShutdownHook(shutdownThread) 추가
     * Core 를 통해서 Service / Worker 에게 Event 형식으로 전달.
     * Service 는 Worker 가 동작을 멈출 때 까지 대기
     * Worker 는 종류에 따라서 동작을 멈춤
        * ThreadWorker / MgtWorker 는 더이상 DB 에서 데이터를 가져오지 않고 현재 데이터 처리만 마무리
        * MqWorker 는 Mq 에서 데이터를 더이상 가져오지 않고 현재 데이터 처리만 마무리
     * InnerWorker(Async Module) 은 기존 처리를 진행.
        * 최대 30 초 대기후 reduceResource 처리 - 남은 데이터를 file 에 저장 후 재시작시 읽어와서 처리

#### 참조

   * [ JAVA ] JVM Shutdown 상황에서도 로그를 출력해 보자.
     > <http://ccambo.blogspot.com/2014/10/java-jvm-shutdown.html>

   * 톰캣(tomcat) 프로세스 우아하게 종료 시키기 - shutdown.sh 대치
     > <https://www.lesstif.com/pages/viewpage.action?pageId=22643003>
     
   * [Spring Boot] gracefully close를 위한 shutdown method 구현
     > <http://blog.naver.com/PostView.nhn?blogId=userspace&logNo=220827459148>
     
   * Apache HTTP Server Version 2.2
     > <https://httpd.apache.org/docs/2.2/ko/stopping.html>