# 안드로이드의 UI 스레드
UI를 업데이트하고 사용자와 상호작용을 리스닝하며, 상호작용에 의해 생선된 이벤트 처리를 전담한다.

## CallFromWrongThreadException
```
Android는 뷰 계층을 생성하지 않은 스레드에서 관련 뷰를 업데이트 한다면 CallFromWrongThreadException가 발생된다.
즉, UI 스레드만이 뷰 계층을 생성할 수 있고 업데이트 가능하다.
```

## NetworkOnMainThreadException
```
자바에서 네트워크 동작은 기본적으로 블로킹 된다. 
즉, UI 스레드에서 네트워크 호출을 하는 경우 응답을 받거나 혹은 타임아웃이 발생될때까지 UI 쓰레드는 멈추게 된다.
UI 쓰레드가 멈추게 되는 경우 뷰의 애니메이션이나 사용자와의 상호작용이 멈추는 것을 의미하므로 UI 쓰레드에서 네트워크 호출을 할때마다 Android 애플리케이션은 중단된다.
이때, Android는 NetworkOnMainThreadException 에러가 발생한다.
그러므로, 개발자는 Network 등 지연되는 작업은 Background Thread에서 처리하고, UI에 반영은 UI 스레드에서 처리 해야 한다.
```

# 스레드 생성
## CoroutineDispatcher
```
코루틴 디스패처를 통해 쉽게 스레드와 스레드풀을 생성할 수 있지만, 직접 억세스 하거나 제어하지는 않는다.
아래 예제는 스레드를 하나만 갖는 CoroutineDispatcher 생성 방법이다.
```
```kotlin
class Foo {
    val netDispatcher: ExecutorCoroutineDispatcher = newSingleThreadContext(name = "ServiceCall")
}
```

## 디스패처에 코루틴 붙이기
```
코루틴 디스패처가 생성이 되었다면, 코루틴에서 해당 디스패처를 사용할 수 있도록 설정할 수 있다.
코루틴에 코루틴 디스패처를 설정하면 해당 스레드에서 동작하게 된다.
```
```kotlin
class Foo {
    val netDispatcher: ExecutorCoroutineDispatcher = newSingleThreadContext(name = "ServiceCall")
    GlobalScope.launch(netDispatcher) {
        doSomething()
    }
}
```

### async 코루틴 시작
```
결과 처리를 위한 목적으로 코루틴을 사용한다면 async()를 사용해야 한다.
async는 Deferred<T>를 반환하며 T는 그 결과의 유형을 나타낸다.
async를 사용할 때 결과를 처리하는 것을 잊으면 안 된다.
```
```kotlin
fun main(args: Array<String>) = runBlocking {
    val task = GlobalScope.async {
        doSomething()
    }
    task.join()
    println("Completed")
}

fun doSomething() {
    throw UnsupportedOperationException("Can't do")
}
```
```
위 코드를 보면 UnsupportedOperationException가 발생되어 프로그램이 비정상 종료할것 같지만
실제 결과는 Completed가 표시되며 정상 종료 된다.
그 이유는 async 블록 안에서 발생되는 에러는 Deffered에 담기게 되고, 결과를 확인해야 전파된다.
결과를 안전하게 확인 하는 방법으로는 isCancelled와 getCancellationException()을 통해 검사할 수 있다.
```
```kotlin
fun main(args: Array<String>) = runBlocking {
    val task = GlobalScope.async {
        doSomething()
    }
    task.join()
    if (task.isCancelled) {
        val exception = task.getCanncellationException()
        println("Error with message: ${exception.cause}")
    } else {
        println("Completed")
    }
}
```
```
만약 에러를 전파하고 싶다면 await()을 이용하면 된다.
task.await()
```

### launch 코루틴 시작
```
결과를 반환하지 않는 코루틴을 시작하려면 launch()를 사용해야 한다.
launch는 연산을 실패한 경우에만 통보 받기를 원하는 fire-and-forget 시나리오를 위해 설계됐다.
또한 필요할때 취소할 수 있는 함수도 제공된다.
```
```kotlin
fun main(args: Array<String>) = runBlocking {
    val task = GlobalScope.launch {
        doSomething()
    }
    task.join()
    println("Completed")
}

fun doSomething() {
    throw UnsupportedOperationException("Can't do")
}
```
```
결과는 예상한 대로 UnsupportedOperationException가 log가 표시되지만, 실행이 중단되지 않고 "Completed" 까지 표시 된다.
```

