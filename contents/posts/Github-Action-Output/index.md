---
IDX: "NUM-121"
tags:
  - Github Actions
description: "Github Action Output을 사용하여 job 사이 변수 공유하기"
update: "2024-02-04T11:11:00.000Z"
date: "2024-02-04"
상태: "Ready"
title: "Github Action Output"
---
## 목적

[깃허브 액션](https://sharknia.github.io/Github-Actions)에서 Job들은 서로 다른 독립된 환경을 가집니다. 각 작업은 독립적으로 실행되며, 각기 다른 러너 환경에서 다른 버전의 도구를 사용할 수 있습니다. 이것은 아주 강점이지만, 때로 다른 Job끼리 변수를 공유해야 하는 상황이 있을 수 있습니다. 

이때 사용하는 것이 Output 입니다. 

## 선언하기

```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
    outputs: # 다른 job에서 사용하기 위해 선언
      output1: ${{ steps.step1.outputs.test }}
      output2: ${{ steps.step2.outputs.test }}
    steps:
      - id: step1 # 해당 output을 특정하기 위한 id 선언
        run: echo "test=hello" >> "$GITHUB_OUTPUT"
      - id: step2
        run: echo "test=world" >> "$GITHUB_OUTPUT"
```

기본적인 선언 방법은 `echo "변수=값" >> "$GITHUB_OUTPUT”` 의 꼴입니다. 예전에는 set_output을 사용했던 것 같은데, 이 방법도 지금 작동은 제대로 하지만 해당 코드로 실행을 해보면 보안상의 문제로 곧 depacreted 될 거라는 warning이 발생합니다. 

반드시 id를 지정해줘야 합니다. 해당 id를 사용해 나중에 원하는 변수를 참조할 수 있습니다. 

output은 동일한 job이 아니라 job2에서 사용을 하고 싶을 경우에 선언해야 합니다. 

## 사용하기 - 동일한 job

```yaml
- name: use output same job
        run: |
          echo ${{ steps.step1.outputs.test}}
          echo ${{ steps.step2.outputs.test}}
```

같은 job 안에서 사용할때에는 `${{ steps.”id”.outputs.”변수명”}}` 의 꼴로 사용하면 됩니다. 

## 사용하기 - 다른 job

```yaml

  job2:
    runs-on: ubuntu-latest
    needs: job1
    steps:
      - env:
          OUTPUT1: ${{needs.job1.outputs.output1}}
          OUTPUT2: ${{needs.job1.outputs.output2}}
        run: echo "$OUTPUT1 $OUTPUT2"
```

다른 job에서 해당 변수를 사용할 경우에도 [needs](https://sharknia.github.io/Github-Actions-Job---needs) 종속성 선언을 해줘야 합니다. job1에서 선언한 output을 사용할 것이므로 needs 설정을 해주고 `${{ needs.”job-이름”.outputs.”output변수명”}}` 의 꼴로 참조할 수 있습니다. 

또한, 변수 명의 경우 대소문자는 구분하지 않습니다. 

## 참조

[https://docs.github.com/ko/actions/using-jobs/defining-outputs-for-jobs#예-작업-출력-정의](https://docs.github.com/ko/actions/using-jobs/defining-outputs-for-jobs#예-작업-출력-정의)

