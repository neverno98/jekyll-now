---
layout: post
title: hbm2ddl.auto 옵션
---

### 현재까지 hibernate.hbm2ddl.auto 옵션을 validate 를 사용하였다.
   > release 시에 db 적용을 확인하고 올려야 원하지 않는 오류를 줄일 수 있기 때문이였다.

### 장점
  * 릴리즈 시에 DML 오류에 의해서 원치 않는 시점에 발생하는 오류를 막을 수 있다.
  * 현재 스크립트로 DML 을 동작시키긴 하지만 완벽하지 않는 경우가 여전히 발생한다.
    > 이전에 비해서는 많이 줄었다.
    
### 단점
  * 우리는 돌아가는 프로세스가 하나가 아니라 여러가지가 돌아간다. 이에 따라 쓸데없이 중복해서 여러번 DML 검사를 하게 된다.
  * 한꺼번에 프로세스를 재시작하게 되는 데 검사를 진행하느라 실제 서비스의 동작이 늦게 시작된다.
    > 이에 따른 불만 사항이 많았다.
    
### 개선 
* 기본 옵션을 none 으로 수정한다.
* 코드 프리즈 / 릴리즈 시에는 특정한 프로세스만 system.property 로 옵션을 주어서 validate 로 실행하여 DML 이 정상적인지 확인 과정을 거친다.


#### 참조

   * hibernate.hbm2ddl.auto 설명
     > <https://github.com/HomoEfficio/dev-tips/blob/master/hibernate.hbm2ddl.auto%20%EC%9C%84%ED%97%98%20%ED%97%B7%EC%A7%80.md>

   * md 문법
     > <https://gist.github.com/ihoneymon/652be052a0727ad59601>