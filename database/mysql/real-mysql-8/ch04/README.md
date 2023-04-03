---
description: Real MySQL 8.0
---

# 04. 아키텍처

MySQL 서버는 **MySQL 엔진**과 **스토리지 엔진**으로 구분할 수 있다.

스토리지 엔진은 핸들러 API를 만족하면 누구든지 스토리지 엔진을 구현해서 MySQL 서버에 추가해서 사용할 수 있다.

이번 장에서는 MySQL 엔진과 MySQL 서버에서 기본으로 제공되는 **InnoDB 스토리지 엔진**, 그리고 **MyISAM 스토리지 엔진**을 구분해서 살펴본다.

## 목차

1. MySQL 엔진 아키텍처
2. InnoDB 스토리지 엔진 아키텍처
3. MyISAM 스토리지 엔진 아키텍처
4. MySQL 로그 파일
