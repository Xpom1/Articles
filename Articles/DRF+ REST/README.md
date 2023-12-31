# DRF + REST
### Что такое REST?
REST (Representational state transfer) – это стиль архитектуры программного обеспечения для распределенных систем.

В общем случае REST является очень простым интерфейсом управления информацией без использования каких-то дополнительных внутренних прослоек. Каждая единица информации однозначно определяется глобальным идентификатором, таким как URL. Каждая URL в свою очередь имеет строго заданный формат.

Как происходит управление информацией сервиса – это целиком и полностью основывается на протоколе передачи данных. Наиболее распространенный протокол конечно же HTTP. Так вот, для HTTP действие над данными задается с помощью методов: GET (получить), PUT (добавить, заменить), POST (добавить, изменить, удалить), DELETE (удалить). Таким образом, действия CRUD (Create-Read-Update-Delete) могут выполняться как со всеми 4-мя методами, так и только с помощью GET и POST.

Вот как это будет выглядеть на примере:

- GET /book/ — получить список всех книг
- GET /book/3/ — получить книгу номер 3
- PUT /book/ — добавить книгу (данные в теле запроса)
- POST /book/3 – изменить книгу (данные в теле запроса)
- DELETE /book/3 – удалить книгу

Архитектура REST очень проста в плане использования. По виду пришедшего запроса сразу можно определить, что он делает, не разбираясь в форматах. Данные передаются без применения дополнительных слоев, поэтому REST считается менее ресурсоемким, поскольку не надо парсить запрос чтоб понять что он должен сделать и не надо переводить данные из одного формата в другой.

### Что такое Django rest framework (DRF)?

DRF - это мощный набор инструментов для создания веб-сервисов и API на основе фреймворка Django. Он является одним из наиболее широко используемых для создания RESTful API в экосистеме Django.

### Для чего нужен DRF? 
- Для быстрой разработки API. Готовые компоненты значительно упрощают создание API. Это позволяет сосредоточиться на самой бизнес-логике, а не на написании основного кода с нуля.
- Для возможности иметь гибкую структуру, которая позволяет разработчикам выбирать наиболее подходящее решение под конкретную задачу.
- Для использования различных методов аутентификации. DRF предоставляет множество способов авторизации, таких как аутентификация на основе токена, OAuth, JWT и другие.
- Для сериализации данных. Мощные инструменты сериализации и десериализации данных API позволяют преобразовывать объекты моделей Django и другие данные в различные форматы данных, такие как JSON или XML, и обратно.
- Для валидации и обработки данных. С помощью инструментов для обработки входных данных можно определить правила валидации в соответствии с требованиями приложения. 
- Для автоматической генерации документации API на основе кода. DRF поддерживает использование инструментов (например, Swagger) для создания более детальной и интерактивной документации.
 
