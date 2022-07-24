---
description: 이펙티브 코틀린 정리하기
---

# 최대한 플랫폼 타입을 사용하지 말라

널 안전성(null-safety)은 코틀린의 주요 기능 중 하나이며, 코틀린에서 NPE(Null-Pointer Exception)는 거의 찾아보기 어렵다.

하지만 null-safety 메커니즘이 없는 자바 등의 프로그래밍 언어와 코틀린을 연결해서 사용할 때는 NPE가 발생할 수 있다.

따라서 최대한 안전하게 접근하려면 nullable로 가정하고 다루어야 한다.

## 플랫폼 타입이란?

코틀린은 자바 등의 다른 프로그래밍 언어에서 넘어온 타입들을 특수하게 다루는데, 이러한 타입을 **플랫폼 타입**이라고 부른다. 플랫폼 타입은 `String!`처럼 타입 이름 뒤에 `!` 기호를 붙여서 표기한다.

```java
// 자바
public class UserRepo {

    public User getUser() { ... }

}
```

```kotlin
val repo = UserRepo()
val user1 = repo.user // User!
val user2: User = repo.user // User
val user3: User? = repo.user // User?
```

{% hint style="warning" %}
플랫폼 타입을 사용할 때는 항상 주의를 기울여야 한다. null이 아니라고 생각되는 것이 null일 가능성이 있기 때문이다.

따라서 설계자가 명시적으로 어노테이션으로 표시하거나, 주석으로 달아두지 않으면, 동작이 변경될 가능성이 있다.
{% endhint %}

{% hint style="success" %}
자바를 코틀린과 함께 사용할 때, 자바 코드를 직접 조작할 수 있다면 가능한 `@Nullable`과 `@NotNull` 어노테이션을 붙여서 사용하는 것이 좋다.
{% endhint %}

## 플랫폼 타입의 위험성

플랫폼 타입은 값을 활용할 때 NPE가 발생할 수 있다. 한두번 안전하게 사용했더라도, 이후에 사용할 때에는 NPE를 발생시킬 가능성이 존재하고, 이는 오류를 찾는 데 오랜 시간을 걸리게 할 것이다.

{% hint style="danger" %}
플랫폼 타입이 전파(다른 곳에서 사용)되는 일은 굉장히 위험하다. 항상 위험을 내포하고 있으므로, 안전한 코드를 원한다면 이런 부분을 제거하는 것이 좋다.
{% endhint %}

## 정리

- 플랫폼 타입을 사용하는 코드는 해당 부분만 위험할 뿐만 아니라, 이를 활용하는 곳까지 영향을 줄 수 있는 위험한 코드이다.
- 따라서 이런 코드를 사용하고 있다면 빨리 해당 코드를 제거하는 것이 좋다.
- 연결되어 있는 자바 생성자, 메서드, 필드에 nullable 여부를 지정하는 어노테이션을 활용하는 것도 좋다. 이러한 정보는 자바 개발자에게도 유용한 정보다.