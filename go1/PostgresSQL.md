https://www.youtube.com/watch?v=i5-1HNf3W_Y - стоит посмотреть его, там реализация в Go

### Удаление базы данных

```PostgreSQL
DROP DATABASE testdb -- любую другую базу данных помимо testdb
```

---
### Создание базы данных

```PostgreSQL
CREATE DATABASE testdb
```

---
### Исключение дубликатов, DISTINCT

Простой пример, вот так выглядят `address` в нашей таблице `publisher`:

```PostgreSQL
SELECT address FROM publisher
```

![[Pasted image 20250315021557.png]]
Здесь присутствуют повторы, для того чтобы от них избавиться воспользуемся `DISTINCT` :

```PostgreSQL
SELECT DISTINCT address FROM publisher
```

![[Pasted image 20250315021611.png]]

---
### Оператор WHERE

Вот наша таблица:

![[Pasted image 20250315021622.png]]

А нам нужны данные только о самолётах `Boeing`, при том те, которые не оправляются из `London`, для этого воспользуемся `WHERE`:
	
```PostgreSQL
SELECT * FROM Trip
WHERE plane = 'Boeing' AND NOT town_from = 'London';
```

![[Pasted image 20250315021634.png]]

---
### Сортировка, оператор ORDER BY

- ASC - сортировка по возрастанию (по умолчанию)
- DESC - сортировка по убыванию

*Общая структура:*
```PostgreSQL
SELECT поля_таблиц FROM наименование_таблицы
WHERE ...
ORDER BY столбец_1 [ASC | DESC][, столбец_n [ASC | DESC]]
```

*Пример:*

```PostgreSQL
SELECT address FROM publisher 
ORDER BY address DESC
```

---
==*Далее база данных - БД/DB*==
### Как создавать таблицу в БД?

```PostgreSQL
CREATE TABLE publisher
(
	publisher_id integer PRIMARY KEY,
	org_name varchar(128) NOT NULL,
	address text NOT NULL
);

CREATE TABLE book
(
	book_id integer  PRIMARY KEY,
	title text NOT NULL,
	isbn varchar(32) NOT NULL
)
```

- *NOT NULL - для того чтобы пользователь не мог передать пустую строку*

---
### Вставить данные в таблицу (заполнение таблиц)

```PostgreSQL
INSERT INTO book
VALUES
(1,'The Dairy of a Young Girl', '0199535566'),
(2,'Pride and Prejudice', '9780307594006'),
(3, 'To Kill a Mockingbird', '0446310786'),
(4, 'The Book of Gutsy Women', '1501178414'),
(5, 'War and Pease', '1788886526');

INSERT INTO publisher 
VALUES
(1,'Everyman`s Library', 'NY'),
(2, 'Oxford University Press', 'NY'),
(3, 'Grand Central Publisher', 'Washington'),
(4, 'Simon & Schuter', 'Chicago');
```

### Извлечь данные из таблицы

```PostgreSQL
SELECT * FROM book
```

![[Pasted image 20250315021654.png]]

-  ==" * " - извлечение всех данных из таблицы== 

```PostgreSQL
SELECT * FROM publisher
```

![[Pasted image 20250315021705.png]]

---
### Отношение "один ко многим"

 У нас есть две таблицы, которые необходимо объединить, для этого используются ==внешние ключи==. Мы уже создавали первичные ключи(id), которые уникально определяют запись в таблице.

**Первичные ключи(id)** - уникально определяют запись в таблице

Cоздадим дополнительный столбец в ***book*** и скажем PostgreSQL, что это колонка будет ссылаться на ***publisher_id*** из таблице ***publisher***

```PostgreSQL
-- Добавление нового столбца fk_publisher_id в таблицу book
ALTER TABLE book
ADD COLUMN fk_publisher_id INT;

-- Добавление ограничения внешнего ключа для этого столбца
ALTER TABLE book
ADD CONSTRAINT fk_book_publisher FOREIGN KEY (fk_publisher_id) REFERENCES publisher (publisher_id);
```

- `fk` - обозначение внешнего ключа
- `fk_book_publisher` - наименование ограничений
- `FOREIGN KEY` - внешний ключ
- `REFERENCES` - то на что ссылается внешний ключ

**Добавление внешнего ключа:**

- `ADD CONSTRAINT`: Указывает имя ограничения (в данном случае `fk_book_publisher`).

- `FOREIGN KEY (fk_publisher_id) REFERENCES publisher (publisher_id)`:
    - Столбец `fk_publisher_id` в таблице `book` ссылается на столбец `publisher_id` в таблице `publisher`.

> [!ПРИМЕЧАНИЕ]
> Также сделать столбец `fk_publisher_id`  можно в момент создания таблицы и указать, что он будет ссылаться на `publisher_id` из `publisher`
> 
> ![[Pasted image 20250315021726.png]]
> 

###### Обновляем наши данные в таблице book

```PostgreSQL
UPDATE book
SET fk_publisher_id = 1
WHERE title = 'The Dairy of a Young Girl';

UPDATE book
SET fk_publisher_id = 1
WHERE title = 'Pride and Prejudice';

UPDATE book
SET fk_publisher_id = 2
WHERE title = 'To Kill a Mockingbird';

UPDATE book
SET fk_publisher_id = 2
WHERE title = 'The Book of Gutsy Women';

UPDATE book
SET fk_publisher_id = 2
WHERE title = 'War and Pease';
```

Наша таблица после изменений:

![[Pasted image 20250315021738.png]]

##### Объединяем наши таблицы

```PostgreSQL
SELECT 
    book.book_id,
    book.title,
    book.isbn,
    publisher.org_name AS org_name
FROM 
    book
INNER JOIN 
    publisher 
ON 
    book.fk_publisher_id = publisher.publisher_id;
```

![[Pasted image 20250315021750.png]]

> [!ОТНОШЕНИЯ]
> Помимо отношения "один ко многим" есть:
> - один к одному
> - многое к многим


> Вся информация бралась из данного видеоролика https://www.youtube.com/watch?v=HVQNxdI6fqY&list=PLBheEHDcG7-k1Y_Uy04Dj2ylWhcfSfqoF&index=1

