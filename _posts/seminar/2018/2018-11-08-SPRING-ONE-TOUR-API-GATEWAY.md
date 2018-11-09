---
layout:     post
title:      SpringOne Tour Seoul : Spring Cloud Gateway
author:     kingbbode
tags:       spring think
subtitle:   `스프링 원 투어 서울 컨퍼런스` 중 정윤진님의 `Spring Cloud Gateway` 세션
category:  seminar
outlink: 0
---


2018년 11월 8일 `스프링 원 투어 서울 컨퍼런스` 중 정윤진님의 `Spring Cloud Gateway` 세션

# Spring Cloud Gateway

## SPEC

- Spring 5 +
- Spring Boot 2 +
- API Gateway Pattern

## What is an API Gateway?

하나로 요청을 받아, 요청을 분리.

- Routing
- Canary-ing
- Security
- Monolith Strangling
- Monitoring
- Resiliency

## Spring Cloud Gateway

### Type

- Appliance
- SAAS (ex: ELB)
- Web Server
- Mesh
  - Side Car Pattern + Proxy Pattern
- Developer Oriented

### History

- 초기 버전인 `spring-cloud-zuul` 에서는 `Netflix Zuul 1` 을 사용

- 비동기를 지원하기 위해 `Netflix` 는 `Zuul 2` 로 업그레이드
  - 하위 호환성 없음
  - `Netflix` 는 자신들의 문제만을 해결

- `Zuul` 은 Spring Eco 시스템의 취지에 맞지 않아, `Spring Cloud Gateway` 에서는 `Zuul 2` 를 사용하지 않고 새로 만들게 됨.

### Spec
- Reactor + Netty
- Non-Blocking

### inside Gateway

Handler -> Filter(pre, post, global) -> Service

### vs Netflix

- 어떤 서비스로 보낼 것인가 - (Netflix : Zuul, Spring : cloud-gateway)
- 서비스 중 어디로 보낼 것인가( Netflix : Riborn, Spring : cloud-discovery )

### Configuration

- 과거 버전에서는 YAML방식을 사용
- 과거 버전도 호환하되, Java Configuration 방식 지원

### Eco

- Redis RateLimit
- discovery
- histrix
- security
- micrometer
- zipkin(request tracing)

## TIP TOOL

- httpbin
- httppie
- wssocket (npm)

## Design Idea

### Embedded

Backend Service 가 깨지기 쉬운 형태

### Facade

- Client 에서 특정 요청은 Backend 로 보내고 싶지 않을 때.
- Micro Service 로 전환.

### Corss-Cutting + App-Specific

- Gateway to Gateway


## END

- slides.com/spencer/spring-cloud-gateway
- github.com/ryanjbaxter/gateway-s1p-2018
