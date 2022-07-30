---
description: 이펙티브 코틀린 정리하기
---

# 사용자 정의 오류보다는 표준 오류를 사용하라

{% hint style="success" %}
가능하다면 직접 오류를 정의하는 것보다는 최대한 표준 라이브러리의 오류를 사용하는 것이 좋다.
{% endhint %}

표준 라이브러리의 오류는 많은 개발자가 알고 있으므로, 이를 재사용하는 것이 좋다. 잘 만들어진 규약을 가진 널리 알려진 요소를 재사용하면, **다른 사람들이 API를 더 쉽게 이해할 수 있다.**

- `IllegalArgumentException`, `IllegalStateException` : `require()`와 `check()`를 사용해 `throw` 할 수 있는 예외
- `IndexOutOfBoundsException` : 인덱스 파라미터의 값이 범위를 벗어났다는 것을 나타낸다. 일반적으로 컬렉션 또는 배열과 함께 사용한다.
- `ConcurrentModificationException` : 동시 수정을 금지했는데 발생한 것을 나타낸다.
- `UnsupportedOperationException` : 사용자가 사용하려고 했던 메서드가 현재 객체에서는 사용할 수 없다는 것을 나타낸다. (기본적으로는 사용할 수 없는 메서드는 클래스에 없는 것이 좋다.)
- `NoSuchElementException` : 사용자가 사용하려고 했던 요소가 존재하지 않음을 나타낸다.