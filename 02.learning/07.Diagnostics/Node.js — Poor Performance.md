# [성능 저하](https://nodejs.org/en/learn/modules/publishing-a-package#poor-performance)

이 문서에서는 Node.js 프로세스의 성능을 프로파일링하는 방법을 배울 수 있습니다.


## [My application has a poor performance](https://nodejs.org/en/learn/modules/publishing-a-package#my-application-has-a-poor-performance)





### 증상

내 애플리케이션의 지연 시간이 높고, 데이터베이스나 하위 서비스와 같은 의존성에서 병목 현상이 발생하지 않음을 이미 확인했습니다. 따라서 애플리케이션이 코드를 실행하거나 정보를 처리하는 데 상당한 시간을 소비한다고 의심됩니다.

여러분은 애플리케이션 성능에 대체로 만족하지만, 어떤 부분을 개선하면 더 빠르고 효율적으로 실행할 수 있는지 알고 싶을 수 있습니다. 이는 사용자 경험을 개선하거나 컴퓨팅 비용을 절약하려는 경우에 유용할 수 있습니다.


### [디버깅](https://nodejs.org/en/learn/modules/publishing-a-package#debugging)

이번 사용 사례에서는 다른 코드보다 더 많은 CPU 사이클을 사용하는 코드 조각에 관심이 있습니다. 이를 로컬에서 수행할 때는 보통 코드를 최적화하려고 합니다.

이 문서는 Node.js 애플리케이션을 프로파일링하는 두 가지 간단한 방법을 제공합니다:

-   [V8 샘플링 프로파일러 사용](https://nodejs.org/en/learn/getting-started/profiling/)
-   [Linux Perf 사용](https://nodejs.org/en/learn/diagnostics/poor-performance/using-linux-perf)


