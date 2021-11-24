Вы можете перенести кластер {{ MS }} в кластер {{ mms-short-name }} с помощью [логического импорта](#snapshot) снапшота базы данных или [транзакционной репликации](#replication). Оба способа имеют свои ограничения:

- Миграция с помощью логического импорта:
  - В процессе создания снапшота используются блокировки таблиц.
  - Процесс импорта снапшота проходит медленно по сравнению с транзакционной репликацией.
  - Данные в кластере-приемнике будут актуальны только на момент создания снапшота и могут устареть.
  - Этот способ не подходит для баз данных с логическими неконсистентностями — например, если в ней удалены зависимости или таблицы, на которые ссылаются представления. Если это ограничение критично, вместо этого вы можете создать [моментальный снимок базы данных](https://docs.microsoft.com/ru-ru/sql/relational-databases/databases/create-a-database-snapshot-transact-sql) и направить его в [службу поддержки]({{ link-console-support }}), чтобы наши специалисты восстановили базу из снапшота вручную.

- Миграция с помощью транзакционной репликации:
  - Однонаправленность — изменения на кластере-приемнике не будут реплицироваться в кластер-источник.
  - Во время работы публикации нельзя менять схему на кластере-приемнике.

## Миграция с помощью логического импорта {#snapshot}

Перенести данные из кластера {{ MS }} в кластер {{ mms-short-name }} можно с помощью программы [sqlpackage](https://docs.microsoft.com/ru-ru/sql/tools/sqlpackage/sqlpackage-download). Она создаст файл снапшота базы {{ MS }} со схемой и данными таблиц и других объектов, а затем импортирует его в кластер {{ mms-short-name }}.

Чтобы мигрировать базу данных из кластера-источника {{ MS }} в кластер-приемник {{ mms-name }} с помощью логического импорта:

1. [Настройте кластер-источник](#configure-source).
1. [Настройте кластер-приемник](#configure-target).
1. [Создайте снапшот базы данных из кластера-источника](#create-snapshot).
1. [Импортируйте снапшот базы данных в кластер-приемник](#import-snapshot).

### Перед началом работы {#before-you-begin-snapshot}

1. [Создайте кластер {{ mms-name }}](../../managed-sqlserver/operations/cluster-create.md). При этом:

   - Хост-мастер кластера-приемника должен находиться в публичном доступе для подключения к нему программы sqlpackage.
   - В кластере-приемнике должен использоваться тот же **SQL Server Collation**, что и в кластере-источнике.

1. Проверьте, что вы можете [подключиться к кластеру-приемнику](../../managed-sqlserver/operations/connect.md#connection-ide) и к кластеру-источнику с помощью {{ ssms }}.

1. [Скачайте и установите программу sqlpackage](https://docs.microsoft.com/ru-ru/sql/tools/sqlpackage/sqlpackage-download).

### Настройте кластер-источник {#configure-source}

1. Удалите в базе-источнике всех пользователей, использующих **Windows Authentication**, оставив только пользователей с **SQL Server Authentication**.

1. Выдайте пользователю `sa` роль `db_owner` в той базе, которую собираетесь переносить, с помощью запроса:

    ```sql
    ALTER AUTHORIZATION ON DATABASE::<имя базы> TO sa;
    ```

1. Включите в базе-источнике компонент **Service Broker** и переключите ее модель восстановления в режим **Full**:
   1. Откройте {{ ssms }}.
   1. Откройте контекстное меню нужной базы данных и выберите пункт **Properties**.
   1. Перейдите на вкладку **Options**.
   1. Поменяйте значение опции **Recovery model** на **Full**.
   1. В блоке **Service Broker** поменяйте значение **Broker Enabled** на **True**.

### Настройте кластер-приемник {#configure-target}

[Добавьте](../operations/cluster-users#adduser) в кластере {{ mms-name }} всех пользователей, которые есть в базе-источнике, с теми же именами и паролями.

### Создайте снапшот базы данных {#create-snapshot}

Чтобы экспортировать базу данных в файл .dacpac, запустите PowerShell, перейдите в директорию с программой sqlpackage и выполните команду:

   ```powershell
   .\sqlpackage.exe `
       /a:Extract `
       /ssn:"<адрес кластера-источника {{ MS }}>" `
       /sdn:"<имя базы данных>" `
       /tf:"<локальный путь к файлу .dacpac>" `
       /p:ExtractAllTableData=True `
       /p:ExtractReferencedServerScopedElements=False
   ```

{% note info %}

Чтобы экспортировать только схему таблиц без самих данных, уберите из команды параметр `/p:ExtractAllTableData=False`. Если нужно экспортировать только определенные таблицы, укажите параметр `/p:TableData=<имя таблицы>` (этот параметр можно указывать несколько раз).

{% endnote %}

Подробнее см. в [документации {{ MS }}](https://docs.microsoft.com/ru-ru/sql/tools/sqlpackage/sqlpackage-extract).

### Импортируйте снапшот базы данных в кластер-приемник {#import-snapshot}

Чтобы импортировать снапшот в кластер {{ mms-name }}, запустите PowerShell, перейдите в директорию с программой sqlpackage и выполните команду:

   ```powershell
   .\sqlpackage.exe `
       /a:Publish `
       /sf:"<локальный путь к файлу .dacpac>" `
       /tsn:"<FQDN хоста-мастера в кластере {{ mms-name }}>,1433" `
       /tdn:"<имя целевой базы данных>" `
       /tec:True `
       /ttsc:True `
       /tu:"<имя пользователя целевой БД с ролью db_owner>" `
       /tp:"<пароль>" `
       /p:AllowIncompatiblePlatform=True `
       /p:IgnoreCryptographicProviderFilePath=True `
       /p:IgnoreExtendedProperties=True `
       /p:IgnoreFileAndLogFilePath=True `
       /p:IgnoreFilegroupPlacement=True `
       /p:IgnoreFileSize=True `
       /p:IgnoreFullTextCatalogFilePath=True `
       /p:IgnoreLoginSids=True `
       /p:ScriptRefreshModule=False
   ```

Подробнее см. в [документации {{ MS }}](https://docs.microsoft.com/ru-ru/sql/tools/sqlpackage/sqlpackage-publish).

## Миграция с помощью транзакционной репликации {#replication}

Перенести данные из кластера {{ MS }} в кластер {{ mms-name }} с минимальным временем простоя можно с помощью [репликации транзакций (transactional replication)](https://docs.microsoft.com/ru-ru/sql/relational-databases/replication/transactional/transactional-replication). Такой тип репликации поддерживается с 2016-й версии {{ MS }} и позволяет мигрировать данные на более поздние версии SQL Server в кластере {{ mms-name }}.

При репликации транзакций:

* При инициализации [агент моментальных снимков](https://docs.microsoft.com/ru-ru/sql/relational-databases/replication/agents/replication-agents-overview#snapshot-agent) создает снапшот базы данных со схемой и файлами данных таблиц и других объектов и копирует его с [издателя (publisher)](https://docs.microsoft.com/ru-ru/sql/relational-databases/replication/publish/replication-publishing-model-overview#publisher) на [распространителя (distributor)](https://docs.microsoft.com/ru-ru/sql/relational-databases/replication/publish/replication-publishing-model-overview#distributor), который управляет переносом данных.

   {% note info %}
   
   Так как при создании снапшота используются блокировки таблиц, то рекомендуется выполнять инициализацию в период минимальных нагрузок на базу.
   
   {% endnote %}

*  [Агент чтения журнала (Log Reader Agent)](https://docs.microsoft.com/ru-ru/sql/relational-databases/replication/agents/replication-agents-overview#log-reader-agent) переносит транзакции из лога транзакций на распространителя.
* Снапшот и транзакции из распространителя переносятся с помощью [агента распространителя (Distribution Agent)](https://docs.microsoft.com/ru-ru/sql/relational-databases/replication/agents/replication-agents-overview#distribution-agent) к [подписчику (subscriber)](https://docs.microsoft.com/ru-ru/sql/relational-databases/replication/publish/replication-publishing-model-overview#subscribers).

В такой схеме издатель, распространитель и оба агента располагаются в кластере-источнике, а подписчик — в кластере-приемнике. Возможны и другие варианты распределения ролей, например выделенные сервера для распространителя.

{% note info %}

Выполнение объемных транзакций на кластере-источнике может замедлить репликацию.

{% endnote %}

Чтобы мигрировать базу данных из кластера-источника {{ MS }} в кластер-приемник {{ mms-name }} с помощью транзакционной репликации:

1. [Создайте публикацию на кластере-источнике](#create-publication).
1. [Создайте подписку на кластере-источнике](#create-subscription).
1. [Остановите процесс репликации и перенесите нагрузку](#transfer-load).

Если созданные ресурсы вам больше не нужны, [удалите их](#clear-out).

### Перед началом работы {#before-you-begin-replication}

1. [Создайте кластер {{ mms-name }}](../../managed-sqlserver/operations/cluster-create.md) любой подходящей конфигурации. При этом:

    * Укажите то же имя базы данных, что и на кластере-источнике. 
    * Хост-мастер кластера-приемника должен находиться в публичном доступе для подключения к нему кластера-источника.
    * Версия {{ MS }} должна быть не ниже, чем на кластере-источнике.

1. Проверьте, что вы можете [подключиться к кластеру-приемнику](../../managed-sqlserver/operations/connect.md#connection-ide) и к кластеру-источнику с помощью {{ ssms }}.

### Создайте публикацию на кластере-источнике {#create-publication}

1. Подключитесь к кластеру-источнику из {{ ssms }}.
1. Разверните список объектов сервера в Object Explorer.
1. Откройте контекстное меню для директории **Replication** и выберите **New → Publication**.
1. Пройдите все шаги мастера создания публикации, в том числе:
   1. Укажите, что сам сервер будет выступать в качестве распространителя.
   1. Укажите директорию для снапшота базы данных.
   1. Выберите базу данных, которую необходимо мигрировать.
   1. Выберите тип публикации **Transactional replication**.
   1. Выберите в списке статей для публикации все сущности базы данных, которые требуется реплицировать (таблицы, представления, хранимые процедуры).
   1. Выберите время создания снапшота базы данных.
   
   Подробнее см. в [документации {{ MS }}](https://docs.microsoft.com/ru-ru/sql/relational-databases/replication/publish/create-a-publication).

1. Нажмите кнопку **Finish**.

{% note info %}

* При переносе нескольких баз для каждой из них создайте отдельную публикацию.

* Вам потребуются права привилегированного пользователя на все таблицы, выбранные для публикации.

{% endnote %}

### Создайте подписку на кластере-источнике {#create-subscription}

1. Подключитесь к кластеру-источнику из {{ ssms }}.
1. Разверните список объектов сервера в **Object Explorer**.
1. Откройте контекстное меню для директории **Replication** и выберите **New → Subscription**.
1. Пройдите все шаги мастера создания подписки, в том числе:
   1. Для агента-распространителя выберите **Run all agents at the Distributor** (запуск на кластере-источнике).
   1. Добавьте подписчика и укажите [данные для подключения к хосту-мастеру кластера-приемника](../../managed-sqlserver/operations/connect.md#connection-ide).
   1. Выберите для подписчика базу данных кластера-приемника.
   1. В параметрах безопасности **Distribution Agent Security** в блоке **Connect to the Subscriber** укажите имя и пароль учетной записи владельца базы данных на кластере-приемнике.

   Подробнее см. в [документации {{ MS }}](https://docs.microsoft.com/ru-ru/sql/relational-databases/replication/create-a-push-subscription).

 1. Нажмите кнопку **Finish**.

Запустится процесс репликации. Чтобы следить за его статусом, [запустите монитор](https://docs.microsoft.com/ru-ru/sql/relational-databases/replication/monitor/start-the-replication-monitor) и [добавьте подписку для отслеживания](https://docs.microsoft.com/ru-ru/sql/relational-databases/replication/monitor/add-and-remove-publishers-from-replication-monitor).

### Остановите процесс репликации и перенесите нагрузку {#transfer-load}

1. Проверьте, что на кластере-приемнике доступны все перенесенные данные с кластера-источника.
1. Переведите кластер-источник в режим <q>только чтение</q>:
   1. Откройте {{ ssms }}.
   1. Откройте контекстное меню для реплицируемой базы данных, затем выберите пункт **Properties**.
   1. Выберите **Database Properties → Options** и в блоке **State** поменяйте значение **Database Read Only** на **True**.
1. [Остановите репликацию](https://docs.microsoft.com/ru-ru/sql/relational-databases/replication/agents/start-and-stop-a-replication-agent-sql-server-management-studio) на кластере-источнике.
1. Удалите подписку и публикацию на кластере-источнике. Подтвердите разрешение {{ ssms }} удалить подписчика на кластере-приемнике.
1. Перенесите нагрузку на кластер-приемник.

### Как удалить созданные ресурсы {#clear-out}

Чтобы перестать платить за созданные ресурсы, [удалите](../../managed-sqlserver/operations/cluster-delete.md) их.

Если вы зарезервировали статический публичный IP-адрес, [удалите](../../vpc/operations/address-delete.md) его.