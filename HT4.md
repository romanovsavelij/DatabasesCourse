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

TODO

### Распределение файлов БД по разным носителям?

### На каком языке программирования написана СУБД?

Написана на С++ и C. Есть биндинги под Go, Java, Python и другие популярные языки. 

### Какие типы индексов поддерживаются в БД? Приведите пример создания индексов.

Индексы не поддерживаются "из коробки", в FoundationDB они реализуются по модели "сделай сам". Благо, ACID транзакции значительно упрощают
обработку concurrency в кейсе, когда надо изменить данные и поменять все соответствующие индексы атомарно. 
