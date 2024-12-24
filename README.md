# todo-app-tests-JAVA-
## Отчет о тестировании приложения `todo-app`

## 1. Подготовка окружения

### Установка Docker
Команда для проверки версии Docker
```bash
docker --version

### результат
Docker version 27.4.1, build b9d17ea

# Загрузка и запуск образа
1. Загружен архив todo-app.tar в рабочую папку
tar -xvzf tester-task.tar.gz
docker load < todo-app.tar

Результат
Loaded image: todo-app:latest

2. Проверка доступных образов
docker images

Результат
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
todo-app      latest    a0f434f1f1eb   2 months ago    7.1MB

3. Запуск контейнера
docker run -d -p 8080:4242 -e VERBOSE=1 todo-app:latest

Результат
CONTAINER ID   IMAGE             COMMAND   CREATED          STATUS          PORTS                                         NAMES
63758838a4be   todo-app:latest   "./app"   18 seconds ago   Up 17 seconds   0.0.0.0:8080->4242/tcp, [::]:8080->4242/tcp   serene_gates

4. Проверка состояний контейнера
docker ps

## 2. Функционльное тестирование API
### Проверка доступных маршрутов








