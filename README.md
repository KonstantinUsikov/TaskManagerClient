# Система управления задачами — Android-клиент

Android-приложение для ведения личного списка задач. Клиент хранит данные не локально, а на отдельном
сервере, с которым обменивается текстовыми сообщениями по TCP-сокету.

## Возможности

- Регистрация и вход по имени пользователя и паролю. Пароль хешируется SHA-256 на стороне клиента
  (`util/HashUtil.java`), на сервер передаётся только хеш.
- Две вкладки: **«Текущие задачи»** и **«Завершенные задачи»**. Задачи отображаются в таблице
  (название, описание, срок).
- Добавление задачи: название, описание и срок выполнения (выбирается в календаре).
- Редактирование задачи, отметка о выполнении или снятие отметки, удаление.

## Технологии

- Java 8
- Android SDK: `minSdk 26`, `targetSdk 34`, `compileSdk 34`
- Gradle 8.0, Android Gradle Plugin 8.1.1 (Kotlin DSL)
- AndroidX AppCompat, Material Components (`TabLayout` + `ViewPager`)
- [SortableTableView](https://github.com/ISchwarz23/SortableTableView) 2.8.1 (подключается через JitPack)

## Сборка и запуск

Требования:

- Android Studio (Giraffe или новее) с JDK 17
- Устройство или эмулятор с Android 8.0 (API 26) и выше
- Доступный сервер системы управления задачами (см. ниже)

Через Android Studio: клонируйте репозиторий, откройте папку проекта и запустите конфигурацию `app`.

Из командной строки:

```bash
git clone https://github.com/KonstantinUsikov/TaskManagerClient.git
cd TaskManagerClient
./gradlew assembleDebug        # в Windows: gradlew.bat assembleDebug
```

Готовый APK появится в `app/build/outputs/apk/debug/`.

## Настройка сервера

Адрес сервера задан в коде: `82.179.140.18:45125`. Он передаётся в `SocketManager.connectParallel(...)`
в следующих классах:

- `MainActivity`
- `LoginActivity`
- `RegisterActivity`
- `TasksFragment`
- `AddTaskActivity`
- `EditTaskActivity`

Чтобы подключиться к другому серверу, замените адрес и порт во всех этих местах.

## Структура проекта

```
app/src/main/java/com/example/taskmanager/
├── MainActivity.java          — стартовый экран: выбор «Войти» / «Зарегистрироваться», подключение к серверу
├── LoginActivity.java         — вход в систему
├── RegisterActivity.java      — регистрация нового пользователя
├── TasksActivity.java         — экран со вкладками текущих и завершённых задач
├── TasksFragment.java         — вкладка с таблицей задач
├── AddTaskActivity.java       — создание задачи
├── EditTaskActivity.java      — редактирование, выполнение и удаление задачи
└── util/
    ├── SocketManager.java              — TCP-соединение с сервером (отправка и приём сообщений)
    ├── HashUtil.java                   — хеширование пароля (SHA-256)
    ├── SampleFragmentPagerAdapter.java — адаптер вкладок для ViewPager
    └── TaskClickListener.java          — открытие задачи на редактирование по нажатию на строку таблицы
```

Переходы между экранами:

```
MainActivity → LoginActivity / RegisterActivity → TasksActivity (2 × TasksFragment)
                                                     ├→ AddTaskActivity
                                                     └→ EditTaskActivity
```

## Протокол обмена с сервером

Запрос — это имя команды и её аргументы, разделённые символом перевода строки `\n`.
В ответе строки разделяются `\n`, поля внутри строки — символом табуляции `\t`.
Даты передаются в формате `год-месяц-день` (например, `2023-10-19`).

| Команда | Аргументы | Ответ |
|---|---|---|
| `FindPersonByUsername` | `username` | `id\tusername\tpasswordHash` или `" "` (пробел), если пользователь не найден |
| `AddPerson` | `username`, `passwordHash` | `1` при успехе |
| `FindNotDoneTasksByPersonId` | `personId` | строки вида `id\tname\tdescription\tend_at` |
| `FindDoneTasksByPersonId` | `personId` | строки вида `id\tname\tdescription\tend_at` |
| `AddTask` | `personId`, `name`, `description`, `end_at` | `1` при успехе |
| `UpdateTask` | `taskId`, `name`, `description`, `end_at` | `1` при успехе |
| `UpdateTaskDoneAt` | `taskId`, `date` | `1` при успехе |
| `UpdateTaskNotDone` | `taskId` | `1` при успехе |
| `DeleteTask` | `taskId` | `1` при успехе |

Пример запроса на добавление задачи:

```
AddTask
1
Купить продукты
Молоко, хлеб
2023-10-25
```
