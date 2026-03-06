# NULL의 의미와 Three-Valued Logic

## NULL

### Null의 의미
- unknown: 모르는 경우
- unavailable or withheld: 공개하지 않음(사용할 수 없다)
- not applicable: 해당 사항이 없음
- null의 의미는 다양함

### 중요한 사실
- 생일이 null이라고 같다 다르다를 판단할 수 있을까?
- 없음 (단정지을 수 없음)

---

## Null을 찾는 비교 연산
```sql
select id 
from employee 
where birth_date = null;
```
- 아무것도 가져오지 않음
- null이 없다는 의미가 아님
- 같다 다르다 자체를 판단할 수 없음

```sql
select id
from employee
where birth_date is null -- is not
```
- is / is not으로 판단해야함
- 그냥 null인지 아닌지 (같다, 다르다가 아님)

---

## Null과 Three-Valued Logic

### 잘못된 예측
```sql
select *
from employee
where birth_date = '1990-03-09';
```
- birth_date이 null인 경우가 있다면 그 행은 false를 반환할까? NO!
- unknown으로 처리함 (알지 못함(true or false))

### Three-Valued Logic
- 비교/논리 연산의 결과로 True / False / Unknown을 가질 수 있음

---

## Null과 비교 연산

### 비교 연산 예제
- 1 = 1 | true
- 1 != 1 | false
- 1 = null | unknown
- 1 != null | unknown
- null = null | unknown
> Null과 비교 연산을 하면 unknown을 반환함

---

## Unknown의 논리 연산

### 예제
- and
    - True and Unknown: unknown
    - False and Unknown: false
    - unknown and unknown: unknown

- or
    - true or unknown: true
    - false or unknown: unknown
    - unknown or unknown: unknown

- not
    - not unknown: unknown

- unknown은 true 혹은 false일 수도 있다를 기억

---

## where절의 condition(s)

### where 절
- where 절은 condition의 결과가 true인 tuple만 선택됨
- 즉, 결과가 false이거나 unknown이 tuple은 선택되지 않음

---

## not in 사용시 주의사항

### 예시
- v not in (v1, v2, v3)는 아래와 같은 의미
- v != v1 and v != b2 and v != v3
- 만약 v1, v2, v3 중에 하나가 null이라면?
    - unknown이 반환되어서 선택되지 않음
    - 3 not in (1, 2, null) => unknown
    - 이렇게 되면 예상치 못한 동작이 발생함
    - 다르긴한데 가져오지 않음

```sql
select d.id, d.name
from department d
where d.id not in (
    select e.dept_id
    from employee e
    where e.birth_date >= '2000-01-01'
);
```
- 만약에 부서가 아직 배정받지 않은 null이 있을 경우
- unknown을 반환해서 아무것도 반환하지 않게됨

```sql
select d.id, d.name
from department d
where d.id not in (
    select e.dept_id
    from employee e
    where e.birth_date >= '2000-01-01'
        and e.dept_id is not null
);
```
- 해결 방안
    - not null constraint를 걸어주는 경우
    - e.dept_id is not null로 null은 가져오지 않도록 하기
    - **not exists로 바꿔서 처리하기**
        - 가장 많이 사용
        - 위의 예제에서 not exists로 바꾼다면 dept가 null은 가져오지 않으므로 null문제가 해결됨

--

## 추가 내용

### null 비교 연산자
- MySQL: <=> (NULL-safe equal operator)
- PostgreSQL: IS NOT DISTINCT FROM