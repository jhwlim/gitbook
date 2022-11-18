---
description: 이펙티브 코틀린 정리하기
---

# 생성자 대신 팩토리 함수를 사용하라

## 팩토리 함수

생성자의 역할을 대신 해주는 함수

### 장점

- 함수에 이름을 붙일 수 있다. 이름은 객체가 생성되는 방법과 아규먼트로 무엇이 필요한지 설명할 수 있다. 이름이 붙어 있다면, 이해하기 쉽다. 또한, 동일한 파라미터 타입을 갖는 생성자의 충돌을 줄일 수 있다.
- 함수가 원하는 형태의 타입을 리턴할 수 있다. 따라서 다른 객체를 생성할 때 사용할 수 있다. 인터페이스 뒤에 실제 객체의 구현을 숨길 때 유용하게 사용할 수 있다.
- 호출될 때마다 새 객체를 만들 필요가 없다. 함수를 사용해서 객체를 생성하면 싱글턴 패턴처럼 객체를 하나만 생성하게 강제하거나, 최적화를 위해 캐싱 매커니즘을 사용할 수 있다. 또한, 객체를 만들 수 없을 경우, `null`을 리턴하게 만들 수도 있다.
- 아직 존재하지 않는 객체를 리턴할 수도 있다. 이를 활용하면 프로젝트를 빌드하지 않고도 앞으로 만들어질 객체를 사용하거나, 프록시를 통해 만들어지는 객체를 사용할 수 있다.
- 객체 외부에 팩토리 함수를 만들면, 그 가시성을 원하는 대로 제어할 수 있다. 예를 들어 톱레벨 팩토리 함수를 같은 파일 또는 같은 모듈에서만 접근하게 할 수 있다.
- 인라인으로 만들 수 있으며, 그 파라미터들은 `reified`로 만들 수 있다.
- 팩토리 함수는 생성자로 만들기 복잡한 객체도 만들어 낼 수 있다.
- 원하는 때에 생성자를 호출할 수 있다. (생성자는 즉시 슈퍼클래스 또는 기본 생성자를 호출해야 한다.)


다만 팩토리함수로 클래스를 생성할 때 서브클래스 생성에는 슈퍼클래스의 생성자가 필요하기 때문에, 서브클래스를 만들어낼 수 없다.

```kotlin
class IntLinkedList: MyLinkedList<Int>() { // MyLinkedList가 open이라면

    constructor(vararg ints: Int): myLinkedListOf(*ints) // 오류

}
```

하지만 팩토리 함수로 슈퍼클래스를 만들기로 했다면, 그 서브클래스에도 팩토리 함수를 만들면 된다.

```kotlin
class MyLinkedIntList(
    head: Int,
    tail: MyLinkedIntList?
) : MyLinkedList<Int>(head, tail)

fun myLinkedIntListOf(vararg elements: Int): MyLinkedIntList? {
    if (elements.isEmpty()) {
        return null
    }

    val head = elements.first()
    val elementsTail = elements.copyOfRange(1, elements.size)
    val tail = myLinkedIntListOf(*elementsTail)
    return MyLinkedIntList(head, tail)
}
```

팩토리 함수는 기본 생성자가 아닌 추가적인 생성자(secondary constructor)와 경쟁 관계이며, 추가적인 생성자보다 팩토리 함수를 많이 사용한다. 팩토리 함수는 다른 종류의 팩토리 함수와 경쟁 관계에 있다고 할 수 있다.

### 종류

- companion 객체 팩토리 함수
- 확장 팩토리 함수
- 톱레벨 팩토리 함수
- 가짜 생성자
- 팩토리 클래스의 메서드

## Companion 객체 팩토리 함수

팩토리 함수를 정의하는 가장 일반적인 방법

### 특징

인터페이스에도 구현할 수 있다.

```kotlin
class MyLinkedList<T>(
    val head: T,
    val tail: MyLinkedList<T>?
) : MyList<T> { ... }

interface MyList<T> {
    // ...

    companion object {

        fun <T> of(vararg elements: T): MyList<T>? { ... } 

    }
}

// 사용
val list = MyList.of(1, 2)
```

### 많이 사용하는 이름



## 확장 팩토리 함수

## 톱레벨 팩토리 함수

## 가짜 생성자

## 팩토리 클래스의 메서드

## 정리