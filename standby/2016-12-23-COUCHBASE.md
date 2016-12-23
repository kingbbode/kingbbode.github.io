CouchBase(Membase) 장애 복구에 관하여
=====================================

환경
----

-	Membase Cluster
	-	DB Server 01 (Membase)
	-	DB Server 02 (Membase)
-	WEB Server
	-	Spring Boot Web Application
		-	SpyMemCached

---

사전 알아야 할 내용
-------------------

### CouchBase(Membase)

**1. Bucket**

**2. Node**

### Spring Security

**1. SecurityContextPersistenceFilter**

**2. SecurityContextRepository**

### SpyMemcached

**1. MemcachedClient**

**2. Session**

1.모든 Membase Server의 장애 상황에서 어플리케이션을 구동할 수 없는 문제
------------------------------------------------------------------------

### 문제 현상

### 문제 원인

**MemcachedClient**

### 해결 방안

**1. 동적 빈 생성**

**2. 복구 스케줄링**

2.일부 Membase 서버의 장애 발생
-------------------------------

### 문제 현상

**OpertationTimeOut**

`OpertationTimeOut: Timeout waiting for value`

### 문제 원인

**Auto Failover**

### 해결 방안

**increasing the DefaultOperationTimeout so that more retry attempts**

3.어플리케이션이 구동 후 모든 Membase Server 장애 발생
------------------------------------------------------

### 해결 방안

**감지 및 복구 스케줄링**

**Server Restart & Pending**
