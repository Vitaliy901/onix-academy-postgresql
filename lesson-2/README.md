### Report of queries
---
#### Create database

```postgresql
sudo psql -U intern -d market -h localhost

CREATE DATABASE db_lesson_2;

\q
```

#### Import database from dump

```postgresql
sudo -i -u postgres

cd /

pg_restore -d db_lesson_2 -U intern -h localhost -O -x --role=intern /mnt/HDD/projects/onix-academy-postgresql/lesson-2/hw_2.dump;

psql -U intern -d db_lesson_2 -h localhost

```

#### Queries

```postgresql
SELECT * FROM member_applications;
```

```postgresql
SELECT * FROM member_applications LIMIT 10;
```

```postgresql
SELECT * FROM member_applications ORDER BY dob ASC LIMIT 10;
```

```postgresql
SELECT * FROM member_applications WHERE mbi='E3FV7SGMDQJ';
```

```postgresql
SELECT * FROM member_applications ORDER BY dob ASC LIMIT 10 OFFSET 10;
```

```postgresql
SELECT * status FROM member_applications WHERE status ILIKE 'completed';
```

```postgresql
SELECT * status FROM member_applications WHERE status ILIKE 'completed';
```

```postgresql
SELECT title, first_name, phone FROM member_applications WHERE status ILIKE 'archived' OR status ILIKE 'rejected';
```

```postgresql
SELECT * FROM member_applications WHERE status NOT IN('ARCHIVED', 'DRAFT', 'REJECTED')
ORDER BY application_date DESC LIMIT 20;
```

```postgresql
SELECT doctor_id, COUNT(*) FROM member_applications WHERE doctor_id IS NOT NULL GROUP BY doctor_id ORDER BY count DESC LIMIT 3;
```

```postgresql
SELECT email AS gmail FROM member_applications WHERE email ILIKE '%gmail.com';
```

```postgresql
SELECT first_name, last_name, mbi FROM member_applications WHERE application_date >= '2021-04-05' AND signed_date IS NOT NULL;
```

```postgresql
SELECT id, CONCAT(first_name,' ',last_name) as full_name FROM member_applications WHERE physical_state IN('UT', 'NM', 'AZ');
```

```postgresql
SELECT physical_address1 FROM member_applications WHERE physical_zip IN('32766', '55359', '30642', '60158', '54229');
```

```postgresql
SELECT id, first_name FROM member_applications WHERE receive_notifications = false AND first_name LIKE '_%' ORDER BY first_name;

SELECT id, first_name FROM member_applications WHERE receive_notifications = false AND first_name LIKE '_%' ORDER BY email;

```

```postgresql
SELECT phone FROM member_applications WHERE phone LIKE '%123';
```

```postgresql
SELECT phone FROM member_applications WHERE phone LIKE '%333%';
```

```postgresql
SELECT first_name, last_name, phone FROM member_applications
WHERE (gender='M' AND physical_state='AZ') OR (gender='F' AND physical_state='NM');
```

```postgresql
SELECT DISTINCT first_name FROM member_applications;
```

```postgresql
SELECT user_id FROM member_applications WHERE deleted - created < INTERVAL '1 hour';
```