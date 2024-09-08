## <a name="application_lifecycle"></a>Жизненный цикл приложения в Nest.js (Lifecycle events)

Жизненный цикл можно разделить на 3 стадии: инициализация, выполнение (запуск) и завершение. События жизненного цикла возникают в процессе запуска и завершения работы приложения. NestJS вызывает методы хуков, зарегистрированные на `modules`, `injectables` и `controllers` для каждого события.

![application_lifecycle](../pictures/lifecycle-events.png)

-   **onModuleInit** — вызывается один раз после разрешения зависимостей модуля;
-   **onApplicationBootstrap** — вызывается один раз после инициализации модулей, но до регистрации обработчиков установки соединения;
-   **onModuleDestroy** — вызывается после получения сигнала о завершении работы (например, SIGTERM);
-   **beforeApplicationShutdown** — вызывается после **onModuleDestroy**; после завершения (разрешения или отклонения промисов), все существующие подключения закрываются (вызывается app.close());
-   **onApplicationShutdown** — вызывается после закрытия всех подключений (разрешения app.close()).

## <a name="request_lifecycle"></a>Request lifecycle в Nest.js

![application_lifecycle](../pictures/request_lifecycle.png)
