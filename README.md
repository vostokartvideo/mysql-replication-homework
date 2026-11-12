# MySQL — репликация и масштабирование БД
# Антон Сап

## Задание 1. 
- **Master–Slave (Source–Replica):** запись ведётся на основном сервере; изменения передаются на реплику. Используется в том числе для масштабирования чтения. При асинхронной репликации возможна задержка передачи изменений.
- **Master–Master:** каждый сервер реплицирует изменения другому. Запись на оба сервера требует предотвращения конфликтов; двусторонняя репликация сама по себе не обеспечивает автоматический failover.

Репликация не заменяет резервное копирование: ошибочные изменения тоже могут быть переданы на реплику.

## Задание 2. 

Два независимых контейнера MySQL в Docker-сети `mysql-replication-net`, каждый со своим томом данных.

| Сервер | Контейнер | `server-id` | Особенности |
|---|---|---:|---|
| Source | `mysql-master` | 1 | Binary Log, GTID |
| Replica | `mysql-replica` | 2 | Relay Log, GTID, `read_only=ON` |

Создан пользователь `repl` с привилегией `REPLICATION SLAVE`.

**Результаты проверки:**

1. `SHOW REPLICA STATUS\G`: `Replica_IO_Running: Yes`, `Replica_SQL_Running: Yes`, ошибок нет, `Seconds_Behind_Source: 0` на момент проверки.
2. Созданные на Source база `replication_lab` и таблица `employees` появились на Replica.
3. Операции `INSERT` и `UPDATE` успешно переданы на Replica.
4. После `STOP REPLICA` на Source добавлен сотрудник Sergey: Source содержал 4 строки, Replica — 3.
5. После `START REPLICA` недостающая запись появилась на Replica; потоки репликации восстановились.

### Скриншоты

| Этап | Подтверждение |
|---|---|
| Настройки Source | [Конфигурация Source](task6-2-1-master-config.jpg) |
| Настройки Replica | [Конфигурация Replica](task6-2-2-replica-config.jpg) |
| Репликация включена | [Статус репликации — часть 1](task6-2-3-replication-status.jpg) · [часть 2](task6-2-3-replication-status-2.jpg) |
| Создание базы | [База появилась на Replica](task6-2-4-database-replicated.jpg) |
| Добавление данных | [Репликация INSERT](task6-5-insert-replication.jpg) |
| Изменение данных | [Репликация UPDATE](task6-6-update-replication.jpg) |
| Остановка | [Потоки репликации остановлены](task6-7-replication-stopped.jpg) |
| Расхождение данных | [Source — 4 записи](task6-8-source-recovery.jpg) · [Replica — 3 записи](task6-9-replica-.jpg) |
| Восстановление | [Replica — 4 записи](task6-10.jpg) · [Статус после восстановления](task6-11.jpg) |