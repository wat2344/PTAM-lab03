# PTAM-lab03
# Laboratory work III

Данная лабораторная работа посвящена изучению систем автоматизации сборки проекта на примере CMake.

```bash
$ open https://cmake.org/
```

## Tasks

1. Создать публичный репозиторий с названием `lab03` на сервисе GitHub.
2. Ознакомиться со ссылками учебного материала.
3. Выполнить инструкцию учебного материала.
4. Составить отчет и отправить ссылку личным сообщением в Slack.

## Tutorial

### Подготовка окружения

```bash
$ export GITHUB_USERNAME=<имя_пользователя>
$ cd ${GITHUB_USERNAME}/workspace
$ pushd .
$ source scripts/activate
```

*Подготовили переменные для дальнейшей работы.*

### Клонирование и настройка репозитория

```bash
$ git clone https://github.com/${GITHUB_USERNAME}/lab02.git projects/lab03
$ cd projects/lab03
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab03.git
```

*Через терминал создали новый репозиторий и отвязали от старого.*

### Сборка без CMake (вручную)

```bash
$ g++ -std=c++11 -I./include -c sources/print.cpp
$ ls print.o
```

**Вывод:**
```
print.o
```

```bash
$ nm print.o | grep print
```

**Вывод:**
```
0000000000000000 T _Z5printRKNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEERSo
000000000000002a T _Z5printRKNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEERSt14basic_ofstreamIcS2_E
```

```bash
$ ar rvs print.a print.o
```

**Вывод:**
```
ar: creating print.a
a - print.o
```

```bash
$ file print.a
```

**Вывод:**
```
print.a: current ar archive
```

```bash
$ g++ -std=c++11 -I./include -c examples/example1.cpp
$ ls example1.o
```

**Вывод:**
```
example1.o
```

```bash
$ g++ example1.o print.a -o example1
$ ./example1 && echo
```

**Вывод:**
```
hello
```

*Созданы сначала объектные файлы, а после — библиотека `print.a` и исполняемый файл `example1`.*

```bash
$ g++ -std=c++11 -I./include -c examples/example2.cpp
$ nm example2.o
```

*Вывод `nm example2.o` опущен из-за большого объема (символы стандартной библиотеки).*

```bash
$ g++ example2.o print.a -o example2
$ ./example2
$ cat log.txt && echo
```

**Вывод:**
```
hello
```

*Всё то же, что с `example1`, только теперь вывод в файл `log.txt`.*

```bash
$ rm -rf example1.o example2.o print.o
$ rm -rf print.a
$ rm -rf example1 example2
$ rm -rf log.txt
```

*Убрали все сгенерированные объекты.*

### Сборка с помощью CMake

#### Создание `CMakeLists.txt`

```bash
$ cat > CMakeLists.txt <<EOF
cmake_minimum_required(VERSION 3.4)
project(print)
EOF
```

*Указали минимальную версию CMake и имя проекта.*

```bash
$ cat >> CMakeLists.txt <<EOF
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
EOF
```

*Установили стандарт C++11 и требование его поддержки.*

```bash
$ cat >> CMakeLists.txt <<EOF
add_library(print STATIC \${CMAKE_CURRENT_SOURCE_DIR}/sources/print.cpp)
EOF
```

*Добавили статическую библиотеку `print` из исходного файла.*

```bash
$ cat >> CMakeLists.txt <<EOF
include_directories(\${CMAKE_CURRENT_SOURCE_DIR}/include)
EOF
```

*Подключили директорию с заголовочными файлами.*

#### Конфигурация и сборка

```bash
$ cmake -H. -B_build
```

**Вывод:**
```
CMake Deprecation Warning at CMakeLists.txt:1 (cmake_minimum_required):
  Compatibility with CMake < 3.10 will be removed from a future version of
  CMake.

  Update the VERSION argument <min> value.  Or, use the <min>...<max> syntax
  to tell CMake that the project requires at least <min> but has been updated
  to work with policies introduced by <max> or earlier.

-- The C compiler identification is GNU 15.2.0
-- The CXX compiler identification is GNU 15.2.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done (0.5s)
-- Generating done (0.0s)
-- Build files have been written to: /home/vboxuser/wat2344/workspace/projects/lab03/_build
```

```bash
$ cmake --build _build
```

**Вывод:**
```
[ 50%] Building CXX object CMakeFiles/print.dir/sources/print.cpp.o
[100%] Linking CXX static library libprint.a
[100%] Built target print
```

*Первый запуск CMake и сборка библиотеки.*

#### Добавление исполняемых файлов

```bash
$ cat >> CMakeLists.txt <<EOF

add_executable(example1 \${CMAKE_CURRENT_SOURCE_DIR}/examples/example1.cpp)
add_executable(example2 \${CMAKE_CURRENT_SOURCE_DIR}/examples/example2.cpp)
EOF
```

*Добавлены два исполняемых файла: `example1` и `example2`.*

```bash
$ cat >> CMakeLists.txt <<EOF

target_link_libraries(example1 print)
target_link_libraries(example2 print)
EOF
```

*При сборке исполняемых файлов использовать библиотеку `print`.*

#### Повторная сборка

```bash
$ cmake --build _build
```

**Вывод:**
```
CMake Deprecation Warning at CMakeLists.txt:1 (cmake_minimum_required):
  Compatibility with CMake < 3.10 will be removed from a future version of
  CMake.

  Update the VERSION argument <min> value.  Or, use the <min>...<max> syntax
  to tell CMake that the project requires at least <min> but has been updated
  to work with policies introduced by <max> or earlier.

-- Configuring done (0.0s)
-- Generating done (0.0s)
-- Build files have been written to: /home/vboxuser/wat2344/workspace/projects/lab03/_build
[ 33%] Built target print
[ 50%] Building CXX object CMakeFiles/example1.dir/examples/example1.cpp.o
[ 66%] Linking CXX executable example1
[ 66%] Built target example1
[ 83%] Building CXX object CMakeFiles/example2.dir/examples/example2.cpp.o
[100%] Linking CXX executable example2
[100%] Built target example2
```

```bash
$ cmake --build _build --target print
```

**Вывод:**
```
[100%] Built target print
```

```bash
$ cmake --build _build --target example1
```

**Вывод:**
```
[ 50%] Built target print
[100%] Built target example1
```

```bash
$ cmake --build _build --target example2
```

**Вывод:**
```
[ 50%] Built target print
[100%] Built target example2
```

*Собрали все цели.*

```bash
$ ls -la _build/libprint.a
```

**Вывод:**
```
-rw-rw-r-- 1 vboxuser vboxuser 2270 Apr 25 21:38 _build/libprint.a
```

```bash
$ _build/example1 && echo
```

**Вывод:**
```
hello
```

```bash
$ _build/example2
$ cat log.txt && echo
```

**Вывод:**
```
hello
```

```bash
$ rm -rf log.txt
```

*Запустили программы и удалили временный файл.*

### Использование готового `CMakeLists.txt` из репозитория

```bash
$ git clone https://github.com/tp-labs/lab03 tmp
```

**Вывод:**
```
Cloning into 'tmp'...
remote: Enumerating objects: 91, done.
remote: Counting objects: 100% (30/30), done.
remote: Compressing objects: 100% (9/9), done.
remote: Total 91 (delta 23), reused 21 (delta 21), pack-reused 61 (from 1)
Receiving objects: 100% (91/91), 1.02 MiB | 3.64 MiB/s, done.
Resolving deltas: 100% (41/41), done.
```

```bash
$ mv -f tmp/CMakeLists.txt .
$ rm -rf tmp
```

*Перенесли файл `CMakeLists.txt` из `tp-labs/lab03`.*

```bash
$ cat CMakeLists.txt
```

**Вывод (содержимое):**
```cmake
cmake_minimum_required(VERSION 3.4)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_EXAMPLES "Build examples" OFF)

project(print)

add_library(print STATIC ${CMAKE_CURRENT_SOURCE_DIR}/sources/print.cpp)

target_include_directories(print PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

if(BUILD_EXAMPLES)
  file(GLOB EXAMPLE_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/examples/*.cpp")
  foreach(EXAMPLE_SOURCE ${EXAMPLE_SOURCES})
    get_filename_component(EXAMPLE_NAME ${EXAMPLE_SOURCE} NAME_WE)
    add_executable(${EXAMPLE_NAME} ${EXAMPLE_SOURCE})
    target_link_libraries(${EXAMPLE_NAME} print)
    install(TARGETS ${EXAMPLE_NAME}
      RUNTIME DESTINATION bin
    )
  endforeach(EXAMPLE_SOURCE ${EXAMPLE_SOURCES})
endif()

install(TARGETS print
    EXPORT print-config
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION include)
install(EXPORT print-config DESTINATION cmake)
```

#### Конфигурация с установкой

```bash
$ cmake -H. -B_build -DCMAKE_INSTALL_PREFIX=_install
```

**Вывод:**
```
CMake Deprecation Warning at CMakeLists.txt:1 (cmake_minimum_required):
  Compatibility with CMake < 3.10 will be removed from a future version of
  CMake.

  Update the VERSION argument <min> value.  Or, use the <min>...<max> syntax
  to tell CMake that the project requires at least <min> but has been updated
  to work with policies introduced by <max> or earlier.

-- Configuring done (0.0s)
-- Generating done (0.0s)
-- Build files have been written to: /home/vboxuser/wat2344/workspace/projects/lab03/_build
```

```bash
$ cmake --build _build --target install
```

**Вывод:**
```
[100%] Built target print
Install the project...
-- Install configuration: ""
-- Installing: /home/vboxuser/wat2344/workspace/projects/lab03/_install/lib/libprint.a
-- Installing: /home/vboxuser/wat2344/workspace/projects/lab03/_install/include
-- Installing: /home/vboxuser/wat2344/workspace/projects/lab03/_install/include/print.hpp
-- Installing: /home/vboxuser/wat2344/workspace/projects/lab03/_install/cmake/print-config.cmake
-- Installing: /home/vboxuser/wat2344/workspace/projects/lab03/_install/cmake/print-config-noconfig.cmake
```

```bash
$ tree _install
```

**Вывод:**
```
_install
├── cmake
│   ├── print-config.cmake
│   └── print-config-noconfig.cmake
├── include
│   └── print.hpp
└── lib
    └── libprint.a

4 directories, 4 files
```

*Установили библиотеку и заголовки в директорию `_install`.*

### Сохранение изменений в репозитории

```bash
$ git add CMakeLists.txt
$ git commit -m"added CMakeLists.txt"
$ git push origin master
```

*Загрузили изменения на GitHub.*

## Homework

### Задание 1: Библиотека `formatter_lib`

Создадим `CMakeLists.txt` для библиотеки форматирования.

```bash
$ cat > CMakeLists.txt <<EOF
cmake_minimum_required(VERSION 3.4)
project(formatter)
EOF

$ cat >> CMakeLists.txt <<EOF
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
EOF

$ cat >> CMakeLists.txt <<EOF
add_library(formatter STATIC \${CMAKE_CURRENT_SOURCE_DIR}/formatter.cpp)
EOF

$ cat >> CMakeLists.txt <<EOF
include_directories(\${CMAKE_CURRENT_SOURCE_DIR})
EOF
```

*Аналогично туториалу, с заменой названий и путей.*

### Задание 2: Библиотека `formatter_ex_lib`

Перейдём в родительскую директорию и создадим новую.

```bash
$ cd ..
$ mkdir formatter_ex_lib
$ cd formatter_ex_lib
```

Создадим `CMakeLists.txt` для библиотеки, использующей `formatter`.

```bash
$ cat > CMakeLists.txt <<EOF
cmake_minimum_required(VERSION 3.4)
project(formatter_ex)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(../formatter_lib formatter_lib)

add_library(formatter_ex STATIC formatter_ex.cpp)

include_directories(\${CMAKE_CURRENT_SOURCE_DIR})

target_link_libraries(formatter_ex PRIVATE formatter)
EOF
```

*Добавили поддиректорию с библиотекой `formatter` и связали её.*

### Задание 3: Исполняемые файлы

#### Библиотека `solver_lib`

```bash
$ cd ..
$ mkdir solver_lib
$ cd solver_lib
$ touch solver.cpp solver.h
$ cat > CMakeLists.txt << EOF
cmake_minimum_required(VERSION 3.4)
project(solver_lib)

add_library(solver_lib STATIC solver.cpp)
include_directories(\${CMAKE_CURRENT_SOURCE_DIR})
EOF
```

*Созданы заглушки для структуры.*

#### Приложение `hello_world`

```bash
$ cd ..
$ mkdir hello_world
$ cd hello_world
$ cat > main.cpp << EOF
#include <iostream>
#include "formatter_ex.h"

int main() {
    std::cout << formatter("Hello, World!") << std::endl;
    return 0;
}
EOF

$ cat > CMakeLists.txt <<EOF
cmake_minimum_required(VERSION 3.4)
project(hello_world)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(../formatter_ex_lib formatter_ex_lib)

add_executable(hello_world main.cpp)

target_link_libraries(hello_world PRIVATE formatter_ex)
EOF
```

*Создан исполняемый файл, использующий `formatter_ex`.*

#### Приложение `solver`

```bash
$ cd ..
$ mkdir solver
$ cd solver
$ touch main.cpp
$ cat > CMakeLists.txt <<EOF
cmake_minimum_required(VERSION 3.4)
project(solver)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(../formatter_ex_lib formatter_ex_lib)
add_subdirectory(../solver_lib solver_lib)

add_executable(solver main.cpp)

target_link_libraries(solver PRIVATE formatter_ex solver_lib)
EOF
```

*Исполняемый файл, использующий обе библиотеки.*

Теперь все компоненты можно собрать с помощью CMake.# PTAM-lab03
