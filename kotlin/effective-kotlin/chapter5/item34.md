---
description: 이펙티브 코틀린 정리하기
---

# 기본 생성자에 이름 있는 옵션 아규먼트를 사용하라

일반적으로는 기본 생성자를 활용해서 객체를 만드는 것이 좋다.

## 점층적 생성자 패턴

여러 가지 종류의 생성자를 사용하는 패턴

```kotlin
class Pizza {
    val size: String
    val cheese: Int
    val olives: Int
    val bacon: Int

    constructor(size: String, cheese: Int, olives: Int, bacon: Int) {
        this.size = size
        this.cheese = cheese
        this.olives = olives
        this.bacon = bacon
    }

    constructor(size: String, cheese: Int, olives: Int): this(size, cheese, olives, 0)

    constructor(size: String, cheese: Int): this(size, cheese, 0)

    constructor(size: String): this(size, 0)
}
```

<br>

{% hint style="success" %}
코틀린에서는 일반적으로 디폴트 아규먼트를 사용한다.
{% endhint %}

```kotlin
class Pizza(
    val size: String,
    val cheese: Int = 0,
    val olives: Int = 0,
    val bacon: Int = 0
)

// 사용
val myFavorite = Pizza("L", olives = 3, cheese = 1)
```

디폴트 아규먼트는 코드를 단순하고 깔끔하게 만들어 줄 뿐만 아니라, 점층적 생성자보다 훨씬 다양한 기능을 제공한다.

### 디폴트 아규먼트가 점층적 생성자보다 좋은 이유

- 파라미터들의 값을 원하는 대로 지정할 수 있다.
- 아규먼트를 원하는 순서로 지정할 수 있다.
- 명시적으로 이름을 붙여서 아규먼트를 지정하므로 의미가 훨씬 명확하다.

## 빌더 패턴

### 장점

- 파라미터에 이름을 붙일 수 있다.
- 파라미터를 원하는 순서로 지정할 수 있다.
- 디폴트 값을 지정할 수 있다.
- 팩토리로 사용할 수 있다.

### 이름 있는 파라미터를 사용하는 것이 좋은 이유

- 더 짧다. 구현하기 더 쉽다.
- 더 명확하다. 객체가 어떻게 생성되는지 확인하고 싶을 때, 빌더 패턴은 여러 메서드들을 확인해야 한다. 디폴트로 어떤 값을 가지는지 내부적으로 어떤 추가적인 처리가 일어나는지 이해하기 어렵다.
- 더 사용하기 쉽다. 빌더 패턴은 추가적인 knowledge가 필요하다.
- 동시성과 관련된 문제가 없다. 코틀린의 함수 파라미터는 항상 immutable 이다. 반면에 대부분의 빌더 패턴에서 프로퍼티는 mutable 이다. 따라서 빌더 패턴의 빌더 함수를 thread-safe하게 구현하는 것은 어렵다.

### 빌더 패턴이 좋은 경우

빌더 패턴은 값의 의미를 묶어서 지정할 수 있다. (setPositiveButton, setNegativeButton, addRoute)

또한, 특정 값을 누적하는 형태로 사용할 수 있다. (addRoute)

```kotlin
val dialog = AlertDialog.Builder(context)
    .setMessage(R.string.fire_missiles)
    .setPositiveButton(R.string.fire, {d, id -> 
        // 미사일 발사!
    })
    .setNegativeButton(R.string.cancel, {d, id -> 
        // 사용자가 대화상자에서 취소를 누른 경우
    })
    .create()

val router = Router.Builder()
    .addRoute(path = "/home", ::showHome)
    .addRoute(path = "/users, ::showUsers)
    .build()
```

빌더 패턴을 사용하지 않고 이를 구현하려면, 추가적인 타입들을 만들고 활용해야 하는데, 코드가 오히려 복잡해진다.

```kotlin
// 빌더 패턴을 사용하지 않은 경우
val AlertDialog(
    context,
    message = R.string.fire_missiles,
    positiveButtonDescription = ButtonDescription(R.string.fire, { d, id -> 
        // 미사일 발사!
    }),
    negativeButtonDescription = ButtonDescription(R.string.cancel, { d, id -> 
        // 사용자가 대화상자에서 취소를 누른 경우
    })
)

val router = Router(
    routes = listOf(
        Route("/home", ::showHome),
        Route("/users", ::showUsers)
    )
)
```

<br>

{% hint style="success" %}
하지만 일반적으로 이런 코드는 다음과 같이 DSL 빌더를 사용한다.
{% endhint %}

```kotlin
val dialog = context.alert(R.string.fire_missiles) {
    positiveButton(R.string.fire) {
        // 미사일 발사!
    }
    negativeButton {
        // 사용자가 대화상자에서 취소를 누른 경우
    }
}

val route = router {
    "/home" directsTo ::showHome
    "/users" directsTo ::showUsers
}
```

DSL 빌더를 활용하는 패턴이 전통적인 빌더 패턴보다 훨씬 유연하고 명확해서, 코틀린은 이와 같은 형태의 코드를 많이 사용한다.

### 빌더 패턴을 사용하는 경우

- 빌더 패턴을 사용하는 다른 언어로 작성된 라이브러리를 그대로 옯길 때
- 디폴트 아규먼트와 DSL을 지원하지 않는 다른 언어에서 쉽게 사용할 수 있는 API를 설계할 때

{% hint style="success" %}
이를 제외하면, 빌더 패턴 대신 디폴트 아규먼트를 갖는 기본 생성자 또는 DSL을 사용하는 것이 좋다.
{% endhint %}

## 정리

코틀린에서는 점층적 생성자 패턴 대신 디폴트 아규먼트를 활용하는 것이 좋다. 디폴트 아규먼트는 더 짧고, 더 명확하고, 더 사용하기 쉽다.

빌더 패턴도 마찬가지로 거의 사용하지 않는다. 기본 생성자를 사용하는 코드로 바꾸거나, DSL을 활용하는 것이 좋다.