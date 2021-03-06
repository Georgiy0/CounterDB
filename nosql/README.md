# DB - cassandra + memcached

В Django проекте DB реализовано кэширование в memcached и работа с постоянной базой данных (Cassandra).
Приложение обрабатывает два POST запроса: на постановку лайка и на получение информации о количестве лайков.

## Постановка лайка

POST запрос на URL: /db/like с одним параметром post_id.
При постановке лайка производится попытка инкрементации счетчика по ключу post_id в memcache. Если операция инкрементации по ключу в memcache завершается успешно, то происходит инкрементация счетчика и в основной базе. При неудаче (cache miss) происходит обращение в основную базу. Если запись с заданным post_id присутствует в базе, то счетчик в основной базе инкрементируется и информайия о посте заносится в кэш. если записи не существует, то она создается со значением счетчика равным 1 и заносится в кэш.

## Получение информации о количестве лайков

POST запрос на URL: /db/get с одним параметром post_id.
Сначала проверяем наличие записи в memcache. В случае cache miss обращаемся в основную базу. Если удается найти запись в базе, то заносим ее в кэш, иначе возращаем информацию о том, что запись не найдена.

## Формат ответа

Приложение форимирует ответ в виде Json:

```json
{
  "likes_num" : "количество лайков",
  "info" : "дополнительная информация"
}
```

В поле 'info' заносится информация о том, где была найдена запись.

## Node scaling

В файл settings.py была добавлена новая настройка, отвечающая за подключенные nodes, по которым будет распределяться нагрузка:

```json
NODES_LIST = {
    "n_list": [
        "URL первого узла",
        "URL второго узла",
        "URL третьего узла",
        "..."
    ],
    "n_number": "количество узлов"
}
```

# Dependencies

## Cassandra

Для работы с кластером cassandra используется django cassandra engine:
```
pip install django-cassandra-engine
```
## Python requests

Для реализации взаимодействия между Django приложениями по протоколу HTTP используется библиотека requests:
```
pip install requests
```
