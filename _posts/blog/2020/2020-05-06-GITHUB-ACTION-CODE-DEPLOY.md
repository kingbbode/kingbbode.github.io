---
layout:     post
title:      Github Action 배포 (Spring Boot Gradle Multi Module)
author:     kingbbode
tags:       study
subtitle:   Jenkins 에서 Github Action 배포로 전환
category:  posts
outlink: 0
---

개인 프로젝트인 `co-duck.com` 은 오랫동안 `Jenkins` + `CodeDeploy` 로 구축한 배포를 사용하고 있었습니다.

사용하고 있던 배포 환경은

- [기억보단 기록을: 1. AWS Code Deploy로 배포 Jenkins에서 배치 Jenkins로 Spring Batch 배포하기 - 기본 환경 구성](https://jojoldu.tistory.com/313)
- [기억보단 기록을: 2. AWS Code Deploy로 배포 Jenkins에서 배치 Jenkins로 Spring Batch 배포하기 - Code Deploy 연동](https://jojoldu.tistory.com/314)
- [기억보단 기록을: 3. AWS Code Deploy로 배포 Jenkins에서 배치 Jenkins로 Spring Batch 배포하기 - 젠킨스 연동](https://jojoldu.tistory.com/315)

에 잘 정리되어 있습니다.

## Jenkins 에서 Github Action 으로

코딩덕후 프로젝트는 개인 프로젝트로 운영하는 만큼 언제나 비용 최적화에 초점을 맞추고 있습니다.

![gazua](/images/2020/GITHUB-ACTION-CODE-DEPLOY/gazua.png)

`Jenkins` 가 구동 중인 인스턴스는 `t2.micro` 타입으로(메모리 1Gib) 메모리 부족으로 인해 일반적인 설정으로는 `npm` 빌드조차 할 수 없습니다. [swap 파일을 사용하도록 하여](https://tecnstuff.net/how-to-add-swap-space-on-debian-10-linux/) 인스턴스의 모든 것을 끌어내 빌드를 하고 있지만, 이 젠킨스에는 빌드 이외에도 데이터 갱신을 위한 수십개의 스케줄 배치 프로그램이 함께 실행되기 때문에 간혹 스케줄이 꼬이면 인스턴스 사망 사태가 벌어지곤 했습니다.

`Github` 에서 `Github Action` 이라는 `CI/CD` 도구를 제공한다고 들어, 베타 테스터까지 신청하였지만 이러 저러한 핑계로 계속 작업을 미루고 있던 중 최근 인스턴스 사망사태가 빈번히 발생하며, 기존의 `Jenkins` 로 구축한 환경을 `Github Action` 으로 이전하게 되었습니다.

## Github Action

Github Action 에 대한 설명은 잘 설명되어 있는 블로그들이 많이 있으므로([Nesoy Blog: Github Action 시작하기](https://nesoy.github.io/articles/2019-12/Github-Actions)) 생략하겠습니다.

Github Action 에서 가장 놀라웠던 것은 가격과 제공하는 스팩이였습니다.

### 스팩

[문서에 따르면](https://help.github.com/en/actions/reference/virtual-environments-for-github-hosted-runners#supported-runners-and-hardware-resources) 액션 수행당 무려 **2-core CPU, 7 GB of RAM memory, 14 GB of SSD disk space** 라는 CI/CD 에 충분하고도 남을 리소스를 제공하고 있습니다.

### 가격

일단 공개 저장소에 한해서는 모두 무료입니다. 마음 것 사용할 수 있습니다. 코덕 프로젝트는 `GitHub Team` 상품을 사용하고 있고, `Private` 저장소로 운영되고 있습니다. 그래서 약간은 마음이 조마조마했지만, 다행히 `배포` 용도로 사용하기에는 충분해 보였습니다.

![gazua](/images/2020/GITHUB-ACTION-CODE-DEPLOY/price.png){: style="width:75%; display:block; margin:40px auto 0;"}

[문서에 따르면](https://help.github.com/en/github/setting-up-and-managing-billing-and-payments-on-github/about-billing-for-github-actions) `GitHub Team` 상품은 2GB 의 저장소를 사용할 수 있고, 한달에 10,000분(2020년 5월 중순부터 3,000분으로 축소)을 사용할 수 있다고 합니다. 무료로 제공되는 플랜 이상 초과되는 저장소 사용이나 시간에 대한 요금이 청구됩니다.

일단 빌드를 위해 사용되는 저장소 공간이 절대 500mb 를 넘지 않을 것이란 확신이 있고, 1회 빌드 시간을 5분 정도라 가정했을 때 한달에 600번은 배포할 수 있으니 충분할 것 같습니다. 조금 체념하고 CI를 위한 브랜치 전략을 조절한다면 무료 플랜에서 `CI/CD` 를 구축할 수도 있겠단 생각도 듭니다.


## WorkFlow 환경 설정

[기억보단 기록을: 1. AWS Code Deploy로 배포 Jenkins에서 배치 Jenkins로 Spring Batch 배포하기 - 기본 환경 구성](https://jojoldu.tistory.com/313) 과정을 통해 `CodeDeploy` 의 환경은 이미 모두 세팅이 되어있습니다만, Github Action 으로 몇 가지를 설정해주어야 합니다.

### IAM

`Jenkins` 에서 사용한 `aws-cli` 는 `instance` 의 역할을 부여한 방식이지만, Github Action 은 외부에서 `AWS` 자원을 접근해야하기 때문에 `액세스 키`가 필요합니다.

`IAM` 에서 사용자를 만들고, `CodeDeploy` 를 사용하기 위한 역할을 부여합니다.

![add-user1](/images/2020/GITHUB-ACTION-CODE-DEPLOY/add-user.png){: style="width:75%; display:block; margin:40px auto 0;"}

![add-user2](/images/2020/GITHUB-ACTION-CODE-DEPLOY/add-user2.png){: style="width:75%; display:block; margin:40px auto 0;"}

![add-user3](/images/2020/GITHUB-ACTION-CODE-DEPLOY/add-user3.png){: style="width:75%; display:block; margin:40px auto 0;"}

![add-user4](/images/2020/GITHUB-ACTION-CODE-DEPLOY/add-user4.png){: style="width:75%; display:block; margin:40px auto 0;"}

![add-user5](/images/2020/GITHUB-ACTION-CODE-DEPLOY/add-user5.png){: style="width:75%; display:block; margin:40px auto 0;"}

만들어진 엑세스 키는 프로젝트의 `Secrets` 에 안전하게 보관하여 `Github Action` 스크립트에서 사용하도록 합니다.

![secrets](/images/2020/GITHUB-ACTION-CODE-DEPLOY/secrets.png){: style="width:75%; display:block; margin:40px auto 0;"}

### WorkFlow 작성

이번에응 [기억보단 기록을: 3. AWS Code Deploy로 배포 Jenkins에서 배치 Jenkins로 Spring Batch 배포하기 - 젠킨스 연동](https://jojoldu.tistory.com/315) 에 작성된 배포스크립트를 `WorkFlow` 로 옮겨야 합니다. `WorkFlow` 가 `Bash` 환경을 지원하여 매우 수월하게 옮길 수 있었습니다.

```yaml
name: DEPLOY CODUCK WEB

on:
  repository_dispatch:
    types: [deploy-web]
env:
  PROJECT_NAME: app-web
  BUCKET: build-files
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
      with:
        ref: master
    - name: Set up JDK 1.8
      uses: actions/setup-java@v1
      with:
        java-version: 1.8
    - name: Grant execute permission for gradlew
      run: chmod +x gradlew
    - name: Build with Gradle
      run: ./gradlew :${PROJECT_NAME}:clean :${PROJECT_NAME}:build
    - name: MAKE DEPLOYMENT DIR
      run: mkdir -p code-deploy-${PROJECT_NAME}
    - name: MOVE DEPLOY CODE DEPLOY SCRIPT
      run: cp ${PROJECT_NAME}/code-deploy/* code-deploy-${PROJECT_NAME}/
    - name: MOVE BUILD JAR
      run: cp ${PROJECT_NAME}/build/libs/*.jar code-deploy-${PROJECT_NAME}/
    - name: ZIP CODE DEPLOY
      run: zip -r -j code-deploy-${PROJECT_NAME} code-deploy-${PROJECT_NAME}/*
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ap-northeast-2
    - name: Upload to S3
      run: aws s3 cp code-deploy-${PROJECT_NAME}.zip s3://${BUCKET}/${PROJECT_NAME}/${PROJECT_NAME}-${GITHUB_SHA}.zip --region ap-northeast-2
    - name: Code Deploy
      run: aws deploy create-deployment --application-name ${PROJECT_NAME}-deploy --deployment-group-name ${PROJECT_NAME}-deploy-group --region ap-northeast-2 --s3-location bucket=${BUCKET},bundleType=zip,key=${PROJECT_NAME}/${PROJECT_NAME}-${GITHUB_SHA}.zip
    - name: Clear Deploy DIR
      run: rm -rf code-deploy-${PROJECT_NAME}
```

거의 그대로 옮겨졌기 때문에 특별히 설명할 부분이 거의 없습니다.

- `checkout` 이나 `java-setup` 과 같은 라이브러리 사용은 예제와 문서를 통해 친절히 사용법을 제공합니다.
- `Jenkins` 의 `Github Plugin` 이 `GIT_COMMIT` 으로 해쉬 환경변수를 지원한 것과 마찬가지로 `Github Action` 에서는 `GITHUB_SHA` 로 해쉬 값을 제공합니다.
- 저장해두었던 `Secrets` 는 `secrets.` 으로 접근이 가능합니다.
- 민감하지 않은 정보는 `env` 에 설정할 수 있습니다.
- `env` 에서 `ZIP_NAME: ${PROJECT_NAME}/${PROJECT_NAME}-${GITHUB_SHA}.zip` 과 같이 `env 환경변수`를 사용한 환경변수를 작성하려했지만, 적용되지 않았습니다.

### Trigger

`Branch Push`, `Webhook` 등 다양한 트리거 방식이 지원되고 있습니다. ([Events that trigger workflows](https://help.github.com/en/actions/reference/events-that-trigger-workflows))

배포에 대해서는 사용자 액션에 의한 방식을 선호하는 편이기에, 젠킨스의 잡 실행과 같은 수동 방식을 찾았지만 지원하지 않았습니다. 대신 `repository_dispatch` 라는 자유도 높은 트리거 방식을 지원했습니다.

![dispatch-api](/images/2020/GITHUB-ACTION-CODE-DEPLOY/dispatch-api.png){: style="width:75%; display:block; margin:40px auto 0;"}

저장소에 API 를 통해 이벤트를 발송시킬 수 있었으며, 이 때 `event_type` 은 자유롭게 작성할 수 있습니다.([Github API](https://developer.github.com/v3/repos/))

여러 어플리케이션 모듈 중 웹어플리케이션 모듈을 배포하기 위한 스크립트이므로, `event_type`을 특정하여 `deploy-web` 으로 지정하여 해당 `WorkFlow` 만 동작시키도록 할 수 있습니다. 아래와 같은 스크립트를 프로젝트에 첨부하여, 로컬 콘솔에서 트리깅을 하는 방식을 선택했습니다.

```
while : ; do
  printf "github id > "
  read githubId

  if [[ -z "$githubId" ]]; then
    printf '%s\n' "github id is required!"
    continue
  else
    break
  fi
done

curl \
-u $githubId \--request POST \
--data '{"event_type": "deploy-web"}' \
https://api.github.com/repos/10duck/coduck-project/dispatches

printf "github action!"
```

(인증 방식은 개인 키를 사용할 수도 있지만, 여러명이 함께 사용하는 프로젝트이기에 간단히 id, password 인증 방식을 사용하도록 했습니다.)


## 끝

![deploy-sh](/images/2020/GITHUB-ACTION-CODE-DEPLOY/deploy-sh.png){: style="width:75%; display:block; margin:40px auto 0;"}

![success](/images/2020/GITHUB-ACTION-CODE-DEPLOY/success.png){: style="width:75%; display:block; margin:40px auto 0;"}

무사히 배포가 되었습니다.
