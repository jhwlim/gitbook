---
description: Real MySQL 8.0
---

# 13. 파티션

파티션 기능은 테이블을 논리적으로는 하나의 테이블이지만, 물리적으로는 여러 개의 테이블로 분리해서 관리할 수 있게 해준다.

파티션 기능은 주로 대용량의 테이블을 물리적으로 여러 개의 소규모 테이블로 분산하는 목적으로 사용한다.

자주 사용하는 파티션 방법과 사용 시 주의해야 할 사항 위주로 살펴본다.

## 목차

1. 개요
2. 주의사항
3. MySQL 파티션의 종류