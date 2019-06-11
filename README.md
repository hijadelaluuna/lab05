## Laboratory work V

Данная лабораторная работа посвещена изучению фреймворков для тестирования на примере **GTest**

```ShellSession
$ open https://github.com/google/googletest
```

## Tasks

- [x] 1. Создать публичный репозиторий с названием **lab05** на сервисе **GitHub**
- [x] 2. Выполнить инструкцию учебного материала
- [x] 3. Ознакомиться со ссылками учебного материала
- [x] 4. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial
Настройка переменных окружения
```ShellSession
$ export GITHUB_USERNAME=hijadelaluuna
$ alias gsed=sed # for *-nix system
```
Подготовка рабочего пространства
```ShellSession
$ cd ${GITHUB_USERNAME}/workspace  # Переход в папку workspace
$ pushd .                       # Сохранение текущей директории

$ source scripts/activate  # Выполнение скрипта подготовки
```
Клонирование репозитория ЛР4 в ЛР5
```ShellSession
$ git clone https://github.com/${GITHUB_USERNAME}/lab04 projects/lab05   # Клонирование репозитория
Клонирование в «projects/lab05»…
remote: Enumerating objects: 34, done.
remote: Counting objects: 100% (34/34), done.
remote: Compressing objects: 100% (23/23), done.
remote: Total 34 (delta 7), reused 30 (delta 6), pack-reused 0
Распаковка объектов: 100% (34/34), готово.
$ cd projects/lab05   # Переход в директорию
$ git remote remove origin  # Удаление связки с репозиторием
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab05  # Добавление связки
```
Добавление подмодуля тестирования
```ShellSession
$ mkdir third-party # Создание папки
$ git submodule add https://github.com/google/googletest third-party/gtest # Скачивание удаленного репозитория в указанную папку
Клонирование в «/home/luna/hijadelaluuna/workspace/projects/lab05/third-party/gtest»…
remote: Enumerating objects: 16892, done.
remote: Total 16892 (delta 0), reused 0 (delta 0), pack-reused 16892
Получение объектов: 100% (16892/16892), 5.96 MiB | 107.00 KiB/s, готово.
Определение изменений: 100% (12445/12445), готово.
$ cd third-party/gtest && git checkout release-1.8.1 && cd ../..   # Переход в указанную папку, переход в указанную ветку, возврат
Примечание: переход на «release-1.8.1».

Вы сейчас в состоянии «отделённого HEAD». Вы можете осмотреться, сделать
экспериментальные изменения и закоммитить их, также вы можете отменить
изменения любых коммитов в этом состоянии не затрагивая любые ветки и
не переходя на них.

Если вы хотите создать новую ветку и сохранить свои коммиты, то вы
можете сделать это (сейчас или позже) вызвав команду checkout снова,
но с параметром -b. Например:

  git checkout -b <имя-новой-ветки>

HEAD сейчас на 2fe3bd99 Merge pull request #1433 from dsacre/fix-clang-warnings
$ git add third-party/gtest   # Фиксирование изменений
$ git commit -m"added gtest framework" # Коммит изменений
[master a78bd29] added gtest framework
 2 files changed, 4 insertions(+)
 create mode 100644 .gitmodules
 create mode 160000 third-party/gtest
```
Редактирование CMakeList.txt
```ShellSession
$ gsed -i '/option(BUILD_EXAMPLES "Build examples" OFF)/a\  # Вставка строки после указанной 
option(BUILD_TESTS "Build tests" OFF)
' CMakeLists.txt
$ cat >> CMakeLists.txt <<EOF   # Дописывание в CMakeLists.txt 

if(BUILD_TESTS)
  enable_testing()
  add_subdirectory(third-party/gtest)
  file(GLOB \${PROJECT_NAME}_TEST_SOURCES tests/*.cpp)
  add_executable(check \${\${PROJECT_NAME}_TEST_SOURCES})
  target_link_libraries(check \${PROJECT_NAME} gtest_main)
  add_test(NAME check COMMAND check)
endif()
EOF
```
Создание кода с тестами
```ShellSession
$ mkdir tests   # Создание  папки
$ cat > tests/test1.cpp <<EOF   # Создание  файла с кодом
#include <print.hpp>

#include <gtest/gtest.h>

TEST(Print, InFileStream)
{
  std::string filepath = "file.txt";
  std::string text = "hello";
  std::ofstream out{filepath};

  print(text, out);
  out.close();

  std::string result;
  std::ifstream in{filepath};
  in >> result;

  EXPECT_EQ(result, text);
}
EOF
```
Сборка проекта
```ShellSession
$ cmake -H. -B_build -DBUILD_TESTS=ON  # Конфигурирование
-- The C compiler identification is GNU 8.3.0
-- The CXX compiler identification is GNU 8.3.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Found PythonInterp: /usr/bin/python (found version "3.7.3") 
-- Looking for pthread.h
-- Looking for pthread.h - found
-- Looking for pthread_create
-- Looking for pthread_create - not found
-- Check if compiler accepts -pthread
-- Check if compiler accepts -pthread - yes
-- Found Threads: TRUE  
-- Configuring done
-- Generating done
-- Build files have been written to: /home/luna/hijadelaluuna/workspace/projects/lab05/_build

$ cmake --build _build   # Компиляция
Scanning dependencies of target print
[  8%] Building CXX object CMakeFiles/print.dir/sources/print.cpp.o
[ 16%] Linking CXX static library libprint.a
[ 16%] Built target print
Scanning dependencies of target gtest
[ 25%] Building CXX object third-party/gtest/googlemock/gtest/CMakeFiles/gtest.dir/src/gtest-all.cc.o
[ 33%] Linking CXX static library libgtest.a
[ 33%] Built target gtest
Scanning dependencies of target gtest_main
[ 41%] Building CXX object third-party/gtest/googlemock/gtest/CMakeFiles/gtest_main.dir/src/gtest_main.cc.o
[ 50%] Linking CXX static library libgtest_main.a
[ 50%] Built target gtest_main
Scanning dependencies of target check
[ 58%] Building CXX object CMakeFiles/check.dir/tests/test1.cpp.o
[ 66%] Linking CXX executable check
[ 66%] Built target check
Scanning dependencies of target gmock
[ 75%] Building CXX object third-party/gtest/googlemock/CMakeFiles/gmock.dir/src/gmock-all.cc.o
[ 83%] Linking CXX static library libgmock.a
[ 83%] Built target gmock
Scanning dependencies of target gmock_main
[ 91%] Building CXX object third-party/gtest/googlemock/CMakeFiles/gmock_main.dir/src/gmock_main.cc.o
[100%] Linking CXX static library libgmock_main.a
[100%] Built target gmock_main

$ cmake --build _build --target test   # Компиляция указанного
Running tests...
Test project /home/luna/hijadelaluuna/workspace/projects/lab05/_build
    Start 1: check
1/1 Test #1: check ............................   Passed    0.00 sec

100% tests passed, 0 tests failed out of 1

Total Test time (real) =   0.01 sec
```
Проверка тестов
```ShellSession
$ _build/check  # Исполнение файла с тестами
Running main() from /home/luna/hijadelaluuna/workspace/projects/lab05/third-party/gtest/googletest/src/gtest_main.cc
[==========] Running 1 test from 1 test case.
[----------] Global test environment set-up.
[----------] 1 test from Print
[ RUN      ] Print.InFileStream
[       OK ] Print.InFileStream (1 ms)
[----------] 1 test from Print (1 ms total)

[----------] Global test environment tear-down
[==========] 1 test from 1 test case ran. (1 ms total)
[  PASSED  ] 1 test.

$ cmake --build _build --target test -- ARGS=--verbose  # Компиляция с выводом всей информации
Running tests...
UpdateCTestConfiguration  from :/home/luna/hijadelaluuna/workspace/projects/lab05/_build/DartConfiguration.tcl
UpdateCTestConfiguration  from :/home/luna/hijadelaluuna/workspace/projects/lab05/_build/DartConfiguration.tcl
Test project /home/luna/hijadelaluuna/workspace/projects/lab05/_build
Constructing a list of tests
Done constructing a list of tests
Updating test list for fixtures
Added 0 tests to meet fixture requirements
Checking test dependency graph...
Checking test dependency graph end
test 1
    Start 1: check

1: Test command: /home/absinthetoxin/CrazyOverdose/workspace/projects/lab05/_build/check
1: Test timeout computed to be: 10000000
1: Running main() from /home/absinthetoxin/CrazyOverdose/workspace/projects/lab05/third-party/gtest/googletest/src/gtest_main.cc
1: [==========] Running 1 test from 1 test case.
1: [----------] Global test environment set-up.
1: [----------] 1 test from Print
1: [ RUN      ] Print.InFileStream
1: [       OK ] Print.InFileStream (1 ms)
1: [----------] 1 test from Print (1 ms total)
1: 
1: [----------] Global test environment tear-down
1: [==========] 1 test from 1 test case ran. (1 ms total)
1: [  PASSED  ] 1 test.
1/1 Test #1: check ............................   Passed    0.00 sec

100% tests passed, 0 tests failed out of 1

Total Test time (real) =   0.00 sec

```
Обновление Travis CI конфига
```ShellSession
$ gsed -i 's/lab04/lab05/g' README.md  # Замена левой строки на правую
$ gsed -i 's/\(DCMAKE_INSTALL_PREFIX=_install\)/\1 -DBUILD_TESTS=ON/' .travis.yml 
# Дописывание к найденной левой строки строке правой строки
$ gsed -i '/cmake --build _build --target install/a\ # Дописывание правой строки после найденной левой строки
- cmake --build _build --target test -- ARGS=--verbose
' .travis.yml
```
Проверка конфига
```ShellSession
$ travis lint
Warnings for .travis.yml:
[x] value for addons section is empty, dropping
[x] in addons section: unexpected key apt, dropping
```
Отправка изменений
```ShellSession
$ git add .travis.yml  # Фиксация указанного файла
$ git add tests        # Фиксация указанного файла
$ git add -p
diff --git a/CMakeLists.txt b/CMakeLists.txt
index 96a361e..89739e7 100644
--- a/CMakeLists.txt
+++ b/CMakeLists.txt
@@ -4,6 +4,7 @@ set(CMAKE_CXX_STANDARD 11)
 set(CMAKE_CXX_STANDARD_REQUIRED ON)
 
 option(BUILD_EXAMPLES "Build examples" OFF)
+option(BUILD_TESTS "Build tests" OFF)
 
 project(print)
 
Stage this hunk [y,n,q,a,d,j,J,g,/,e,?]? y
y - stage this hunk
n - do not stage this hunk
q - quit; do not stage this hunk or any of the remaining ones
a - stage this hunk and all later hunks in the file
d - do not stage this hunk or any of the later hunks in the file
g - select a hunk to go to
/ - search for a hunk matching the given regex
j - leave this hunk undecided, see next undecided hunk
J - leave this hunk undecided, see next hunk
e - manually edit the current hunk
? - print help
@@ -4,6 +4,7 @@ set(CMAKE_CXX_STANDARD 11)
 set(CMAKE_CXX_STANDARD_REQUIRED ON)
 
 option(BUILD_EXAMPLES "Build examples" OFF)
+option(BUILD_TESTS "Build tests" OFF)
 
 project(print)
 
Stage this hunk [y,n,q,a,d,j,J,g,/,e,?]? y
@@ -34,3 +35,11 @@ install(TARGETS print
 
 install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION include)
 install(EXPORT print-config DESTINATION cmake)
+if(BUILD_TESTS)
+  enable_testing()
+  add_subdirectory(third-party/gtest)
+  file(GLOB ${PROJECT_NAME}_TEST_SOURCES tests/*.cpp)
+  add_executable(check ${${PROJECT_NAME}_TEST_SOURCES})
+  target_link_libraries(check ${PROJECT_NAME} gtest_main)
+  add_test(NAME check COMMAND check)
+endif()
Stage this hunk [y,n,q,a,d,K,g,/,e,?]? y

diff --git a/README.md b/README.md
index 55b5a9f..609c771 100644
--- a/README.md
+++ b/README.md
@@ -9,7 +9,7 @@ $ open https://travis-ci.org
 ## Tasks
 
 - [x] 1. Авторизоваться на сервисе **Travis CI** с использованием **GitHub** аккаунта
-- [x] 2. Создать публичный репозиторий с названием **lab04** на сервисе **GitHub**
+- [x] 2. Создать публичный репозиторий с названием **lab05** на сервисе **GitHub**
 - [x] 3. Ознакомиться со ссылками учебного материала
 - [x] 4. Включить интеграцию сервиса **Travis CI** с созданным репозиторием
 - [x] 5. Получить токен для **Travis CLI** с правами **repo** и **user**
Stage this hunk [y,n,q,a,d,j,J,g,/,e,?]? y
@@ -163,16 +163,16 @@ Done installing documentation for multipart-post, faraday, faraday_middleware, h

 Клонирование репозитория ЛР3 в ЛР4

-$ git clone https://github.com/${GITHUB_USERNAME}/lab03 projects/lab04 # Клонирование репозитория
-Клонирование в «projects/lab04»…
+$ git clone https://github.com/${GITHUB_USERNAME}/lab03 projects/lab05 # Клонирование репозитория
+Клонирование в «projects/lab05»…
 remote: Enumerating objects: 27, done.
 remote: Counting objects: 100% (27/27), done.
 remote: Compressing objects: 100% (19/19), done.
 remote: Total 27 (delta 4), reused 23 (delta 3), pack-reused 0
 Распаковка объектов: 100% (27/27), готово.
-$ cd projects/lab04  # Переход в директорию
+$ cd projects/lab05  # Переход в директорию
 $ git remote remove origin # Удаление связки с репозиторием
-$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab04 # Добавление связки
+$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab05 # Добавление связки

 Записывание в .travis.yml информации о языке

Stage this hunk [y,n,q,a,d,K,j,J,g,/,s,e,?]? y
@@ -217,7 +217,7 @@ Warnings for .travis.yml:
 
 Добавление в начало файла строки с фрагментом вставки значка сервиса Travis CI в формате Markdown
 
-$ sed -i '1i|[![Build Status](https://travis-ci.org/CrazyOverdose/lab04.svg?branch=master)](https://travis-ci.org/CrazyOverdose/lab04)' README.md   # Команда изменена
+$ sed -i '1i|[![Build Status](https://travis-ci.org/CrazyOverdose/lab05.svg?branch=master)](https://travis-ci.org/CrazyOverdose/lab05)' README.md   # Команда изменена
 
 Отправка изменений
Stage this hunk [y,n,q,a,d,K,j,J,g,/,e,?]? y
@@ -237,7 +237,7 @@ Password for 'https://CrazyOverdose@github.com':
 Запись объектов: 100% (31/31), 14.38 KiB | 3.59 MiB/s, готово.
 Всего 31 (изменения 6), повторно использовано 0 (изменения 0)
 remote: Resolving deltas: 100% (6/6), done.
-To https://github.com/CrazyOverdose/lab04
+To https://github.com/CrazyOverdose/lab05
  * [new branch]      master -> master
 
 Работа с Travis CI
Stage this hunk [y,n,q,a,d,K,j,J,g,/,e,?]? y
@@ -269,7 +269,7 @@ Description: ???
hijadelaluuna/lab03 (active: no, admin: yes, push: yes, pull: yes)
 Description: ???
 
-hijadelaluuna/lab04 (active: yes, admin: yes, push: yes, pull: yes)
+hijadelaluuna/lab05 (active: yes, admin: yes, push: yes, pull: yes)
 Description: ???
 
 hijadelaluuna/lab05 (active: no, admin: yes, push: yes, pull: yes)
Stage this hunk [y,n,q,a,d,K,j,J,g,/,e,?]? y
@@ -291,10 +291,10 @@ CrazyOverdose/labaa02 (active: no, admin: yes, push: yes, pull: yes)
 Description: Изучение систем контроля версий на примере Git
 
 $ travis enable   # Активация проекта
-Detected repository as hijadelaluuna/lab04, is this correct? |yes| yes
-hijadelaluuna/lab04: enabled :)
+Detected repository as hijadelaluuna/lab05, is this correct? |yes| yes
+hijadelaluuna/lab05: enabled :)
 $ travis whatsup   # Список последних сборок
-hijadelaluuna/lab04 passed: #1
+hijadelaluuna/lab05 passed: #1
 $ travis branches    # Список последних сборок по веткам проекта
 master:  #1    passed     added CI
 $ travis history  # История сборок для проекта
Stage this hunk [y,n,q,a,d,K,j,J,g,/,s,e,?]? y
@@ -304,7 +304,7 @@ Job #1.1:  added CI
 State:         passed
 Type:          push
 Branch:        master
-Compare URL:   https://github.com/hijadelaluuna/lab04/compare/89df61653546^...da3553aea864
+Compare URL:   https://github.com/hijadelaluuna/lab05/compare/89df61653546^...da3553aea864
 Duration:      28 sec
 Started:       2019-06-09 19:09:31
 Finished:      2019-06-09 19:09:59
Stage this hunk [y,n,q,a,d,K,g,/,e,?]? y


$ git commit -m"added tests"

[master da72cc9] added tests
 4 files changed, 42 insertions(+), 13 deletions(-)
 create mode 100644 tests/test1.cpp
 
$ git push origin master  # Отправка изменений в удаленный репозиторий
Username for 'https://github.com'CrazyOverdose
Password for 'https://CrazyOverdose@github.com': 
Перечисление объектов: 45, готово.
Подсчет объектов: 100% (45/45), готово.
Сжатие объектов: 100% (38/38), готово.
Запись объектов: 100% (45/45), 21.24 KiB | 4.25 MiB/s, готово.
Всего 45 (изменения 12), повторно использовано 0 (изменения 0)
remote: Resolving deltas: 100% (12/12), done.
To https://github.com/CrazyOverdose/lab05
 * [new branch]      master -> master

```
Авторизация и активация репозитория в travis
```ShellSession
$ travis login --auto
Successfully logged in as hijadelaluuna!
$ travis enable
Detected repository as hijadelaluuna/lab05, is this correct? |yes| y
hijadelaluuna/lab05: enabled :)
```
Сохранение результата
```ShellSession
$ mkdir artifacts
$ sleep 20s && gnome-screenshot --file artifacts/screenshot.png
# for macOS: $ screencapture -T 20 artifacts/screenshot.png
# open https://github.com/${GITHUB_USERNAME}/lab05
```

## Report

```ShellSession
$ popd
$ export LAB_NUMBER=05
$ git clone https://github.com/tp-labs/lab${LAB_NUMBER} tasks/lab${LAB_NUMBER}
$ mkdir reports/lab${LAB_NUMBER}
$ cp tasks/lab${LAB_NUMBER}/README.md reports/lab${LAB_NUMBER}/REPORT.md
$ cd reports/lab${LAB_NUMBER}
$ edit REPORT.md
$ gistup -m "lab${LAB_NUMBER}"
```

## Homework

### Задание
1. Создайте `CMakeList.txt` для библиотеки *banking*.
2. Создайте модульные тесты на классы `Transaction` и `Account`.
    * Используйте mock-объекты.
    * Покрытие кода должно составлять 100%.
3. Настройте сборочную процедуру на **TravisCI**.
4. Настройте [Coveralls.io](https://coveralls.io/).

## Links

- [C++ CI: Travis, CMake, GTest, Coveralls & Appveyor](http://david-grs.github.io/cpp-clang-travis-cmake-gtest-coveralls-appveyor/)
- [Boost.Tests](http://www.boost.org/doc/libs/1_63_0/libs/test/doc/html/)
- [Catch](https://github.com/catchorg/Catch2)

```
Copyright (c) 2015-2019 The ISC Authors
```
