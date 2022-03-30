### Запуск базы

```sh
docker run -e COUCHDB_USER=admin -e COUCHDB_PASSWORD=password -p 5984:5984 -d couchdb
```

### Правим скрипт

```js
 Remote: new PouchDB('http://admin:password@localhost:5984/person')
```

### Настраиваем CORS

В разделе CORS разрешаем доступ со всех адресов

### Создаем базу и заводим документ

<img width="430" alt="Screen Shot 2022-03-30 at 19 40 43" src="https://user-images.githubusercontent.com/32338211/160887749-a9dd3ac0-4cde-43fb-8fb1-dad0762c3842.png">

### Делаем sync на фронте

Видим, что запись подтянулась в локальную базу

<img width="229" alt="Screen Shot 2022-03-30 at 19 31 02" src="https://user-images.githubusercontent.com/32338211/160887914-eb81a58d-8caa-47bd-a5dd-aefae5dc9174.png">

### Отключаем базу и опять делаем sync

Запись остается

<img width="229" alt="Screen Shot 2022-03-30 at 19 31 02" src="https://user-images.githubusercontent.com/32338211/160887914-eb81a58d-8caa-47bd-a5dd-aefae5dc9174.png">

### Добавим своих записей на фронте через add при отключенной базе

<img width="286" alt="Screen Shot 2022-03-30 at 19 36 38" src="https://user-images.githubusercontent.com/32338211/160888288-b46b28f1-541b-4b23-aab6-bc703cde2597.png">

### Поднимаем базу

Видим, что в админке базы новые записи появились

<img width="821" alt="Screen Shot 2022-03-30 at 19 36 32" src="https://user-images.githubusercontent.com/32338211/160888385-566de618-a1f7-4359-81e2-b197f49e6ec2.png">

