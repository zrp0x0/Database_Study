# subquery

## 다양한 subquery 사용 예제

### ID가 14인 임직원보다 생일이 빠른 임직원의 ID, 이름, 생일을 알고 싶다
ID가 14인 임직원의 생일 구하기
```sql
select birth_date
from employee
where id = 14;
```
- 결과: '1992-08-04'

위에서 구한 생일보다 빠른 임직원의 id, name, birth_date 구하기
```sql
select id, name, birth_date
from employee
where birth_date < '1992-08-04';
```

위의 두 개의 쿼리를 한 번에 실행하고 싶다면
```sql
select id, name, birth_date
from employee
where birth_date < (
    select birth_date
    from employee
    where id = 14;
)
```
- subquery(nested query or inner query): select, insert, update, delete에 포함된 query
- outer query(main query): subquery를 포함하는 query
- subquery는 반드시 ( ) 안에 기술됨

### ID가 1인 임직원과 같은 부서 같은 성별인 임직원들의 ID와 이름과 직군을 알고 싶다

ID가 1인 임직원의 부서와 성별 구하기
```sql
select dept_id, sex
from employee
where id = 1;
```

위에서 구한 결과에 해당하는 (dept_id, sex)를 비교해서 구하기
```sql
select id, name, position
from employee
where (dept_id, sex) = ('DESIGN', 'MALE');
```

subquery를 사용하여 하나의 쿼리로 작성하기
```sql
select id, name, position
from employee
where (dept_id, sex) = (
    select dept_id, sex
    from employee
    where id = 1
);
```

---

## IN 사용 예제

### ID가 5인 임직원과 같은 프로젝트에 참여한 임직원들의 ID를 알고 싶다

ID가 5인 임직원이 참여한 프로젝트 찾기
```sql
select proj_id
from work_on 
where impl_id = 5;
```

위에서 나온 결과를 가지고 참여한 임직원들의 ID 찾기
```sql
select distinct impl_id
from work_on
where empl_id != 5 and (proj_id = 2001 or proj_id = 2002);
-- proj_id in (2001, 2002)
```
- distinct: 여러 개의 프로젝트에 참여한 사람이 있다면 해당 ID가 중복되서 나오기 때문에 중복을 제거해주어야함

위 두 개의 쿼리를 하나의 쿼리로 작성하면
```sql
select distinct empl_id
from works_on
where empl_id != 5 and proj_id in (
    select proj_id
    from works_on
    where empl_id = 5
);
```
- v in (v1, v2, v3, ...): 하나라도 포함되면 True를 반환
- v not in (v1, v2, v3, ...): 하나라도 포함되지 않으면 True를 반환
- (v1, v2, v3, ...): 명시적인 값의 집합일 수도 있고, subquery의 결과(set or multiset(중복허용))일 수도 있음

### unqualified attribute
명시적으로 어떤 테이블에 포함되는지 명시하지 않은 attr들은 사용된 query를 포함하여 그 query의 바깥쪽으로 존재하는 모든 queries 중에 해당 attr 이름을 가지는 가장 가까이에 있는 table을 참조함

### ID가 5인 임직원과 같은 프로젝트에 참여한 임직원들의 ID와 이름을 알고 싶다

이름을 알고 싶다면 works_on에서 찾은 id를 가지고 employee의 name에 접근해야함
```sql
select distinct e.id, e.name
from employee e, works_on w
where e.id = w.empl_id -- 조인 조건
    and w.empl_id != 5 
    and w.proj_id in (
        select proj_id
        from works_on
        where empl_id = 5
    );
```
- distinct는 (e.id, e.name)의 중복을 판단함

```sql
select id, name
from employee
where id in (
    select distinct empl_id
    from works_on
    where empl_id != 5 and proj_id in (
        select proj_id
        from works_on
        where empl_id = 5
    )
);
```

**from 위치에 subquery를 넣기**
```sql
select id, name
from employee, (
    select distinct empl_id
    from works_on
    where empl_id != 5 and proj_id in (
        select proj_id
        from works_on
        where empl_id = 5
    )
) as dstnct_e -- 가상의 테이블
where id = dstnct_e.empl_id;
```
- 중요한 사실 from에서 두 개의 테이블을 작성하는 건 카르테시안 곱을 하겠다는 의미
- 근데 where id = dstnct_e.empl_id처럼 조인(곱)의 조건을 명시하는 것이 맞음
- 왜냐하면 id랑 empl_id가 달라고 곱을 할 수도 있기 때문임

---

## EXISTS 사용 예제

### ID가 7 또는 12인 임직원이 참여한 프로젝트의 ID와 이름을 알고 싶다

ID가 7 또는 12인 임직원이 참여한 프로젝트 ID 구하기
```sql
select distinct proj_id
from works_on
where empl_id in (7, 12);
```

프로젝트 id를 기반해서 project 이름 구하기
```sql
select id, name
from project
where id in (
    select distinct proj_id
    from works_on
    where empl_id in (7, 12)
);
```

```sql
select distinct p.id, p.name
from project p, works_on w
where p.id = w.proj_id
    and w.impl_id in (7, 12)
```

EXIST를 사용한 쿼리
```sql
select p.id, p.name
from project p
where exists (
    select *
    from works_on w
    where w.proj_id = p.id
        and w.empl_id in (7, 12)
);
```
- 상관 조인
- project에서 튜플을 순차적으로 하나씩 읽음(선택)
- 이때 이걸 가지고 w.proj_id = p.id이고 w.empl_id in (7, 12)가 하나라도 있다면 exist가 true를 반환
- 선택된 튜플이 이제 진짜 선택됨
- 이하 반복

### 정리
- EXISTS: subquery의 결과가 최소 하나의 row라도 있다면 true를 반환
- NOT EXISTS: subquery의 결과가 단 하나의 row도 없다면 true를 반환
- correlated query: subquery가 바깥쪽 query의 attr을 참조할 때, 연관 쿼리라고 부름

```sql
select p.id, p.name
from project p
where p.id in (
    select w.proj_id
    from works_on w
    where w.empl_id in (7, 12)
);
```
- in과 exist는 바꿔가면서 사용할 수 있음

---

## NOT EXISTS 사용 예제

### 2000년대생이 없는 부서의 ID와 이름을 알고 싶다

```sql
select d.id, d.name
from department
where id not in (
    select distinct dept_id
    from employee
    where birth_date >= '2002-01-01'
        and dept_id is not null;
)
```

```sql
select d.id, d.name
from department d
where not exists (
    select *
    from employee e
    where d.id = e.dept_id
        and e.birth_date >= '2000-01-01'
);
```

---

## ANY 사용 예제

### 리더보다 높은 연봉을 받는 부서원을 가진 리더의 ID와 이름과 연봉을 알고 싶다

```sql
select L.id, L.name, L.salary
from employee L
where L.id in (
    select leader_id
    from department
)
    and exists (
        select *
        from employee M
        where M.dept_id = L.dept_id
            and M.salary > L.salary
    );
```

ANY 사용하기
```sql
select e.id, e.name, e.salary
from department d, employee e
where d.leader_id = e.id
    and e.salary < any (
        select salary
        from employee
        where id <> d.leader_id
            and dept_id = e.dept_id
    );
```
- any: 단 하나라도 경우가 있다면 true
- v comparison_operator any (subquery): subquery가 반환한 결과들 중에 단 하나라도 v와의 비교 연산이 true라면 true를 반환함 ( == some)

### 해당 부서의 최고 연봉도 같이 보고 싶다면
```sql
select e.id, e.name, e.salary, (
    select max(salary)
    from employee
    where dept_id = d.id
) as dept_max_salary
from department d, employee e
where d.leader_id = e.id and e.salary < any (
    select salary
    from employee
    where id <> d.leader_id and dept_id = e.dept_id
)
```

---

## All 사용 예제

### ID가 13인 임직원과 한 번도 같은 프로젝트에 참여하지 못한 임직원들의 ID, 이름, 직군을 알고 싶다

```sql
select distinct e.id, e.name, e.position
from employee e, works_on w
where e.id = w.empl_id 
    and w.proj_id <> all (
        select proj_id
        from works_on
        where empl_id = 13
    );
```
- v comparison_operator all (subquery): subquery가 반환한 결과들과 v와의 비교 연산이 모두 true라면 true를 

```sql
select e.id, e.name, e.position
from employee e
where e.id != 13
    and not exists (
        select *
        from works_on w1, works_on w2
        where w1.proj_id = w2.proj_id
            and w1.empl_id = e.id
            and w2.empl_id = 13
    ) 
```

---

## 추가사항

### 성능 비교: IN vs EXISTS
- 최근에는 IN과 EXISTS의 성능차이가 거의 없는 것으로 알면 됨

### 또한 join condition
- w1, w2는 원래의 테이블의 근복적인 위치를 가지고 있음
