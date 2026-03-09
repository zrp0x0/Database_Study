# stored function

## stored function의 뜻과 예제

### stored function 뜻
- 사용자가 정의한 함수
- DBMS에 저장되고 사용되는 함수
- SQL의 select, insert, update, delete statement에서 사용할 수 있음

### stored function 예제 1
- 임직원의 ID를 열자리 정수로 램덤하게 발급하고 싶다
- ID의 맨 앞자리는 1로 고정이다

```sql
delimiter $$ -- delimiter를 $$로 바꿔야 나중에 ;를 사용해도 작성이 끝나지 않았다는 것을 알 수 있음
create function id_generator()
returns int
no sql
begin
    return (1000000000 + floor(rand() * 100000000));
end
$$
delimiter ; -- 다시 바꿔줌
```
- create function id_generator(): 만들고자 하는 함수 이름
- returns int: 반환하는 타입
- no sql: mysql에서만 사용하는 부분
- begin ~ end (body 부분): 동작해야하는 내용을 작성해주면 됨

### 실제 사용
```sql
insert into employee
values (id_generator(), 'JEHN', '1991-08-04', 'F', 'PO', 1000000000, 1005);
```

### stored function 예제 2
- 부서 ID를 파라미터로 받으면 해당 부서의 평균 연봉을 알려주는 함수를 작성하자

```sql
delimiter $$
create function dept_avg_salary(d_id int)
returns int
reads sql data
begin
    declare avg_sal int; -- 변수 선언
    select avg(salary) into avg_sal
        from employee
        where dept_id = d_id; -- 여기서 group을 사용하면 안되는 이유: into는 하나만 넣을 수 있음
    return avg_sal;

    또는
    
    select avg(salary) into @avg_sal -- 이렇게 변수를 사용하지 않고도 사용할 수 있음
        from employee
        where dept_id = d_id;
    return @avg_sal;
end
$$
delimiter ;
```

### 실제 사용
```sql
select *, dept_avg_salary(id)
from department;
```

### stored function 예제 3
- 졸업 요건 중 하나인 토익 800 이상을 충족했는지를 알려주는 함수를 작성하자

```sql
create function toeic_pass_fail(toeic_score_int)
returns char(4)
no sql
begin
    declare pass_fail char(4)
    if toeic_score is null then set pass_fail = 'fail'
    elseif toeic_score < 800 then set pass_fail = 'fail'
    else set pass_fail = 'pass'
    end if; -- 이거 반드시 해야함!!
    return pass_fail;
end
$$
delimiter ;
```

### 실제 사용
```sql
select *, toeic_pass_fail(toeic)
from student;
```

### stored function
- 이외에도 loop을 돌면서 반복적인 작업을 수행하거나
- case 키워드를 사용해서 값에 따라 분기 처리하거나
- 에러를 핸들링하거나 에러를 일으키는 등의 다양한 동작을 정의할 수 있음

### stored function 삭제하기
- drop function stored_function_name;

---

## 등록된 stored function 파악하기

### 회사에 가면 그 회사 DB에서 사용하고 있는 SF가 무엇이 있는지 파악
```sql
show function status where db = 'company';
```
- 만약 데이터 베이스를 명시해서 stored function을 만들고 싶다면?
    - create function db.function -- 선택할 데이터베이스
    - create function function -- 현재 선택된 데이터베이스

### 코드가 궁금하다면?
```sql
show create function id_generator;
``` 

---

## stored function은 언제 써야할까?

### Three-tier architecture
- presentation tier
    - 사용자에게 보여지는 부분을 담당하는 티어
    - HTML / JS / CSS / native app / desktop app

- logic tier
    - 서비스와 관련된 기능과 정책 등등 비즈니스 로직을 담당하는 티어
    - application tier, middle tier라고도 불림
    - Java-Spring

- data tier
    - 데이터를 저장하고 관리하고 제공하는 역할을 하는 티어
    - mysql, sql server

### 그래서 언제 써야할까?
- util 함수로 쓰기에는 괜찮을 것 같다
- 비즈니스 로직을 stored function에 두는 것은 좋지 않을 것 같음
    - 비즈니스 로직이 데이터 티어에 있는 것은 별로 좋지 않음

### 앞의 3개의 function
- dept_avg_salary는 비즈니스 로직을 가지지 않음
- id_generator는 비즈니스 로직이라기에 조금은 애매함
- toiec_pass_fail는 비즈니스 로직을 가짐 800이라는 점수 자체가 바뀔 수도

---

## 추가 정리 내용

### Stored Function 필수 옵션들 (Data Access Characteristics)
- No SQL: SQL문을 전혀 사용하지 않음
- READS SQL DATA: select 문으로 데이터를 읽기만 함
- MODIFIES SQL DATA: insert, update, delete 등으로 데이터를 수정함
- DETERMINISTIC: 동일한 파라미터를 넣으면 항상 같은 결과를 반환할 때 사용 (MySQL이 이 결과를 캐싱하여 성능을 높임)
- NOT DETERMINISTIC: 같은 파라미터를 넣어도 실행할 때마다 결과가 달라질 수 있음
