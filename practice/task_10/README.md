# Задача 10

# Добавить основу игровой логики

- Добавить `EdgeDuration`:
  ```cpp
  using EdgeDuration = int;
  struct IEdge {
    EdgeDuration duration() const = 0;
  };
  ```
  - `duration` может принимать рандомное значение из соответствующего диапазона:
    - Grey = [1, 2]
    - Green = [1, 2]
    - Yellow = [1, 3]
    - Red = [2, 4]
- Расширить `IGraphPath`, добавить продолжительность пути:
  ```cpp
  struct IGraphPath {
    EdgeDuration duration() const = 0;
  };
  ```
- Создать интерфейс `IGame` и класс `Game`, который будет инкапсулировать игровую логику:
  - Игровую карту
  - Расположение рыцаря
  - Расположение принцессы
  - Поиск кратчайшего пути между рыцарем и принцессой
    - Параметр поиска - `PathDistance`, количество шагов пути
  - Поиск скорейшего пути между рыцарем и принцессой
    - Параметр поиска - `Graph::Edge::Duration`, продолжительность пути
  ```cpp
  class Game {
   public:
    // Traverse by `PathDistance`
    GraphPath find_shortest_path() const;
    // Traverse by `EdgeDuration`
    GraphPath find_fastest_path() const;
   private:
    Graph map_;
    Graph::VertexId knight_position_;
    Graph::VertexId princess_position_;
  };
  ```
- Соответственно, для поиска скорейшего пути расширить класс `GraphTraverser`.
- Создать класс `GameGenerator`, который принимает параметры графа `GraphGenerator::Params` и генерирует игру `Game`.
- Расширить логику `printing`:
  ```cpp
  namespace printing {

  std::string print_game(const IGame& game);

  }  // namespace printing
  ```
  - Методы `print_edge` и `print_path` должены печатать новый параметр `Duration`.
  - Пример файла можете посмотреть здесь: [map.json](map.json).
- Ваша программа должна создать игру и залогировать её данные. Пример:
  ```
  2021.12.20 10:44:42 Game is Preparing...
  2021.12.20 10:44:42 Game is Ready {
    map: {
      depth: 7,
      vertices: {amount: 44, distribution: [1, 2, 4, 7, 10, 9, 8, 3]},
      edges: {amount: 89, distribution: {grey: 43, green: 5, blue: 7, yellow: 26, red: 8}}
    },
    knight position: {vertex_id: 0, depth: 0},
    princess position: {vertex_id: 17, depth: 7}
  }
  2021.12.20 10:44:42 Searching for Shortest Path...
  2021.12.20 10:44:42 Shortest Path: {vertices: [0, 1, 22, 38, 39, 16, 17], edges: [0, 4, 8, 12, 15, 19], distance: 6, duration: 11}
  2021.12.20 10:44:42 Searching for Fastest Path...
  2021.12.20 10:44:42 Fastest Path: {vertices: [0, 1, 4, 10, 12, 14, 16, 17], edges: [0, 3, 11, 15, 14, 21, 23], distance: 7, duration: 9}
  ```

## Логика программы

1. Получаете параметры графа от пользователя.
    - `depth`
    - `new_vertices_count`
1. Подготавливаете окружение.
    - `prepare_temp_directory()`
1. Генерируете игру.
    - `const auto game = game_generator.generate();`
1. Выполняете поиск кратчайшего пути.
    - `game.find_shortest_path();`
1. Выполняете поиск скорейшего пути.
    - `game.find_fastest_path();`
1. Параллельно с выполнением программы логируете данные.
    1. `Game is Preparing...`
    1. `Game is Ready`
    1. `Searching for Shortest Path...`
    1. `Shortest Path: { ... }`
    1. `Searching for Fastest Path...`
    1. `Fastest Path: { ... }`
1. Распечатываете игровую карту в файл `map.json` в корень вашей папки.
    - Повторяю, не в `/temp`, а в корень `./`.

# Функция `main` вашей программы

```cpp
int main() {
  const int depth = handle_depth_input();
  const int new_vertices_count = handle_new_vertices_count_input();
  prepare_temp_directory();

  auto& logger = Logger::get_logger();
  logger.log(game_preparing_string());

  auto params = GraphGenerator::Params(depth, new_vertices_count);
  const auto game_generator = GameGenerator(std::move(params));
  const auto game = game_generator.generate();

  logger.log(game_ready_string(*game));
  logger.log(shortest_path_searching_string());

  const auto shortest_path = game->find_shortest_path();

  logger.log(shortest_path_ready_string(shortest_path));
  logger.log(fastest_path_searching_string());

  const auto fastest_path = game->find_fastest_path();

  logger.log(fastest_path_ready_string(fastest_path));

  const auto map_json = printing::json::print_map(game->map());
  write_to_file(map_json, "map.json");

  return 0;
}
```

# Содержание `Pull Request`

- `*.cpp` и `*.hpp` исходные файлы.
- `map.json` результат выполнения программы.
- `makefile` по желанию.

# Время Выполнения

1 Неделя.
