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

팩토리 함수는 캐시(cache)를 가질 수 있다. 그래서 팩토리 함수는 항상 같은 객체를 리턴하게 만들 수도 있다.

모든 순수 함수는 캐싱을 활용할 수 있다. 이를 메모이제이션(memoization)이라고 부른다.

하지만 캐시를 위해 더 많은 메모리를 사용한다. 만약 메모리 문제로 크래시가 생긴다면 메모리를 해제해 주면 된다.

참고로 메모리가 필요할 때 가비지 컬렉터가 자동으로 메모리를 해제해 주는 SoftReference를 사용하면 더 좋다.

{% hint style="info" %}
- WeakReference : 가비지 컬렉터가 값을 정리하는 것을 막지 않는다. 따라서 다른 레퍼런스(변수)가 이를 사용하지 않으면 곧바로 제거된다.
- SoftReference : 가비지 컬렉터가 값을 정리할 수도 있고, 정리하지 않을 수도 있다. 일반적인 JVM 구현의 경우, 메모리가 부족해서 추가로 필요한 경우에만 정리한다. 따라서 캐시를 만들 때는 SoftReference를 사용하는 것이 좋다.
{% endhint %}

{% hint style="success" %}
캐시는 언제나 메모리와 성능의 트레이드 오프가 발생하므로, 여러 가지 상황을 잘 고려해서 현명하게 사용해야 한다.
{% endhint %}

### 무거운 객체를 외부 스코프로 보내기

무거운 객체를 외부 스코프로 보내는 방법이 있다. 컬렉션 처리에서 이루어지는 무거운 연산은 컬렉션 처리 함수 내부에서 외부로 빼는 것이 좋다.

```kotlin
// before
fun <T: Comparable<T>> Iterable<T>.countMax(): Int = count { it == this.max() }

// after
fun <T: Comparable<T>> Iterable<T>.countMax(): Int {
    val max = this.max()
    return count { it == max }
}
```

```kotlin
// before
fun String.isValidIpAddress(): Boolean {
    return this.matches("\\A(?:(?:25[0-5]2[0-4]....\\z").Regex())
}

// after
private val IS_VALID_EMAIL_REGEX = "\\A(?:(?:25[0-5]2[0-4]....\\z".toRegex()

fun String.isValidIpAddress(): Boolean = matches(IS_VALID_EMAIL_REGEX)
```

함수를 사용하지 않는다면 정규 표현식이 만들어지는 것 자체가 낭비이다. 이러한 경우 지연 초기화하면 된다.

```kotlin
private val IS_VALID_EMAIL_REGEX by lazy {
    "\\A(?:(?:25[0-5]2[0-4]....\\z".toRegex()
}
```

### 지연 초기화

무거운 클래스를 만들 때는 지연되게 만드는 것이 좋을 때가 있다.

```kotlin
// before
class A {
    val b = B()
    val c = C()
    val d = D()

    // ...
}

// after
class A {
    val b by lazy { B() }
    val c by lazy { C() }
    val d by lazy { D() }
}
```

A 클래스에 B, C, D라는 무거운 인스턴스가 필요하다면 A 객체를 생성하는 과정이 굉장히 무거워진다. 이러한 경우 내부에 있는 인스턴스들을 지연 초기화하면, A 객체를 생성하는 과정을 가볍게 만들 수 있다.

하지만 무거운 객체를 가졌지만, 메서드의 호출은 빨라야 하는 경우, 처음 호출될 때 무거운 객체들의 초기화가 필요하기 때문에 첫 번째 호출 때 응답 시간이 굉장히 길다. 그래서 백엔드 애플리케이션에서 좋지 않을 수 있다.

또한 지연되게 만들면, 성능 테스트가 복잡해지는 문제가 있다.

### 기본 자료형 사용하기

JVM은 숫자와 문자 등의 기본적인 요소를 나타내기 위한 특별한 기본 내장 자료형을 갖고 있는데, 이를 **기본 자료형**이라고 부른다. 코틀린/JVM 컴파일러는 내부적으로 최대한 이러한 기본 자료형을 사용한다.

다만 아래의 두 가지 상황에서는 기본 자료형을 랩(wrap)한 자료형이 사용된다.

1. nullable 타입을 연산할 때
2. 타입을 제네릭으로 사용할 때

랩한 자료형 대신 기본 자료형을 사용하게 코드를 최적화할 수 있다. 굉장히 큰 컬렉션을 처리할 때 차이를 확인할 수 있다.

하지만 숫자와 관련된 연산은 어떤 형태의 자료형을 사용하나 성능적으로는 큰 차이가 없다. 또한, 기존의 코드에서 사용되던 자료형을 일괄 변경하면, 코드를 읽기 힘들어질 수 있다.

{% hint style="success" %}
결과적으로 코드와 라이브러리의 성능이 굉장히 중요한 부분에서만 이를 적용하는 것이 좋다.
{% endhint %}

{% hint style="info" %}
프로파일러를 활용하면, 어떤 부분이 성능에 중요한 역할을 하는지 쉽게 찾을 수 있다.
{% endhint %}

## 정리

위에서 소개한 방법 중 코몇 가지는 코드의 가독성을 향상시켜 주는 장점도 있으므로 적극적으로 사용하는 것이 좋다. (무거운 객체를 외부 스코프로 보내기)

성능이 중요한 코드에서 성능을 조금이라도 향상시킬 수 있는 방법의 경우, 큰 변경이 필요하거나, 다른 코드에 문제를 일으킬 수 있다면 최적화를 미루는 것도 방법이다.