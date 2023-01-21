---
description: 이펙티브 코틀린 정리하기
---

# 컬렉션 처리 단계 수를 제한하라

모든 컬렉션 처리 메서드는 비용이 많이 든다. 표준 컬렉션 처리는 내부적으로 요소들을 활용해 반복을 돌며, 내부적으로 계산해 반복을 돌며, 내부적으로 계산을 위해 추가적인 컬렉션을 만들어 사용한다. 시퀀스 처리도 시퀀스 전체를 랩하는 객체가 만들어지며, 조작을 위해서 또 다른 추가적인 객체를 만들어 낸다. (람다 표현식을 객체로 만들어 사용해야 한다.) 두 처리 모두  요소의 수가 많다면, 꽤 큰 비용이 들어간다.

{% hint style="success" %}
적절한 메서드를 활용해서, 컬렉션 처리 단계 수를 적절하게 제한하는 것이 좋다.
{% endhint %}

```kotlin
class Student(
    val name: String?
)

// 작동은 한다.
fun List<Student>.getName(): List<String> = this
    .map { it.name }
    .filter { it != null }
    .map { it!! }

// 더 좋다.
fun List<Student>.getNames(): List<String> = this
    .map { it.name }
    .filterNotNull()

// 가장 좋다.
fun List<Student>.getNames(): List<String> = this
    .mapNotNull { it.name }
```

## 정리

대부분의 컬렉션 처리 단계는 '전체 컬렉션에 대한 반복'과 '중간 컬렉션 생성'이라는 비용이 발생한다. 이 비용은 적절한 컬렉션 처리 함수들을 활용해서 줄일 수 있다.