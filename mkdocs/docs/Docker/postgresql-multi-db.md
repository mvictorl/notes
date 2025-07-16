# Создание контейнера PostgreSQL с несколькими БД

При создании контейнера PostgreSQL с помощью `yaml` файла Docker Compose переменная `POSTGRES_DB` из блока `environment` позволяет указать имя базы данных, которая будет создана на данном сервере. Однако, если понадобиться создать вторую (и более) БД, то стандартными средствами Docker Compose этого сделать не получится.

В этом случае можно использовать инициализационные скрипты.

Ниже пример `docker-compose.yaml` файла для создания контейнера PostgreSQL с несколькими БД:

```yaml
services:
  {{ SERVICE_NAME }}-db:
    image: postgres:latest
    container_name: {{ CONTAINER_NAME }}
    ports:
        - 5432:5432
    volumes:
        - {{ VOLUME_NAME }}-data:/var/lib/postgresql/data
        - ./init/create-multiple-postgresql-databases.sh:/docker-entrypoint-initdb.d/create-multiple-postgresql-databases.sh
    environment:
        POSTGRES_MULTIPLE_DATABASES: "{{ DATABASE_1 }}, {{ DATABASE_2 }}, {{ DATABASE_3 }}"
        POSTGRES_USER: {{ DB_USER }}
        POSTGRES_PASSWORD: {{ PASSWORD }}
    networks:
      - {{ NETWORK_NAME }}

volumes:
 {{ VOLUME_NAME }}-data:
```

Необходимо вписать в файл необходимые значения для шаблонов в двойных фигурных скобках.

Имена баз данных задаются через запитую в переменной окружения `POSTGRES_MULTIPLE_DATABASES`.

Эти значения будут подставлены в скрипт `create-multiple-postgresql-databases.sh`, который монтируется, как том (`volume`) к директории контейнера `/docker-entrypoint-initdb.d/`, которая предназначениа для пользовательских скриптов инициализации БД в контейнере Docker.

Файлы скриптов (`*.sql`, `*.sql.gz` или `*.sh`) из этой директории будут выполнены, **только** если директория данных СУБД пуста.

Ниже представлен скрипт `create-multiple-postgresql-databases.sh` для создания нескольких БД, имена которых содержатся с переменной окружения `POSTGRES_MULTIPLE_DATABASES`:

```sh
#!/bin/bash

set -e
set -u

function create_database_grant_privilege() {
        local database=$1
        echo "  Creating database '$database' and granting privileges to '$POSTGRES_USER'"
        psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" <<-EOSQL
            CREATE DATABASE $database;
            GRANT ALL PRIVILEGES ON DATABASE $database TO $POSTGRES_USER;
EOSQL
}

if [ -n "$POSTGRES_MULTIPLE_DATABASES" ]; then
        echo "Multiple database creation requested: $POSTGRES_MULTIPLE_DATABASES"
        for db in $(echo $POSTGRES_MULTIPLE_DATABASES | tr ',' ' '); do
                create_database_grant_privilege $db
        done
        echo "Multiple databases created"
fi
```
