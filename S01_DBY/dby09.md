# group by, aggregate function, order by

## ORDER BY

### ORDER BY
- 조회 결과를 특성 attributes 기준으로 정렬하여 가져오고 싶을 때 사용함
- default 정렬 방식은 오름차순
- 오름차순 정렬은 ASC로 표기
- 내림차순 정렬은 DESC로 표기

### ORDER BY 예제
임직원들의 정보를 연봉 순서대로 정렬해서 알고 싶다

```sql
select * from employee order by salary;
```

높은 순서부터 보고 싶다
```sql
select * from employee order by salary desc;
```

부서별로 높은 연봉 순으로 보고 싶다
```sql
select * from employee order by dept_id asc, salary desc;
```
- null이 있다면 null로 먼저 정렬함
- 순서대로(dept_id, salary) 정렬을 해줌

### 추가사항
- 성능 주의사항
    - order by는 데이터베이스에 상당한 부하를 줄 수 있는 연산
    - 정렬하려는 컬럼에 인덱스가 걸려있지 않다면 메모리나 디스크를 사용해 직접 정렬해야하므로 쿼리 속도가 급격히 느려질 수 있음

---

## aggregate function

### aggregate function
- 여러 tuple들의 정보를 요약해서 하나의 값으로 추출하는 함수
- 대표적으로 count, sum, max, min, avg 함수가 있음
- 주로 관심있는 attr에 사용됨
    - avg(salary), max(birth_date)
- null은 제외함

### 예제
임직원 수를 알고 싶다
```sql
select count(*) from employee;
select count(position) from employee;
select count(dept_id) from employee;
```
- *: 여기서는 튜플의 수를 의미
- position의 중복을 포함해도 셈
- 만약 dept_id로 한다면? (이건 null이라는 가정하에)
    - null인 경우가 있다면 카운트하지 않음
- 그래서 *를 적는 것이 좋음

프로젝트 2002에 참여한 임직원 수와 최대 연봉과 최소 연봉과 평균 연봉을 알고 싶다
```sql
select count(*), max(e.salary), min(e.salary), avg(e.salary)
from works_on w join employee e
    on w.empl_id = e.id
where w.proj_id = 2002;
```

### 디테일
- count(*) vs count(attr) vs count(distinct attr)
    - count(*): null 여부와 상관없이 테이블의 전체 행 수를 셈
    - count(attr): 해당 컬럼에서 null을 제외한 값의 개수만 셈
    - count(distict attr): 해다 컬럼에서 null을 제외하고 중복도 제거한 고유한 값의 개수만 셈
        - 직급 종류 수 (select count(distinct position))

    - null도 sum으로 계산하고 싶다면 null을 0으로 바꾸어주어야함
        - avg(ifnull(salary, 0))


---

## group by

### 예제
```sql
select count(*), max(e.salary), min(e.salary), avg(e.salary)
from works_on w join employee e
    on w.empl_id = e.id
where w.proj_id = 2002;
```
- 특정 project에 한정에서가 아닌 프로젝트 별로 알고 싶다면?

```sql
select w.proj_id, count(*), max(e.salary), min(e.salary), avg(e.salary)
from works_on w join employee e
    on w.empl_id = e.id
group by w.proj_id;
```
- 그룹별로 aggregate function을 수행함!!

### 정리
- 관심있는 attrs 기준으로 그룹을 나눠서 그룹별로 aggregate function을 적용하고 싶을 때 사용
- grouping attrs: 그룹을 나누는 기준이 되는 attrs
- grouping attrs에 null 값이 있을 때는 null 값을 가지는 tuple끼리 묶임

### 디테일
- 원칙
    - select 절에는 group by에 명시된 컬럼 또는 집계함수만 올 수 있음

---

## Having

### 예제
프로젝트 **참여 인원이 7명 이상인** 프로젝트들에 대해서 각 프로젝트에 참여한 임직원 수와 최대 연봉과 최소 연봉과 평균 연봉을 알고 싶다
```sql
select w.proj_id, count(*), max(e.salary), min(e.salary), avg(e.salary)
from works_on w join employee e
    on w.empl_id = e.id
group by w.proj_id
having count(*) >= 7;
```
- Group by와 함께 사용함
- aggregate function의 결과값을 바탕으로 그룹을 필터링하고 싶을 때 사용함
- having절에 명시된 조건을 만족하는 그룹만 결과에 포함됨

### 디테일
- where: group by를 하기 전에 개별 튜플들을 필터링함
- having: group by를 한 후에 만들어진 그룹들을 필터링함
- 만약 having으로 필터링할 수 있는 조건 중, 굳이 집게함수가 필요없는 조건이라면 무조건 where 절로 옮겨야 미리 데이터를 걸러내어 group by가 할 일을 줄일 수 있음


---

## 예제

### 각 부서별 인원수를 인원 수가 많은 순서대로 정렬해서 알고 싶다
```sql
select dept_id, count(*) as empl_count
from employee
group by dept_id
order by impl_count DESC;
```
- dept_id로 그룹핑을 했기 때문에 dept_id로 select를 해주어야함
- null인 tuple도 묶어서 계산함

### 각 부서별 - 성별 인원수를 인원수가 많은 순서대로 정렬해서 알고 싶다
```sql
select dept_id, sex, count(*) as empl_count
from employee
group by dept_id, sex
order by empl_count desc;
```

### 회사 전체 평균 연봉보다 평균 연봉이 적은 부서들의 평균 연봉을 알고 싶다
```sql
select dept_id, avg(salary)
from employee
group by dept_id
having avg(salary) < (
    select avg(salary) from employee
);
```

### 각 프로젝트 별로 프로젝트에 참여한 90년대생들의 수와 이들의 평균 연봉을 알고 싶다
```sql
select w.proj_id, count(*), round(avg(salary), 0) 
from works_on w join employee e on w.empl_id = e.id
where e.birth_date between '1990-01-01' and '1999-12-31'
group by w.proj_id;
```
- 생일을 한정지어주고 나서 그룹핑
- rount(avg(salary), 0): 소수점이하 반올림

### 각 프로젝트 별로 정렬해서 프로젝트에 참여한 90년대생들의 수와 이들의 평균 연봉을 알고 싶다
```sql
select w.proj_id, count(*), round(avg(salary), 0) 
from works_on w join employee e on w.empl_id = e.id
where e.birth_date between '1990-01-01' and '1999-12-31'
group by w.proj_id
order by w.proj_id;
```

### 프로젝트 참여 인원이 7명 이상인 프로젝트에 한정해서 각 프로젝트별로 프로젝트에 참여한 90년대생들의 수와 이들의 평균 연봉을 알고 싶다
```sql
select w.proj_id, count(*), round(avg(salary), 0) 
from works_on w join employee e on w.empl_id = e.id
where e.birth_date between '1990-01-01' and '1999-12-31'
group by w.proj_id
having count(*) >= 7
order by w.proj_id;
```
- 이거는 잘못됨!!
- 현재 count(*)는 90년대 생의 수를 나타내는 것
    - 프로젝트 참여 인원 자체를 의미하는 것이 아님

```sql
select w.proj_id, count(*), round(avg(salary), 0) 
from works_on w join employee e on w.empl_id = e.id
where e.birth_date between '1990-01-01' and '1999-12-31'
    and w.proj_id in (
        select proj_id 
        from works_on
        group by proj_id
        having count(*) >= 7
    ) -- 프로젝트 참여 인원 수가 7명 이상인 사람 수
group by w.proj_id
order by w.proj_id;
```

---

## select 요약 & 실행 순서

### select 실행 순서
```sql
5. select attrs or aggregate functions
1. from tables
2. [where conditions]
3. [group by group attrs]
4. [having group conditions]
6. [order by attributes]
```
- select 쿼리에서 각 절(phrase)의 실행 순서는 개념적인 순서임
- select 쿼리의 실제 실행 순서는 각 RDBMS에서 어떻게 구현했는지에 따라 다름
    - Optimize를 해주는게 
    
- 디테일
    - 저 순서대로 작성되기 때문에 별칭을 사용할 수 있는 곳이 있고 없는 곳이 생김
    - 예를 들면 select에서 별칭을 만들면 where 절에서 사용할 수 없음
    - 위의 예로 order by는 select의 별칭을 사용할 수 있었음