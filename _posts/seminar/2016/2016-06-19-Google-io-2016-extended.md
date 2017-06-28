---
layout:     post
title:      구글 IO 2016 EXTENDED
author:     kingbbode
tags:       spring
subtitle:   세미나 정리 및 후기.
category:  seminar
outlink: 0
---

## **Session1. Progress Web App**
#### Web이 점차 App 형태로(App이 아닌) 변할 것
* 모바일 성장률이 엄청 남

#### 기존 플랫폼을 활용
* Minimal payload & overhead
    - Web Components를 사용 권장.
    - HTTPS
        * HTTP/2
        * Service Worker
        * Push notification
        * Geolocation and More Platfrom APIs
        * Firebase, broti
* Minimal time to first interacation
    - Polymer-CLI, Angularjs2
    - 캐시 지원 환경에서는 Response와 Animation에 더 중점을 두어야 함.
    - Chrome Develop Tools TimeLine 활용
    - Service Worker
        * BackGround Java Script
        * 도구
            - sw-toolbox 관리 툴(fastest, cacheOnly, network 등)
            - Test 도구 : lighthouse,  Chrome Develop Tool - application
* Offline-first
    - Cache
        * Service Work에서 설정 가능.
    - Installable App
        * Manifast 등록
    - Storage
        * IndexedDB??
            - 설정 어려움 -> Idb 활용.(google? 제공 npm)
    - Sync
    - Web PUSH
        * Facebook, jumia, flicart
#### Why?
* MS Serivce Worker Dev Open!
* Native app 보다 훨씬 데이터 절감 등 여러가지 효과.
* 모바일 발전은 앞으로도 더 클 것.
	
***

## **Session2. What's next for the web**
##### Progressive란?
>구글에서 진행하는 캠페인

#### 새 기능
* Promise
    - Web Standards(Callback -> Promise)
* Fetch
    - 리소스 호출
        * 리소스 순서 등등 여러가지를 편리하게.
        * Promise 기반
* Stream
    - 기존 Network의 Response가 도착할 때까지 컨트롤할 수 없는 것을 보완하여 Network 통신 데이터가 쪼개져 버퍼에 저장되고 전체 도착하기 전에 처리 가능.
* RequestIdleCallback
    - 중요하지 않은 일들을 Idle때 처리할 수 있는 콜백
* PassiveEventListener
    - 이벤트 리스너도 패시브하게 동작할 수 있게 하는 옵션
    - Threead bottleneck 해결
* Intersection Observer
    - Dom observe. 통계 데이터 작성시 유용할 듯??.
* Resource Hints
    
* MediaRecorder
    - 동영상 녹화 등 여러가지 가능.
* Media Session
* CSS 기타 등등등..
* Web MIDI
* Web BlueTooth
* Physical Web
    - 새로운 triggering 방식. 
    - Bicon 등 방식을 이용해서 
        * Shorten URL <= 31byte에 주소를 다 저장할 수 없어 크롬 서버 통해 통신을 통해 해당 URL로 전송시킴.?
					
***

## **Session3. Google's PRPL web development pattern**
#### HTML5 Web Components 개요
* 정리
    - HTML import
        * 외부 HTML 참조
    - Shadow DOM
        * HTML 서브셋이 독립된 DOM을 가질 수 있음.
    - Custom Elements
        * 브라우저 기본 DOM의 leaf object /class를 개발자가 임의로 만들 수 있음.
    - Templates
        * 템플릿을 만들고 디자인 가능.
* WebComponents.js를 통해 지원. (IE 11+)
* Polymer
    - HTML5 컴포넌트 기반 라이브러리
    - 단점
        * 크고 느림.
        * 의존성 체크 오버헤드.
#### PRPL 개발 패턴
* 딥링크/사용자 액션/앱과 유사/오프라인대체가 잘되야 좋은 앱?(구글 기준)
* Push Render Pre-Cache Lazy-Loading이란 패턴.
    - 푸시하고
    - 랜더링하고
    - 캐시해놓고
    - 레이지로드하고 페이지 만듬.
    
* IE 비중이 높은 우리나라에서는 안댐..
* Custom Element
    - HTML에는 CustomElement로 구성된 소스만 들어있고, 정적인 요소까지도 App Route에서 처리.
* Import
    - Component Load
* HTTP/2
    - HTTP는 자잘한 리소르르 다루기 힘듬 -> 복잡한 툴체인, 비효율적 번들
    - 한번 요청으로 여러가지 요청을 하고 리스폰스를 받음. 
* Service Worker
    - 일반 웹은 네트워크가 없으면 동작하지 않으며, 불필요한 리퀘스트가 많음.
    - Request 가로채고 처리, 적절 캐시, 오프라인 캐시.
