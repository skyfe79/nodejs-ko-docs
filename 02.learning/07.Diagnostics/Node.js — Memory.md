# Table of Contents

- [메모리](#메모리)
  - [프로세스가 메모리 부족으로 종료됨](#프로세스가-메모리-부족으로-종료됨)
    - [증상](#증상)
    - [부작용](#부작용)
  - [My process utilizes memory inefficiently](#my-process-utilizes-memory-inefficiently)
    - [증상](#증상)
    - [사이드 이펙트](#사이드-이펙트)
  - [디버깅](#디버깅)

# [메모리](https://nodejs.org/en/learn/modules/publishing-a-package#memory)

이 문서에서는 메모리 관련 문제를 디버깅하는 방법을 배울 수 있습니다.


## [프로세스가 메모리 부족으로 종료됨](https://nodejs.org/en/learn/modules/publishing-a-package#my-process-runs-out-of-memory)

Node.js(자바스크립트)는 가비지 컬렉션을 사용하는 언어입니다. 따라서 메모리 누수가 발생할 가능성이 있습니다. Node.js 애플리케이션은 일반적으로 다중 테넌트 환경에서 비즈니스적으로 중요한 역할을 하며, 장시간 실행되는 경우가 많습니다. 따라서 메모리 누수를 찾을 수 있는 접근 가능하고 효율적인 방법을 제공하는 것이 필수적입니다.


### 증상

사용자는 지속적으로 증가하는 메모리 사용량을 관찰합니다. *(빠르거나 느리게, 며칠 또는 몇 주에 걸쳐 발생할 수 있음)* 그런 다음 프로세스 관리자에 의해 프로세스가 충돌하고 재시작되는 것을 확인합니다. 프로세스가 이전보다 느리게 실행될 수 있으며, 재시작으로 인해 일부 요청이 실패할 수 있습니다. *(로드 밸런서가 502 오류로 응답함)*


### [부작용](https://nodejs.org/en/learn/modules/publishing-a-package#side-effects)

- 메모리 고갈로 인해 프로세스가 재시작되고 요청이 무시될 수 있습니다.
- 가비지 컬렉션(GC) 활동 증가로 CPU 사용량이 늘어나고 응답 시간이 느려질 수 있습니다.
    - GC가 이벤트 루프를 블로킹하여 속도가 느려지는 현상이 발생할 수 있습니다.
- 메모리 스와핑 증가로 프로세스가 느려질 수 있습니다 (GC 활동으로 인해).
- 힙 스냅샷을 얻기 위한 충분한 메모리가 없을 수도 있습니다.


## [My process utilizes memory inefficiently](https://nodejs.org/en/learn/modules/publishing-a-package#my-process-utilizes-memory-inefficiently)





### 증상

애플리케이션이 예상보다 많은 메모리를 사용하거나, 가비지 컬렉터의 활동이 증가하는 현상을 관찰할 수 있습니다.


### [사이드 이펙트](https://nodejs.org/en/learn/modules/publishing-a-package#side-effects-1)

- 페이지 폴트(page fault) 발생 횟수 증가
- 가비지 컬렉션(GC) 활동 및 CPU 사용량 증가


## 디버깅

대부분의 메모리 문제는 특정 타입의 객체가 차지하는 공간과 가비지 컬렉션을 방해하는 변수를 파악함으로써 해결할 수 있습니다. 또한 프로그램의 메모리 할당 패턴을 시간에 따라 이해하는 것도 도움이 됩니다.

-   [힙 프로파일러 사용하기](https://nodejs.org/en/learn/diagnostics/memory/using-heap-profiler/)
-   [힙 스냅샷 사용하기](https://nodejs.org/en/learn/diagnostics/memory/using-heap-snapshot/)
-   [GC 트레이스 사용하기](https://nodejs.org/en/learn/diagnostics/memory/using-gc-traces/)


