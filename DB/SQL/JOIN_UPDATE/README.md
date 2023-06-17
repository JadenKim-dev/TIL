# JOIN UPDATE

보통의 update 문은 아래와 같은 형식을 가지고 있다

```sql
Update EMPLOYEE set department = '경영지원' where name = '박지현'
```

앞 부분에 수정 내용을 적고, where 문을 이용해서 어떤 데이터에 대한 수정/삭제가 이루어질지 제한한다.  
위의 형식은 set 절과 where 조건에 사용할 row가 수정할 테이블 내에 존재할 때 사용할 수 있다.  
(위 예시에서는 name 열이 EMPLOYEE 테이블에 포함되어 있으므로 사용이 가능했다.)

만약 set 절이나 where 문 작성 과정에서 다른 테이블의 데이터를 이용해야 한다면 join update를 사용해야 한다.

## Set 절에서 다른 테이블의 데이터를 이용하는 경우

set 절에서 join 데이터를 사용하는 예시는 다음과 같이 작성할 수 있다.
특정 직원의 RENT_LOG 개수를 세서 EMPLOYEE 테이블에 개수를 저장하는 쿼리문이다.

```sql
update EMPLOYEE set rent_num = b.cnt
from EMPLOYEE e
join (select employee_id, count(*) cnt  from RENT_LOG group by employee_id) b
on e.employee_id = b.id;
```

서브쿼리를 이용하여 각 직원의 렌트 수를 구하고, 이 결과와 join을 수행하여 employee 테이블의 rent_num 값을 변경했다.

## Where 문에서 다른 테이블의 데이터를 이용하는 경우

Where 문에서 다른 테이블의 데이터를 이용하는 예시는 아래와 같다.
'김가람' 직원의 렌트 로그에 대해서 렌트 비용을 1000원 증액하는 쿼리이다.

```sql
update RENT_LOG set price = price + 1000
from RENT_LOG R join EMPLOYEE E on R.employee_id = E.id;
where E.name = '김가람'
```

EMPLOYEE 테이블에 존재하는 name 필드의 데이터로 where 조건을 걸고, 해당 조건에 맞는 RENT_LOG의 price 값을 수정했다.

다른 테이블의 정보를 이용하여 join delete를 수행해야 하는 경우에도 위와 동일한 방식으로 쿼리를 작성할 수 있다.
