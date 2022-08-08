### Report of queries
---

#### Queries

1. Количество пользователей растёт, при логине поиск пользователя по имейлу работает медленно, нужно ускорить его с помощью индекса.

		CREATE INDEX email_idx ON users (email);

2. Из-за изменений в функционале, наличие нескольких адресов у пользователя больше не поддерживается.
Нужно с минимально возможным количеством запросов, и так чтобы ничего не сломалось, перенести адреса в таблицу users. Для этого должна использоваться следующая логика:
основным адресом пользователя считается тот, на который было отправлено наибольшее количество заказов.
При равенстве количества заказов выбирается тот, сумма заказов на который больше. При равенстве суммы, нужно выбирать последний использованный адрес.
Если заказов не было - последний добавленный.
Таблицу user_addresses, и все места где она использовалась необходимо удалить.


		CREATE VIEW user_address_view AS

		WITH temp AS (SELECT SUM(amount) AS amount, user_address_id FROM (SELECT SUM(p.price * op.quantity) AS amount, user_address_id FROM users AS u
		INNER JOIN orders AS o ON u.id = o.user_id
		INNER JOIN orders_products AS op ON o.id = op.order_id
		INNER JOIN products AS p ON op.product_id = p.id GROUP BY o.id) AS amount_price_address GROUP BY user_address_id)

		SELECT id, address, state, postcode, ua.user_id, temp.amount FROM (SELECT ua.id, ua.address, ua.state, ua.postcode, ua.user_id, (SELECT COUNT(user_id) FROM orders WHERE ua.id = user_address_id) AS orders_quantity FROM user_addresses AS ua) AS ua
		INNER JOIN (SELECT DISTINCT user_id, MAX(quantity) OVER (PARTITION BY user_id) AS orders_quantity FROM (SELECT DISTINCT user_id, COUNT(user_id) OVER (PARTITION BY user_address_id) AS quantity FROM orders) as uq) AS uq ON uq.user_id = ua.user_id AND uq.orders_quantity = ua.orders_quantity
		INNER JOIN temp ON temp.user_address_id = id;

		---

		CREATE MATERIALIZED VIEW final_user_address_mview AS
		WITH temp2 AS (SELECT uav.id, uav.address, uav.state, uav.postcode, uav.user_id, uav.amount, o.order_date FROM user_address_view AS uav
		INNER JOIN (SELECT DISTINCT user_id, MAX(amount) OVER (PARTITION BY user_id) AS amount FROM user_address_view) AS max ON max.user_id = uav.user_id AND max.amount = uav.amount
		INNER JOIN orders AS o ON o.user_address_id = uav.id)

		SELECT u.id, u.name, u.email, address, state, postcode FROM temp2
		INNER JOIN (SELECT DISTINCT user_id, MAX(order_date) OVER (PARTITION BY user_id) AS last_order_date FROM (SELECT uav.id, uav.address, uav.state, uav.postcode, uav.user_id, uav.amount, o.order_date FROM user_address_view AS uav 
		INNER JOIN (SELECT DISTINCT user_id, MAX(amount) OVER (PARTITION BY user_id) AS amount FROM user_address_view) AS max ON max.user_id = uav.user_id AND max.amount = uav.amount
		INNER JOIN orders AS o ON o.user_address_id = uav.id) AS uav) AS uav ON uav.user_id = temp2.user_id AND uav.last_order_date = temp2.order_date
		FULL JOIN users AS u ON u.id = temp2.user_id
		UNION
		SELECT id, name, email, address, state, postcode FROM users
		INNER JOIN (SELECT address, state, postcode, user_id FROM user_addresses
		WHERE id IN (SELECT DISTINCT MAX(id) OVER (PARTITION BY user_id) AS id FROM user_addresses
		WHERE user_id NOT IN (SELECT DISTINCT user_id FROM orders))) AS address ON address.user_id = users.id;

		---

		ALTER TABLE users ADD address varchar(255), ADD state varchar(20), ADD postcode varchar(5);

		WITH insert_table AS (SELECT fu1.id, fu1.name, fu1.email, fu2.address, fu2.state, fu2.postcode FROM (SELECT DISTINCT id, name, email FROM final_user_address_mview) AS fu1
		LEFT JOIN (SELECT id, address, state,postcode FROM final_user_address_mview WHERE address IS NOT NULL)AS fu2 ON fu2.id = fu1.id)

		UPDATE users SET id = new.id, name = new.name, address = new.address, state = new.state, postcode = new.postcode FROM insert_table AS new WHERE users.id = new.id;

		DROP TABLE user_addresses CASCADE;

		ALTER TABLE orders DROP COLUMN user_address_id;

3. В магазине появляются сотрудники (таблица employees) у сотрудника есть как минимум name, email, salary, role. Роли хранятся в отдельной таблице.
Также каждый из сотрудников отвечает за одну или несколько категорий товаров. Нужно создать по несколько записей в каждой таблице.

		CREATE TABLE employees (id bigserial PRIMARY KEY, name varchar(150) NOT NULL, email varchar(150) NOT NULL, salary int NOT NULL CHECK (salary >= 0), role_id int NOT NULL CHECK (role_id >= 0));

		CREATE TABLE roles (id serial PRIMARY KEY, name varchar(255) UNIQUE NOT NULL);

		CREATE TABLE employees_categories (employee_id int NOT NULL CHECK (employee_id >= 0) REFERENCES employees (id), category_id int NOT NULL CHECK (category_id >= 0) REFERENCES product_categories (id));

		INSERT INTO roles (name) VALUES ('administrator'), ('manager'), ('seller-consultant');

		INSERT INTO employees (name, email, salary, role_id)
		VALUES ('Oliver', 'oliver@email.com', 250, 3), ('Isabella', 'isabella@email.com', 270, 3), ('Henry', 'henry@email.com', 260, 3);

		INSERT INTO employees_categories VALUES (1,31), (1,32), (2,30), (3,29);

4. Оказалось, что при наполнении таблицы связывающей сотрудников, и категории товаров произошла ошибка. Очистите таблицу, сбросьте последовательность id, наполните таблицу заново.

		TRUNCATE employees_categories RESTART IDENTITY;
		
		BEGIN;
		INSERT INTO employees_categories VALUES (1,31), (1,32), (2,30), (3,29);
		COMMIT;
	