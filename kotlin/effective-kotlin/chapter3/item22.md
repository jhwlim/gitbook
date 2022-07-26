---
description: 이펙티브 코틀린 정리하기
---

# 일반적인 알고리즘을 구현할 때 제네릭을 사용하라

타입 아규먼트를 사용하면 함수에 타입을 전달할 수 있다. 타입 아규먼트를 사용하는 함수(즉, 타입 파라미터를 갖는 함수)를 **제네릭 함수**라고 한다.

```kotlin
inline fun <T> Iterable<T>.filter(
    predicate: (T) -> Boolean
) { ... }
```

타입 파라미터는 컴파일러에 타입과 관련된 정보를 제공하여 컴파일러가 타입을 조금이라도 더 정확하게 추측할 수 있게 해준다. 따라서 프로그램이 조금 더 안전해지고, 개발자는 프로그래밍이 편해진다.

## 제네릭 제한

타입 파라미터의 중요한 기능 중 하나는 구체적인 타입의 서브타입만 사용하게 타입을 제한하는 것이다.

```kotlin
fun <T : Comparable<T>> Iterable<T>.sorted(): List<T> { ... } 

fun <T, C : MutableCollection<in T>> Iterable<T>.toCollection(
    destination: C
): C { ... }

class ListAdapter<T : ItemAdapter>(
    /* ... */
) { ... }
```

타입에 제한이 걸리므로, 내부에서 해당 타입이 제공하는 메서드를 사용할 수 있다.

많이 사용하는 제한으로는 `Any`가 있다. 이는 nullable이 아닌 타입을 나타낸다.

## 정리

일반적으로 타입 파라미터를 사용해서 type-safe 제네릭 알고리즘과 제네릭 객체를 구현한다.

타입 파라미터는 구체 자료형의 서브타입을 제한할 수 있다. 이렇게 하면 특정 자료형이 제공하는 메서드를 안전하게 사용할 수 있다.