# 서브쿼리

서브쿼리는 하나의 쿼리 안에 또 다른 쿼리가 중첩된 형식으로 작성된 쿼리문이다.  
서브쿼리는 SELECT 절, FROM 절, WHERE 절의 총 세 위치에서 사용 가능하다.  
각 위치에 따라서 subquery가 수행하는 역할에 차이가 존재한다.

## SELECT 절에서 사용(스칼라 서브쿼리)

SELECT 절에서 서브쿼리를 사용하는 경우에는 스칼라 서브쿼리라고 부른다.  
주의할 점은 내부에서 중첩해서 사용하는 쿼리가 스칼라 값을 결과로 가져야 한다는 점, 즉 하나의 레코드만을 가져야 한다는 점이다.

스칼라 서브쿼리의 예시는 아래와 같다.  
각 성별에 해당하는 직원의 수를 구하는 서브쿼리문이다.

```sql
select (select count(*) from employee where gender = 0) 여성_직원_수,
       (select count(*) from employee where gender = 1) 남성_직원_수
```

쿼리의 결과는 다음과 같다.
![1](./images/1.png)
하나의 record에 모든 데이터가 저장되어 있다.  
여성 적원 수, 남성 직원 수에 대한 집계가 각 열에 입력된 형태이다.

이와 유사한 결과를 group by 문을 통해 얻을 수도 있다.

```sql
select case
    when gender = 0 then '여성'
    when gender = 1 then '남성'
end as '성별', count(*) '직원 수'
from EMPLOYEE
group by gender;
```

해당 쿼리의 결과는 아래와 같다.
![2](./images/2.png)
서브쿼리문의 결과와 차이점은 각 집계 결과가 서로 다른 record에 위치하게 된다는 점이다.  
`직원수`라는 동일한 열의 데이터이면서 서로 다른 레코드에 포함된다.
