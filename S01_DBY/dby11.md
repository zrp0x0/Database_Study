# Stored Procedure

## stored procedure란
- 사용자가 정의한 프로시저
- DRBMS에 저장되고 사용되는 프로시저
- 구체적인 하나의 태스크(task)를 수행함

### 예제 1
두 정수의 곱셈 결과를 가져오는 프로시저를 작성

```sql
delimiter $$
create procedure product(in a int, in b int, out result int)
begin
    set result = a * b;
end
$$
delimiter ;
```
- in parameter(default): 
- out parameter: 받드시 명시해야함
- set

```sql
call product(5, 7, @result);
select @result;
```
- @result: 사용자가 정의한 변수

### 예제 2
두 정수를 맞바꾸는 프로시저를 작성

```sql
delimiter $$
create procedure swap(inout a int, inout b in)
begin
    set @temp = a;
    set a = b;
    set b = @teamp;
end
$$
delimiter ;
```
- in: 값을 파라미터로 받을 순 있지만 값을 바꿀 수 없음
- out: 반환값을 전달하려는 용도로 사용
- inout: 값을 전달할 수도 있고 반환값을 전달할 수도 있음

```sql
set @a = 5, @b = 7;
call swap(@a, @b);
select @a, @b; -- 7, 5
```

### 예제 3
각 부서별 평균 연봉을 가져오는 프로시저를 작성

```sql
delimiter $$
create procedure get_dept_avg_salary()
begin
    select dept_id, avg(salary)
    from employee
    group by dept_id;
end
$$
delimiter ;
```
- sql을 사용함으로써 in, out, inout 없이 결과를 반환할 수 있음

```sql
call get_dept_avg_salary();
```

### 예제 4
사용자가 프로필 닉네임을 바꾸면 이전 닉네임을 로그에 저장하고 새 닉네임으로 업데이트하는 프로시저를 작성

```sql
delimiter $$
create procedure change_nickname(user_id int, new_nick varchar(30))
begin
    insert into nickname_logs(
        select id, nickname, now() from users where id = user_id
    );
    update users set nickname = new_nick where id = user_id;
end
$$
delimiter ;
```
- 현재 유저 닉네임을 로그에 저장
- 새 닉네임으로 설정

```sql
call change_nickname(1, 'ZIDANE');
```

### stored procedure
- 이외에도 조건문을 통해 분기처리를 하거나
- 반복문을 수행하거나
- 에러를 핸들링하거나 에러를 일으키는 등의 다양한 로직을 정의할 수 있음

---

## stored procedure vs stored function
틀린 내용이 있을 수도 있음

### diff
- create 문법: create procedure / create function
- return 키워드로 값 반환: 불가능 / 가능
- 파라미터로 값(들) 반환: 가능 / 일부 가능
- 값을 꼭 반환해야 하나: 필수 아님 / 필수
- SQL statement: 불가능 / 가능
- transaction 사용: 가능 / 대부분 불가능
- 주된 사용 목적: business logic / computation

- 그 외
    - 다른 function/procedure를 호출할 수 있는지
    - resultset(= table)을 반환할 수 있는지
    - precompiled execution plan을 만드는지
    - try-catch를 사용할 수 있는지
