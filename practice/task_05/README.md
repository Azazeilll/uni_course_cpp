# Задача 5

# 1. Разбить вашу программу на отдельные файлы

Разбить вашу программу на отдельные `header` и `source` файлы.
Соответственно у вас должна получиться примерно такая структура:
- `main.cpp`
- `graph.hpp`
- `graph.cpp`
- `graph_generator.hpp`
- `graph_generator.cpp`
- `graph_json_printing.hpp`
- `graph_json_printing.cpp`
- `graph_printing.hpp`
- `graph_printing.cpp`
- `logger.hpp`
- `logger.cpp`
- `config.hpp`

`.hpp` файлы включают в себя только декларацию (интерфейс).
Исключение только однострочные методы.

`.cpp` файлы включают в себя только имплементацию (реализацию).

Возьмём, к примеру, файлы `graph.hpp` и `graph.cpp`:
```cpp
// graph.hpp
class Graph {
  Vertex& add_vertex();
  Edge& add_edge(VertexId from_vertex_id,
                 VertexId to_vertex_id);
  int depth() const { return depth_map_.size(); }
  // ...
};
```
```cpp
// graph.cpp
#include "graph.hpp"

Vertex& Graph::add_vertex() {
  // function body
}

Edge& Graph::add_edge(VertexId from_vertex_id,
                      VertexId to_vertex_id) {
  // function body
}

// ...
```

Все свободные функции и константы, которые используются только локально и не имеют смысла как часть публичного интерфейса,
должны находиться в `.cpp` файлах.

## Какие файлы нужно указывать в `#include`

В `#include` нужно указывать только `.hpp` файлы.
Возьмём, к примеру, файлы `main.cpp`, `graph.hpp` и `graph.cpp`:
```cpp
// graph.hpp
class Graph {
  Vertex& add_vertex();
  // ...
};
```
```cpp
// graph.cpp
#include "graph.hpp"

Vertex& Graph::add_vertex() {
  // function body
}
```
```cpp
// main.cpp
#include "graph.hpp"
int main() {
  auto graph = Graph();
  // ...
  return 0;
}
```

## Include Guard

Чтобы избежать множественного копирования `.hpp` файлов компилятором,
нужно обязательно использовать `include guards` - это директивы компилятора,
которые помогут ему понять, заходил ли он уже внутрь данного файла, или нет.
Возьмём, к примеру, файл `graph.hpp`:
```cpp
#pragma once // <-- Это и есть Include Guard
// ...
class Graph {
  // ...
};
```

Повторяю, они указываются только внутри `.hpp` файлов.

# 2. Обернуть код в `namespace`

Обернуть ваш код в `namespace uni_course_cpp`.
Оба типа файлов, и `.hpp` и `.cpp`.

`main.cpp` не нужно оборачивать.

Все локальные функции и константы внутри `.cpp` файлов записывайте в приватный `namespace`:

```cpp
namespace {
// This is a private namespace
// It doesn't have a name
}  // namepsace

namespace uni_course_cpp {
// your
// code
// is
// here
}  // namepsace uni_course_cpp
```

# 3. Генерировать множество графов

- Запрашивать у пользователя новый входной параметр:
  - `graphs_count`: Количество генерируемых графов.
- Доработать программу, чтобы последовательно генерировать `graphs_count` графов.
- Запрашивать входные параметры только 1 раз, и использовать их для всех графов.
- Для каждого графа создавать отдельный `JSON` файл.

# 4. Создать файл конфигурации

1. Создайте файл `config.hpp`, в котором будет храниться конфигурация нашего проекта.
  Добавьте туда путь к папке `temp`, где теперь должны создаваться `JSON` файлы:
    ```cpp
    namespace config {

    inline constexpr const char* kTempDirectoryPath = "./temp/";

    }  // namespace config
    ```
1. Пример пути до директории `temp`:
    - `/surname_name/temp/`
1. Добавить директорию `temp` в конец файла `/.gitignore`:
1. Соответственно, этой директории в вашем ПР быть не должно.

# 5. Добавить Логирование

1. Написать `Logger`, который будет использоваться для вывода следующей информацию в консоль и в файл с названием `log.txt`:
    ```
    date_time Graph 0, Generation Started
    date_time Graph 0, Generation Finished {
      depth: number,
      vertices: {amount: number, distribution: [depth_0_count, depth_1_count, ...]},
      edges: {amount: number, distribution: {grey: count, green: count, yellow: count, red: count}}
    }
    ```
    Пример:
    ```
    2021.11.16 00:43:38 Graph 0, Generation Started
    2021.11.16 00:43:39 Graph 0, Generation Finished {
      depth: 8,
      vertices: {amount: 1360, distribution: [1, 4, 16, 54, 129, 247, 364, 371, 174]},
      edges: {amount: 3079, distribution: {grey: 1359, green: 128, yellow: 962, red: 276}}
    }
    ```
1. Реализовать используя паттерн `Singleton`.
    - [Wiki: Singleton Pattern](https://en.wikipedia.org/wiki/Singleton_pattern)
1. В публичном интерфейсе, для логирования данных, у него должен быть только один метод:
    ```cpp
    class Logger {
     public:
      void log(const std::string& string);
    };
    ```
    - Добавлять дату и время логирования автоматически внутри метода `log()`.
      Получать дату и время можно следующим образом:
      ```cpp
      #include <chrono>
      #include <ctime>
      #include <iomanip>
      #include <sstream>

      std::string get_current_date_time() {
        const auto date_time = std::chrono::system_clock::now();
        const auto date_time_t = std::chrono::system_clock::to_time_t(date_time);
        std::stringstream date_time_string;
        date_time_string << std::put_time(std::localtime(&date_time_t),
                                          "%Y.%m.%d %H:%M:%S");
        return date_time_string.str();
      }
      ```
1. Файл `log.txt` создавать внутри поддиректории `temp`.
    - Пример:
      - `/surname_name/temp/log.txt`
1. Обновить `config` и забирать из него путь до файла `log.txt`:
    ```cpp
    namespace config {

    inline constexpr const char* kLogFilename = "log.txt";
    inline const std::string kLogFilePath = kTempDirectoryPath + kLogFilename;

    }  // namespace config
    ```
1. Непосредственную генерацию строки c данными графа добавить в `printing`:
    ```cpp
    namespace printing {

    std::string print_graph(const Graph& graph);

    }  // namespace printing
    ```

# Частые Ошибки

- [Cвободные функции](/practice/common_mistakes.md#free-functions)

# Функция `main` вашей программы

```cpp
// ... some other logic ...

int main() {
  const int depth = handle_depth_input();
  const int new_vertices_count = handle_new_vertices_count_input();
  const int graphs_count = handle_graphs_count_input();
  prepare_temp_directory();

  const auto params = GraphGenerator::Params(depth, new_vertices_count);
  const auto generator = GraphGenerator(std::move(params));
  auto& logger = Logger::get_logger();

  for (int i = 0; i < graphs_count; i++) {
    logger.log(generation_started_string(i));
    const auto graph = generator.generate();

    const auto graph_description = printing::print_graph(graph);
    logger.log(generation_finished_string(i, graph_description));

    const auto graph_json = printing::json::print_graph(graph);
    write_to_file(graph_json, "graph_" + std::to_string(i) + ".json");
  }

  return 0;
}
```

# Содержание `Pull Request`

- `*.cpp` и `*.hpp` исходные файлы.
- Изменения в `/.gitignore`.
- `makefile` по желанию.

# Время Выполнения

1 Неделя.

# Полезные ссылки

- [Singleton Pattern](https://en.wikipedia.org/wiki/Singleton_pattern)
