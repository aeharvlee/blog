---
title: "How to renew entire table's rows safely?"
date: 2020-05-19
---

안전하게 테이블의 내용을 갱신하는 방법에 대해 공유드립니다.

![mysql_logo](/images/2020-05-19-renew-table-safely/0.png)

## 1. Introduction

이름은 동일하지만 매일 혹은 주기적으로 완전히 새로운 내용으로 갱신되어야 하는 MySQL 테이블이 있으신가요?

혹시 `TRUNCATE TABLE important_table` 를 한 뒤에 새로운 데이터를 삽입해주는 프로세스를 진행하고 있진 않으신가요? 

만약 그렇다면 `TRUNCATE` 이후 에러가 발생하면 사실상 빈 테이블이 되는 것이고 서비스 장애상태가 발생하게 되는데, 어떻게 하면 안전하게 테이블을 새로운 데이터로 깔끔하게 교체할 수 있을까요?

위와 같은 고민을 하고 계시다면 이 글이 도움이 되실겁니다!  :raised_hands:

본 글에서는 "어떻게 하면 안전하게 테이블을 새 데이터로 교체할 수 있을까?"에 대해 다룹니다. :slightly_smiling_face:

## 2. Body

### 2.1. 상황(문제) 설명

매일 정해진 시간에 테이블의 데이터를 갱신하는 Python 스크립트가 있습니다. 스크립트의 역할은 pandas 프레임워크로 방대한 양의 데이터를 처리한 뒤 테이블의 내용을 교체해주는 겁니다.

처음 필자가 사용했던 방법론은 아래와 같습니다.

1. `important_table`에 새롭게 삽입할 예정인 다량의 데이터를 pandas로 연산
2. `TRUNCATE TABLE important_table`
3. 1번에서 계산하여 도출한 다량의 데이터를 테이블에 삽입 `pandas_dataframe.to_sql(important_table, alchemy_engine, schema, if_exists='append', index=False)`

이 프로세스에는 치명적인 오류가 있는데요, "2번까지 성공하더라도 3번에서 실패하면 서비스는 장애상태가 된다."라는 점입니다.

실제로 여러 환경적 요인으로 3번과 같은 대량의 insert문은 간헐적으로 실패를 할 수 있습니다. **따라서 3번이 한큐에 성공한다는 보장은 정말 위험한 가정입니다.**

**"2번과 3번을 같은 Transaction에 묶고 완료가 된 이후 commit하면 안전하지 않나요?"** 라는 의문을 가지실 수 있는데, 아래의 문장을 읽어보시면 바로 이해가 되실 겁니다.

**truncate** is not  "transactional" in the sense that it commits and can't be rolled back,  and can modify object storage attributes. So it's not ordinary **DML** - Oracle classifies it as **DDL**. delete is an ordinary **DML** statement.

truncate 명령은 **DDL**(Data Description Language)로 분류되기 때문에 **DML**(Data Manipulation Language)처럼 트랜잭션처럼 처리할 수 없습니다. 롤백될 수도 없구요.

### 2.2. 해결 방법

제가 문제 해결을 위해 사용한 방법은 **"다른 테이블에 대량의 데이터를 insert하고, 성공했다면 기존 테이블과 교체"** 였습니다.

{{< gist aeharvlee f399dcb42b5ece1f429f15b6b68b26f3 >}}

이렇게 코드를 작성하면 실패했을 때 기존 테이블의 데이터가 소실될 염려가 없고 실패 하더라도 원본 테이블의 내용은 보존됩니다. 

## 3. Summary

Database 관련 연산을 다룰 때는 항상 **실행 도중 실패할 가능성을 염두해두는 것이 중요합니다.** 각 연산의 단계별로 "만약 이 단계에서 실패하면 어떤 일이 발생하지...?" 라는 의문을 가지고 코드를 작성한다면 훨씬 안전한 코드, 예상한 대로 동작하는 코드가 될 것 같습니다.
