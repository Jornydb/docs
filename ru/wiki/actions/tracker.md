# Задачи из {{ tracker-full-name }}

[{{ tracker-full-name }}]({{ link-tracker }}) — это сервис для управления проектами и процессами. Подробнее о возможностях сервиса читайте в [документации {{ tracker-full-name }}](../../tracker/).

## Ссылка на задачу {#ticket}

Вы можете вставлять на вики-страницы _магические ссылки_ на отдельные задачи и списки задач. Такие ссылки всегда содержат ключ, название, статус задачи и логин исполнителя. Чтобы вставить ссылку на задачу, скопируйте ее ключ и вставьте в текст страницы. 

## Список задач {#ticket-list}

Блок `not_var{{tasks}}` содержит список задач {{ tracker-name }}, которые входят в очередь или удовлетворяют определенному фильтру.

### Вызов блока {#task-call}

```
{{tasks url="адрес фильтра или очереди"}}
```

Дополнительных параметров у блока `not_var{{tasks}}` нет.

* Чтобы вывести все задачи очереди, в параметре `url` укажите ее [ключ](../../tracker/manager/create-queue.md#key) или ссылку из адресной строки браузера.

* Чтобы вывести [фильтр](../../tracker/user/create-filter.md) задач, в параметре `url` укажите ссылку на фильтр из адресной строки браузера. 

    {% note info %}

    Параметр `url` блока `tasks` не поддерживает символ `"`. Если в ссылке на фильтр из адресной строки встречается такой символ, замените его на `%22`.

    {% endnote %}