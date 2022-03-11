### Установка

Я запустил монгу в докере, пробросив порт. Подключился к этой базе через `mongosh`, работал через эту консольку. 

### Данные

Данные я взял с [kaggle](https://www.kaggle.com/sbhatti/news-summarization). Это датасет выжимок из новостных статей обьемом порядка 4Гб. 

Заполнял данными базу с помощью `mongoimport`:
```sh 
mongoimport --db data --collection news --type csv --headline --file data.csv
```

### Получение данных
Возьмем новость с ID=35586320:
```
db.news.find({ID: {$eq: 35586320}})
```

Новости не из источника CNN/Daily Mail:
```
db.news.find({Dataset: {$ne: 'CNN/Daily Mail'}}).limit(5)
```

Запрос на получение новостей, у которых в Summary есть подстрока Russia, а в Content есть Russia и war. При этом оставим только поля Summary и Dataset:
```
db.news.find({ Content: /.*\sRussia\s.*\swar\s.*/, Summary: /Russia/ }, {Summary: 1, Dataset: 1}).limit(5)
```

Посчитаем количество таких новостей:
```
db.news.find({ Content: /.*\sRussia\s.*\swar\s.*/, Summary: /Russia/ }, {Summary: 1, Dataset: 1}).count()
```

Получили 838 штук. 

А всего новостей 870521 штук:
```
db.news.count()
```

Получается, не так уж мы популярны по историческим меркам =) 

Возьмем новости из одного из двух источников - XSum или CNN:
```
db.news.find({ Dataset: { $in: ['CNN', 'XSum'] }}).limit(5)
```


### Обновление данных
```
db.news.updateMany({Dataset: {$eq: 'CNN/Daily Mail'}}, { $set: { Dataset: 'CNN' }})
```

### Индексы
Сделаем поиск по фиксированному `ID` без индекса: 
```
db.news.find({ID: {$eq: 36475048}}).explain("executionStats")
```

Вижу в статистике этого запроса, что время исполнения - 44 секунды. 

Добавляю индекс:
```
db.news.createIndex( { ID: 1 } )
```

Поиск с индексом занял всего 30мс! Аналогичного рода огромное ускорение может быть получено при поиске, например, по полю `Dataset`. 

Потом я решил, создать текстовый индекс на поле Content. Хотел например поискать по нему по подстроке документы: 
```
db.news.createIndex({Content: "text"})
```

Монга шебуршала минут 15, после чего просто упала. Видимо слишком много контента для нее, не потянула =)
