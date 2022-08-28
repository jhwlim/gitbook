# 스프링과 함께 사용하기

<https://kotest.io/docs/extensions/spring.html>

## 기타

### @Transactional 으로 영속성(persistence)이 유지 안되는 문제

#### Solution

```kotlin
override fun extensions() = listOf(SpringTestExtension(SpringTestLifecycleMode.Root))
```
