DEVEIW 2016
===========

안녕하세요. 포털개발팀 권용근 담당입니다!<br>BackEnd 위주로 진행되는 DEVIEW 2016 2일차 후기를 올립니다.<br> 개발적인 내용은 최대한 줄여 간단한 참석 후기 및 생각을 정리해보았습니다.

![main](/zuminternet/images/main.JPG)

정말 많은 개발자들이 왔습니다. 코엑스 그랜드볼륨에서 총 4개의 트랙으로 진행되었고, 다양한 부스가 준비되었습니다.

**다양한 부스들!**
------------------

### 1. 개발 서적 할인 판매 부스

![book](/zuminternet/images/b1.JPG)

다양한 개발서적 출판사에서 서적 홍보 및 판매를 위해서 부스를 열었습니다! 개발 컨퍼런스에서 흔히 볼 수 있는 광경이지만, 컨퍼런스 규모가 큰만큼 정말 많은 출판사에서 왔습니다!

### 2. 오픈소스 홍보 부스들

**Naver OpenSource**

![o1](/zuminternet/images/nl.PNG)

최근 도입을 고려했던 APM PINPOINT와 사내에서 잘 사용하고 있는 nGrinder 등을 오픈소스로 제공해주는 Naver의 OpenSource 프로젝트를 홍보하기 위해서 부스를 열었네요!

**Scouter**

![o2](/zuminternet/images/scouter.JPG)

개인적으로 관심을 가지고 지속적으로 보고 있는 Scouter APM도 홍보 부스를 열었습니다! <br>Scouter APM은 이번 DEVIEW 2일차 2번 트랙의 세번째 세션의 주제로도 진행되었습니다. Scouter TeamUP 플러그인을 개인적으로 만들어보았는데, 제작자분께 컨트리뷰트를 제안했더니 흔쾌히 허락해주셨습니다ㅎㅎ 잘하면 조만간 Scouter에 TeamUP 플러그인이 공식적으로 올라갈 수도 있을 것 같습니다!

### 3. 스타트업 부스들

![s1](/zuminternet/images/devs.JPG)

쿠키런으로 잘 알려진 데브시스터즈의 부스입니다. AI를 이기면 피규어를 주는 이벤트를 진행하고 있었습니다. 게임에는 소질이 없어서, 참여는 못했습니다..

![s2](/zuminternet/images/b234.png)

사진으로 다 담지는 못했지만 여러 스타트업 부스들에서 제품을 홍보하고 있었습니다!

> 그런데! 이 많은 스타트업이 어째서 굳이 개발 컨퍼런스에 제품을 홍보하러 왔을까 의문이 들었습니다. 많은 사람들이 온다지만 전부 개발자들인데(?)!<br>그래서 부스에 다가가 부스에 방문한 사람들이 하는 대화를 들어보았습니다. <br> "무슨 언어로 개발되었나요?", "어떤 기술을 사용하나요?" 등의 대화와 결정적으로 `"채용을 하고 있나요?"`라는 대화!<br> 제가 생각한 결론은 `채용 홍보`를 위한 부스라고 생각을 했습니다. 많은 경력 개발자들 중 열에 반은 스타트업으로 이직을 한다고 들었습니다. 아마 더 새롭고 재미있는 기술들을 사용하고 싶어하고, 도전하고 싶어하는 개발자들 특유의 욕구때문이라고 생각하고 있습니다. 이런 부스가 개발자들의 욕구에 어필을 많이 하고있지 않을까 생각합니다.

![s1](/zuminternet/images/pr.JPG)

![s1](/zuminternet/images/st.JPG)

부스들을 전부 다 돌고나니 선물들도 많이 챙기게 되었습니다!ㅎㅎ

**들은 세션들!**
----------------

### Session 1. 나는 서버를 썰 터이니 너는 개발만 하여라

![s1](/zuminternet/images/devops.JPG)

개발자가 개발 업무에만 충실 할 수 있도록 신뢰 할 수 있는 인프라 시스템 구축! <br>즉 `DevOps`를 말합니다.

`DevOps Korea group`에서 활동하고 계신 양지욱님께서 발표를 해주셨습니다. 주 내용은 오픈소스 Devops툴인 `Zinst`였지만, 발표 내용으로 `Devops`를 알 수 있었습니다. 인프라팀과 포털개발팀에서 준비 중인 인프라를 갖추는데도 도움이 될만한 좋은 발표였습니다!

발표자료 : [http://www.slideshare.net/deview/231-67605834](http://www.slideshare.net/deview/231-67605834)

---

### Session 2. QUIC을 이용한 네트워크 성능 개선

쿠키런의 라인 진출로 네트워크 CS가 증가하였다고 합니다.`HTTP/1.1`과 `SPDY`의 `모바일에서의 문제`때문이였는데요! 이를 해결하고자 Google이 주도하고 개발한 `QUIC`을 활용한 사례를 소개해주었습니다.

발표자료 : [http://www.slideshare.net/deview/quic-67614063](http://www.slideshare.net/deview/quic-67614063)

---

### Session 3. 스카우터

성능 메트릭의 Visualization으로 결과는 알 수 있다. 그러나 원인을 알 수 없다! 그래서 등장한 APM(성능 메트릭 + 원인 분석을 위한 @) 중 Scouter를 소개하는 세션입니다.

Scouter APM을 구현하는 핵심 기술인 **BCI(Byte code instrumentation)** 를 소개해주어 정말 유익했던 발표였습니다! 이 외의 내용은 이전 제가 들었던 세미나에서 했던 내용과 같았습니다. 정보가 필요하신 분은 [제 블로그로](https://kingbbode.github.io/seminar/2016/08/10/Domain-driven-scouter.html)!

발표자료 : http://www.slideshare.net/deview/213monitoringwithscouter

---

### Session 4는 부스를 돌며 휴식..

딱히 관심있는 분야의 내용이 없어서 Session 4 시간에는 부스를 돌며 휴식을 했습니다 ㅎㅎ

---

### Session 5. 루빅스

다음, 카카오에서 개발하여 활용 중인 `루빅스`를 소개해주었습니다. 마지막 페이지에서 요약까지 해주셔서 정리하기가 좋았습니다!

![루빅스 요약](/zuminternet/images/rbs.png)

발표자료 : [http://www.slideshare.net/deview/235-67609014](http://www.slideshare.net/deview/235-67609014)

루빅스는 관심있어 하시는 분들이 꽤 있을 수도 있어서 정리한 내용을 작성합니다!

루빅스 세션을 들으며 정리한 내용
--------------------------------

*루빅스 총 개발기간 21개월 (2015.02년부터 시작)*

### 루빅스란?

코드네임 Tangram

-	유한한 리소스를 가지고, 다양한 형태를 보여준다는 의미

루빅스는 **콘텐츠 추천 시스템**!

### Model

MAB의 문제점인 Cold Start Problem

-	Cold Start User : 의미있는 통계를 주는 사용자가 많지 않았다.
	-	해결 방법
		-	Multi-Armed Bandit 전략
			-	탐색(Explore) -> 계산(Exploit)
			-	평가가 좋은 곳을 자주 간다
			-	많이 가면 확실(탐색을 줄여도), 적으면 불확실(탐색을 더)
-	지속적 변화
	-	CTR : 관심은 계속 떨어짐
	-	Position, User Cluster 편향

루빅스는 변화를 반영(실시간)하도록 노력했다.

### System

`Request = 50억`

속도에 따라 다른 파이프라인

-	streaming(실시간) : couchbase
-	mini batch(단기) : hbase
-	batch(몇 일) : elastic, openTSDB

#### 목표

-	데이터에 기반해 진화하는 시스템
-	고성능
-	확장성
-	장애 내구성

#### 모댈 개선 과정

다양한 모델들을 계속하여 테스트하며, 가장 좋은 모델을 선정.

-	Offline Test

	-	과거 Log를 이용해서 모델 성능 검증
	-	Online의 환경을 완벽 재현은 힘듬
	-	위험성이 없는 상태에서 모델들을 테스트

-	Online Bucket Test

	-	A/B 테스트
	-	실시간 테스트 분석
	-	테스트 비율을 점차적으로 늘림
	-	Data Flow V2

#### High Performance

서버의 99% Request 응답 시간 : 3ms

**Serving Layer**

Serving Layer는 단순할 수록 강한 성능을 낼 수 있다.

-	nginx
-	play-scala : non blocking의 솔루션
-	Couchbase: **latency 문제에서 가장 강력**
-	zukeeper

latency를 줄이기 위한 노력! 데이터 형태 변경!

*서빙에 최적화된 데이터 준비*

**Network I/O 줄이기**

-	버켓, 콘텐츠 등 미리 가져올 수 있는 정보 Local Cache로 사용
-	ZK watch로 변화 감지하여 Cache

**수평적 확장 가능**

-	노드 추가만으로 확장 가능.

#### 장애 가용성

**Micro service**

작업을 목적에 따라 쪼갤 수 있음.

-	독립적 개발 가능
-	일부 장애가 전체 장애로 전파되지 않음
-	관리 복잡도는 증가

**Service Orchestration Tools**

관리 복잡도를 해결 가능

-	Task 죽지 않도록 관리
-	일관된 배포&롤백, 모니터링할 수 있음

**Test Automation**

-	지속적으로 전체 데이터 파이프라인 테스트
-	state를 확인 가능하도록
-	Alert까지

---

최대한 개발에 대한 내용을 빼서 보고 느낀 것들을 정리해보았습니다!<br>부족한 내용 읽어주셔서 감사합니다!<br>**팀장님, 본부장님 감사합니다 ㅎㅎ**
