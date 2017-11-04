---
layout:     post
title:      젠킨스 파이프라인 사용하여 자동 배포환경 만들어보기!
author:     kingbbode
tags:       think, ci
subtitle:   Build Pipeline 으로 작성했던 배포 구조를 Jenkins Pipeline으로 이전!
category:  posts
outlink: 0
---

[지난 글(젠킨스 사용하여 자동 배포환경 만들어보기)](https://kingbbode.github.io/posts/jenkins-build) 에서 Jenkins Build Pipeline 으로 자동 배포 시스템을 구축하는 것에 대해 공유를 했었습니다.

![huk](/images/2017/2017-11-04-JEKINS-PIPELINE/huk.png)

댓글을 통해 `Jenkins Pipeline` 이라는게 있다는 것을 처음 알고 되고..!!

그 때서야, 젠킨스 2.0 의 가장 큰 변화가 `Jenkins Pipeline` 이라는 것을 알게 되었습니다. 이 때부터 마음의 숙제로 가지고 있었지만, 최근 블로그 통계로 젠킨스 블로깅 글의 방문이 높다는 것을 알고 급하게(groovy를 제대로 숙지하지 못한 채..ㅜㅜ) 글을 작성하게 되었습니다.

![late](/images/2017/2017-10-29-JAVA9-PRACTICE/late.png) (최근 이 짤을 굉장히 많이 쓰게 됩니다..)

`Jenkins Pipeline` 이란 스크립트를 통해 파이프라인의 흐름을 정의하는 기능입니다. 이 스크립트는 `groovy`로 작성되며, Pipeline DSL을 통해 전달 파이프라인을 작성할 수 있습니다.

**Jenkins Pipeline에 대한 자세한 설명은 생략!**

그리하여, 저 또한 지난 `Build Pipeline`으로 작성했던 시스템을, `Pipeline Script` 기반으로 모두 전환하였습니다.

구축한 내용과 결과를 공유합니다.

구축하고자 하는 환경
--------------------

구축하고자 하는 환경은 지난 Build Pipeline 으로 구축했을 당시의 조건을 그대로 구축하는 것 입니다.

Git 연동이나, Gradle, Maven 빌드 등 기본적인 조건을 제외하고 가장 원했던 조건은 아래와 같습니다.

-	공통 내용을 모듈화하여 공용으로 사용할 수 있는 구조.
-	각 단계에 대한 수동 실행이 가능한 구조.

구축
----

### Jenkins File

젠킨스는 `Pipeline Script`를 Admin Web에서 직접 작성하여 저장하는 방법과, Git을 통해 JenkinsFile 을 읽는 방법을 제공합니다.

만들고자 하는 스크립트는, **배포 Flow가 지나치게 다르지 않은 프로젝트에 한하여, 모두 동일한 스크립트를 사용할 수 있도록 구성하여 파라미터 기반으로 실행될 수 있는 스크립트** 입니다.

그렇기 때문에 저장소에 대한 수정만으로 모든 프로젝트에 적용할 수 있는 Git을 통해 JenkinsFile 을 읽는 방법을 사용합니다.

*groovy 사용이 미숙하여, 코드가 더러운 점은 양해 부탁드립니다. groovy 사용성을 숙지하여 추후 리펙토링할 예정입니다.*

#### Stage

젠킨스 `Pipeline Script`는 실행에 대한 단위인 `node` 와 파이프라인을 구성하는 `stage`가 있습니다. `node` 와 `stage` 간에는 먼저 선언되야 하는 것은 없이 작성할 수 있습니다.

```
node('', {
  stage('example1', {
    echo ""
  })
})

stage('example2', {
  node('', {
    echo ""
  })
})
```

`Pipeline Script`는 병렬 실행을 지원하기 때문에 실행 단위를 분리해야할 경우 `node`를 사용하여 실행 단위를 분리할 수 있고, `Pipeline`의 단계를 `stage`로 구성하게 됩니다.

자동 배포를 위해 만드려고 하는 Stage는 아래와 같습니다.

> `Flow Check` - `Parameter Check` - `Git CheckOut` - `Test` - `Build` - `Deploy` - `Switch`

##### 파라미터 정의

**Flow**

각 잡 실행 Pipeline에서 진행할 `Stage` 를 정의하는 파라미터들입니다.

`USE_TEST` : Test 단계 사용 유무

`USE_BUILD` : Build 단계 사용 유무

`USE_DEPLOY` : Deploy 단계 사용 유무

`USE_SWITCH` : Switch 단계 사용 유무

**Module**

각 프로젝트 Pipeline Job에서 넘겨주어야 할 모듈 정보입니다.

`GRADLE_VERSION` : Gradle Version

`JAVA_VERSION` : Java Version

`NODE_VERSION` : Node Version (Optional)

`SLACK_TOKEN` : Slack Token (Optional)

**Git**

소스 Check Out 을 위한 정보입니다.

`GIT_URL` : Git Repository Url

`BRANCH_SELECTOR` : 대상 Branch

**Deploy**

그리고 각 배포 환경에 따라 정의되어야 하는 파라미터들이 있습니다. 아래 파라미터는 제가 배포해야할 환경에 필요한 파라미터로 각 배포 환경에 맞게 구성하시면 됩니다.

`CONFIG_NAME`, `REMOTE_PATH`, `TARGET_USER`, `TARGET_SERVER`

##### 변수 정의

**Flow**

```
def useTest = true
def useBuild = true
def useDeploy = true
def useSwitch = true
```

`각 단계에 대한 수동 실행이 가능한 구조` 를 위해 사용되는 변수들입니다. 이 변수들은 `Stage` 내부에서 확인될 것 입니다.(Stage를 분기처리하면 전체 Flow가 망기지기 때문에) 기본 값은 모두 실행되는 것이며, 이후 아래에서 설명할 Pipeline Job을 만드는 과정에서 이 값들이 어떻게 쓰일지 다시 한번 설명하도록 하겠습니다.

**Module**

```
def useNode = true
def useSlack = true
```

해당 모듈을 사용할지 결정하는 변수입니다. 모든 프로젝트가 Gradle Build라는 가정하에 적성하여, `useGradle`, `useMaven` 는 작성하지 않았습니다. 필요하다면 작성하셔서 사용하면 좋을 것 같습니다.

##### 1. Flow Check

이 과정은 해당 빌드가 어떤 스테이지를 실행하는지 확인할 수 있도록 작성한 `Stage` 입니다.

```
stage("Flow Check", {
    try {
        println "  TEST FLOW = $USE_TEST"
        useTest = "$USE_TEST" == "true"
    }
    catch (MissingPropertyException e) {
        println "  TEST FLOW = true"
    }

    try {
        println "  BUILD FLOW = $USE_BUILD"
        useBuild = "$USE_BUILD" == "true"
    }
    catch (MissingPropertyException e) {
        println "  BUILD FLOW = true"
    }

    try {
        println "  DEPLOY FLOW = $USE_DEPLOY"
        useBuild = "$USE_DEPLOY" == "true"
    }
    catch (MissingPropertyException e) {
        println "  BUILD DEPLOY = true"
    }

    try {
        println "  SWITCH FLOW = $USE_SWITCH"
        useBuild = "$USE_SWITCH" == "true"
    }
    catch (MissingPropertyException e) {
        println "  SWITCH FLOW = true"
    }
})
```

`$`로 시작하는 이름들은 파라미터로 넘겨받을 환경 변수들입니다. 해당 환경변수가 설정되지 않았을 때 `MissingPropertyException`이 발생하지만, 이 값들은 없으면 Default 값으로 동작하도록 예외처리를 하였습니다.

##### 2. ParameterCheck

```
stage("Parameter Check", {
    println " BUILD_USER = " + BUILD_USER
    println " CONFIG_NAME = $CONFIG_NAME"
    println "  REMOTE_PATH = $REMOTE_PATH"
    println "  TARGET_USER = $TARGET_USER"
    println "  TARGET_SERVER = $TARGET_SERVER"
    println "  GIT_URL = $GIT_URL"
    println "  BRANCH_SELECTOR = $BRANCH_SELECTOR"
    println "  GRADLE_VERSION = $GRADLE_VERSION"
    println "  JAVA_VERSION = $JAVA_VERSION"

    env.JAVA_HOME="${tool name : JAVA_VERSION}"
    env.PATH="${env.JAVA_HOME}/bin:${env.PATH}"

    try {
        println "  SLACK_TOKEN = $SLACK_TOKEN"
    }
    catch (MissingPropertyException e) {
        useSlack = false
    }
    try {
        println "  NODE_VERSION = $NODE_VERSION"
    }
    catch (MissingPropertyException e) {
        useNode = false
    }
})
```

파라미터를 검증하는 `Stage` 입니다. 환경변수가 없을 때는 예외를 발생시키므로, 필수 사용 파라미터에 대해서는 예외처리를 하지 않습니다.

##### 3. Git CheckOut

```
stage("Git CheckOut", {
    if (useTest || useBuild) {
        println "Git CheckOut Started"
        checkout(
                [
                        $class                           : 'GitSCM',
                        branches                         : [[name: '${BRANCH_SELECTOR}']],
                        doGenerateSubmoduleConfigurations: false,
                        extensions                       : [],
                        submoduleCfg                     : [],
                        userRemoteConfigs                : [[url: '${GIT_URL']]
                ]
        )
        println "Git CheckOut End"
    } else {
        println "Git CheckOut Skip"
    }
})
```

넘겨받은 `GitUrl`과 `Branch`를 사용하여 Git Check Out 을 실행하는 `Stage` 입니다. `Test`나 `Build` 중 하나라도 Flow에 포함되어 있다면 이 단계가 실행됩니다.

##### 4. Test

```
stage('Test') {
    if (useTest) {
        println "Test Started"
        try {
            sh "${tool name: GRADLE_VERSION, type: 'hudson.plugins.gradle.GradleInstallation'}/bin/gradle test -Dorg.gradle.daemon=true"
        } finally {
            junit allowEmptyResults: true, keepLongStdio: true, testResults: 'build/test-results/*.xml'
        }
        println "Test End"
    } else {
        println "Test Skip"
    }
}
```

넘겨받은 Gradle Version을 사용하여, test Task 를 실행시킵니다. 테스트가 완료된 후에는 테스트 결과를 수집합니다. testResults는 각 프로젝트 환경에 맞는 `Path`를 작성합니다.

##### 5. Deploy, Switch(Manual Flow)

두 `Stage`는 독립된 `Stage` 이지만 자세한 내용을 공개할 수 없으므로, 간단히 작성합니다.

```
stage("Deploy", {
    if(useDeploy) {
        println "Deploy Started"
        sh "****"
        println "Deploy End"
    } else {
        println "Deploy Skip"
    }
})

stage("Switch", {
    if(useSwitch) {
        try {
            input("스위칭 하시겠습니까?")
            println "Switch Started"
            sh "****"
            println "Switch End"
        } catch (Exception e) {
            println "Switch Skip"
        }
    } else {
        println "Switch Skip"
    }
})
```

`Manual Flow`를 위해 input을 사용합니다. flow는 input을 만나게되면, 사용자가 `proceed` 혹은 `abort` 를 선택하기 전까지는 대기 상태로 들어갑니다.

`abort`를 선택하면 예외가 발생하지만, 그렇다고 해당 `Stage`가 실패한 것은 아니라고 생각하여 예외처리를 작성하였습니다.

##### 기타. Node 사용하기

gradle 빌드 과정에 node 를 사용해야 할 task가 구성되어 있을 때 [Nvm Wrapper Plugin](https://wiki.jenkins.io/display/JENKINS/Nvm+Wrapper+Plugin)을 사용합니다.

```
if (useNode) {
    nvm('version' : "${NODE_VERSION}") {
        stage('Test') {
            if (useTest) {
                println "Test Started"
                ~~
                println "Test End"
            } else {
                println "Test Skip"
            }
        }
        stage("Build", {
            if(useBuild) {
                println "Build Started"
                ~~
                println "Build End"
            } else {
                println "Build Skip"
            }
        })
    }
} else {
    stage('Test') {
        if (useTest) {
            println "Test Started"
            ~~
            println "Test End"
        } else {
            println "Test Skip"
        }
    }
    stage("Build", {
        if(useBuild) {
            println "Build Started"
            ~~
            println "Build End"
        } else {
            println "Build Skip"
        }
    })
}
```

nvm이 `{}` scope 안에서만 사용 가능하여 위와 같이 else문에 똑같은 중복 코드가 발생하는 로직을 일단 사용하였습니다...

> groovy 학습 후 리펙토링을..ㅜㅜ

---

### Pipeline Template Job 만들기

위의 스크립트는 많은 파라미터를 필요로 합니다. 프로젝트를 추가해야할 때마다 매번 파라미터를 추가하는 노가다를 피하기 위하여, Template로 사용할 수 있는 `Pipeline Template Job` 을 만드려고 합니다.

#### 생성

![newjob](/images/2017/2017-11-04-JEKINS-PIPELINE/newjob.png)

new Job에서 Pipeline을 선택합니다.

#### Jenkins File Git SCM 연동

![scm](/images/2017/2017-11-04-JEKINS-PIPELINE/pipeline.png)

Pipeline script from SCM 를 사용하여, 필요한 정보를 채워줍니다.

![jenkinsfile](/images/2017/2017-11-04-JEKINS-PIPELINE/jenkinsfile.png)

`Script Path`에 작성한 Jenkins File 경로를!

#### Parameter 설정

**Flow**

![flow](/images/2017/2017-11-04-JEKINS-PIPELINE/flow.png)

캡쳐본과 위에서 설명한 스크립트가 조금 다릅니다. SCP, DRONE, QUEEN이 DEPLOY, SWITCH 입니다. 해당 파라미터를 `Boolean Parameter`로 설정합니다.

**Module**

이 단계에서는 [Extensible Choice Parameter plugin](https://wiki.jenkins.io/display/JENKINS/Extensible+Choice+Parameter+plugin)를 잠시 소개하겠습니다.

`Extensible Choice Parameter plugin`은 Global 로 등록해놓은 Choice Parameter 를 사용할 수 있도록 되어 있습니다. 즉 Choice Parameter 이지만, 중앙 관리가 가능하도록 제공하는 Plugin 입니다.

![extensible](/images/2017/2017-11-04-JEKINS-PIPELINE/extensiblechoice2.png)

이 Plugin을 통해 Jenkins Global Tool Installation을 연동한 듯한(?) 효과를 주기 위함입니다. `System Config` 에서 `Jenkins Global Tool Installation`에 정의한 이름으로 매칭하여 `Extensible Choice Parameter`를 위와 같이 구성했습니다.

![extensible](/images/2017/2017-11-04-JEKINS-PIPELINE/ex3.png)

그리고 `Extensible Choice`로 설정한 Choice Parameter 를 정의하였습니다.

> `Jenkins Global Tool Installation` 관련 `pipeline script` 에서 사용 가능한 Choice Parameter Plugin이 있다면 공유 부탁드립니다..ㅜㅜ(JDK Parameter Plugin을 시도했으나 기대와 다르게 동작하여 적용하지 못했습니다.)

**Git**

`String Parameter`로 아래 정보를 구성합니다.

![giturl](/images/2017/2017-11-04-JEKINS-PIPELINE/giturl.png)

![branch](/images/2017/2017-11-04-JEKINS-PIPELINE/branch.png)

`${BRANCH_TO_BUILD}` 환경 변수는 Hook으로 인해 발생한 Branch를 읽는 환경 변수입니다. Template에 유일하게 이 부분만 디폴트 Value를 지정해줍니다. (캡처 본에서 Default Value가 채워진 곳이 몇 군데 있지만, 실제 계속 재사용되는 것은, Branch 뿐입니다.)

### Template 사용하기

*Git 계정 연동 및 Hook 설정은 생략합니다*

#### Job 생성

![newjob](/images/2017/2017-11-04-JEKINS-PIPELINE/template.png)

new Job에서 제일 아래 Copy from을 통해 Template Job을 복사합니다.

#### 기본 값 채워넣기

대부분의 설정은 기본 설정을 유지하고, 파라미터의 `Default Value` 만 각 프로젝트에 맞게 설정하여줍니다.

![flow](/images/2017/2017-11-04-JEKINS-PIPELINE/flow.png)

![extensible](/images/2017/2017-11-04-JEKINS-PIPELINE/ex3.png)

![giturl](/images/2017/2017-11-04-JEKINS-PIPELINE/giturl.png)

![branch](/images/2017/2017-11-04-JEKINS-PIPELINE/branch.png)

#### 수동 실행

수동으로 Job을 빌드하려면 `Build with Parameter`를 사용합니다.

![bwp](/images/2017/2017-11-04-JEKINS-PIPELINE/bwp.png)

그러면 아래와 같이 기본 값들이 채워진 상태로 나오며 실행을 하면 끝!

![parameter](/images/2017/2017-11-04-JEKINS-PIPELINE/parameter.png)

#### 다시 실행

이미 진행된 Job에 대해서 다시 실행하려면 [Rebuild Plugin](https://wiki.jenkins.io/display/JENKINS/Rebuild+Plugin) 을 사용합니다.

이 플러그인은 이미 종료된 실행 잡을 다시 빌드할 수 있도록 도와주는 플러그인입니다. `Parameterized` 를 지원하여, 해당 실행 잡이 가지고 있던 파라미터를 그대로 사용할 수 있으며, 수정도 가능합니다.

사용 방법은 매우 간단합니다. 플러그인을 설치하면, 아래와 같이 파이프 라인 뷰의 실행 잡에 Rebuild 탭이 보이게 됩니다.

![rebuild1](/images/2017/2017-10-30-JENKINS-PLUGIN/rebuild1.png)

클릭하게 되면 해당 잡 실행기록이 가지고 있던 파라이터가 그대로 기본 설정이 되어 동일한 조건으로 잡의 실행이 가능합니다.

#### Script 기능 확인!

`Pipeline Job`이 구성된 후 Job이 실행되면 `Stage View`에 아래와 같은 화면을 보여줍니다! 기존 `Build Pipeline`에서는 `View` 를 따로 만들어야해서 불편했었습니다.

![flow](/images/2017/2017-11-04-JEKINS-PIPELINE/full.png)

**Flow**

수동 실행 시 단계를 Test 단계를 스킵하면, 설정한 스크립트대로 아래와 같이!

![testskip1](/images/2017/2017-11-04-JEKINS-PIPELINE/testskip1.png)

![flow](/images/2017/2017-11-04-JEKINS-PIPELINE/full.png)

![testskip2](/images/2017/2017-11-04-JEKINS-PIPELINE/testskip2.png)

**수동 파이프라인**

![input](/images/2017/2017-11-04-JEKINS-PIPELINE/input.png)

위와 같이 수동 단계에서 대기하며, 버튼이 제공됩니다!

### TIP

**IntelliJ에서 GDSL 사용하기**

[https://gist.github.com/arehmandev/736daba40a3e1ef1fbe939c6674d7da8](https://gist.github.com/arehmandev/736daba40a3e1ef1fbe939c6674d7da8)

*gdsl 다운*

![stx](/images/2017/2017-11-04-JEKINS-PIPELINE/stx.png)

![gdsl](/images/2017/2017-11-04-JEKINS-PIPELINE/gdsl.png)

**Snippet Generator**

Pipeline gdsl 로 코드를 작성하기 어렵다면, 제공해주는 Snippet Generator를 사용합니다.

![stx](/images/2017/2017-11-04-JEKINS-PIPELINE/stx.png)

![snippet](/images/2017/2017-11-04-JEKINS-PIPELINE/snippet.png)

`Build Pipeline` 사용성으로 groovy script 를 생성해주며, 다양한 snippet 이 있습니다! 저 같은 경우는 `Docs`와 `Stack Overflow` 을 찾아보면서 했을 때 생기던 오류들이, 정말 쉽게 해결..!

---

마무리
------

`Jenkins Pipeline`은 강력했습니다. `Build Pipeline`으로 만들었던 모든 기능을 그대로 옮길 수 있을 뿐더라, 제약으로 막혀있던 기능을 스크립트 기반이기에 해결할 수 있었습니다. 고로 `Jenkins Pipeline`은 뭐든지 가능하다!

그러나 단점도 존재하는 것 같습니다.

일단 groovy를 사용해야 한다는 점! snippet 생성기가 어느 정도 이 부분을 해소해주지만, **groovy에 대한 거부감, 혹은 언어를 배우고 싶지 않다면** `Build Pipeline`으로 구성하는 것이 쉽습니다. 스크립트 작성없이 클릭만(?)으로 만들 수 있기 때문입니다.

또 한가지 단점은 아직도 지원하지 않는 `Plugin`이 많다는 점 입니다. 인기있는 플러그인 중에서도 `Jenkins Pipeline` 을 지원하지 않거나, 안정적이지 않은 플러그인들을 꽤 보았습니다. 이 부분은 분명히 시간이 해결해주리라! 믿습니다.

저는 개인적으로 `Jenkins Pipeline`이 굉장히 마음에 듭니다. `maven`보다 `gradle`을 선호하듯, `개발자 친화적` + `CoC`가 저는 좋습니다.
