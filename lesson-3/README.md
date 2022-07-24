### Report of queries
---
#### Create database

```postgresql
sudo psql -U intern -d market -h localhost

CREATE DATABASE db_lesson_3;

\q
```

#### Import database from dump

```postgresql
sudo -i -u postgres

cd /

pg_restore -d db_lesson_3 -U intern -h localhost -O -x --role=intern /mnt/HDD/projects/onix-academy-postgresql/lesson-3/hw_3.dump;

psql -U intern -d db_lesson_3 -h localhost
```
#### Queries

1. удалить таблицы user_data, user_discounts, discounts;

		DROP TABLE user_data, user_discounts, discounts;

2. очистить все таблицы;

		DELETE FROM product_categories;

		DELETE FROM user_addresses;

		DELETE FROM users;

3. добавить несколько пользователей;

		INSERT INTO users (name, email)
		VALUES ('Liam', lower('Liam@email.com')),
		('Ashlay', lower('Ashlay@email.com')),
		('Carl', lower('carl@email.com'));

4. добавить им минимум по одному адресу для доставки (у пользователя может быть несколько адресов);

		ALTER TABLE user_addresses DROP CONSTRAINT user_id_fk;

		ALTER TABLE user_addresses 
		ADD CONSTRAINT user_id_fk FOREIGN KEY (user_id)
		REFERENCES users (id);

		INSERT INTO user_addresses (address, state, postcode, user_id)
		VALUES ('Viktora Chmilenko, 24b', 'KR', '25001', 5),
		('Vokzalna, 28, entrance №3', 'KR', '25001', 5),
		('Korolenko, 32 a,', 'KR', '25006', 6),
		('Sadova Street, 43,', 'KR', '25001', 7)
		('Shevchenko, 16b,', 'KR', '25005', 7);

5. добавить несколько категорий для товаров;

		INSERT INTO product_categories (name, parent_id)
		VALUES ('Home appliances', null),
		('PC and laptops', null);

		INSERT INTO product_categories (name, parent_id)
		VALUES ('Fridges', 27),
		('Washing machines', 27),
		('Processors', 28),
		('SSD', 28),
		('Laptops', 28);

		INSERT INTO product_categories (name, parent_id)
		VALUES ('Bosch Fridges', 29),
		('LG Fridges', 29),
		('Bosch Washing machines', 30),
		('LG Washing machines', 30),
		('Intel', 31),
		('AMD', 31),
		('Apple', 33),
		('Dell', 33),
		('Kingston', 32),
		('Samsung', 32);

6. добавить несколько товаров;

		INSERT INTO products (name,price,description, category_id)
		VALUES ('Bosch H34', 10599,'some fridge',34),
		('LG fd-345', 9349,'some fridge',35),
		('Bosch ds-300', 13499, 'some washing machine', 36),
		('LG n-200', 24320, 'some washing machine', 37),
		('Apple MacBook Air 13', 50499, 'some laptop', 40),
		('Dell Inspiron-7633', 43033, 'some laptop', 41),
		('Kingston A-400', 1230, 'some SSD', 42),
		('Samsung 870 Evo', 3023, 'some SSD', 43),
		('Intel core I-5', 4555, 'some processor', 38),
		('AMD-8660', 3235, 'some processor', 39);

7. связать таблицы orders и products с помощью таблицы orders_products (в заказе может быть множество товаров, и каждый товар может быть во многих заказах). В этой же таблице нужно хранить количество единиц товара в заказе.

		CREATE TABLE orders_products (order_id bigint NOT NULL,
		product_id bigint NOT NULL, 
		quantity smallint NOT NULL DEFAULT 1 CHECK (quantity > 0), 
		UNIQUE (order_id, product_id),
		FOREIGN KEY (order_id) REFERENCES orders (id),
		FOREIGN KEY (product_id) REFERENCES products (id));

8. создать несколько заказов

		ALTER TABLE orders ADD CONSTRAINT user_id_fk FOREIGN KEY (user_id) REFERENCES users (id) ON DELETE CASCADE;

		ALTER TABLE orders ADD CONSTRAINT user_addressid_id_fk FOREIGN KEY (user_address_id) REFERENCES user_addresses (id);

		INSERT INTO orders (user_id, order_date, user_address_id)
		VALUES (5, NOW(), 3),
		(6, NOW(), 4);
		
		INSERT INTO orders_products (order_id,product_id, quantity)
		VALUES (1,24,1),(1,25,1),(2,28,2),(2,29,1);

9. вывести список всех адресов. В каждом адресе должно быть указано имя пользователя, которому принадлежит адрес.

		SELECT ua.address, name FROM user_addresses AS ua
		INNER JOIN users ON ua.user_id = users.id;

10. вывести список всех адресов одного из пользователей. В каждом адресе должно быть указано имя пользователя, которому принадлежит адрес.

		SELECT ua.address, name FROM user_addresses AS ua
		LEFT JOIN users ON ua.user_id = users.id
		WHERE users.id = 7;

11. вывести названия всех продуктов с указанием названия категории которй принадлежит продукт.

		SELECT p.name AS product, pc.name AS category FROM products AS p
		INNER JOIN product_categories AS pc ON p.category_id = pc.id;

12. вывести названия всех категорий (даже тех, в которых нет ни одного продукта), и названия продуктов, которые в них находятся.

		SELECT pc.name AS category, pc2.name AS subcategory_1, pc3.name AS subcategory_2, p.name AS product 
		FROM product_categories AS pc
		INNER JOIN product_categories AS pc2 ON pc.id = pc2.parent_id
		INNER JOIN product_categories AS pc3 ON pc2.id = pc3.parent_id
		LEFT JOIN products AS p ON p.category_id = pc3.id;

13. главная страница с категориями: вывести названия всех категорий в которых нет продуктов.

		SELECT pc.name AS category, pc2.name AS subcategory_1, pc3.name AS subcategory_2 FROM product_categories AS pc
		INNER JOIN product_categories AS pc2 ON pc.id = pc2.parent_id
		INNER JOIN product_categories AS pc3 ON pc2.id = pc3.parent_id
		LEFT JOIN products AS p ON p.category_id = pc3.id WHERE p.name IS NULL;

14. страница одной из категорий товаров: вывести название родителя категории с названием %name%.

		SELECT pc.name FROM product_categories AS pc
		INNER JOIN product_categories AS pc2 ON pc.id = pc2.parent_id WHERE pc2.id = 42;

15. просмотр страницы товара: вывести название товара, название его категории, и название родительской категории для товара с id %id%. Колонка с категорией должна при этом называться category_name, с родительской категорией - parent_category_name.

		SELECT p.name AS product, pc.name AS category_name, pc2.name AS parent_category_name FROM products AS p
		INNER JOIN product_categories AS pc ON p.category_id = pc.id
		INNER JOIN product_categories AS pc2 ON pc.parent_id = pc2.id;

16. смотрим статистику заказов: вывести id заказа, и имя пользователя для заказов сделанных после %date%.

		SELECT o.id, u.name FROM orders AS o
		INNER JOIN users AS u ON o.user_id = u.id WHERE o.order_date > '2022-07-23';

17. просматриваем сделанный заказ: вывести список записей вида: название товара, цена, количество из заказа с id %id%

		SELECT p.name, p.price, op.quantity, o.id FROM orders AS o 
		INNER JOIN orders_products AS op ON o.id = op.order_id
		INNER JOIN products AS p ON op.product_id = p.id;

18. история заказов: вывести список записей вида: имя пользователя, id заказа, дата заказа, тотал - сумма стоимости всех товаров в заказе (нужно учитывать, что в заказе бывает несколько единиц одного товара)

		SELECT u.name, o.id, o.order_date, SUM(p.price * op.quantity) AS amaunt FROM users AS u
		INNER JOIN orders AS o ON u.id = o.user_id
		INNER JOIN orders_products AS op ON o.id = op.order_id
		INNER JOIN products AS p ON op.product_id = p.id GROUP BY u.name, o.id, o.order_date ORDER BY o.order_date DESC;

19. считаем деньги: вывести список записей вида: имя пользователя, сумма всех заказов пользователя.

		SELECT u.name, SUM(p.price * op.quantity) AS amaunt FROM users AS u
		INNER JOIN orders AS o ON u.id = o.user_id
		INNER JOIN orders_products AS op ON o.id = op.order_id
		INNER JOIN products AS p ON op.product_id = p.id GROUP BY u.name;
