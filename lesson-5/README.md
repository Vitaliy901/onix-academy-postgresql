### Report of queries

---

#### Queries

1.  Количество пользователей растёт, при логине поиск пользователя по имейлу работает медленно, нужно ускорить его с помощью индекса.

        CREATE INDEX email_idx ON users (email);

2.  Из-за изменений в функционале, наличие нескольких адресов у пользователя больше не поддерживается.
    Нужно с минимально возможным количеством запросов, и так чтобы ничего не сломалось, перенести адреса в таблицу users. Для этого должна использоваться следующая логика:
    основным адресом пользователя считается тот, на который было отправлено наибольшее количество заказов.
    При равенстве количества заказов выбирается тот, сумма заказов на который больше. При равенстве суммы, нужно выбирать последний использованный адрес.
    Если заказов не было - последний добавленный.
    Таблицу user_addresses, и все места где она использовалась необходимо удалить.

        ALTER TABLE users ADD address varchar(255), ADD state varchar(20), ADD postcode varchar(5);

        ---

        CREATE VIEW user_addresses_view AS
        SELECT DISTINCT ON (user_id) user_id, users.id, users.name, users.email, addresses.address, addresses.state, addresses.postcode FROM (SELECT ua.user_id, ua.address, ua.state, ua.postcode, COUNT(orders.id) AS orders_count, SUM(products.price * orders_products.quantity) total_price FROM user_addresses AS ua
        FULL JOIN orders ON orders.user_address_id = ua.id
        LEFT JOIN orders_products ON orders.id = orders_products.order_id
        LEFT JOIN products ON orders_products.product_id = products.id GROUP BY orders.user_address_id, ua.id ORDER BY orders_count DESC, total_price DESC, ua.id) AS addresses

        RIGHT JOIN users ON users.id = user_id ORDER BY user_id,total_price DESC;

        ---

        UPDATE users SET id = new.id, name = new.name, address = new.address, state = new.state, postcode = new.postcode FROM user_addresses_view AS new WHERE users.id = new.id;

        DROP TABLE user_addresses CASCADE;

        ALTER TABLE orders DROP COLUMN user_address_id;

3.  В магазине появляются сотрудники (таблица employees) у сотрудника есть как минимум name, email, salary, role. Роли хранятся в отдельной таблице.
    Также каждый из сотрудников отвечает за одну или несколько категорий товаров. Нужно создать по несколько записей в каждой таблице.

        CREATE TABLE employees (id bigserial PRIMARY KEY, name varchar(150) NOT NULL, email varchar(150) NOT NULL, salary int NOT NULL CHECK (salary >= 0), role_id int NOT NULL CHECK (role_id >= 0));

        CREATE TABLE roles (id serial PRIMARY KEY, name varchar(255) UNIQUE NOT NULL);

        CREATE TABLE employees_categories (employee_id int NOT NULL CHECK (employee_id >= 0) REFERENCES employees (id), category_id int NOT NULL CHECK (category_id >= 0) REFERENCES product_categories (id));

        INSERT INTO roles (name) VALUES ('administrator'), ('manager'), ('seller-consultant');

        INSERT INTO employees (name, email, salary, role_id)
        VALUES ('Oliver', 'oliver@email.com', 250, 3), ('Isabella', 'isabella@email.com', 270, 3), ('Henry', 'henry@email.com', 260, 3);

        INSERT INTO employees_categories VALUES (1,29), (1,30), (2,33), (3,32);

4.  Оказалось, что при наполнении таблицы связывающей сотрудников, и категории товаров произошла ошибка. Очистите таблицу, сбросьте последовательность id, наполните таблицу заново.

TRUNCATE employees_categories RESTART IDENTITY;

BEGIN;
INSERT INTO employees_categories VALUES (1,29), (1,30), (2,31), (2,33), (3,32);
COMMIT;
