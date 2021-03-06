# 프로세스, 스레드, 코루틴

## 프로세스
```
프로세스는 메모리에 올라와 실행되고 있는 애플리케이션의 인스턴스다.
운영체제로부터 시스템 자원을 할당받는 작업의 단위이며, 내부 스레드에서 접근 가능하다.
프로세스는 1개 이상의 스레드를 가지고 있다.
```

## 스레드
```
스레드는 프로세스에서 실행되는 여러 흐름의 단위이다.
프로세스는 최소 하나의 프로세스를 포함하는데 이 스레드는 애플리케이션의 진입점을 실행한다.
스레드를 프로세스 내에 스레드끼리 자원들을(Code, Data, Heap) 공유하며 실행된다.
```

## 코루틴
```
코루틴은 경량 스레드라고 불리며, 스레드와 마찬가지로 실행될 명령어 집합을 정의하며 스레드에서 실행된다.
하나의 스레드에서 여러개의 코루틴이 있을 수 있지만, 명령어가 처리될때에는 1개의 스레드에서 1개의 코루틴이 실행된다.
코루틴은 실행중 중단 지점에 도달하면 스레드를 떠나 대기중인 다른 코루틴을 실행시킨다.
코트린의 코루틴은 스택리스 코루틴이라고 불리며, 특정 스레드에 종속되지 않기때문에 프로세스에서 Context Switching이 필요하지 않는다.
그러므로, 수천개의 스레드보다 수천개의 코루틴을 생성하는것이 더 빠르고 적게 자원을 소모한다.
```

[* 스택리스 코루틴이란?](https://www.charlezz.com/?p=44635)

# 코틀린에서의 동시성
## 동시성
```
동시에 여러 작업들이 실행되는 것처럼 보이는 논리적인 개념이다.
싱글 코어에서 여러 스레드를 동작시키는 방식을 가리킨다.
```

## 병렬성
```
동시에 여러 작업들이 병렬적으로 실행되는 물리적인 개념이다.
다중 코어에서 여러 작업들이 동시에 처리되는 방식을 가리킨다.
```

# CPU 바운드와 I/O 바운드
## CPU 바운드
```
작업의 처리속도가 CPU의 진행속도에 제한됨을 의미한다.
즉, 코드변경 없이 CPU의 성능에 비례하여 처리속도가 증가 및 감소한다.
```

## I/O 바운드
```
작업의 처리속도가 입출력 장치 시스템 속도에 의해 제한됨을 의미한다.
즉, CPU가 아무리 빨리 일을 처리하더라도 입출력 장치 성능에 비례하여 처리속도가 단축 및 증가될 것이다.
```

# CPU 바운드 알고리즘에서의 동시성과 병렬성
## 단일 코어에서의 CPU 바운드 알고리즘
```
단일 코어에서 n개의 데이터를 동시성 코드로 처리 하는 경우 다음과 같은 경우가 발생할 수 있다.
데이터들은 여러개의 스레드 사이에 교차 배치되며 매번 일정량의 데이터를 처리 후 다음 스레드로 전환될 것이다.
다음 스레드로 전환이 되면서 현재 작업중인 스레드의 상태를 저장하고 다음 실행될 스레드의 상태를 불러온다.
이 작업을 컨텍스트 스위칭이라고 하며 전체 프로세스에 오버헤드가 발생된다.
즉, 단일 스레드에서 처리되는 시간이 여러 스레드에서 처리되는 시간보다 빠를 수 있다.
```

## 멀티 코어에서의 CPU 바운드 알고리즘
```
멀티 코어에서 n개의 데이터를 코어 갯수만큼 생성하여 병렬 실행할 경우 시간이 단일코어에 비해 시간이 단축될것이다.
```

## IO 바운드 알고리즘에서 동시성 대 병렬성
```
IO 바운드 작업은 입출력 장치의 결과를 기다린다.
따라서, IO 바운드 작업은 동시성이나 병렬성에 상관없이 더 나은 성능을 제공한다.
```

# 동시성이 어려운 이유
## 레이스 컨디션
```
두개 이상의 프로세스에서 공용 데이터를 병행적으로 접근하여 읽거나 쓸 때, 데이터에 접근한 순서에 따라 그 실행 결과가 달라지는 상황을 말한다.
```

## 원자성 위반
```
원자성 위반은 메모리로 접근을 동기화하여 보호되지 않았을때 발생된다.
예를들면, 여러 스레드에서 하나의 데이터를 접근하여 업데이트를 하는 경우, 다른 스레드에서 수정하고 있는 데이터를 변경하여 예상되지 않은 결과가 나올 수 있다.
```

## 교착상태
```
상호 배제에 의해 나타나는 문제점으로, 둘 이상의 프로세스가 다른 프로세스가 점유하고 있는 자원을 서로 기다릴때 무한 대기에 빠지는 상황이다.
```

## 라이브 락
```
여러 스레드가 계속해서 자원을 점유했다 풀었다 하는 상황이 반복되어, 정상적으로 실핼할 수 없는 상황을 말한다.
```