---
description: 이펙티브 코틀린 정리하기
---

# use를 사용하여 리소스를 닫아라

더 이상 필요하지 않을 때, `close()`를 호출해서 명시적으로 닫아야 하는 리소스가 있다. 이러한 리소스들은 `AutoCloseable`을 상속받는 `Closeable` 인터페이스를 구현하고 있다.

리소스에 대한 레퍼런스가 없어질 때, 가비지 컬렉터가 리소스를 처리하기는 하지만 굉장히 느리며 그동안 리소스를 유지하는 비용이 많이 들어간다.

따라서 더 이상 필요하지 않다면 명시적으로 `close()`를 호출해주는 것이 좋다.

## try ~ finally 블록 사용하기

전통적으로 이러한 리소스는 try ~ finally 블록을 사용해서 처리했다.

```kotlin
fun countCharactersInFile(path: String): Int {
    val reader = BufferedReader(FileReader(path))
    try {
        return reader.lineSequence().sumBy { it.length }
    } finally {
        reader.close()
    }
}
```

### 단점

코드가 굉장히 복잡하고 좋지 않다. 리소스를 닫을 때 예외가 발생할 수도 있는데, 이러한 예외를 따로 처리하지 않기 때문이다.

또한, try 블록과 finally 블록 내부에서 오류가 발생하면 둘 중 하나만 전파된다. 둘 다 전파하도록 구현하려면 코드가 굉장히 길고 복잡해진다.

## use 사용하기

코틀린 표준 라이브러리에서는 `Closeable` 인터페이스에 대한 확장함수로 `use()`를 제공한다. finally 블록에서 `close()`를 호출하고 있으며, 예외가 발생하더라도 `close()`를 호출하도록 구현되어 있다.

```kotlin
fun countCharactersInFile(path: String): Int {
    val reader = BufferedReader(FileReader(path))
    reader.use {
        return reader.lineSequence().sumBy { it.length }
    }
}
```

아래와 같이 줄여서 작성할 수도 있다.

```kotlin
fun countCharactersInFile(path: String): Int {
    BufferedReader(FileReader(path)).use { reader ->
        return reader.lineSequence().sumBy { it.length }
    }
}
```

### useLines

코틀린 표준 라이브러리는 <u>파일을 한 줄씩 처리</u>할 때 활용할 수 있는 `useLines()`를 제공한다.

```kotlin
fun countCharactersInFile(path: String): Int {
    File(path).useLines { lines -> 
        return lines.sumBy { it.length }
    }
}
```

**메모리에 파일의 내용을 한 줄씩만 유지하므로, 대용량 파일도 적절하게 처리할 수 있다.**

다만 파일의 줄을 한 번만 사용할 수 있다는 단점이 있다. 파일을 특정 줄을 두 번 이상 반복 처리하려면, 파일을 두 번 이상 열어야 한다.

## 정리

use를 사용하면 `Closeable`/`AutoCloseable`을 구현한 객체를 쉽고 안전하게 처리할 수 있다.

또한 파일을 처리할 때는 파일을 한 줄씩 읽어 들이는 `useLines()`를 사용하는 것이 좋다.