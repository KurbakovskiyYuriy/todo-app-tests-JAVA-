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

1. Получение списка задач
curl http://localhost:8080/todos

Результат
[]

2. Добавление задачи
curl -X POST -H "Content-Type: application/json" -d '{"id":1,"text":"Test TODO","completed":false}' http://localhost:8080/todos

Результат
HTTP 200 OK

3. Проверки добавленной задачи
curl http://localhost:8080/todos

Результат
[{"id":1,"text":"Test TODO","completed":false}]

4.Добавление второй задачи
curl -X POST -H "Content-Type: application/json" -d '{"id":2,"text":"Another TODO","completed":false}' http://localhost:8080/todos

Результат
HTTP 200 OK

5. Проверка добавленных задач
curl http://localhost:8080/todos

Результат
[{"id":1,"text":"Test TODO","completed":false},{"id":2,"text":"Another TODO","completed":false}]

## Проверка неподдерживаемых методов
### 1. Обновление задачи (PUT)
curl -X PUT -H "Content-Type: application/json" -d '{"id":1,"text":"Updated TODO","completed":true}' http://localhost:8080/todos

Результат
HTTP 405 Method Not Allowed

### 2. Удаление задачи (DELETE)
curl -X DELETE http://localhost:8080/todos/1

Результат
HTTP 405 Method Not Allowed

### 3. Проверка поддерживаемых методов
curl -X OPTIONS http://localhost:8080/todos

Результат
HTTP 200 OK
Allow: GET, POST

## 3. Выводы
1. Приложение корректно запускается в Docker-контейнере.
2. Поддерживаются только методы GET и POST. Методы PUT и DELETE не реализованы, что подтверждается статусом 405 Method Not Allowed.
3. Приложение корректно добавляет и возвращает задачи, однако обновление и удаление задач невозможно без доработки кода.

## 4. Рекомендации
1. Реализовать обработчики методов PUT и DELETE для обеспечения функционала обновления и удаления задач.
2. Провести повторное тестирование после реализации данных методов.
3. Добавить обработку ошибок и документацию для доступных маршрутов API.

_______________________________________________________________________________________________________________

# 5. Нагрузочное тестирование
## Шаги для проведения нагрузочного теста с использованием JMH (Java Microbenchmark Harness)

1. Создать новый проект для теста
mkdir -p ~/shared/tester-task/load-test
cd ~/shared/tester-task/load-test

2. Инициализировать Maven-проект
mvn archetype:generate -DgroupId=com.example -DartifactId=todo-app-load-test -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
cd todo-app-load-test

3. Добавить зависимости JMH
nano pom.xml

Добавьте в секцию <dependencies> следующее
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>1.37</version>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>1.37</version>
    <scope>provided</scope>
</dependency>

4. Создать JMH Benchmark
Создать файл для тестирования производительности
mkdir -p src/main/java/com/example
nano src/main/java/com/example/LoadTestBenchmark.java

Добавьте в файл следующий код
package com.example;

import org.openjdk.jmh.annotations.*;

import java.util.concurrent.TimeUnit;
import java.net.HttpURLConnection;
import java.net.URL;

@BenchmarkMode(Mode.Throughput)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
@State(Scope.Thread)
public class LoadTestBenchmark {

    @Benchmark
    public void testGetTodos() {
        try {
            URL url = new URL("http://localhost:8080/todos");
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            connection.setRequestMethod("GET");
            int responseCode = connection.getResponseCode();
            if (responseCode != 200) {
                throw new RuntimeException("Failed with response code: " + responseCode);
            }
        } catch (Exception e) {
            throw new RuntimeException("Error during GET request", e);
        }
    }
}


5. Собрать проект
mvn clean install

6. Запустить нагрузочное тестирование
java -jar target/benchmarks.jar

Запуск
package com.api.performance;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.stream.IntStream;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Test;

import static com.TodoAppMethods.createTaskWithDetails;
import static com.model.TodoTask.getTodoTask;
import static com.utils.UtilJava.cleanApp;
import static java.util.concurrent.TimeUnit.MINUTES;
import static org.junit.jupiter.api.Assertions.assertTrue;

public class PostTodoPerformanceTest {

    private static final int THREAD_COUNT = 5; 
    private static final int TASKS_PER_THREAD = 50; 

    @Test
    public void testPostTodoPerformance() throws InterruptedException {
        ExecutorService executor = Executors.newFixedThreadPool(THREAD_COUNT);
        List<Future<Long>> futures = new ArrayList<>();
        CountDownLatch latch = new CountDownLatch(THREAD_COUNT);

        IntStream.range(0, THREAD_COUNT).forEach(i -> futures.add(executor.submit(() -> {
            long totalTime = 0;
            latch.countDown();
            latch.await();

            for (int j = 0; j < TASKS_PER_THREAD; j++) {
                long startTime = System.nanoTime();
                createTaskWithDetails(getTodoTask(), "Detailed description " + j); 

                long endTime = System.nanoTime();
                totalTime += (endTime - startTime);
            }
            return totalTime;
        })));

        executor.shutdown();
        executor.awaitTermination(1, MINUTES);

        long totalRequests = THREAD_COUNT * TASKS_PER_THREAD;
        long totalTime = 0;
        long maxTime = 0;
        long minTime = Long.MAX_VALUE;

        for (Future<Long> future : futures) {
            try {
                long threadTime = future.get();
                totalTime += threadTime;
                maxTime = Math.max(maxTime, threadTime);
                minTime = Math.min(minTime, threadTime);
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }

        double averageTime = totalTime / (double) totalRequests;
        System.out.println("Total requests: " + totalRequests);
        System.out.println("Average time per request: " + (averageTime / 1_000_000) + " ms");
        System.out.println("Max time per thread: " + (maxTime / 1_000_000) + " ms");
        System.out.println("Min time per thread: " + (minTime / 1_000_000) + " ms");

        assertTrue(averageTime < 700_000_000, "Average request time is too high!"); // Порог 700ms
    }

    @AfterEach
    public void cleanUp() {
        cleanApp(); // Очистка приложения после каждого теста
    }
}

Результат




