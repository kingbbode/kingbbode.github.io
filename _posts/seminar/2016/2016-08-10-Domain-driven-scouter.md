---
layout: post
title: 도메인 드리븐 정기 세미나 - Scouter
categories: [seminar]
tags: [seminar]
fullview: false
comments: true
published: true
outlink: 0
---

세미나 정리 및 후기.

# **Domain Driven APM(SCOUTER)**
>2016년 8월 10일

### 모니터링?
 - UXM
 - NMS
 - SMS
 - APM
 - EMS

### 왜 필요한가?
 > 대부분 문제는 Application.  
문제의 원인을 찾는 것이 중요하기 때문.


## Scouter

#### Scouter 장점
 - 반응속도가 빠르다.
 - Object Request에 다양한 옵션으로 정보를 확인 가능.
 - TPS에서 시간대별 확인 가능()
 - XLog(Affected low count 제공)
 - WAS 한대에 100ms 줬을 때 5%?(제니퍼보다 작음)
 - 수집 서버가 달라도 그룹핑 가능
 - 모든게 바이너리기때문에 hbase기반보다는 훨씬 더 적게 쓸 수 있음(DB가 아닌 파일)
 - 구성요소 배치가 매우 자유로움.
 - 플러그인 달기 편함.

#### 아키텍처
 - Collector : 성능정보 수집
 - Client : 수집된 정보 요청
 - Agent : Host, Java

#### 어떤 문제?
 - THread hang
 - Slow transaction
 - out of memory
 - unhandled exception

#### 문제 
* Slow transaction
    - Active Service를 통해 모니터링 가능(THread 탐색)
    - XLog - Thread/Time 분포도 제공
        - 상세 profile(쿼리(기본), method(옵션 설정))
        - Fetch 시간까지 제공해서 Gap 확인 용이(DB는 쿼리가 끝난 시간만 봄)

* Thread hang
    - Thread List, Dump(Object List)
    - List로 확인 후 Dump로 자세히 확인

* out of memory
    - Memory leak
        - Heap Memory를 확인 가능(GC)
            - Heap Dump로 확인
    - 대량건 조회 
        - Thread Dump로 확인
    - Heap histogram, Heap Dump***
    - XLog에 기본 설정으로 10000건 초과시(too many record exception)

* Resource leak
    - 스카우터 옵션으로 알아서 찾아줌
        - profile_connection_open_enabled=true
        - tarce_rs_leak_enabled=false
        - tarce_stmt_leak_enabled=false

* inefficient Logic / Logic outside of app
    - XLog, Thread Dump를 봐도 별 이상 없을 때
    - 좋은 기능! SFA -(옵션으로 사용 가능, 다른 APM에 없음)
        - 일정 시간 간격으로 Thread Dump를 모아 통계적 분석을 통해

  

### Scouter for service architecture
> 관계의 복잡성으로 인한 문제 발생.  
**cascading failure**

모니터링 cross service tarcing이 중요.


##Scouter 좋은 기능!!

###PLUGIN
* Simple Scripting Plugin
    - Apgent Plugin
        - Http Service Plugin
            - 모든 HTTP 리퀘스트 요청 시작, 끝
        - Http call Plugin
        - capture plugin
            - hook_args_patterns(ex)org.mybatis.spring~~)
                - redefine class - Object Request - Loaded Class - 우클릭 Redefine
            - class, method capture 가능.(hook_args_pattern으로))

    - collector server Plugin
* Built-in Plugin
    - Scouter-Alert-telegram
    - Scouter-Alert-Slack
    - Scouter-Alter-~~?
    - 제작할 수 있음. 어렵지않음.만들어서 jar를 스카우터 서버의 lib에 넣고 스카우터 서버만 재실행
    - 가이드 문서 있음

* Influxdb-plugin
    - 블로그있음, 불완전;
    - demo.scouterapm.com:3000 scouter/scouter

###PULSE(릴리즈는 안됌)
 - http프로토콜을 이용한 성능 카운터 수집 I/F
    - 숫자로 이루어진 데이터의 수집
 - 속도가 빠르진 않음.
 - 쉽게 제작 가능
 - 활용 예
    - Redis / Apache HTTPD 등의 perf stat 정보를 scouter로 전송하여 모니터링

## 정리
elk 
centralized logging / monitoring -> 어플리케이션 수정 없이..

