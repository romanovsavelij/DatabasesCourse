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
```

### Распределение файлов БД по разным носителям?

За Data Distribution отвечает отдельный модуль. Этот механизм переодически перераспределяет ключи по нодам кластера. Например, если какая-то нода перегружена, часть ключей может быть перенесена на слабо загруженную ноду.

Дизайн этого механизма описан в отдельном [дизайн доке](https://github.com/apple/foundationdb/blob/main/design/data-distributor-internals.md). 

### На каком языке программирования написана СУБД?

Написана на С++ и C. Есть биндинги под Go, Java, Python и другие популярные языки. 

### Какие типы индексов поддерживаются в БД? Приведите пример создания индексов.

Индексы не поддерживаются "из коробки", в FoundationDB они реализуются по модели "сделай сам". Благо, ACID транзакции значительно упрощают
обработку concurrency в кейсе, когда надо изменить данные и поменять все соответствующие индексы атомарно. 

TODO: создание индекса

### Как строится процесс выполнения запросов в вашей СУБД?

### Есть ли для вашей СУБД понятие «план запросов»? Если да, объясните, как работает данный этап.

### Поддерживаются ли транзакции в вашей СУБД? Если да, то расскажите о нем. Если нет, то существует ли альтернатива?

### Какие методы восстановления поддерживаются в вашей СУБД. Расскажите о них.

### Расскажите про шардинг в вашей конкретной СУБД. Какие типы используются? Принцип работы.

### Возможно ли применить термины Data Mining, Data Warehousing и OLAP в вашей СУБД?

### Какие методы защиты поддерживаются вашей СУБД? Шифрование трафика, модели авторизации и т.п.

### Какие сообщества развивают данную СУБД? Кто в проекте имеет права на коммит и создание дистрибутива версий? Расскажите об этих людей и/или компаниях.

Это open-source проект, соответственно законтрибьютить может любой желающий. В основном развивает его Apple, потому что эта база используется
у них во многих внутренних системах.

### Как продолжить самостоятельное изучение языка запросов с помощью демобазы. Если демобазы нет, то создайте ее.

### Где найти документацию и пройти обучение

Документация есть на [официальном сайте](https://apple.github.io/foundationdb/) FoundationDB. Курсов не нашел, но зато у них очень подробная и хорошо написанная документация, ее должно хватить. 

### Как быть в курсе происходящего

FoundationDB ведет свой [twitter](https://twitter.com/foundationdb). 
