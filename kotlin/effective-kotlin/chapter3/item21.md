---
description: 이펙티브 코틀린 정리하기
---

# 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라

프로퍼티 위임을 사용하면 일반적인 프로퍼티의 행위를 추출해서 재사용할 수 있다.

## lazy 프로퍼티 패턴

지연 프로퍼티는 lazy 프로퍼티는 이후에 처음 사용하는 요청이 들어올 때 초기화되는 프로퍼티를 의미한다.

stdlib의 lazy 함수를 사용하여 lazy 프로퍼티 패턴을 쉽게 구현할 수 있다.

```kotlin
val value by lazy { createValue() }
```

## observable 패턴

프로퍼티 위임을 사용하면, 변화가 있을 때 이를 감지하는 observable 패턴을 쉽게 만들 수 있다.

stdlib의 observable 델리게이트를 기반으로 간단하게 구현할 수 있다.

```kotlin
var items: List<Item> by
    Delegates.observable(listOf()) { _, _, _ -> 
        notifyDataSetChanged()
    }

var key: String? by
    Delegates.observable(null) { _, old, new ->
        Log.e("key changed from $old to $new")
    }
```

