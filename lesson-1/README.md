### Report of queries
---
#### Creating a user
```postgresql
- sudo -u postgres createuser intern -d -r -P;
- sudo psql -U intern -d postgres -h localhost; *password:123*
```
#### Creating a database
```postgresql
- create database market;
```
#### Creating a table of users
```postgresql
- create table users (id bigserial PRIMARY KEY,
	name varchar(100),
	email varchar(100) UNIQUE not null,
	password varchar(200),
	birth_date date not null,
	created_at timestamp not null);
```
#### Adding users to table
```postgresql
- insert into users (name, email,password, birth_date, created_at)
	values ('Liam', 'liam@gmail.com', '48g4958j4ofw', '2000-12-04', date_trunc('seconds', now()::timestamp));
```
#### Creating a table of products with relations and owner
```postgresql
- create table owners (id serial PRIMARY KEY, name varchar(100), phone varchar(15) UNIQUE not null, created_at timestamp not null);
- create table products (id bigserial PRIMARY KEY,
	name varchar(255) not null,
	description text not null,
	price numeric(7,2) not null,
	created_at timestamp not null,
	owner_id bigserial not null CHECK (owner_id > 0) REFERENCES owners (id) ON DELETE CASCADE);
- create table orders (id bigserial PRIMARY KEY,
	user_id bigint not null CHECK (user_id > 0) REFERENCES users (id),
	created_at timestamp not null);
- create table orders_products (order_id bigint not null CHECK (order_id > 0) REFERENCES orders (id),
	product_id bigint not null CHECK (product_id > 0) REFERENCES products (id) ON DELETE CASCADE,
	created_at timestamp not null);
- insert into owners (name, phone, created_at)
	VALUES ('Ashlay', '(050) 234-23-46', date_trunc('seconds', now()::timestamp)),
	('Carl', '(063) 642-23-46', date_trunc('seconds', now()::timestamp));
```
#### Adding a few products
```postgresql
- insert into products (name,description,price,created_at, owner_id)
	values ('Iphone','some description', 439.99,date_trunc('seconds', now()::timestamp),1),
	('Mac mini','some description', 759.99,date_trunc('seconds', now()::timestamp),2),
	('Apple watch','some description', 59.99,date_trunc('seconds', now()::timestamp),2),
	('Apple iPhone XS','some description', 129.99,date_trunc('seconds', now()::timestamp),1);
- insert into orders (user_id,Created_at)
	values (2,date_trunc('seconds', now()::timestamp)),
	(3,date_trunc('seconds', now()::timestamp));
- insert into orders_products (order_id,product_id,created_at)
	values (1,1,date_trunc('seconds', now()::timestamp)),
	(1,1,date_trunc('seconds', now()::timestamp)),
	(1,2,date_trunc('seconds', now()::timestamp)),
	(2,3,date_trunc('seconds', now()::timestamp)),
	(2,4,date_trunc('seconds', now()::timestamp));
```
#### Deleting an owner
```postgresql
- delete from owners where id=2;
```
#### Adding an age restriction
```postgresql
- delete from users where name='Liam';
- alter table users add check (date_part('year', age(birth_date)) >= 18);
```
#### Adding an owner
```postgresql
- insert into owners (name, phone, created_at)
	VALUES ('Carl', '(063) 642-23-46', date_trunc('seconds', now()::timestamp));
```
#### Adding a few products
```postgresql
- insert into products (name,description,price,created_at, owner_id)
	values ('Iphone','some description', 439.99,date_trunc('seconds', now()::timestamp),5),
	('Apple iPhone XS','some description', 129.99,date_trunc('seconds', now()::timestamp),5);
```
#### Change of product owners
```postgresql
- update products set owner_id = CASE WHEN owner_id = 1 THEN 5 ELSE 1 END;
```