```sql
show databases;

create database company;

select database();

use company;

select database();

-- drop database company;

create table department (
	id int primary key,
	name varchar(20) not null unique,
	leader_id int
);

create table employee (
	id int primary key,
	name varchar(30) not null,
	birth_date date,
	sex char(1) check(sex in ('M', 'F')),
	position varchar(10),
	salary int default 50000000,
	dept_id int,
	foreign key (dept_id) references department(id)
		on delete set null
		on update CASCADE,
	check (salary >= 50000000)
);

create table project(
	id int primary key,
	name varchar(20) not null unique,
	leader_id int,
	start_date date,
	end_date date,
	foreign key (leader_id) references employee(id)
		on delete set null 
		on update cascade,
	check(start_date < end_date)
);

create table works_on (
	empl_id int,
	proj_id int,
	primary key(empl_id, proj_id),
	foreign key(empl_id) references employee(id)
		on delete CASCADE
		on update CASCADE, 
	foreign key(proj_id) references project(id)
		on delete CASCADE
		on update CASCADE
);

alter table department add foreign key(leader_id)
references employee(id)
on update CASCADE
on delete SET NULL;

SHOW CREATE TABLE employee;

alter table employee drop check employee_chk_2;

ALTER TABLE employee
ADD CONSTRAINT employee_chk_2 CHECK (salary >= 50000000);
```