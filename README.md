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
|  `vector_start_systemd`        | Параметр применения systemd в системе  | `true`                        |
|  `vector_check_service`        | Параметр доступности clickhouse-server | `true`                        |

vector_check_service - при использовании данного параметра как true, необходимо наличие уже подготовленного clickhouse-server с возможностью авторизации, можно использовать готовый официальный docker образ "clickhouse/clickhouse-server:latest, см. раздел Tox. Он проверяет доступность clickhouse-server ```curl <IP>:8123```
vector_start_systemd - параметр используется для определения запуска Vector, true - управление через systemd, false - shell запуск.

Environment
----------------
При формировании конфигурационного файла Vector через шаблон vector.yaml.j2, используется переменные из переменного окружения.
```yml
user: ${CLICKHOUSE_USER}
password: ${CLICKHOUSE_PASSWORD}
```
Это необходимо в случаи правил авторизации на clickhouse-server. Перед началом применения данной роли необходимо, определить переменные в переменном окружении.
```bash
export CLICKHOUSE_USER="<пользователь clickhouse-server>"
export CLICKHOUSE_PASSWORD="<пароль clickhouse-server>" 
```

Example Playbook
----------------

    - hosts: servers
      roles:
         - { role: vector }

Molecule
-------
Molecule выполняет тестирование deploy Vector с использованием Docker.

Vector разворачивается на 3-х images:
  - name: molecule-netology
    image: rockylinux:9
  - name: molecule-oracle
    image: oraclelinux:9
  - name: molecule-ubuntu
    image: ubuntu:latest  

Выполняет следующие сценария:
scenario:
  name: default
  test_sequence:
    - dependency
    - cleanup
    - destroy
    - syntax
    - create
    - prepare
    - converge
    - idempotence
    - verify
    - idempotence
    - cleanup
    - destroy

Для выполнения deploy in docker используется collections community.docker
```yml
---
collections:
  - name: community.docker
    version: ">=3.10.4"
```

В файле molecule.yml выполнено переопределение path role  
```ini
roles_path: /<путь до размещения директории с ролью или ролями где находиться Vector>/
```

Molecule пример тестирования
-------
1. Добавляем авторизационные данные для clickhouse в переменное окружение (можно не добавлять, а указать их в момент передачи env в контенер)
```bash
export CLICKHOUSE_USER="admin" && export CLICKHOUSE_PASSWORD="admin"
```
2. Поднимаем docker container clickhouse с дополнительными аргументами env и port forward
   ```bash
   docker run --rm -d -it -e CLICKHOUSE_USER="$CLICKHOUSE_USER" -e CLICKHOUSE_PASSWORD="$CLICKHOUSE_PASSWORD" -p 8127:8123 -p 9001:9000 clickhouse/clickhouse-server:latest
   ```
3. Проверяем доступность clickhose по протоколу http на порт 8127 -> 8123
```bash
 curl 127.0.0.1:8127
```
4. Пробуем подключится к native clickhouse на порт 9001 -> 9000 (clickhouse-client должен предварительно установлен на control node)
```bash
clickhouse-client --host 127.0.0.1 --port 9001 --user admin --password admin
```
5. Получим информацию об IP container clickhouse в docker networ bridge.
```bash
docker inspect <ID_container>  --format '{{json .NetworkSettings.Networks}}' | jq
```

<img width="1833" height="1223" alt="Vector" src="https://github.com/user-attachments/assets/7357ccc2-4212-4ec5-a7cc-30f5d25a2683" />

Tox
-------
Выполнение тестирования запуска Molecule в разных версия виртуального окружения.
Тестирование производилось в окружении python3.12, выполнение molecule test на версии ansible-core 2.20.5 и минимальной версии 2.17.0.

Перед запуском тестирования через Tox измените, при необходимости, параметры в tox.ini переменных для передачи в переменное окружение при создании виртуального окружения
```ini
setenv =
    CLICKHOUSE_USER = admin
    CLICKHOUSE_PASSWORD = admin
```
Также,чтобы при проверки валидации конфигурационного файла Vector, валидация не ушла в ошибку, должен быть предварительно подготовленный clickhouse-server с возможностью авторизации, можно использовать готовый официальный docker образ "clickhouse/clickhouse-server:latest".
```bash
 docker run --rm -d -it -e CLICKHOUSE_USER=admin -e CLICKHOUSE_PASSWORD=admin -p 8127:8123 clickhouse/clickhouse-server:latest
```
При использовании docker образа необходимо указать port forward, например ```-p 8127:8123 ``` и изменить порт по умолчанию в переменной ```vector_clickhouse_http_port``` на порт через который происходит транспорт до clickhouse-server. 
Для определения IP адреса clickhouse-server для конфигурационного файла, необходимо в defaults/main.yml (переменные по умолчанию) прописать адрес сервера 
```yml
vector_clickhouse_http_host: "<IP clickhouse-server>"
```

License
-------

MIT

Author Information
------------------
Max Maxi is a devops student
