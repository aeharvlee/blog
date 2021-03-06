---
title: "Improve Perofrmance of MySQL query in 3 minutes"
date: 2019-12-22
---

> 대상 독자: MySQL 디비 성능을 개선하고 싶은데 아직 관련 경험이 전무한 개발자 혹은 엔지니어

실습을 진행하는 필자의 환경은 다음과 같습니다:

* **OS:** MacOS 10.15.2
* **MySQL:** Ver 8.0.18
* **MySQL Workbench**

OS 관계 없이 MySQL과 Workbench만 있으면 됩니다. Workbench가 아닌 다른 툴을 사용해도 좋습니다.


## 1. 개요

5만 개의 데이터를 다루는 연산을 수행합니다. INSERT, SELECT에 대해 다룰 것이며 어떻게 성능을 올릴 수 있을지 고민하면서 직접 실습을 진행해보도록 합시다.

### 1.1 MySQL Workbench 설정

Preference > MySQL Session

* DBMS connection read timeout interval (in seconds): from 30 to 300
* DBMS connection timeout interval (in seconds): from 60 to 600

많은 데이터를 다루다 보면 요청이 다 처리되지 못한채 커넥션이 끊길 수 있기 때문에 위와 같이 설정해줍니다.



### 1.2 Create Schema

이번 실습에 사용할 스키마를 생성 해줍니다. 필자의 경우 `test_schema`라는 이름으로 생성해주었습니다.



## 2. 실습

직접 코드를 한 단위씩 실행하면서 처리되는 시간을 살펴보도록 합시다.

{{< gist aeharvlee ac075359456480601ade6efe8f973551 >}}

아래는 실행 결과를 도식화한 표입니다.

![result-of-performance-test](/images/2019-12-22-improve-mysql-performance/0.png)



## 3. 결론

**INSERT (UPDATE, DELETE의 경우에도 해당 될 겁니다.)**

* 디폴트 옵션(autocommit = 1)으로 5만 개의 데이터를 삽입할 때 걸리는 시간: 약 19초

* autocommit 옵션을 끄고 트랜잭션과 커밋으로 나누어 처리했을 때 시간: 약 2초

**적용: 많은 양의 데이터를 다룰 때는 반드시 트랜잭션과 커밋을 나누어 처리를 해야합니다.**



**SELECT**

* START TRANSACTION READ ONLY로 처리했을 경우, 성능 개선은 거의 없다.
* 그러나 읽기 트랜잭션 중 쓰기 트랜잭션을 방지한다는 면에서 필요할 때가 분명 있을 것이다.

**적용: 읽기 목적에 따라 트랜잭션을 적절히 사용하는 게 좋을 것 같습니다. 성능 향상은 크게 없었습니다.**

