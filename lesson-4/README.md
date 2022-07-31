### Report of queries
---

#### Queries

1. Добавить таблицу discounts, с полями id, name, type, и amount:

		CREATE TABLE discounts (id bigserial PRIMARY KEY, type varchar(100), amount int CHECK (amount >= 0));

2. Добавить связную таблицу users_discounts с полями: user_id, discount_id, expiration_date, записи должны быть уникальными по набору этих 3х полей:

		CREATE TABLE users_discounts 
		(user_id bigint UNIQUE NOT NULL CHECK (user_id >= 0) REFERENCES users (id), 
		discount_id bigint UNIQUE NOT NULL CHECK (discount_id >= 0) REFERENCES discounts (id), 
		expiration_date date UNIQUE);

3. Создать 3 скидки:
	- минус 20% на первый заказ, тип "one-time";
	- минус 5% на все заказы, тип "bulk";
	- минус 10% на все заказы, тип "bulk".
	
```postgresql
		INSERT INTO discounts (type,amount)
		VALUES ('one-time', 20),
		('bulk', 5),
		('bulk', 10);
```
4. Вывести список всех пользователей и количество их заказов (users.id, users.name, orders_quantity) c помощью джойна:

		SELECT u.id, u.name, COUNT(o.user_id) AS orders_quantity FROM users AS u
		LEFT JOIN orders AS o ON u.id = o.user_id GROUP BY u.id ORDER BY u.id ASC;

5. Вывести то же самое с помощью сабквери:

		SELECT u.id, u.name, (SELECT COUNT(orders.user_id) FROM orders WHERE u.id = orders.user_id) AS orders_quantity FROM users AS u;

6.С помощью джойна соединить результат предыдущего запроса с таблицей discounts, так чтобы получилось 2 колонки user_id, discount_id. Соединять по такому набору условий: 0 заказов - минус 20% на первый заказ, 1 заказ - минус 5% на все заказы.2 и больше заказов - минус 10% на все заказы:

		SELECT u.id AS user_id, d.id AS discount_id FROM discounts AS d
		LEFT JOIN (SELECT u.id, u.name, (SELECT COUNT(orders.user_id) FROM orders WHERE u.id = orders.user_id) AS orders_quantity FROM users AS u) AS u
		ON (u.orders_quantity = 0 AND d.amount = 20) OR (u.orders_quantity = 1 AND d.amount = 5) OR (u.orders_quantity >= 2 AND d.amount = 10);

7.Из полученного запроса создать view users_discounts_view:

		CREATE VIEW users_discounts_view AS
		SELECT u.id AS user_id, d.id AS discount_id FROM discounts AS d
		LEFT JOIN (SELECT u.id, u.name, (SELECT COUNT(orders.user_id) FROM orders WHERE u.id = orders.user_id) AS orders_quantity FROM users AS u) AS u
		ON (u.orders_quantity = 0 AND d.amount = 20) OR (u.orders_quantity = 1 AND d.amount = 5) OR (u.orders_quantity >= 2 AND d.amount = 10);

8.Результат выборки из view вставить в таблицу users_discounts:

		INSERT INTO users_discounts (user_id, discount_id)
		SELECT * FROM users_discounts_view;

9.С помощью джойна user_discounts_view и таблицы discounts вывести id пользователя, имя пользователя, название скидки.

		SELECT (SELECT id FROM users WHERE id = udv.user_id) AS id, (SELECT name FROM users WHERE id = udv.user_id) AS name, d.type AS discount FROM discounts AS d
		LEFT JOIN users_discounts_view AS udv ON udv.discount_id = d.id;

10.Создать таблицу invoices с колонками user_id, user_name, dicscount_name, order_date, order_total, discount_amount, total_after_discount.

		CREATE TABLE invoices (user_id serial PRIMARY KEY, user_name varchar(200) NOT NULL, dicscount_name varchar(100) NOT NULL, order_date date NOT NULL, order_total int NOT NULL, discount_amount int NOT NULL, total_after_discount int NOT NULL);

11.Вывести последний (самый новый) заказ каждого пользователя, в виде user_id, user_name, order_total, order_date
		
		CREATE VIEW last_user_orders_view AS
		SELECT u.user_id, u.user_name, order_history.order_total, u.order_date FROM
		(SELECT u.id AS user_id, u.name AS user_name, MAX(o.order_date) AS order_date FROM users AS u
		INNER JOIN orders AS o ON o.user_id = u.id GROUP BY u.id) AS u
		INNER JOIN (SELECT u.id AS user_id, SUM(p.price * op.quantity) AS order_total, o.order_date FROM users AS u
		INNER JOIN orders AS o ON u.id = o.user_id
		INNER JOIN orders_products AS op ON o.id = op.order_id
		INNER JOIN products AS p ON op.product_id = p.id GROUP BY u.id, o.order_date) AS order_history
		ON u.user_id = order_history.user_id AND u.order_date = order_history.order_date ORDER BY u.order_date DESC;

12.Сделать выборку джойном предыдущего запроса с users_discounts_view так, чтобы получились поля из пункта 10. Нужно посчитать тотал заказа с учётом той скидки, которая есть у пользователя.


		SELECT luov.user_id, luov. user_name, d.type AS discount_name, luov.order_date, luov.order_total, d.amount, (luov.order_total - luov.order_total / 100 * d.amount) AS total_after_discount FROM last_user_orders_view AS luov
		INNER JOIN users_discounts_view AS udv ON luov.user_id = udv.user_id
		INNER JOIN discounts AS d ON udv.discount_id = d.id;


13.Сохранить полученный результат в invoices.

		INSERT INTO invoices SELECT luov.user_id, luov. user_name, d.type AS discount_name, luov.order_date, luov.order_total, d.amount, (luov.order_total - luov.order_total / 100 * d.amount) AS total_after_discount FROM last_user_orders_view AS luov
		INNER JOIN users_discounts_view AS udv ON luov.user_id = udv.user_id
		INNER JOIN discounts AS d ON udv.discount_id = d.id;
