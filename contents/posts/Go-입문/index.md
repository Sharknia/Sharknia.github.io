---
IDX: "NUM-191"
tags:
  - Go
  - OAuth2
description: "MAC Go 설치"
update: "2024-05-28T08:39:00.000Z"
date: "2024-05-28"
상태: "Ready"
title: "Go 입문"
---
![](image1.png)
## Install

맥에서는 다음의 명령어로 간단하게 Go를 설치할 수 있습니다. 

```bash
brew install go
```

### 설치 확인

다음의 명령어로 설치를 확인합니다. 

```bash
$ go version
go version go1.22.3 darwin/arm64
```

또는 다음의 명령어로 설치를 확인합니다. 

```bash
$ go env
GO111MODULE=''
GOARCH='arm64'
GOBIN=''
GOCACHE='/Users/furychick/Library/Caches/go-build'
GOENV='/Users/furychick/Library/Application Support/go/env'
GOEXE=''
GOEXPERIMENT=''
GOFLAGS=''
GOHOSTARCH='arm64'
GOHOSTOS='darwin'
GOINSECURE=''
GOMODCACHE='/Users/furychick/go/pkg/mod'
GONOPROXY=''
GONOSUMDB=''
GOOS='darwin'
GOPATH='/Users/furychick/go'
GOPRIVATE=''
GOPROXY='https://proxy.golang.org,direct'
GOROOT='/opt/homebrew/Cellar/go/1.22.3/libexec'
GOSUMDB='sum.golang.org'
GOTMPDIR=''
GOTOOLCHAIN='auto'
GOTOOLDIR='/opt/homebrew/Cellar/go/1.22.3/libexec/pkg/tool/darwin_arm64'
GOVCS=''
GOVERSION='go1.22.3'
GCCGO='gccgo'
AR='ar'
CC='cc'
CXX='c++'
CGO_ENABLED='1'
GOMOD='/dev/null'
GOWORK=''
CGO_CFLAGS='-O2 -g'
CGO_CPPFLAGS=''
CGO_CXXFLAGS='-O2 -g'
CGO_FFLAGS='-O2 -g'
CGO_LDFLAGS='-O2 -g'
PKG_CONFIG='pkg-config'
GOGCCFLAGS='-fPIC -arch arm64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -ffile-prefix-map=/var/folders/55/rk2pgkpj32d494v1hkj7j5zc0000gn/T/go-build2326145907=/tmp/go-build -gno-record-gcc-switches -fno-common'
```

## Go의 Workspace

Go의 워크스페이스와 프로젝트 관리는 다른 언어와 비교했을 때 독특한 면이 있습니다. Go의 워크스페이스는 Go 언어에서 소스 코드와 컴파일된 바이너리, 패키지를 관리하는 디렉터리 구조를 의미합니다. 

### **기본 워크스페이스 구조**

Go의 워크스페이스는 보통 세 가지 주요 디렉터리로 구성됩니다:

- src: 소스 코드 파일이 위치하는 디렉터리입니다. 각 프로젝트는 src 디렉터리 하위에 폴더로 존재하며, 폴더 구조는 패키지 경로를 반영합니다.

- pkg: 패키지가 컴파일된 결과물이 저장되는 디렉터리입니다. 이는 src 디렉터리의 코드가 컴파일될 때 생성되는 파일들이 위치하는 곳입니다.

- bin: 실행 파일이 저장되는 디렉터리입니다. src 디렉터리의 코드가 컴파일되어 생성된 실행 파일이 여기 저장됩니다.

## **Go Modules**

Go 1.11 이후로는 Go Modules가 도입되어, GOPATH의 중요성이 줄어들었습니다. Go Modules는 프로젝트별로 의존성을 관리할 수 있도록 하며, 프로젝트 디렉터리 내에서 독립적으로 동작합니다. Go Modules를 사용하면 go.mod 파일을 통해 의존성을 명시하고 관리할 수 있습니다. 

고로, 예전처럼 반드시 워크스페이스 안에 프로젝트가 위치할 필요가 없으며 여타 언어처럼 아무 곳에나 프로젝트를 생성할 수 있습니다. 

### 예제 프로젝트 생성

#### 프로젝트 디렉토리 생성

```bash
mkdir myproject
cd myproject
```

### 모듈 초기화

```bash
go mod init myproject
```

### 의존성 관리

```bash
go mod tidy
```

`go mod tidy` 명령어는 Go Modules와 관련된 명령어로, `go.mod` 파일과 `go.sum` 파일을 정리하고 최신 상태로 유지하는 데 사용됩니다. 이 명령어는 다음과 같은 작업을 수행합니다:

- 사용되지 않는 의존성 제거: 프로젝트 코드에서 더 이상 사용되지 않는 패키지를 `go.mod`와 `go.sum` 파일에서 제거합니다.

- 필요한 의존성 추가: 코드에서 사용하고 있지만 `go.mod`에 명시되지 않은 패키지를 찾아서 추가합니다.

- 정확한 버전 관리: 모든 의존성이 올바른 버전으로 명시되어 있는지 확인하고, 이를 `go.sum` 파일에 기록합니다.

### 실행

```bash
go run main.go
```

`go run` 명령어는 지정된 Go 소스 파일을 컴파일하고 즉시 실행합니다. 

컴파일된 실행 파일을 디스크에 저장하지 않고, 메모리에서 직접 실행합니다. 따라서 빠르게 코드를 테스트하거나 간단한 스크립트를 실행하는 데 유용합니다.

배포하거나 다른 시스템에서 실행할 프로그램을 만들 때에는 `go build` 명령어를 사용해야 합니다. 

## Hello World 찍어보기

### `main.go`  

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

### 파일 실행

```bash
$ go run main.go
```

## Go의 메서드 선언 기본 형식

Go 언어에서 메서드는 다음과 같은 형식을 가집니다. 

```go
func (receiver ReceiverType) MethodName(parameters) (returnTypes) {
    // 메서드 본문
}
```

receiver: 메서드가 호출될 때, 해당 타입의 인스턴스를 참조할 수 있는 변수입니다.

ReceiverType: 메서드가 속하는 타입입니다. 구조체 타입일 때가 많지만, 기본 타입일 수도 있습니다.

MethodName: 메서드의 이름입니다.

parameters: 메서드에 전달되는 인수입니다.

returnTypes: 메서드가 반환하는 값의 타입입니다.

### receiver(수신자)

수신자는 메서드가 어떤 타입에 속하는지 지정합니다. 수신자는 값 수신자와 포인터 수신자로 나눌 수 있습니다.

#### **값 수신자(Value Receiver)**

값 수신자는 메서드가 호출될 때, 수신자의 복사본을 사용합니다. 수신자의 필드를 변경하더라도 원본에는 영향을 미치지 않습니다.

```go
type Person struct {
    Name string
}

// 값 수신자 메서드
func (p Person) Greet() string {
    return "Hello, " + p.Name
}
```

#### **포인터 수신자(Pointer Receiver)**

포인터 수신자는 메서드가 호출될 때, 수신자의 포인터를 사용합니다. 수신자의 필드를 변경하면 원본에도 영향을 미칩니다.

```go
type Person struct {
    Name string
}

// 포인터 수신자 메서드
func (p *Person) SetName(name string) {
    p.Name = name
}
```

#### 예시

`Person` 구조체를 사용하여 값 수신자와 포인터 수신자의 예시입니다. 

```go
package main

import "fmt"

// Person 구조체 정의
type Person struct {
    Name string
    Age  int
}

// 값 수신자 메서드
func (p Person) Greet() string {
    return "Hello, " + p.Name
}

// 포인터 수신자 메서드
func (p *Person) SetAge(age int) {
    p.Age = age
}

func main() {
    // Person 인스턴스 생성
    person := Person{Name: "Alice", Age: 30}

    // 값 수신자 메서드 호출
    greeting := person.Greet()
    fmt.Println(greeting) // "Hello, Alice"

    // 포인터 수신자 메서드 호출
    person.SetAge(35)
    fmt.Println(person.Age) // 35
}
```

위 예시에서 `Greet` 메서드는 값 수신자를 사용하여 `Person`의 이름을 반환합니다. 반면, `SetAge` 메서드는 포인터 수신자를 사용하여 `Person`의 나이를 변경합니다.]

### **메서드와 함수의 차이점**

Go에서 메서드와 일반 함수의 주요 차이점은 메서드는 특정 타입에 속한다는 점입니다. 일반 함수는 특정 타입에 속하지 않으며, 메서드는 특정 타입의 인스턴스에서 호출됩니다.

#### **일반 함수 예시**

```go
func Greet(name string) string {
    return "Hello, " + name
}
```

#### **메서드 예시**

```go
type Person struct {
    Name string
}

func (p Person) Greet() string {
    return "Hello, " + p.Name
}
```

## 파이썬과의 비교로 Go 이해하기

클래스(구조체)의 메소드가 클래스(구조체) 바깥에서 정의되는 느낌이라고 이해했습니다. 파이썬과 비교하며 코드를 다시 살펴보겠습니다. 

### 구조체(클래스) 정의

#### 파이썬

클래스 내부에 초기화 메서드(`__init__`)를 정의합니다.

```python
class Person:
    def __init__(self, name):
        self.name = name
```

#### Go

구조체를 정의하여 데이터를 저장합니다.

```go
type Person struct {
    Name string
}
```

### 메서드 정의

#### 파이썬

파이썬에서는 클래스 내부에서 메서드를 정의합니다. 메서드는 클래스의 인스턴스에서 호출되며, `self` 키워드를 사용하여 인스턴스 변수와 다른 메서드에 접근합니다.

```python
class Person:
    def __init__(self, name):
        self.name = name

    def greet(self):
        return f"Hello, {self.name}"

person = Person("Alice")
print(person.greet())  # "Hello, Alice"
```

#### Go

Go에서는 메서드를 구조체 바깥에서 정의하지만, 해당 구조체와 연관시키기 위해 수신자(Receiver)를 사용합니다. 수신자는 메서드가 호출될 때, 해당 구조체의 인스턴스를 참조할 수 있게 해줍니다.

```go
package main

import "fmt"

// Person 구조체 정의
type Person struct {
    Name string
}

// 수신자를 사용하는 메서드 정의
func (p Person) Greet() string {
    return "Hello, " + p.Name
}

func main() {
    person := Person{Name: "Alice"}
    fmt.Println(person.Greet())  // "Hello, Alice"
}
```

### 메서드 호출

#### 파이썬

인스턴스를 생성하고 메서드를 호출합니다.

```python
person = Person("Alice")
print(person.greet())  # "Hello, Alice"
```

#### Go

구조체 인스턴스를 생성하고 메서드를 호출합니다.

```go
person := Person{Name: "Alice"}
fmt.Println(person.Greet())  // "Hello, Alice"
```

