![](../../common_files/header.png)

*назад на [курс](../../course.md) / [roadmap](../../roadmap.md)*

***

   

Многопоточность. java.util.concurrent. Потокобезопасные коллекции
=================================================================

Сегодня мы познакомимся с одной из важнейших частей пакета _java.util.concurrent_, которая предоставляет **потокобезопасные коллекции**, более эффективные, чем потокобезопасные legacy-коллекции (_Vector_, _Hashtable_, _Stack_) собственные механизмы для безопасной работы с потоконебезопасными коллекциями, с которыми мы познакомились в рамках Collections Framework.

Legacy-коллекции строятся на базе механизма _synchronized_, объявляя каждый из методов по взаимодействию с коллекцией синхронизированным. Очевидно, что такое решение является достаточно грубым.

Сразу отметим, что в этой статье мы не будем говорить о **блокирующих** и **неблокирующих очередях**, которые, безусловно, являются коллекциями (реализуют один из базовых интерфейсов – _Queue_), но имеют несколько отличные от остальных коллекций сценарии использования. Для знакомства с ними будет выделен отдельный урок.

В данном же уроке сконцентрируемся на коллекциях, решающих те же задачи, что и привычные нам реализации _List_, _Set_ и _Map_, но гарантирующих безопасность взаимодействия в многопоточной среде.

### Класс Collections

Однако начнем мы не с пакета _java.util.concurrent_.

Коллекции в нем покрывают лишь общие сценарии использования, что логично. Но существуют также и узконаправленные коллекции, которые служат для решения специфических задач. Например, _WeakHashMap_, которая использует механизм **слабых ссылок** (**weak reference**) для избавления от элементов, которые уже не востребованы. В нашем курсе мы вряд ли коснемся типов ссылок в Java, в силу малой (но имеющейся) потребности в реальной разработке. Однако вы можете ознакомиться с тем, какие ссылки бывают в Java. Это не слишком сложная (для ознакомления) тема, хотя ее применение и требует понимания особенностей работы Garbage Collector’а.

Кроме того, к ситуациям, когда может потребоваться специфическая потокобезопасная коллекция могут быть отнесены и собственные реализации коллекций на базе алгоритмов, отличных от тех, что применяются в реализациях из _java.util_. Скажем, супер-эффективный список вашего производства.

Так или иначе, если вы имеете коллекцию, которую требуется сделать потокобезопасной, вы можете обратиться к утилитному классу _Collections_. Он имеет массу полезных методов, но нас интересуют те из них, которые позволят сделать коллекцию сихнорнизированной:

·  _synchronizedCollection(Collection<T> c);_

·  _synchronizedSet(Set<T> s);_

_· synchronizedSortedSet(SortedSet<T> s);_

_· synchronizedNavigableSet(NavigableSet<T> s);_

_· synchronizedList(List<T> list);_

_· synchronizedMap(Map<K,V> m);_

_· synchronizedSortedMap(SortedMap<K,V> m);_

_· synchronizedNavigableMap(NavigableMap<K,V> m)._

Обратите внимания, что методов для синхронизации очередей (и деков) не предусмотрено (у этого есть причины, но о них в следующей статье). В остальном покрыты все основные интерфейсы коллекций, что позволяет использовать в полученной синхронизированной коллекции любой набор доступных методов – от базовых из _Collection_, до специфических, как в продвинутых интерфейсах вроде _NavigableMap_.

В целом, принцип работы данных методов прост – они создают объект класса-обертки над переданной коллекцией, объявляя все ее методы как _synchronized_. Т.е. делают коллекцию потокобезопасной с помощью того же подхода, который используется в потокобезопасных legacy-коллекциях. Это, конечно, не очень эффективно, но лучше, чем ничего.

Стоит отметить, что монитором выступает не сама коллекция, а объект-**мьютекс**. Этот термин, исторически, не так распространен в Java, хотя является одним из базовых терминов в многопоточности. В рамках текущего контекста его можно воспринимать как объект, единственная цель которого – быть монитором при синхронизации. Однако сам термин рекомендую запомнить – вы его еще не раз встретите.

В целом, рекомендую ознакомиться с возможностями класса _Collections_ и за пределами многопоточности – там есть достаточно много методов, полезных при разработке в императивной парадигме.

### Коллекции в java.util.concurrent

#### List

Традиционно начнем со списков. Рассматриваемый пакет предлагает только одну реализацию – _CopyOnWriteArrayList_.

Полагаю, вы уже успели убедиться на практике, что _List_ – не самый популярный тип коллекций. По крайней мере, в контексте длительного хранения данных в памяти, а не в рамках метода.

Данную коллекцию (как и другие _CopyOnWrite_\-коллекции) рекомендуют применять в ситуациях, когда чтение происходит чаще записи (добавления, замены или удаления элементов). Это связано с алгоритмом **Copy-On-Write**, на базе которого и построена коллекция (о нем чуть ниже). Однако в случае с листом альтернатив (кроме синхронизированных коллекций, включая _Vector_) нет. Впрочем, как и широкой потребности в списках как разделяемых ресурсах.

В данном пункте гораздо важнее понять, в чем принцип алгоритма _Copy-On-Write_ (**копирование при записи**) – он применяется как в ряде других коллекций, так и, в широком смысле, компьютерной архитектуре в целом, включая многие процессы операционных систем, связанные с низкоуровневой записью (как в оперативной памяти, так и при работе с файлами).

Основной принцип достаточно прост: при изменении ресурса (в данном случае – массива, лежащего внутри списка) происходит его копирование. При этом чтение из неизмененного массива может продолжаться. Как только процесс записи: вставки, удаления или замены (по сути, создания нового массива через копирование с произведением необходимой операции) – элемента завершится – копия заменит исходный массив и чтение будет происходить уже из нее.

Таким образом, теоретически возможна ситуация, когда операция изменения была вызвана одним потоком, но другой поток прочел устаревшую информацию, поскольку обновленный массив еще не заменил исходный.

К слову данный подход делает _CopyOnWriteArrayList_ даже более простым в восприятии реализации, чем обычный _ArrayList_ – поскольку любое изменение все равно копирует весь массив, нет необходимости в не самом очевидном механизме, представленном в методе _ArrayList#grow()_, который будет увеличивать размер массива при его заполнении – достаточно увеличивать (или уменьшать) размер массива на нужное число элементов при копировании, остальные манипуляции лишены смысла.

К сожалению, подобный подход достаточно затратен по памяти, особенно для листов с множеством элементов. Кроме того, любая операция записи все еще работает на базе _synchronized_, а значит, множественная запись может привести к голоданию отдельных потоков – они будут ждать освобождение монитора.

#### Map

В случае с мапами выбор несколько шире.

Наиболее распространенная реализация – _ConcurrentHashMap_.

По сути, напоминает привычную нам _HashMap_. Ее плюс в том, что все методы являются потокобезопасными – у другой реализации могут быть исключения, будьте осторожны.

Потокобезопасность достигается за счет синхронизации на объекте **чанка** (также распространен термин «**сегмент**» – именно в контексте _ConcurrentHashMap_) при записи, а также концепции **CAS** (**Compare-And-Swap**, **сравнение с обменом**) при получении доступа к чанку. **CAS-операции** нам уже встречались при знакомстве с atomic-типами – именно на них построена работа методов вроде _AtomicInteger#getAndIncrement()_.

Суть CAS-операций заключается в том, что они являются атомарными (вплоть до атомарности на уровне процессора) и представляют из себя сравнение известного потоку значения с текущим (реальным) и его замену на новое значение, если результат сравнения положителен. Таким образом, CAS-операции содержат два параметра – текущее значение и обновленное.

Таким образом, _ConcurrentHashMap_ является достаточно оптимизированной несортированной мапой, гарантирующей потокобезопасность собственных операций.

Альтернативой _ConcurrentHashMap_, если требуется упорядоченность элементов по компаратору, может стать _ConcurrentSkipListMap_ – реализация интерфейса _ConcurrentNavigableMap_, потокобезопасного наследника хорошо известного нам интерфейса _NavigableMap_ (его основная реализация – _TreeMap_).

Минусы данной реализации (кроме худшей алгоритмической сложности операций) – в том, что не все ее методы потокобезопасны (например, _computeIfAbsent()_). Поэтому рекомендую обращаться к документации, если планируете использовать что-то вне базовых методов чтения/записи.

Данная мапа работает на базе структуры данных **Skip List** – **Список с пропусками**. Эта структура однозначно не относится к числу базовых, но рекомендую ознакомиться с ней самостоятельно. При определенных сложностях реализации и расчета эффективности, сама концепция является интересной и достаточно простой. Утрируя, данную структуру можно рассматривать как список с особенностями, более характерными для деревьев. В меньшей степени в контексте самой структуры (тут больше применимо сравнение с несколькими отдельными списками), в большей – в контексте алгоритмической сложности операций

Ссылка на вики – [клац](https://ru.wikipedia.org/wiki/%D0%A1%D0%BF%D0%B8%D1%81%D0%BE%D0%BA_%D1%81_%D0%BF%D1%80%D0%BE%D0%BF%D1%83%D1%81%D0%BA%D0%B0%D0%BC%D0%B8) (рекомендую обратить внимание на пункт _Описание_, он написан достаточно простым языков).

Выбор реализации зависит, как правило, от тех же критериев, что и выбор между _HashMap_ и _TreeMap_ – необходимость базовой сортировки данных.

#### Set

С сетами, на самом деле, традиционно просто.

Основных реализаций две – _CopyOnWriteArraySet_ и _ConcurrentSkipListSet_.

_CopyOnWriteArraySet_ работает на базе _CopyOnWriteArrayList_ и обладает всеми его недостатками – тяжелая операция копирования, особенно для больших объемов информации, синхронизированные методы записи. По сути, его можно рассматривать как некий аналог _LinkedHashSet_ из-за сохранения порядка элементов (пусть и посредством иного механизма), но он лишен константной сложности операций. В целом, _CopyOnWriteArraySet_ является даже менее эффективным, чем _CopyOnWriteArrayList_ из-за необходимости проверять наличие эквивалентного элемента.

_ConcurrentSkipListSet_ – реализация, работающая на базе _ConcurrentSkipListMap_ (или другой реализации _ConcurrentNavigableMap_, если захотеть). По сути, потокобезопасный аналог _TreeMap_. В целом, вариант менее дорогой для записи даже на больших объемах, с поправкой на необходимость сортировки по компаратору.

Еще один вариант потокобезопасной мапы, хоть и не имеющий собственного полноценного класса – _ConcurrentHashMap.KeySetView_. Создать его можно через статический метод _ConcurrentHashMap.newKeySet()_. По сути, представляет собой потокобезопасный вариант _HashSet_. И, очевидно, работает на базе _ConcurrentHashMap_. В целом, достаточно удобный вариант для сценариев, когда не требуется сортировка элементов.

### Что дальше?

Данный урок позволит работать с потокобезопасными коллекциями и даже понимать их основные отличия и поверхностно (если речь не о _Copy-On-Write_ коллекциях – они очень просты) – принципы работы. Но данная тема также открывает простор в изучении как сложных структур данных (_Skip List_) и механизмов синхронизации (_Copy-And-Swap_), так и позволяет шире взглянуть на уже знакомые нам итераторы (а далее - Spliterator’ы и Stream API) в контексте концепций **Fail Fast** и **Fail Safe**.

Точкой входа в последнее может стать _ConcurrentModificationException_. Подробнее можно ознакомиться в [Задаче 2](#%D0%97%D0%B0%D0%B4%D0%B0%D1%87%D0%B0-2). Пример, возможно, не самый корректный, но вполне показательный.

Впереди нас ждет еще один урок, который можно отнести к потокобезопасным коллекциям. Но это уже совсем другая история.

С теорией на сегодня все!

![](../../common_files/footer.png)

Переходим к практике:

### Задача 1

Реализуйте [Задачу из Урока 61](/Mnogopotochnost-Sinhronizaciya-potokov-Ponyatie-monitora-Klyuchevoe-slovo-synchronized-03-30#%D0%97%D0%B0%D0%B4%D0%B0%D1%87%D0%B0), используя потокобезопасные коллекции вместо механизма synchronized.

### Задача 2

Создайте стрим на базе _ArrayList_ и внутри стрима удаляйте элементы исходного списка. Например, пусть каждый элемент удаляет сам себя.

Попробуйте сделать то же самое, но заменив _ArrayList_ на _CopyOnWriteArrayList_. Как изменился результат?

Можете попробовать также и с другими потокобезопасными и потоконебезопасными коллекциями.

  

Если что-то непонятно или не получается – welcome в комменты к посту или в лс:)

Канал: [https://t.me/ViamSupervadetVadens](https://t.me/ViamSupervadetVadens)

Мой тг: [https://t.me/ironicMotherfucker](https://t.me/ironicMotherfucker)

_Дорогу осилит идущий!_

***

*назад на [курс](../../course.md) / [roadmap](../../roadmap.md)*

***

_редакция 29.04.2023_, [_оригинал_](https://telegra.ph/Mnogopotochnost-javautilconcurrent-Potokobezopasnye-kollekcii-04-29)