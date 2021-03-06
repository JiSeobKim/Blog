---
layout: post                       
title: Tool - Fastlane Lane 사용법
categories: [Tool]
---

> 이전 포스팅
> - [Fastlane 개념](https://jiseobkim.github.io/tool/2019/03/30/Tool-Fastlane을-알아보자.html)
> - [Fastlane 설치 방법과 1개월 사용 후기](https://jiseobkim.github.io/swift/2019/04/21/Tool-Fastlane-설치-방법.html)


포스팅 할건 많은데, 느낌이 왔을때 새로 쓰는게 나을지, 아니면 이것처럼 시작한것은 끝내고 쓰는게 좋을지 모르겠다.
뭔가 읽을때는 순차적인게 좋을듯 한데, 느낌이 왔을때 바로 못쓰니 김새는 느낌이 있다.
잡담 끝.

---

<br>
# Lane에 대하여

FastLane의 꽃은 Lane이 아닐까 하는 생각이 든다.
FastLane의 장점은 Xcode에서 여러번 클릭을 여러번 해야하는 부분들을 명령어 하나로 해결해주는 점인듯 했다.

그리고, 그 단순화 과정 마저도 한번에 묶어주는 `Lane`, 그래서 꽃이라 칭해보았다.


<br>
명령어 몇가지를 보자
- **Derived 파일 제거**
```
fastlane action clear_derived_data
```

- **IPA 추출하기**
```
fastlane gym
```

`Derived 제거` 관련의 경우, 가끔 빌드시 문제가 생길시 여기 파일을 지워주면 해결되는 경우가 많다. 빌드시 계속 생성되기 때문에 언제든 지워도 되는 파일!
`IPA 추출`은 TestFlight를 안쓴다면 사내 테스트용으로 필요로 뽑아두는 분들이 많을거라 생각된다. 명령어 하나로 손쉽게 추출!
(IPA 추출의 경우 dSYM 파일도 같이 나온다.)

<br>

두 명령어다 아주 좋은 명령어이다. 그렇지만 사람은 언제나 귀찮은 작업 하나라도 줄이려는게 분명하다.
위와 같이 `IPA 추출`을 했을 경우 `Clean`과 `Derived 제거`를 같이 할 경우가 많다.
한줄 한줄 입력하는것도 귀찮았나보다. 그래서 여러가지 명령어를 한번에 실행하고 싶었던걸까?

그래서 `Lane`이라는 것이 나온듯 하다.

<br><br>
# Lane 생성하기
이전 포스팅 내용해 `fastlane init`을 했다면, 해당 프로젝트 폴더내에 `fastlane` 이라는 폴더가 있으며, 그안에 `Fastfile`이란 파일이 존재한다. 
내용은 아래와 같다.
```
# This file contains the fastlane.tools configuration
# You can find the documentation at https://docs.fastlane.tools
#
# For a list of all available actions, check out
#
#     https://docs.fastlane.tools/actions
#
# For a list of all available plugins, check out
#
#     https://docs.fastlane.tools/plugins/available-plugins
#

# Uncomment the line if you want fastlane to automatically update itself
# update_fastlane

default_platform(:ios)

platform :ios do
  desc "Description of what the lane does"
  lane :custom_lane do
    # add actions here: https://docs.fastlane.tools/actions
  end
end
```

(#) 부분은 한번쯤 읽어보면 되고,
<br>
중요한 부분은 여기다.

```
desc "Description of what the lane does"
lane :custom_lane do
    # add actions here: https://docs.fastlane.tools/actions
end
```

이것이 `lane`의 기본 틀이라 보면 되고, 각 설명은 아래와 같다.
- **desc**: lane에 대한 설명.
- **custom_lane**: lane를 실행시킬 명령어.
- **do ~ end**: 이 lane으로 인해 실행될 명령어들을 정의.


<br> 
<br>
위에 예를 들었던 명령어 두개를 이용하여 `lane`을 생성해보자

```
desc "Test Lane"
lane :test_lane do
    clear_derived_data
    gym
end
```

이렇게하면 생성완료!
<br>
그 후에 아래 명령어를 실행하면 된다.
```
fastlane test_lane
```
그럼 위 `test_lane`내의 명령어들이 순차적으로 진행되는 것을 볼 수 있다.

그런데, 여기서 끝나면 아쉬울것이다. gym(아카이브) 명령어를 실행할때 옵션을 주는 경우가 필요하다.

예를 들면 인증서를 선택(Distribution/Adhoc/Development)해야하는 경우도 필요하며, 아카이브 전에 빌드 클린은 필요하기에
이러한 명령어들에 대한 옵션들이 필요하다.

<br><br>

# Lane 명령어에 옵션주기

옵션 주는것은 간단하다. 해당 명령어 뒤에 괄호()를 열어 주고 적으면 된다.

```
desc "Test Lane"
lane :test_lane do
    clear_derived_data
    gym(
        scheme: "Foo",
        export_method: "ad-hoc",
        clean: true,
        output_directory: "path/to/dir"
    )
end
```

`gym`뒤에 괄호를 열어줬고, 그 안에 옵션들을 나열했다.
- **scheme**: 해당 프로젝트의 타겟을 설정해준다. (한 workspace안에 여러 앱이 있을 경우 필요하다.)
- **export_method**: 인증서 옵션 주는 부분이다. (종류: app-store, ad-hoc, package, enterprise, development, developer-id)
- **clean**: 빌드 클린 여부이다, true or false 주면 된다.
- **output_directory**: 결과물들을 저정할 위치를 셋팅해주는 값이다. (안적으면 해당 프로젝트 폴더)

<br><br>


# PS.
마지막 부분처럼 옵션이 생기면, 생각보다? 줄이 길어진다. 외우기도 힘들며, 하나가 아니라 여러게 생길수 있다. 
하지만 `lane`덕에 미리 셋팅만 해둔다면, 단 1줄의 명령어로 끝낼 수 있다.

슬랙으로 알림도 줄 수 있고, 알아서 스크린샷도 찍어주고, 추출도 해주고, 업로드도 해주고, 인증서 관리부터 생성까지 해주니
알아두면 좋은 도구임은 확실한듯하다. 나중에 고급편도 포스팅 해야겠다.

이걸로 *fastlane*은 일단 끝!
