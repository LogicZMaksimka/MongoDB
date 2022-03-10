# MongoDB introduction
## _Stackoverflow questions database_

База данных содержит 40 миллионов записей вопросов на stackoverflow начиная с 2008 года и до 2020.
Подробнее поля документов можно посмотреть в примере

**Создание базы данных**
```
use stackoverflow
db
db.createCollection("posts_data")
// Заполнение данными через MongoDB Compass
```

**Пример документа**
```json
{
    "_id": {
        "$oid": "6228fc0381fd6c70646ed988"
    },
    "AcceptedAnswerId": 7,
    "AnswerCount": 13,
    "CommentCount": 1,
    "CommunityOwnedDate": {
        "$date": "2012-10-31T12:42:47.213Z"
    },
    "CreationDate": {
        "$date": "2008-07-31T17:42:52.667Z"
    },
    "FavoriteCount": 41,
    "Id": 4,
    "LastActivityDate": {
        "$date": "2018-02-22T13:40:13.577Z"
    },
    "LastEditDate": {
        "$date": "2017-09-27T02:52:59.927Z"
    },
    "LastEditorDisplayName": "Rich B",
    "LastEditorUserId": 3151675,
    "OwnerUserId": 8,
    "PostTypeId": 1,
    "Score": 557,
    "Tags": "<c#><winforms><type-conversion><decimal><opacity>",
    "Title": "While applying opacity to a form, should we use a decimal or a double value?",
    "ViewCount": 35933
}
```

**Примеры результатов запросов**

> _Запрос_
```
db.posts_data.find(
    {Title : { $regex : "MongoDB", $options : "i" }},
    {Title: 1, Tags: 1}
)
```

> _Вывод_

| \_id | Tags | Title |
| :--- | :--- | :--- |
| 6228fc3b81fd6c706478f2be | &lt;database&gt;&lt;mongodb&gt;&lt;couchdb&gt; | MongoDB or CouchDB - fit for production? |
| 6228fc4881fd6c70647c1990 | &lt;mongodb&gt; | Updating nested documents in mongodb |
| 6228fc4b81fd6c70647c90f6 | &lt;c++&gt;&lt;visual-studio&gt;&lt;visual-c++&gt;&lt;mongodb&gt; | MongoDB and visual C++ 2008 linker errors |
| 6228fc5481fd6c70647eafdc | &lt;mongodb&gt;&lt;uniqueidentifier&gt;&lt;nosql&gt; | Unique IDs with mongodb |
| 6228fc5981fd6c70647fdafe | &lt;php&gt;&lt;json&gt;&lt;symfony1&gt;&lt;mongodb&gt; | Symfony \(PHP framework\) and MongoDB \( or any json-based database\) |
| 6228fc5981fd6c70647fe6a1 | &lt;python&gt;&lt;django&gt;&lt;mongodb&gt; | Custom storage system for GridFS \(MongoDB\)? |
| 6228fc5a81fd6c7064804089 | &lt;mysql&gt;&lt;mongodb&gt; | When to use MongoDB or other document oriented database systems? |
| 6228fc6881fd6c706482d9c0 | &lt;mongodb&gt;&lt;sphinx&gt; | using sphinx search with mongodb as datasource |
| 6228fc6a81fd6c7064831a8c | &lt;mongodb&gt;&lt;nosql&gt; | How to run this query in mongodb? |
| 6228fc6c81fd6c70648399f6 | &lt;mongodb&gt;&lt;document&gt;&lt;nosql&gt; | MongoDB: How to update multiple documents with a single command? |

> _Запрос_
```
db.posts_data.find(
{
    Score : { $gte : 1000 },
    CreationDate : { $lte : ISODate("2009-01-01T23:59:59Z")},
},
{
    Title: 1, Score: 1, CreationDate: 1
}
).sort({Score : -1}).limit(10)
```
> _Вывод_

| \_id | CreationDate | Score | Title |
| :--- | :--- | :--- | :--- |
| 6228fc1181fd6c7064709242 | 2008-10-07T11:50:34.630Z | 14653 | null |
| 6228fc1481fd6c706471266e | 2008-10-23T18:48:44.797Z | 11858 | null |
| 6228fc1681fd6c706471d349 | 2008-11-15T06:51:09.660Z | 9894 | What is the difference between 'git pull' and 'git fetch'? |
| 6228fc1681fd6c706471d34b | 2008-11-15T06:52:40.267Z | 8295 | null |
| 6228fc1181fd6c7064709038 | 2008-10-07T09:30:22.217Z | 8169 | null |
| 6228fc1a81fd6c7064726f6a | 2008-12-07T19:30:48.340Z | 8152 | null |
| 6228fc1481fd6c706471262e | 2008-10-23T18:21:11.623Z | 8091 | What does the "yield" keyword do? |
| 6228fc0c81fd6c70646f5f11 | 2008-09-13T08:30:26.950Z | 7806 | null |
| 6228fc1181fd6c706470922f | 2008-10-07T11:44:47.920Z | 7682 | How to modify existing, unpushed commits? |
| 6228fc0e81fd6c70646fdd05 | 2008-09-21T10:12:07.003Z | 7654 | How do JavaScript closures work? |

**Индексы**
Созданы в MongoDB Compass
| key | name | v | background | default\_language | language\_override | textIndexVersion | weights |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| {"\_id": 1} | \_id\_ | 2 | null | null | null | null | null |
| {"\_fts": "text", "\_ftsx": 1} | Title\_text\_Tags\_text | 2 | false | english | language | 3 | {"Tags": 1, "Title": 1} |
| {"FavoriteCount": -1} | FavoriteCount\_-1 | 2 | false | null | null | null | null |
| {"CreationDate": -1} | CreationDate\_-1 | 2 | false | null | null | null | null |

**Время работы запросов**
Все запросы далее вида: `db.posts_data.find( ... ).explain("executionStats")`
| Запрос | Код | Без индексов | С индексами |
| --- | --- | --- | --- |
| Все документы в коллекции | `db.posts_data.find().explain("executionStats")` | 208.071s | 98.446s|
| Документы с фразой "MongoDB" в заголовке| `Title: { $regex : "MongoDB" }` | 374.329s| 27.573s|
| Документы с фразой "MongoDB" в заголовке или тегах, без учёта регистра | `$or:[{ Title : { $regex : "MongoDB", $options : "i" }, }, { Tags : { $regex : "MongoDB", $options : "i" }}]}`| 802.965s | 295.516s|
| Значение FavoriteCount >= 1000 | `FavoriteCount : { $gte : 1000 }`| 149.753s | 6.228s |
| Значение FavoriteCount >= 1000 и дата создания до 2009-01-01| `FavoriteCount : { $gte : 1000 }, CreationDate : { $lte : ISODate("2009-01-01T23:59:59Z")}`| 122.947s | 0.555s |
| В заголовке есть пара фигурных кавычек| `Title : {$regex : "{.*?}"}`| 132.628s | 350.112s|

В результате можно заметить, что добавление индексов значительно ускорило большинство запросов

Работа выполнена: _Савкиным Максимом Константиновичем_, группа: _Б05-927_
