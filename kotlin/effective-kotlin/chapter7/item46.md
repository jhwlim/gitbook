---
description: 이펙티브 코틀린 정리하기
---

# 함수 타입 파라미터를 갖는 함수에 inline 한정자를 붙여라

`inline` 한정자의 역할은 컴파일 시점에 '함수를 호출하는 부분'을 '함수의 본문'으로 대체하는 것이다.

```kotlin
repeat(10) {
    print(it)
}

// 컴파일 시점
for (index in 0 until 10) {
    print(index)
}
```

일반적인 함수를 호출하면 함수 본문으로 점프하고, 본문의 모든 문장을 호출한 뒤에 함수를 호출했던 위치로 다시 점프하는 과정을 거친다.

하지만 '함수를 호출하는 부분'을 '함수의 본문'으로 대체하면 이러한 점프가 일어나지 않는다.

장점

- 타입 아규먼트에 `reified` 한정자를 붙여서 사용할 수 있다.
- 함수 타입 파라미터를 가진 함수가 훨씬 빠르게 동작한다.
- 비지역(non-local) 리턴을 사용할 수 있다.

단점

- `inlin` 한정자를 붙였을 때 발생하는 비용이 존재한다.

## 타입 아규먼트를 reified로 사용할 수 있다

JVM 바이트 코드에는 제네릭이 존재하지 않는다. 따라서 컴파일을 하면, 제네릭 타입과 관련된 내용이 제거 된다.

```kotlin
any is List<Int>    // 오류
any is List<*>      // OK

fun <T> printTypeName() {
    print(T::class.simpleName)  // 오류
}
```

함수를 인라인으로 만들면, 이러한 제한을 무시할 수 있다. 함수 호출이 본문으로 대체되므로, `reified` 한정자를 지정하면, 타입 파라미터를 사용한 부분이 타입 아규먼트로 대체된다.

```kotlin
inline fun <reified T> printTypeName() {
    print(T::class.simpleName)
}

// 사용
printTypeName<Int>()    // Int
printTypeName<Char>()   // Char
printTypeName<String>() // String
```

컴파일하는 동안 `printTypeName`의 본문이 실제로 대체된다.

```kotlin
print(Int::class.simpleName)    // Int
print(Char::class.simpleName)   // Char
print(String::class.simpleName) // String
```

## 함수 타입 파라미터를 가진 함수가 훨씬 빠르게 동작한다