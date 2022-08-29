---
description: 이펙티브 코틀린 정리하기
---

# 리시버를 명시적으로 참조하라

## 여러 개의 리시버

{% hint style="success" %}
스코프 내부에 둘 이상의 리시버가 있는 경우, 리시버를 명시적으로 나타내면 좋다.
{% endhint %}

```kotlin
// bad
class Node(
    val name: String
) {
    fun makeChild(childName: String) =
        create("$name.$childName")
            .apply { print("Created ${name}") }

    fun create(name: String): Node? = Node(name)
}

fun main() {
    val node = Node("parent")
    node.makeChild("child") // Created parent
}
```

리시버가 명확하지 않다면, 명시적으로 리시버를 적어서 이를 명확하게 하자. **레이블 없이 리시버를 사용하면, 가장 가까운 리시버를 의미한다. 외부에 있는 리시버를 사용하려면, 레이블을 사용해야 한다.**

```kotlin
// good
class Node(
    val name: String
) {
    fun makeChild(childName: String) =
        create("$name.$childName")
            .apply { print("Created ${this?.name} in ${this@Node.name") }

    fun create(name: String): Node? = Node(name)
}

fun main() {
    val node = Node("parent")
    node.makeChild("child") // Created parent.child in parent
}
```

명확하게 작성하면, 코드를 안전하게 사용할 수 있을 뿐만 아니라 가독성도 향상된다.

## DSL 마커

코틀린 DSL을 사용할 때는 여러 리시버를 가진 요소들이 중첩되더라도 리시버를 명시적으로 붙이지 않는다. DSL은 원래 그렇게 사용하도록 설계되었기 때문이다.

```kotlin
// normal use
table {
    tr {
        td { +"Column 1" }
        td { +"Column 2" }
    }
    tr {
        td { +"Value 1" }
        td { +"Value 2" }
    }
}
```

그런데 DSL에서는 외부의 함수를 사용하는 것이 위험한 경우가 있다.

```kotlin
// misuse
table {
    tr {
        td { +"Column 1" }
        td { +"Column 2" }
        tr {
            td { +"Value 1" }
            td { +"Value 2" }
        }
    }
}
```
기본적으로 모든 스코프에서 외부 스코프에 있는 리시버의 메서드를 사용할 수 있는데, 위와 같이 잘못된 사용을 막으려면 **암묵적으로 외부 리시버를 사용하는 것을 막는 `@DslMarker` 메타 어노테이션(어노테이션을 위한 어노테이션)을 사용해야 한다.**

```kotlin
@DslMarker
annotation class HtmlDsl

fun table(f: TableDsl.() -> Unit) { ... }

@HtmlDsl
class TableDsl { ... }
```

만약 외부 리시버의 함수를 사용하려면, 다음과 같이 명시적으로 사용해야 한다.

```kotlin
table {
    tr {
        td { +"Column 1" }
        td { +"Column 2" }
        this@table.tr {
            td { +"Value 1" }
            td { +"Value 2" }
        }
    }
}
```

DSL 마커는 가장 가까운 리시버만을 사용하게 하거나, 명시적으로 외부 리시버를 사용하지 못하게 할 때 활용할 수 있는 굉장히 중요한 메커니즘이다. DSL 설계에 따라서 사용 여부를 결정하는 것이 좋다.

## 정리

짧게 적을 수 있다는 이유만으로 리시버를 제거하지 말자.

여러 개의 리시버가 있는 상황 등에는 리시버를 명시적으로 적어주는 것이 좋다. 어떤 리시버의 함수인지를 명확하게 알 수 있으므로, 가독성이 향상된다.

DSL에서 외부 스코프에 있는 리시버를 명시적으로 적게 강제하고 싶다면, `@DslMarker` 메타 어노테이션을 사용한다.