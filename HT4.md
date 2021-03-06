### История развития СУБД

FoundationDB начала разрабатываться Nick Lavezzo и Dave Rosenthal в 2009 году. В 2013 году был опубликован первый beta релиз в открытый доступ. 
В 2015 году компания была поглощена Apple и более не стала доступна для скачивания. В 2018 году Apple вернула исходный код FoundationDB в открытый доступ
с лицензией Apache 2.0. 

### Инструменты для взаимодействия с СУБД

С FoundationDB можно работать при помощи клиента `fdbcli`. Также можно обращаться к базе из своего любимого языка при помощи биндингов: C++, C, Python, Java, Go, Erlang, ...

Также для работы с базой можно использовать специально разработанный для нее язык Flow, который во многом похож на С/Go. 

Простейшая команда `fdbcli status`, которую имеет смысл вызвать в перед началом работы с базой: 
```sh
➜  ~ fdbcli
Using cluster file `/usr/local/etc/foundationdb/fdb.cluster'.

The database is available.

Welcome to the fdbcli. For help, type `help'.
fdb> status

Using cluster file `/usr/local/etc/foundationdb/fdb.cluster'.

Configuration:
  Redundancy mode        - single
  Storage engine         - memory-2
  Coordinators           - 1
  Usable Regions         - 1

Cluster:
  FoundationDB processes - 1
  Zones                  - 1
  Machines               - 1
  Memory availability    - 4.3 GB per process on machine with least available
  Fault Tolerance        - 0 machines
  Server time            - 05/04/22 12:21:45

Data:
  Replication health     - Healthy
  Moving data            - 0.000 GB
  Sum of key-value sizes - 0 MB
  Disk space used        - 105 MB

Operating space:
  Storage server         - 1.0 GB free on most full server
  Log server             - 6.5 GB free on most full server

Workload:
  Read rate              - 19 Hz
  Write rate             - 0 Hz
  Transactions started   - 4 Hz
  Transactions committed - 0 Hz
  Conflict rate          - 0 Hz

Backup and DR:
  Running backups        - 0
  Running DRs            - 0

Client time: 05/04/22 12:21:45
```


### Какой database engine используется в вашей СУБД?

FoundationDB использует одноименный database engine.

### Как устроен язык запросов в вашей СУБД? Разверните БД с данными и выполните ряд запросов.
 
Пример работы с базой:
```sh
fdb> writemode on
fdb> set student_123_name Saveliy
Committed (12128976652)
fdb> set student_123_surname Romanov
Committed (12137361608)
fdb> set student_123_dbmark 10
Committed (12150900873)
fdb> set student_124_name Max
Committed (12175624425)
fdb> set student_124_surname Lobanov
Committed (12189517708)
fdb> set student_124_dbmark 7
Committed (12201329183)
fdb> get student_123_name
`student_123_name' is `Saveliy'
fdb> set student_125_name Saveliy
Committed (21245107540)
fdb> set student_125_surname Petrov
Committed (21256512895)
fdb> getrange student_123 student_123_\xFF

Range limited to 25 keys
`student_123_dbmark' is `10'
`student_123_name' is `Saveliy'
`student_123_surname' is `Romanov'
```

### Распределение файлов БД по разным носителям?

Есть два конфигурации хранения данных: `ssd` и `memory`.

При задании типа `ssd` данные будут храниться на диске и браться именно оттуда. Этот тип оптимизирован под `ssd` 
и c другими типами диском вроде `hdd` очень не рекомендуется использовать. Данные хранятся на диске в виде B-дерева. 
При удалении данных память может высвобождаться не сразу, FoundationDB будет потихоньку вычищать удаленные сегменты по возможности. 

При типе `memory` чтения обслуживаются из оперативной памяти, а записи все еще идут на диск. 
Все данные должны помещаться в оперативную память. Также в этом случае запуск ноды может занимать минуты, 
потому что при старте надо сформировать нужные структуры данных из диска. 

### На каком языке программирования написана СУБД?

Написана на С++ и C. Есть биндинги под Go, Java, Python и другие популярные языки. 

### Какие типы индексов поддерживаются в БД? Приведите пример создания индексов.

Индексы не поддерживаются "из коробки", в FoundationDB они реализуются по модели "сделай сам". Благо, ACID транзакции значительно упрощают
обработку concurrency в кейсе, когда надо изменить данные и поменять все соответствующие индексы атомарно. 

```sh
fdb> set student_name_Saveliy_123 ""
Committed (21307570403)
fdb> set student_name_Saveliy_125 ""
Committed (21311785572)
fdb> getrange student_name_Saveliy student_name_Saveliy_\xFF

Range limited to 25 keys
`student_name_Saveliy_123' is `'
`student_name_Saveliy_125' is `'
```

Пример кода работы с индексом из статьи в доке: 
```python
user = fdb.Subspace(('user',))
index = fdb.Subspace(('zipcode_index',))

@fdb.transactional
def set_user(tr, ID, name, zipcode):
        tr[user[ID][zipcode]] = name
        tr[index[zipcode][ID]] = ''

# Normal lookup
@fdb.transactional
def get_user(tr, ID):
    for k,v in tr[user[ID].range()]:
        return v

    return None

# Index lookup
@fdb.transactional
def get_user_IDs_in_region(tr, zipcode):
    return [index.unpack(k)[1] for k, _ in tr[index[zipcode].range()]]
```

### Как строится процесс выполнения запросов в вашей СУБД?

База однопоточная. Если у вас на машине несколько процессоров, надо поднять несколько инстансов, чтобы их утилизировать.

Сама по себе база предоставляет только key-value модель хранения и примитивный язык запросов, тем самым отделяя движок от API и модели хранения данных.
Поэтому полезно использовать надстройки, называемые Layer-ами. Они реализованы как отдельные модули. Например, есть [fdb-document-layer
](https://github.com/FoundationDB/fdb-document-layer), который реализует документную модель хранения данных и протокол `MongoDB wire`.

Поэтому, процесс вылонения запроса выглядит так:

<img width="534" alt="Screen Shot 2022-05-04 at 20 27 25" src="https://user-images.githubusercontent.com/32338211/166746546-184a6196-af94-4962-b4a3-f7a4e57e743c.png">

### Есть ли для вашей СУБД понятие «план запросов»? Если да, объясните, как работает данный этап.

Нет, такого понятия как "план запроса" не имеется. Это key-value хранилище, поэтому тут все нет сложносоставленных операций, 
которые можно было бы исполнять многими разными способами. 

Тут для каждой операции, которая может долго исполняться, понятно почему. Например, если `getrange` запрос долго исполнуятся, то это потому что range слишком большой. Такой запрос можно разбить на несколько более маленьких. 

### Поддерживаются ли транзакции в вашей СУБД? Если да, то расскажите о нем. Если нет, то существует ли альтернатива?

Да, можно сказать, что любая операция над базой по умолчанию оборачивается в транзакцию. Транзакции удовлетворяют `ACID` свойствам + `Causality` (гарантируется, что транзакция увидет все изменения в транзакциях, закомиченных до начало текущей). Причем уровень изоляции по умолчанию - `strictly serializable`.

Эти гарантии реализуются при помощи `MVCC` на чтение и `optimistic concurrency` на запись. Таким образом, во время коммита транзакция может быть
откачена, тогда клиентской библиотеке надо будет ее рестартануть.

При этом гарантии можно ослабить при желании. Например, при помощи `Snapshot Reads` можно значительно уменьшить задержки на чтение при высоком contention и в этом случае гарантируется, что транзакцию не надо будет рестартить.

### Какие методы восстановления поддерживаются в вашей СУБД. Расскажите о них.

FoundationDB умеет переживать отказы любых отдельтных узлов. Весь пул нод выделяется в "команды". Реплики одного шарда лежат на узлах одной "команды". 
"команда" составляется с учетом локальности, чтобы там были узлы из разных ДЦ или из разных стоек в ДЦ. 

FoundationDB умеет восстанавливаться после ошибок любого типа. Для нее специально написан детерменированный симулятор распределенной системы, 
на котором она ежедневно тестируется на миллионах симуляций со всевозможными отказами разных компонентов системы. 
Поэтому, ее отказоустойчивости можно доверять =)

### Расскажите про шардинг в вашей конкретной СУБД. Какие типы используются? Принцип работы.

За Data Distribution отвечает отдельный модуль. Поддерживается как горизонтальное, так и вертикальное шардирование в полностью автоматизированном режиме. Этот механизм переодически перераспределяет ключи по нодам кластера. Например, если какая-то нода перегружена, часть ключей может быть перенесена на слабо загруженную ноду.

Дизайн этого механизма описан в отдельном [дизайн доке](https://github.com/apple/foundationdb/blob/main/design/data-distributor-internals.md). 

### Возможно ли применить термины Data Mining, Data Warehousing и OLAP в вашей СУБД?

Это база не заточена под аналитические задачи. Она оптимизирована под большой throughput входящих запросов, а не под аналитические нужды. 

### Какие методы защиты поддерживаются вашей СУБД? Шифрование трафика, модели авторизации и т.п.

По умолчанию трафик не шифруется, но при желании можно подключить TLS. Тогда все взаимодействие между клиентами и серверами будет шифроваться с помощью асинхронного шифрования. 

### Какие сообщества развивают данную СУБД? Кто в проекте имеет права на коммит и создание дистрибутива версий? Расскажите об этих людей и/или компаниях.

Это open-source проект, соответственно законтрибьютить может любой желающий. В основном развивает его Apple, потому что эта база используется
у них во многих внутренних системах.

### Где найти документацию и пройти обучение

Документация есть на [официальном сайте](https://apple.github.io/foundationdb/) FoundationDB. Курсов не нашел, но зато у них очень подробная и хорошо написанная документация, ее должно хватить. 

### Как быть в курсе происходящего

FoundationDB ведет свой [twitter](https://twitter.com/foundationdb) и [блог](https://www.foundationdb.org/blog/).
