Vector
=========

Данная роль выполняет:
- Устанавливает сборщик журналов Vector
- Настраивает репозиторий
- Развертывает шаблон Vector конфигурации
- Собирает логи доступа nginx
- Отправляет логи в ClickHouse

Role Variables
--------------


|          Variable              |              Description               |     Default                   |
| ------------------------------ | -------------------------------------- | ----------------------------- |
|  `vector_repo_url`             | URL адрес репозитория                  | `https://packages.timber.io`  |
|  `vector_repo_arch`            | Тип архитектуры ОС                     | `x86_64`                      |
|  `vector_include_logs`         | Path parsing лога                      | `/var/log/nginx/my_access.log`|
|  `vector_conf_dest`            | Path до файла конфигурации на instance | `/etc/vector/vector.yaml`     |
|  `vector_clickhouse_http_host` | HTTP адрес clickhouse                  | `127.0.0.1`                   |
|  `vector_clickhouse_http_port` | HTTP порт clickhouse службы            | `8123`                        |
|  `vector_clickhouse_db_name`   | Имя БД на clickhouse                   | `nginx`                       |
|  `vector_clickhouse_table_name`| Имя таблицы в БД clickhouse            | `my_access_logs`              |


Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: vector }

License
-------

MIT

Author Information
------------------
Max Maxi is a devops student
