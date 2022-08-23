---
description: 이펙티브 코틀린 정리하기
---

# 코딩 컨벤션을 지켜라

## 코딩 컨벤션을 지켜야하는 이유

- 어떤 프로젝트를 접해도 쉽게 이해할 수 있다.
- 다른 외부 개발자도 프로젝트의 코드를 쉽게 이해할 수 있다.
- 다른 개발자도 코드의 작동 방식을 쉽게 추측할 수 있다.
- 코드를 병합하고, 한 프로젝트의 코드 일부를 다른 코드로 이동하는 것이 쉽다.

## 컨벤션을 지킬 때 도움이 되는 도구

- IntelliJ Formatter : 공식 코딩 컨벤션 스타일에 맞춰서 코드를 변경해준다.
- ktlink : 많이 사용되는 코드를 분석하고 컨벤션 위반을 알려 주는 linter

## 자주 위반 되는 규칙

많은 파라미터를 갖고 있는 클래스는 다음과 같이 각각의 파라미터를 한 줄씩 작성하는 방법을 사용한다.

```kotlin
class Person(
    val id: Int = 0,
    val name: String = "",
    val surname: String = ""
) : Human(id, name) { ... }
```

<br>

함수도 파라미터들을 많이 갖고 있고 길다면 다음과 같이 작성한다.

```kotlin
public fun <T> Iterable<T>.joinToString(
    separator: CharSequence = ", ",
    prefix: CharSequence = "",
    postfix: CharSequence = "",
    limit: Int = -1,
    truncated: CharSequence = "...",
    transform: ((T) -> CharSequence)? = null
):  String { ... }
```

<br>

아래의 코드는 두 가지 측면에서 문제가 될 수 있다.

- 모든 클래스의 아규먼트가 클래스 이름에 따라서 다른 크기의 들여쓰기를 갖는다. 클래스 이름을 변경할 때 모든 기본 생성자 파라미터의 들여쓰기를 조정해야 한다.
- 클래스가 차지하는 공간의 너비가 너무 크다.

```kotlin
// 이렇게 하지 마세요.
class Person(val id: Int = 0,
             val name: String = "",
             val surname: String = "") : Human(id, name) { ... }
```

{% hint style="success" %}
프로젝트의 컨벤션은 반드시 지켜 주는 것이 좋다. 프로젝트의 모든 코드는 마치 한 사람이 작성한 것처럼 작성되어야 한다.
{% endhint %}

코딩 컨벤션을 확실하게 읽고, 정적 검사기를 활용해서 프로젝트의 코딩 컨벤션 일관성을 유지하자.