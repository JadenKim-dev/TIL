# 트랜잭션

트랜잭션이란 데이터베이스와의 소통 단위이다.  
`begin transaction` 구문을 통해 트랜잭션을 시작하고, `rollback` 또는 `commit` 구문으로 트랜잭션을 끝맺는다.

```sql
begin transaction;

-- 여러 쿼리문들

rollback;
```

트랜잭션 안에 포함된 쿼리문들은 임시 실행되며, commit 명령으로 트랜잭션을 끝낸 경우 쿼리문들의 실행 결과가 database에 반영된다.

만약 rollback 명령으로 트랜잭션을 끝낸다면 쿼리문들의 실행 내역은 제거되어 database에 반영되지 않게 된다.
