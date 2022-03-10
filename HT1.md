### Установка

Я запустил монгу в докере, пробросив порт. Подключился к этой базе через `mongosh`, работал через эту консольку. 

### Данные

Данные я взял с [kaggle](https://www.kaggle.com/sbhatti/news-summarization). Это датасет выжимок из новостных статей обьемом порядка 4Гб. 

Заполнял данными базу с помощью `mongoimport`:
```sh 
mongoimport --db data --collection news --type csv --headline --file data.csv
```

### Получение данных
```sh 
db.news.find({Dataset: {$ne: 'CNN/Daily Mail'}}).limit(5)

db.news.find({ID: {$eq: 35586320}})
```

### Обновление данных
```sh
db.news.updateMany({Dataset: {$eq: 'CNN/Daily Mail'}}, { $set: { Dataset: 'CNN' }})
```
### Индексы
Сделаем поиск по фиксированному `ID` без индекса: 
```sh 
db.news.find({ID: {$eq: 36475048}}).explain("executionStats")
```

Вижу в статистике этого запроса, что время исполнения - 44 секунды. 

Добавляю индекс:
```sh 
db.news.createIndex( { ID: 1 } )
```

Поиск с индексом занял всего 30мс! Аналогичного рода огромное ускорение было получено для поиска по полю `Dataset`. 


