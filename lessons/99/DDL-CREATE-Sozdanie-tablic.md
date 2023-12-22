![](../../common_files/header.png)

*назад на [курс](../../course.md) / [roadmap](../../roadmap.md)*

***

   

DDL. CREATE. Создание таблиц
============================

В рамках сегодняшнего урока разберем синтаксис создания таблиц в SQL, который уже несколько раз использовали.

В дальнейших уроках также будем возвращаться к созданию таблиц, но уже в контексте расширения базовой функциональности и применения новых изучаемых концепций и конструкций.

Итак, базовый синтаксис. За основу возьмем уже хорошо знакомую нам таблицу passenger и запрос, с помощью которого ее создали:

```java
create table passenger (
  id                  bigserial,
  first_name          varchar(100),
  last_name           varchar(100),
  birth_date          date,
  male                boolean         default true, 
  last_purchase       timestamp,
  favorite_airports   text[]
);
```

Полагаю, синтаксис достаточно очевиден, но есть моменты, которые стоит подсветить.

Оператор **_CREATE_** является одним из основополагающих в DDL и позволяет создать какой-либо элемент структуры в рамках СУБД. Какой элемент требуется создать - указывается следующим (если опустить необязательные модификаторы) оператором.

Мы встречали как минимум три структурных элемента, создаваемых через _CREATE_:

1.  _DATABASE_;
2.  _SCHEMA_;
3.  _TABLE_.

Есть и другие, с рядом из них мы познакомимся в следующих уроках.

После этого указывается имя элемента. В нашем случае - таблицы.

> Как и в других языках программирования, в SQL есть зарезервированные слова. И, что логично, эти слова нельзя использовать в качестве названий таблиц, колонок, алиасов и прочих имен. Но иногда очень хочется (чаще всего - при создании таблицы user, а это слово - тоже зарезервировано в SQL). Стоит ли так делать - вопрос не однозначный, но обычно - нет.  
> В любом случае, возможность использовать ключевое слово для названия есть - с поправкой на СУБД, обычно она заключается в необходимости заключить такое слово в кавычки (в PostgreSQL - двойные, для других СУБД - уточняйте в документации).  
> То есть: **create table user (...);** - ошибка;  
> **create table "user" (...);** - все ок:)

Далее внутри круглых скобок перечисляются колонки, которые нужны данной таблице, с указанием типа данных и других атрибутов колонки.

В целом, кроме колонок, в этом блоке может быть много всего полезного, но об этом поговорим в дальнейших уроках.

Рассматривая запрос выше, можно увидеть, что не всегда дело ограничивается названием колонки и типом данных - есть, например, такое:

```java
 male                boolean         default true, 
```

На самом деле, в описании колонки тоже можно указать массу полезных атрибутов, которые мы постепенно разберем. Пока же остановимся на **_DEFAULT_**.

В случае использования этого оператора к описанию колонки, он позволяет задать значение по умолчанию, которое будет использовано, если при добавлении записи в таблицу не было явно указано значения для соответствующего столбца. Так, в нашем случае будет проставляться true в колонке _male_, если не указано иного.

На этом можно было бы не заострять особого внимания, если бы в _DEFAULT_ можно было поместить только литерал - вполне логичная и очевидная функциональность.

Но в _DEFAULT_ можно указать, например, и функцию (правда, в параметрах нельзя использовать столбцы таблицы), что открывает более широкие возможности. Например, распространено заполнение даты создания записи через _DEFAULT now()_, что позволяет автоматически записывать текущую дату и/или время (с поправкой на тип самого поля).

  

В целом, на этом можно завершить описание базового синтаксиса - он довольно прост.

![](../../common_files/footer.png)

И перейти к практике:)

Ее я предлагаю сделать полезной и добавить таблицы, которые позволят нам расширить возможности по построению запросов в дальнейшем.

  

### Задача 1

Добавьте в БД таблицу _airport_. В состав колонок в обязательном порядке должны входить автоинкрементирующийся _id_ и _имя_ аэропорта. Остальные атрибуты могут быть добавлены на ваш вкус.

  

### Задача 2

Добавьте промежуточную таблицу между _пассажиром_ и _аэропортом_. Полагаю, вы помните, что сейчас пассажир содержит колонку с массивом любимых аэропортов. Кажется, пришло время закрепить эту связь в виде классического M2M.

Данная промежуточная таблица должна позволить через функциональность _JOIN_ связывать пассажиров с их любимыми аэропортами. Обычно такого рода таблицы именуются как _table1\_table2_.

  

### Задача 3

Добавьте таблицу _flight_. Полагаю, очевидно, что указывать в каждом билете _время_ и _аэропорты отправления_ и _прибытия_ \- несколько избыточно, сущность, эквивалентная рейсу, позволит хранить и обрабатывать эту информацию гораздо компактнее. Исходя из этой информации, постарайтесь подобрать адекватный набор колонок

Обновление таблицы билетов сделаем уже в ближайших уроках:)

  

### Задача 4

Добавьте таблицу _user_. Она должна обеспечивать информацию об учетной записи в нашей условной информационной системе. Полагаю, в нее стоит добавить _id_, _username_ и _пароль_, остальные колонки - на ваш вкус.

  

Если что-то непонятно или не получается – welcome в комменты к посту или в лс:)

Канал: [https://t.me/ViamSupervadetVadens](https://t.me/ViamSupervadetVadens)

Мой тг: [https://t.me/ironicMotherfucker](https://t.me/ironicMotherfucker)

_Дорогу осилит идущий!_

***

*назад на [курс](../../course.md) / [roadmap](../../roadmap.md)*

***

_редакция 12.08.2023_, [_оригинал_](https://telegra.ph/DDL-CREATE-Sozdanie-tablic-08-12)