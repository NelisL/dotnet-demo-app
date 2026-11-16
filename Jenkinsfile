pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                sh '''
                    NET="ci-net-$BUILD_NUMBER"
                    DB="ci-mysql-$BUILD_NUMBER"

                    cleanup() {
                      docker rm -f "$DB" >/dev/null 2>&1 || true
                      docker network rm "$NET" >/dev/null 2>&1 || true
                    }
                    trap cleanup EXIT

                    docker network create "$NET"

                    docker run -d --name "$DB" --network "$NET" \
                      -e MYSQL_ROOT_PASSWORD=test \
                      -e MYSQL_DATABASE=todo_test_db \
                      -e MYSQL_USER=todo_usr \
                      -e MYSQL_PASSWORD=letmeinplz \
                      mysql:8.4

                    i=0
                    until docker exec "$DB" mysqladmin ping -h127.0.0.1 -uroot -ptest --silent 2>/dev/null; do
                      i=$((i+1))
                      if [ "$i" -ge 60 ]; then
                        echo "MySQL werd niet op tijd klaar" >&2
                        docker logs "$DB"
                        exit 1
                      fi
                      sleep 2
                    done

                    docker exec -i "$DB" mysql -uroot -ptest todo_test_db <<'SQL'
CREATE TABLE IF NOT EXISTS todos (
  id INT AUTO_INCREMENT PRIMARY KEY,
  title VARCHAR(255) NOT NULL,
  is_done TINYINT(1) NOT NULL DEFAULT 0,
  created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);
SQL

                    docker run --rm \
                      --network "$NET" \
                      --volumes-from "$(hostname)" \
                      -w "$WORKSPACE" \
                      -e HOME=/tmp -e DOTNET_CLI_HOME=/tmp \
                      -e "ConnectionStrings__TodoDb=Server=$DB;Port=3306;Database=todo_test_db;User=todo_usr;Password=letmeinplz;" \
                      mcr.microsoft.com/dotnet/sdk:10.0 \
                      dotnet test TodoApp.Tests
                '''
            }
        }

        stage('Docker context') {
            steps {
                sh '''
                    mkdir -p ~/.ssh
                    ssh-keyscan -H 192.0.2.10 >> ~/.ssh/known_hosts
                    docker context rm -f remote || true
                    docker context create remote --docker "host=ssh://vagrant@192.0.2.10"
                '''
            }
        }

        stage('Preparation') {
            environment {
                DOCKER_CONTEXT = 'remote'
            }
            steps {
                sh '''
                    docker rm -f todoapp || true
                    docker rm -f todoappdb || true
                '''
            }
        }

        stage('Build & Deploy') {
            environment {
                DOCKER_CONTEXT = 'remote'
            }
            steps {
                sh '''
                    docker network create todo-net || true
                    docker volume create mariadb-data

                    docker run -d --name todoappdb --network todo-net --restart unless-stopped \
                      -e MARIADB_ROOT_PASSWORD=sekrit \
                      -e MARIADB_DATABASE=todo_db \
                      -e MARIADB_USER=todo_usr \
                      -e MARIADB_PASSWORD=letmeinplz \
                      -v mariadb-data:/var/lib/mysql \
                      mariadb:11

                    i=0
                    until docker exec todoappdb mariadb -h127.0.0.1 -utodo_usr -pletmeinplz todo_db -e "SELECT 1" >/dev/null 2>&1; do
                      i=$((i+1))
                      if [ "$i" -ge 30 ]; then
                        echo "MariaDB werd niet op tijd klaar" >&2
                        docker logs todoappdb
                        exit 1
                      fi
                      sleep 2
                    done

                    docker exec -i todoappdb mariadb -utodo_usr -pletmeinplz todo_db < ./TodoApp/schema.sql

                    docker build -t dotnet-demo-app ./TodoApp
                    docker run -d --name todoapp --network todo-net --restart unless-stopped \
                      -p 6060:8080 \
                      -e "ConnectionStrings__TodoDb=Server=todoappdb;Port=3306;Database=todo_db;User=todo_usr;Password=letmeinplz" \
                      dotnet-demo-app
                '''
            }
        }
    }
}
