---
layout:     post
title:      Nexus 3.X - Maven, NPM 저장소로 이용하기
author:     kingbbode
tags:       web ci repository
subtitle:   Nexus에서 Maven 저장소, NPM 저장소를 구성하고 사용하는 내용을 정리
category:  posts
outlink: 0
---

**대부분의 Nexus 관련 글이 2.x.x 버전을 기준으로 작성되어 있어 3.x.x 기준으로 간단히 정리를 해보려고 합니다.**

Nexus란?
--------

다양한 Format 의 저장소를 제공하는 Sonatype 에서 진행 중인 Repository Manager 프로젝트 입니다. Maven 에서 사용할 수 있는 가장 널리 사용되는 무료 저장소(정확하게는 무료 제품인 Nexus OSS)로 잘 알려져있습니다.

Nexus는 사설 저장소를 제공해주는 도구입니다.

사설 저장소의 필요 이유는 [(가리사니-사설 Repository Nexus 설치 / 연동)](https://gs.saro.me/#!m=elec&jn=774) 에 박용서님이 작성해주신 글에 잘 설명되어있습니다.

![why](/images/2017/2017-07-04-NEXUS-3XX-MAVEN-NPM/why.png)

*(출처 : [가리사니-사설 Repository Nexus 설치 / 연동](https://gs.saro.me/#!m=elec&jn=774)\)*

**몇 가지 추가하자면,**

-	개발팀에서 사용하는 공통 라이브러리들을 공유
-	특정 솔루션을 사용하기 위한 3rd Party 라이브러리의 관리

를 위해서도 필요하다고 생각됩니다.

---

### DIFF NEXUS 2.X, 3.X

Nexus Repository 3은 아키텍처와 기능을 완전히 새로 작성하여 만들어졌다고 합니다. 그래서 2.X 버전과의 동등성과 하위호환성이 보장되지 않습니다.

![jeff](/images/2017/2017-07-04-NEXUS-3XX-MAVEN-NPM/jeff.png)

2.x 버전에서 3.x 버전으로 업그레이드를 고려하신다면[(Nexus Repository Manager 2.x to 3.x Feature Equivalency)](https://support.sonatype.com/hc/en-us/articles/226489388-Nexus-Repository-Manager-2-x-to-3-x-Feature-Equivalency) 를 꼭 참고하시길 바랍니다!

### 지원 포맷

#### NEXUS 2.X

![oss2](/images/2017/2017-07-04-NEXUS-3XX-MAVEN-NPM/oss2.png)

#### NEXUS 3.X

![oss3](/images/2017/2017-07-04-NEXUS-3XX-MAVEN-NPM/oss3.png)

2.X 에서 3.X 로 업그레이드되면서 지원 포맷이 확장되었습니다!

**Docker** 와 **npm** 까지!! ( 제가 .NET과 Python, Ruby를 안써서 나머지는 인기있다는 정도만 압니다! )

2.X 버전 때는 Nexus는 `Maven 저장소` 라는 인식이 강했는데, 3.X 버전의 Nexus는 **Maven 뿐만아니라, 다양한 포맷을 지원하는 저장소 관리자** 라는 생각이 확 듭니다!

이제 본격적으로 Nexus `설치`부터, `Maven`, `NPM` 까지의 사용 방법입니다.

설치
----

Nexus를 설치하는 방법은 크게 2가지 입니다.

Sonatype 사이트에서 제공하는 압축 파일을 통해 설치 및 실행하는 방법과, Docker 이미지로 설치 및 실행하는 방법입니다.

### 파일 설치 및 실행

Sonatype 홈페이지(https://www.sonatype.com/download-oss-sonatype)에서 OSS 3.X 버전을 다운로드하고 압축을 해제합니다.

압축을 해제했을 때 아래의 2개의 폴더가 보입니다.

![folder](/images/2017/2017-07-04-NEXUS-3XX-MAVEN-NPM/folder.png)

-	nexus 3.x

	-	Nexus Repository Manager 어플리케이션 폴더

-	sonatype-work

	-	Data-store로 저장소, 설정, 캐시 등의 모든 데이터가 이 폴더 하위에 저장

`nexus 3.x` 폴더 하위의 `bin` 폴더에는 Nexus 어플리케이션 구동에 관한 스크립트가 있습니다. `bin` 폴더 하위의 `nexus`를 실행합니다.

**UNIX**

```
./nexus run
```

**WINDOW**

```
nexus.exe /run
```

실행에 성공하면!

![run](/images/2017/2017-07-04-NEXUS-3XX-MAVEN-NPM/run.png)

이 외의 다양한 설정은 [공식 Document](https://books.sonatype.com/nexus-book/reference3/install.html)를 참고! (위 내용도 이 문서에 다 있습니다.)

### Docker

[Sonaytype Github](https://github.com/sonatype/docker-nexus3)에 설명이 잘 되어있어 링크로 대체합니다!

#### 설치 끝

-	요약
	-	다운 받는다.
	-	실행한다.

![welcome](/images/2017/2017-07-04-NEXUS-3XX-MAVEN-NPM/welcome.png)

기본 ID 와 비밀번호는 `admin / admin123` 입니다!

공통
----

### Blob Stores

모든 저장소는 Blob Stores를 지정해줘야 합니다. Blob Store는 실제 데이터가 저장되는 장소이며, sonatype-work의 하위 디렉토리와 매칭됩니다.

![blob](/images/2017/2017-07-04-NEXUS-3XX-MAVEN-NPM/blob.png)

### Repository Type

모든 Format에 3가지 타입이 있습니다.

-	proxy

	-	말그대로 외부의 다른 경로를 proxy하는 저장소입니다. 이 proxy를 통해 외부(공식 라이브러리 저장소 등)의 데이터를 캐시할 수 있습니다.

-	hosted

	-	하나의 사설 라이브러리 집단입니다.

-	group

	-	proxy 와 hosted 들을 묶을 수 있는 단위 집단입니다. 내부에서 정하는 순서에 의해 내부 라이브러리를 탐색합니다.

Gradle에서 Maven Repository로 사용
----------------------------------

이 곳에서 Maven의 Snapshots과 releases 구성에 대한 설명은 하지 않겠습니다. 기본적으로 아래와 같은 샘플들이 주어집니다.

![maven-example](/images/2017/2017-07-04-NEXUS-3XX-MAVEN-NPM/maven-example.png)

설정을 참고하여 구조를 작성할 수 있습니다.

### Upload

> build.gradle

```java
apply plugin: 'maven'

group = "com.kingbbode"
version = '0.0.1-SNAPSHOT'

project.group = 'com.kingbbode'

uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: 'http://your.nexus.repository.com/nexus/content/repositories/releases/') {
                authentication(userName: '계정명', password: '비밀번호')
            }
            snapshotRepository(url: 'http://your.nexus.repository.com/nexus/content/repositories/snapshot/') {
                authentication(userName: '계정명', password: '비밀번호')
            }
        }
    }
}
```

[ 출처 : [( 권남 : Gradle Maven Deployment )](http://kwonnam.pe.kr/wiki/gradle/maven)\]

`project.name`이 `artifactId` 가 됩니다.

![upload](/images/2017/2017-07-04-NEXUS-3XX-MAVEN-NPM/upload.png)

`uploadArchives` 를 실행하면 Nexus에 업로드가 됩니다.

Nexus의 Component를 살펴보면, 업로드한 프로젝트가 올라와 있는 것을 볼 수 있습니다.

![d1](/images/2017/2017-07-04-NEXUS-3XX-MAVEN-NPM/d1.png)

![d2](/images/2017/2017-07-04-NEXUS-3XX-MAVEN-NPM/d2.png)

![d3](/images/2017/2017-07-04-NEXUS-3XX-MAVEN-NPM/d3.png)

### Download

Nexus를 통해 Dependency를 가져올 때는 해당 Hosted를 직접 지정하거나 상위 Group을 지정할 수 있습니다.

> build.gradle

```java

repositories {
  maven {
    credentials {
      username "admin"
      password "admin123"
    }
    url "http://your.nexus.repository.com/nexus/content/repositories/public/"
  }
}
```

NPM Mirror Server로 사용
------------------------

아래와 같은 구조를 기본 구조로 사용합니다.

![npm](/images/2017/2017-07-04-NEXUS-3XX-MAVEN-NPM/npm.png)

`npm-registry` 는 http://registry.npmjs.org 를 Proxy하며, `npm-private`에 npm 모듈을 올리려고 합니다.

`npm` 은 내부 Scope가 겹칠 수 있으므로, 한 `Hosted` 저장소에 한 모듈만 올라가도록 구성합니다.

![npmg](/images/2017/2017-07-04-NEXUS-3XX-MAVEN-NPM/npmg.png)

`npm-group` 은 두 저장소를 묶습니다.

### Upload

`addUser`를 통해 `global` 로 인증을 선언할 수 있지만, Nexus 저장소의 특성상 계정별 접근 권한 및 다양한 구성과 환경이 있을 것을 감안하여, 저는 각 프로젝트에 `.npmrc` 파일을 생성하였습니다.

> .npmrc

```java
_auth=YWRtaW46YWRtaW4xMjM=
```

`.npmrc`의 `_auth`에는 `<username>:<password>`의 형태가 base64 인코딩되어 작성되야 합니다.

```shell
echo -n 'myuser:mypassword' | openssl base64
```

위 스크립트를 통해 추출할 수 있습니다.

그리고 `package.json` 에 publish 될 저장소 url을 명세합니다.

> package.json

```javascript
{
  ...

  "publishConfig": {
    "registry": "http://your.nexus.repository.com/repository/npm-private/"
  }
}
```

여기까지 완료되었다면 아래 명령을 통해 Nexus로 배포할 수 있습니다.

```
npm publish
```

![npmp](/images/2017/2017-07-04-NEXUS-3XX-MAVEN-NPM/npmp.png)

배포를 위한 package.json 및 모듈 작성 방법은 아웃사이더님의 [Node.js 모듈을 npm 저장소에 배포하기](https://blog.outsider.ne.kr/829)를 추천!

### Download

Nexus를 미러 서버로 사용하기 위해서도 global 로 선언이 가능지만,

```
npm --registry http://your.nexus.repository.com/repository/npm-group/ install -g your-pac
```

각 프로젝트에서 `.npmrc` 를 통해 설정하는 방법도 있습니다.

> .npmrc

```
registry=http://your.nexus.repository.com/repository/npm-group/
```

![npmd](/images/2017/2017-07-04-NEXUS-3XX-MAVEN-NPM/npmd.png)

마무리
------

Nexus를 이용해 사설 저장소를 구성하니, 프로젝트의 모듈을 구성하거나, 라이브러리를 제공하는 등 여러가지 면에서 좋은 점이 많은 것 같습니다. 젠킨스도 달아보고, 도커도 해보고 여러가지를 더 해봐야겠습니다.
