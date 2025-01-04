# Table of Contents

- [실시간 디버깅](#실시간-디버깅)
  - [My application doesn’t behave as expected](#my-application-doesnt-behave-as-expected)
    - [증상](#증상)
    - [디버깅](#디버깅)

# [실시간 디버깅](https://nodejs.org/en/learn/modules/publishing-a-package#live-debugging)

이 문서에서는 Node.js 프로세스를 실시간으로 디버깅하는 방법을 배울 수 있습니다.


## [My application doesn’t behave as expected](https://nodejs.org/en/learn/modules/publishing-a-package#my-application-doesnt-behave-as-expected)





### 증상

사용자는 특정 입력에 대해 애플리케이션이 예상한 결과를 제공하지 않는 것을 확인할 수 있습니다. 예를 들어, HTTP 서버가 특정 필드가 비어 있는 JSON 응답을 반환하는 경우가 있습니다. 이 과정에서 다양한 문제가 발생할 수 있지만, 이번 사례에서는 주로 애플리케이션 로직과 그 정확성에 초점을 맞추겠습니다.


### 디버깅

이번 사용 사례에서는 특정 트리거(예: 들어오는 HTTP 요청)에 대해 애플리케이션이 실행하는 코드 경로를 이해하고자 합니다. 또한 코드를 단계별로 실행하고, 실행을 제어하며, 메모리에 저장된 변수 값을 확인하고 싶을 수 있습니다.

-   [Inspector 사용하기](https://nodejs.org/en/learn/diagnostics/live-debugging/using-inspector)


