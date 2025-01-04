# Table of Contents

- [[Flame Graphs](https://nodejs.org/en/learn/modules/publishing-a-package#flame-graphs)](#flame-graphshttpsnodejsorgenlearnmodulespublishing-a-packageflame-graphs)
  - [플레임 그래프는 어떤 용도로 사용되나요?](#플레임-그래프는-어떤-용도로-사용되나요)
  - [[플레임 그래프 생성 방법](https://nodejs.org/en/learn/modules/publishing-a-package#how-to-create-a-flame-graph)](#플레임-그래프-생성-방법httpsnodejsorgenlearnmodulespublishing-a-packagehow-to-create-a-flame-graph)
    - [[미리 패키징된 도구 사용하기](https://nodejs.org/en/learn/modules/publishing-a-package#use-a-pre-packaged-tool)](#미리-패키징된-도구-사용하기httpsnodejsorgenlearnmodulespublishing-a-packageuse-a-pre-packaged-tool)
    - [시스템 perf 도구로 플레임 그래프 만들기](#시스템-perf-도구로-플레임-그래프-만들기)
    - [실행 중인 프로세스를 샘플링하기 위해 `perf` 사용하기](#실행-중인-프로세스를-샘플링하기-위해-perf-사용하기)
    - [Node.js 내부 함수 필터링하기](#nodejs-내부-함수-필터링하기)
    - [[Node.js의 프로파일링 옵션](https://nodejs.org/en/learn/modules/publishing-a-package#nodejss-profiling-options)](#nodejs의-프로파일링-옵션httpsnodejsorgenlearnmodulespublishing-a-packagenodejss-profiling-options)
    - [[왜 이것들이 필요할까요?](https://nodejs.org/en/learn/modules/publishing-a-package#why-do-i-need-them-at-all)](#왜-이것들이-필요할까요httpsnodejsorgenlearnmodulespublishing-a-packagewhy-do-i-need-them-at-all)
  - [[`perf` output issues](https://nodejs.org/en/learn/modules/publishing-a-package#perf-output-issues)](#perf-output-issueshttpsnodejsorgenlearnmodulespublishing-a-packageperf-output-issues)
    - [[Node.js 8.x V8 파이프라인 변경 사항](https://nodejs.org/en/learn/modules/publishing-a-package#nodejs-8x-v8-pipeline-changes)](#nodejs-8x-v8-파이프라인-변경-사항httpsnodejsorgenlearnmodulespublishing-a-packagenodejs-8x-v8-pipeline-changes)
    - [[Node.js 10+](https://nodejs.org/en/learn/modules/publishing-a-package#nodejs-10)](#nodejs-10httpsnodejsorgenlearnmodulespublishing-a-packagenodejs-10)
    - [[플레임 그래프에서 깨진 라벨](https://nodejs.org/en/learn/modules/publishing-a-package#broken-labels-in-the-flame-graph)](#플레임-그래프에서-깨진-라벨httpsnodejsorgenlearnmodulespublishing-a-packagebroken-labels-in-the-flame-graph)
  - [[예제](https://nodejs.org/en/learn/modules/publishing-a-package#examples)](#예제httpsnodejsorgenlearnmodulespublishing-a-packageexamples)

# [Flame Graphs](https://nodejs.org/en/learn/modules/publishing-a-package#flame-graphs)





## 플레임 그래프는 어떤 용도로 사용되나요?

플레임 그래프는 함수에서 소비된 CPU 시간을 시각화하는 방법입니다. 이 그래프를 통해 동기 작업에 너무 많은 시간을 소비하는 부분을 정확히 파악할 수 있습니다.


## [플레임 그래프 생성 방법](https://nodejs.org/en/learn/modules/publishing-a-package#how-to-create-a-flame-graph)

Node.js에서 플레임 그래프를 만드는 것이 어렵다고 들어본 적이 있을지 모르지만, 이제는 그렇지 않습니다. 더 이상 Solaris 가상 머신이 필요하지 않습니다!

플레임 그래프는 `perf`의 출력 결과로 생성됩니다. `perf`는 Node.js 전용 도구는 아니지만, CPU 시간을 시각화하는 가장 강력한 방법입니다. 다만 Node.js 8 이상에서 JavaScript 코드가 최적화되는 방식 때문에 문제가 발생할 수 있습니다. 자세한 내용은 아래 [perf 출력 문제](https://nodejs.org/en/learn/modules/publishing-a-package#perf-output-issues) 섹션을 참고하세요.


### [미리 패키징된 도구 사용하기](https://nodejs.org/en/learn/modules/publishing-a-package#use-a-pre-packaged-tool)

로컬에서 플레임 그래프를 한 번에 생성하고 싶다면 [0x](https://www.npmjs.com/package/0x)를 사용해 보세요.

프로덕션 환경 배포 문제를 진단하려면 다음 문서를 참고하세요: [0x 프로덕션 서버](https://github.com/davidmarkclements/0x/blob/master/docs/production-servers.md).


### 시스템 perf 도구로 플레임 그래프 만들기

이 가이드는 플레임 그래프를 만드는 단계를 보여주고, 각 단계를 직접 제어할 수 있도록 도와줍니다.

각 단계를 더 자세히 이해하고 싶다면, 뒤에 나오는 섹션을 참고하세요.

이제 시작해봅시다.

1. `perf`를 설치합니다. (보통 linux-tools-common 패키지를 통해 설치할 수 있습니다. 이미 설치되어 있다면 생략합니다.)
   
2. `perf`를 실행해봅니다. 커널 모듈이 없다는 오류가 발생하면 해당 모듈도 설치합니다.
   
3. perf를 활성화한 상태로 node를 실행합니다. (Node.js 버전별 팁은 [perf 출력 문제](https://nodejs.org/en/learn/modules/publishing-a-package#perf-output-issues)를 참고하세요.)
   
    ```bash
    perf record -e cycles:u -g -- node --perf-basic-prof app.js
    ```
   
4. 패키지가 없어서 perf를 실행할 수 없다는 경고가 아니라면, 다른 경고는 무시해도 됩니다. 커널 모듈 샘플에 접근할 수 없다는 경고는 신경 쓰지 않아도 됩니다.
   
5. `perf script > perfs.out`을 실행하여 시각화할 데이터 파일을 생성합니다. 더 읽기 쉬운 그래프를 위해 [일부 정리 작업](https://nodejs.org/en/learn/modules/publishing-a-package#filtering-out-nodejs-internal-functions)을 적용하는 것이 유용합니다.
   
6. 아직 설치하지 않았다면 stackvis를 설치합니다. `npm i -g stackvis`
   
7. `stackvis perf < perfs.out > flamegraph.htm`을 실행합니다.

이제 플레임 그래프 파일을 좋아하는 브라우저에서 열어 확인하세요. 색상이 구분되어 있으므로 가장 진한 오렌지색 막대를 먼저 살펴보세요. 이 부분이 CPU를 많이 사용하는 함수일 가능성이 높습니다.

참고로, 플레임 그래프의 요소를 클릭하면 해당 부분을 확대한 화면이 그래프 위에 표시됩니다.


### 실행 중인 프로세스를 샘플링하기 위해 `perf` 사용하기

이 방법은 이미 실행 중인 프로세스에서 플레임 그래프 데이터를 기록할 때 유용합니다. 특히 중단하기 어려운 프로덕션 환경에서 재현하기 힘든 문제를 분석할 때 효과적입니다.

```bash
perf record -F99 -p `pgrep -n node` -g -- sleep 3
```

`sleep 3`은 왜 필요할까요? 이 명령어는 `perf`가 계속 실행되도록 유지하기 위해 사용됩니다. `-p` 옵션으로 다른 프로세스 ID를 지정하더라도, `perf`는 명령어가 실행되는 프로세스에서 시작되고 그 프로세스가 종료될 때 함께 종료됩니다. `perf`는 전달된 명령어의 수명 동안 실행되며, 실제로 프로파일링하는 명령어인지 여부는 중요하지 않습니다. `sleep 3`은 `perf`가 3초 동안 실행되도록 보장합니다.

`-F`(프로파일링 빈도)가 99로 설정된 이유는 무엇일까요? 이는 합리적인 기본값입니다. 필요에 따라 조정할 수 있습니다. `-F99`는 `perf`에게 초당 99개의 샘플을 취하라고 지시합니다. 더 높은 정밀도를 원한다면 이 값을 증가시키면 됩니다. 낮은 값은 출력을 줄이고 결과의 정밀도를 낮춥니다. 필요한 정밀도는 CPU 집약적인 함수가 실제로 실행되는 시간에 따라 달라집니다. 눈에 띄는 속도 저하의 원인을 찾고 있다면, 초당 99프레임으로도 충분할 것입니다.

3초 동안 `perf` 기록을 완료한 후, 위에서 설명한 마지막 두 단계를 따라 플레임 그래프를 생성하면 됩니다.


### Node.js 내부 함수 필터링하기

일반적으로 여러분은 자신이 작성한 코드의 성능만 확인하고 싶을 것입니다. 이때 Node.js와 V8의 내부 함수를 필터링하면 그래프를 훨씬 더 쉽게 읽을 수 있습니다. 다음 명령어를 사용해 `perf` 파일을 정리할 수 있습니다.

```bash
sed -i -r \
  -e "/( __libc_start| LazyCompile | v8::internal::| Builtin:| Stub:| LoadIC:|\[unknown\]| LoadPolymorphicIC:)/d" \
  -e 's/ LazyCompile:[*~]?/ /' \
  perfs.out
```

만약 플레임 그래프를 확인했을 때 이상하거나, 대부분의 시간을 차지하는 주요 함수가 누락된 것처럼 보인다면, 필터 없이 플레임 그래프를 다시 생성해 보세요. 아마도 Node.js 자체의 문제가 발생한 드문 경우일 수 있습니다.


### [Node.js의 프로파일링 옵션](https://nodejs.org/en/learn/modules/publishing-a-package#nodejss-profiling-options)

`--perf-basic-prof-only-functions`와 `--perf-basic-prof`는 여러분의 자바스크립트 코드를 디버깅할 때 유용한 두 가지 옵션입니다. 다른 옵션들은 Node.js 자체를 프로파일링하는 데 사용되며, 이 가이드의 범위를 벗어납니다.

`--perf-basic-prof-only-functions`는 출력이 적기 때문에 오버헤드가 가장 적은 옵션입니다.


### [왜 이것들이 필요할까요?](https://nodejs.org/en/learn/modules/publishing-a-package#why-do-i-need-them-at-all)

이 옵션 없이도 플레임 그래프를 얻을 수 있지만, 대부분의 막대가 `v8::Function::Call`로 표시됩니다.


## [`perf` output issues](https://nodejs.org/en/learn/modules/publishing-a-package#perf-output-issues)





### [Node.js 8.x V8 파이프라인 변경 사항](https://nodejs.org/en/learn/modules/publishing-a-package#nodejs-8x-v8-pipeline-changes)

Node.js 8.x 이상 버전에서는 V8 엔진의 자바스크립트 컴파일 파이프라인에 새로운 최적화가 적용되었습니다. 이로 인해 성능 향상을 위해 함수 이름이나 참조가 때때로 접근 불가능해질 수 있습니다. (이를 Turbofan이라고 합니다.)

이러한 변경으로 인해 플레임 그래프에서 함수 이름이 정확하게 표시되지 않을 수 있습니다.

기대했던 함수 이름 대신 `ByteCodeHandler:`와 같은 내용이 나타날 수 있습니다.

[0x](https://www.npmjs.com/package/0x)는 이러한 문제를 완화하기 위한 기능을 내장하고 있습니다.

자세한 내용은 다음 링크를 참고하세요:

-   [https://github.com/nodejs/benchmarking/issues/168](https://github.com/nodejs/benchmarking/issues/168)
-   [https://github.com/nodejs/diagnostics/issues/148#issuecomment-369348961](https://github.com/nodejs/diagnostics/issues/148#issuecomment-369348961)


### [Node.js 10+](https://nodejs.org/en/learn/modules/publishing-a-package#nodejs-10)

Node.js 10.x는 `--interpreted-frames-native-stack` 플래그를 사용하여 Turbofan의 문제를 해결합니다.

JavaScript를 컴파일할 때 V8이 어떤 파이프라인을 사용했는지와 상관없이, 플레임 그래프에서 함수 이름을 얻으려면 다음 명령어를 실행하세요:

```bash
node --interpreted-frames-native-stack --perf-basic-prof-only-functions
```


### [플레임 그래프에서 깨진 라벨](https://nodejs.org/en/learn/modules/publishing-a-package#broken-labels-in-the-flame-graph)

만약 다음과 같은 라벨을 보게 된다면

```
node`_ZN2v88internal11interpreter17BytecodeGenerator15VisitStatementsEPNS0_8ZoneListIPNS0_9StatementEEE
```

이는 여러분이 사용 중인 Linux perf가 demangle 지원 없이 컴파일되었음을 의미합니다. 자세한 내용은 [https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1396654](https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1396654)를 참고하세요.


## [예제](https://nodejs.org/en/learn/modules/publishing-a-package#examples)

직접 플레임 그래프를 만들어보고 싶다면 [플레임 그래프 연습](https://github.com/naugtur/node-example-flamegraph)을 통해 실습해보세요!


