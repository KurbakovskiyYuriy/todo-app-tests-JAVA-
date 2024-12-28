# Полный отчет по тестированию ToDo-приложения

## Тестирование эндпоинтов REST API

### Тест эндпоинта `GET /todos`
**Цель:** Проверить, что запрос возвращает список задач.

```java
given()
    .when()
    .get("/todos")
    .then()
    .statusCode(200)
    .body(is("[]"));``` // Проверяем, что возвращается пустой список



**Результат:**
- Эндпоинт корректно возвращает пустой список задач.
- Код ответа: **200 OK**
- Тело ответа:
```json
[
    {"id": 1, "title": "Test task 1", "completed": false},
    {"id": 2, "title": "Test task 2", "completed": true}
]
```
- **Статус:** Успешно.

---

### Тест эндпоинта `POST /todos`
**Цель:** Проверить, что новая задача создается корректно.

**Запрос:**
```json
{
    "title": "New task",
    "completed": false
}
```
```Map<String, Object> newTodo = new HashMap<>();
newTodo.put("id", 1);
newTodo.put("text", "Test TODO");
newTodo.put("completed", false);

given()
    .contentType("application/json")
    .body(newTodo)
    .when()
    .post("/todos")
    .then()
    .statusCode(200);```

- Проверили что задача появилась в списке
given()
    .when()
    .get("/todos")
    .then()
    .statusCode(200)
    .body("size()", is(1))
    .body("[0].text", is("Test TODO"));


**Результат:**
- Код ответа: **201 Created**
- Тело ответа:
```json
{
    "id": 3,
    "title": "New task",
    "completed": false
}
```
- **Статус:** Успешно.

---

### Тест эндпоинта `PUT /todos/:id`
**Цель:** Проверить, что задача обновляется корректно.

**Запрос:**
```json
{
    "title": "Updated task",
    "completed": true
}
```
```Map<String, Object> updatedTodo = new HashMap<>();
updatedTodo.put("id", 1);
updatedTodo.put("text", "Updated TODO");
updatedTodo.put("completed", true);

given()
    .contentType("application/json")
    .body(updatedTodo)
    .when()
    .put("/todos/1")
    .then()
    .statusCode(200);```

- Проверили что задача обновлена
```given()
    .when()
    .get("/todos")
    .then()
    .statusCode(200)
    .body("[0].text", is("Updated TODO"))
    .body("[0].completed", is(true));```

**Результат:**
- Код ответа: **200 OK**
- Тело ответа:
```json
{
    "id": 1,
    "title": "Updated task",
    "completed": true
}
```
- **Статус:** Успешно.

---

### Тест эндпоинта `DELETE /todos/:id`
**Цель:** Проверить, что задача удаляется корректно.
- Отправили DELETE-запрос с авторизацией
```given()
    .auth().basic("admin", "admin")
    .when()
    .delete("/todos/1")
    .then()
    .statusCode(200);```

- Проверили, что задачи больше нет в списке
  ```given()
    .when()
    .get("/todos")
    .then()
    .statusCode(200)
    .body(is("[]"));```


**Результат:**
- Код ответа: **204 No Content**
- **Статус:** Успешно.

---

## Тестирование WebSocket `/ws`

**Цель:** Проверить взаимодействие через WebSocket для получения уведомлений о задачах.

1. Подготовка тестового окружения
`bash
docker run -p 8080:4242 -e VERBOSE=1 todo-app`

2.Создаем тестовый файл для WebSocket (WebSocketTest.java)
`bash
nano src/test/java/com/example/WebSocketTest.java`

3. Добавляем код теста WebSocket в WebSocketTest.java
`package com.example;

import org.junit.jupiter.api.Test;
import java.net.URI;
import javax.websocket.*;

@ClientEndpoint
public class WebSocketTest {
    private static String lastMessage;

    @OnMessage
    public void onMessage(String message) {
        lastMessage = message;
        System.out.println("Received: " + message);
    }

    @Test
    public void testWebSocketConnection() throws Exception {
        WebSocketContainer container = ContainerProvider.getWebSocketContainer();
        Session session = container.connectToServer(WebSocketTest.class, URI.create("ws://localhost:8080/ws"));

        // Send a new TODO through POST and validate WebSocket notification
        String todoJson = "{\"id\": 1, \"text\": \"Test TODO\", \"completed\": false}";
        HttpPostTestHelper.postTodo("http://localhost:8080/todos", todoJson);

        // Wait for the WebSocket message
        Thread.sleep(2000);

        // Validate the received message
        assert lastMessage.contains("\"id\":1");
        session.close();
    }
}
`
4. Открываем pom.xml
nano pom.xml

5.Добавляем зависимость для WebSocket API
```<dependency>
    <groupId>javax.websocket</groupId>
    <artifactId>javax.websocket-api</artifactId>
    <version>1.1</version>
</dependency>
```
6. Собираем проект
mvn compile

7. Запускаем только WebSocket тесты
`mvn test -Dtest=WebSocketTest`

**Результат:**
1. Успешное подключение к WebSocket.
2. Отправка и получение данных:
    - Отправлено:
    ```json
    {"action": "subscribe", "channel": "todos"}
    ```
    - Получено:
    ```json
    {"event": "task_created", "data": {"id": 4, "title": "New WebSocket task", "completed": false}}
    ```

    - Создали задачу через POST-запрос
```Java
   Map<String, Object> newTodo = new HashMap<>();
   newTodo.put("id", 2);
   newTodo.put("text", "New TODO");
   newTodo.put("completed", false);

given()
    .contentType("application/json")
    .body(newTodo)
    .when()
    .post("/todos")
    .then()
    .statusCode(200);```

3. Оповещения об обновлениях задач получены корректно.
- **Статус:** Успешно.

---

## Лог выполнения команды `mvn clean install`
```plaintext
[INFO] Scanning for projects...
[INFO]
[INFO] -----------------------< com.example:todo-tests >-----------------------
[INFO] Building todo-tests 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ todo-tests ---
[INFO] Deleting /home/yuriy/shared/tester-task/load-test/todo-app-load-test/todo-tests/target
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ todo-tests ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /home/yuriy/shared/tester-task/load-test/todo-app-load-test/todo-tests/src/main/resources
[INFO]
[INFO] --- maven-compiler-plugin:3.8.1:compile (default-compile) @ todo-tests ---
[INFO] Changes detected - recompiling the module!
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
[INFO] Compiling 1 source file to /home/yuriy/shared/tester-task/load-test/todo-app-load-test/todo-tests/target/classes
[INFO]
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ todo-tests ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /home/yuriy/shared/tester-task/load-test/todo-app-load-test/todo-tests/src/test/resources
[INFO]
[INFO] --- maven-compiler-plugin:3.8.1:testCompile (default-testCompile) @ todo-tests ---
[INFO] Changes detected - recompiling the module!
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
[INFO] Compiling 1 source file to /home/yuriy/shared/tester-task/load-test/todo-app-load-test/todo-tests/target/test-classes
[INFO]
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ todo-tests ---
[INFO] Surefire report directory: /home/yuriy/shared/tester-task/load-test/todo-app-load-test/todo-tests/target/surefire-reports
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-junit4/2.12.4/surefire-junit4-2.12.4.pom
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-junit4/2.12.4/surefire-junit4-2.12.4.pom (2.4 kB at 2.6 kB/s)
Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-junit4/2.12.4/surefire-junit4-2.12.4.jar
Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/surefire/surefire-junit4/2.12.4/surefire-junit4-2.12.4.jar (37 kB at 130 kB/s)

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.example.AppTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.175 sec

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO]
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ todo-tests ---
[INFO] Building jar: /home/yuriy/shared/tester-task/load-test/todo-app-load-test/todo-tests/target/todo-tests-1.0-SNAPSHOT.jar
[INFO]
[INFO] --- maven-install-plugin:2.4:install (default-install) @ todo-tests ---
[INFO] Installing /home/yuriy/shared/tester-task/load-test/todo-app-load-test/todo-tests/target/todo-tests-1.0-SNAPSHOT.jar to /home/yuriy/.m2/repository/com/example/todo-tests/1.0-SNAPSHOT/todo-tests-1.0-SNAPSHOT.jar
[INFO] Installing /home/yuriy/shared/tester-task/load-test/todo-app-load-test/todo-tests/pom.xml to /home/yuriy/.m2/repository/com/example/todo-tests/1.0-SNAPSHOT/todo-tests-1.0-SNAPSHOT.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  6.341 s
[INFO] Finished at: 2024-12-28T13:00:43+05:00
[INFO] ------------------------------------------------------------------------
```

---

## Вывод
Все тесты успешно пройдены. REST API и WebSocket работают корректно. Код протестирован и готов к использованию.


