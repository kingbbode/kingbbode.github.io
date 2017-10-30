지난 젠킨스 자동 배포 환경에 이어 이전 글에서 소개하지 못했던 환경을 구성하는데 유용했던 플러그인을 소개드립니다.

자동 배포환경을 만드는데 유용했던 젠킨스 플러그인!
--------------------------------------------------

### 특정 다시 잡을 실행시키고 싶을 땐!? - Rebuild Plugin

첫 번째 소개할 플러그인은 [Rebuild Plugin](https://wiki.jenkins.io/display/JENKINS/Rebuild+Plugin) 입니다.

이 플러그인은 이미 종료된 실행 잡을 다시 빌드할 수 있도록 도와주는 플러그인입니다. `Parameterized` 를 지원하여, 해당 실행 잡이 가지고 있던 파라미터를 그대로 사용할 수 있으며, 수정도 가능합니다.

사용 방법은 매우 간단합니다. 플러그인을 설치하면, 아래와 같이 파이프 라인 뷰의 실행 잡에 Rebuild 탭이 보이게 됩니다.

![rebuild1](/images/2017/2017-10-30-JENKINS-PLUGIN/rebuild1.png)

클릭하게 되면 해당 잡 실행기록이 가지고 있던 파라이터가 그대로 기본 설정이 되어 동일한 조건으로 잡의 실행이 가능합니다.

![rebuild2](/images/2017/2017-10-30-JENKINS-PLUGIN/rebuild2.png)

---

### Git PUSH Trigger 방식을 그대로 두고, 수동 실행 때는 Branch 를 선택하고 싶을 땐!? - Git Parameter Plugin

두 번째 소개할 플러그인은 `Git` 으로부터 `Parameter` 를 발생시킬 수 있게 도와주는 Plugin 입니다.

이 플러그인은 Git Plugin 기반으로 연동된 Git 으로부터 태그, 브랜치 등의 정보를 읽을 수 있습니다.

저의 경우 아래와 같이 해당 잡에서 사용할 수 있는 브랜치를 release 브랜치 하위로 강제하도록 설정을 하였습니다.

![git](/images/2017/2017-10-30-JENKINS-PLUGIN/git_parameter1.png)

`Default Value` 는 Git Plugin 이 생성해주는 `BRANCH_TO_BUILD` 파라미터를 사용하여 Push 로 인한 자동 트리거 상황에서는 Push 트리거를 발생시킨 브랜치를 추적하도록 하였습니다. 그리고 잡의 Git 설정의 브랜치를 `${BRANCH_SELECTOR}` 로 설정합니다.

![git2](/images/2017/2017-10-30-JENKINS-PLUGIN/git_parameter2.png)

자동 트리거 방식을 유지한 채로, 수동 실행을 위한 파라미터 설정이 완료되었습니다. Run 실행 시 아래와 같이 Git 을 추적하여 조건의 Branch에 대한 Select Box 를 제공하여 줍니다.

![git3](/images/2017/2017-10-30-JENKINS-PLUGIN/git_parameter3.png)

이제 `Build`, `Rebuild` 등 수동 트리거 방식에서 아래와 같이 브랜치를 선택하여 Build 를 할 수 있도록 되었습니다.
