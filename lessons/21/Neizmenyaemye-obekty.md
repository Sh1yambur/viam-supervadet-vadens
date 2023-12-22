![](../../common_files/header.png)

*назад на [курс](../../course.md) / [roadmap](../../roadmap.md)*

***

   

Неизменяемые объекты
====================

Скоро мы будем знакомиться более подробно с классом _String_ и его альтернативами, а также классами-обертками над примитивными типами.

Для этого нам стоит разобраться с понятием неизменяемого объекта и актуальностью его применения.

#### Изменяемые и неизменяемые объекты

В большинстве ООП языков, любой объект можно отнести к одной из двух категорий:

· **Изменяемый** – **mutable**;

· **Неизменяемый** – **immutable**. В рамках профессионального общения вы можете услышать вариант «**имьютабельный**»;

Основным признаком изменяемости объекта можно считать возможность изменить значения его полей после инициализации. По сути, если у вашего объекта есть хоть один сеттер или поле, которое не помечено _final_ или модификатором доступа, отличным от _private_ – он однозначно изменяем.

Изменяемыми являются абсолютное большинство классов-сущностей, что логично. Какой смысл от объекта типа «Счетчик», если мы не можем изменить его значение?

Неизменяемыми же часто делают классы бизнес-логики (по крайней мере, в современном Java-мире), иногда – классы-сущности. Кроме того, ряд базовых классов Java являются immutable: _String_, _Integer_, _Double_, _Character_ и другие классы обертки.

Зачем делать объект неизменяемым?

Причины могут быть разными, но почти всегда сводятся к одному: изменение состояния объекта в процессе его жизни может привести к нежелательным изменениям поведения.

Вспоминая задачу с _counterAggregation_ (Урок 12), представьте, что ваш массив в объекте класса _AggregationService_ был изменен на другой, содержащий другие элементы. Не самый удачный пример, но даже в нем логика работы могла оказаться нарушена. Например, потому что вы не смогли найти счетчик, который точно туда добавляли. Или хуже – полю с массивом присвоили _null_ – тогда практически любое обращение к методам _AggregationService_ порождало бы исключение.

Итак, как же гарантировать неизменяемость объекта в Java:

1\. Класс должен быть _final_. Безусловно, это не влияет на изменяемость полей. Но для не финализированного класса нет гарантии, что в переменной этого типа не окажется записан объект класса-наследника с отличающейся логикой. Это правило выполняется далеко не всегда (особенно, для классов бизнес-логики), но о нем обязательно захотят услышать на собеседовании;

2\. Все поля должны быть помечены _private final_. Это гарантирует неизменность примитивных полей и ссылок для полей ссылочных типов;

3\. Инициализация полей должна производиться в конструкторе. Это вытекает из того, что поля помечены как _final_;

4\. Отсутствие сеттеров. В целом, они и так бесполезны, если поля помечены как _final_, но не всем это может быть очевидно;

5\. Геттеры возвращают только примитивные типы или копии объектов, если геттер написан для поля ссылочного типа. Это связано с тем, что _final_ поле гарантирует неизменность ссылки. Но никак не влияет на возможность изменить поля объекта, который по этой ссылке доступен.

Неизменяемые объекты, кроме большей предсказуемости их поведения, в сравнении с изменяемыми, имеют и другие плюсы.

Во-первых, их удобно использовать как идентификаторы. Скажем, объекты класса «Человек» могут иметь массу изменяемых параметров: семейный статус, количество детей, половая принадлежность… В общем-то, большинство полей, которыми можно охарактеризовать человека, могут изменить свое значение.

Но если нам нужно найти человека среди множества других людей (скажем, в массиве) – нам понадобятся неизменные поля. В каких-то системах это может быть одно поле: например, уникальный номер паспорта или ИНН. В других системах это может быть несколько параметров. Скажем, дата и место рождения, ФИО при рождении. В таком случае может быть полезно вынести неизменяемые поля в отдельный класс, объекты которого и использовать как идентификатор.

К сожалению, пример вышел несколько натянутым, поскольку мы еще не знакомы с некоторыми механиками языка, в которых это может быть полезно.

Во-вторых, неизменяемые объекты можно переиспользовать. Например, работая с классом «Машина» мы можем иметь набор изменяемых параметров для каждого конкретного объекта машины. Цвет, дата последнего прохождения ТО, наличие тонировки, работоспособность поворотников… Но, при этом, технические характеристики будут неизменны для каждой модели. В таком случае может быть полезным вынести их в отдельный класс и создавать объекты характеристик не для каждой машины, а для каждой модели (очевидно, что количество моделей автомобилей намного меньше, чем самих автомобилей). Сама машина будет лишь иметь поле типа «технические характеристики», причем все машины одной модели будут содержать в этом поле ссылку на один и тот же объект. Такой подход может сильно сэкономить память.

**!NB:** если вы предполагаете использование объектов класса в качестве идентификатора (никогда не используйте в этих целях изменяемые объекты) – подумайте о том, чтобы вынести хэшкод такого объекта в отдельное поле, возможно, его даже стоит сделать константой. Это может помочь с ускорением поиска по массиву (в более широком смысле – коллекции) таких объектов.

#### В качестве итога

_Immutable-объекты_ – очень удобный инструмент в достаточно широком диапазоне задач. Не все случаи применения мы можем рассмотреть сейчас в силу недостатка знаний, но даже озвученные примеры, на мой взгляд, являются убедительной демонстрацией его полезности.

Выше даны классические для Java правила достижения имьютабельности объекта. Не все из них и не всегда применяются в полной мере, все, как обычно, зависит от специфики конкретной задачи. На данном этапе рекомендую обратить внимание именно на имьютабельность для классов-сущностей, с классами логики все немного сложнее, но с этим мы будем разбираться позже.

С теорией на сегодня все!

![](../../common_files/footer.png)

  

Переходим к практике:

#### Задача

Реализуйте задачу из [урока 19](/Metody-klassa-Object-12-01).

На свое усмотрение, вынесите неизменяемые поля, используемые для идентификации и поиска машины в отдельный immutable класс или сделайте весь класс «Машина» неизменяемым. Правильный выбор зависит от набора полей, который существует в вашей текущей реализации класса «Машина».

  

Если что-то непонятно или не получается – welcome в комменты к посту или в лс:)

Канал: [https://t.me/+relA0-qlUYAxZjI6](https://t.me/+relA0-qlUYAxZjI6)

Мой тг: [https://t.me/ironicMotherfucker](https://t.me/ironicMotherfucker)

_Дорогу осилит идущий!_

***

*назад на [курс](../../course.md) / [roadmap](../../roadmap.md)*

***

_редакция 02.12.2022_, [_оригинал_](https://telegra.ph/Neizmenyaemye-obekty-12-02)