---
description: 이펙티브 코틀린 정리하기
---

# 불필요한 객체 생성을 피하라

불필요한 객체 생성을 피하는 것이 최적화의 관점에서 좋다.

## 객체 생성 비용은 항상 클까?

어떤 객체를 랩(wrap)하면, 크게 세 가지 비용이 발생한다.

1. 객체는 더 많은 용량을 차지한다.
2. 요소가 캡슐화되어 있다면, 접근에 추가적인 함수 호출이 필요하다. 함수를 사용하는 처리는 굉장히 빠르므로 큰 비용이 발생하지는 않는다.
3. 객체는 생성되어야 한다. 객체는 생성되고, 메모리 영역에 할당되고, 이에 대한 레퍼런스를 만드는 등의 작업이 필요하다.

객체를 제거함으로써 위의 세 가지 비용을 모두 피할 수 있다. 특히 객체를 재사용하면 첫 번째와 세 번째에 설명한 비용을 제거할 수 있다.

## 불필요한 객체를 제거하는 방법

### 객체 선언 (싱글톤)

```kotlin
sealed class LinkedList<T>

class Node<T>(
    val head: T,
    val tail: LinkedList<T>
): LinkedList<T>()

class Empty<T>: LinkedList<T>()

// 사용
val list: LinkedList<Int> = Node(1, Node(2, Node(3, Empty())))
val list2: LinkedList<String> = Node("A", Node("B", Empty()))
```

위 코드의 문제점

- 리스트를 만들 때마다 `Empty` 인스턴스를 만들어야 한다.

개선된 코드

```kotlin
sealed class LinkedList<out T>

class Node<out T>(
    val head: T,
    val tail: LinkedList<T>
) : LinkedList<T>()

object Empty : LinkedList<Nothing>()

val list: LinkedList<Int> = Node(1, Node(2, Node(3, Empty())))
val list2: LinkedList<String> = Node("A", Node("B", Empty))
```

- Nothing 리스트를 만들어서 사용한다. Nothing은 모든 타입의 서브타입이다. 따라서 `LinkedList<Nothing>`은 리스트가 covariant 이라면(out 한정자), 모든 LinkedList의 서브타입이 된다. 리스트는 immutable이고, 이 타입은 out 위치에서만 사용되므로, 현재 상황에서는 타입 아규먼트를 covariant로 만드는 것은 의미 있는 일이다.

이러한 트릭은 immutable sealed 클래스를 정의할 때 자주 사용된다. 만약 mutable 객체에 사용하면 공유 상태 관리와 관련된 버그를 검출하기 어려울 수 있으므로 좋지 않다.

{% hint style="success" %}
mutable 객체는 캐시하지 않는다는 규칙을 지키는 것이 좋다.
{% endhint %}

### 캐시를 활용하는 팩토리 함수