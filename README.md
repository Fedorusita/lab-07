# lab-07
# Docker  

1.Создадим файл ```.hpp```
```
#include <fstream>
#include <iostream>
#include <string>

void print(const std::string& text, std::ofstream& out);
void print(const std::string& text, std::ostream& out = std::cout);
```
2. Создадим файл ```.cpp```
```
#include <print.hpp>

void print(const std::string& text, std::ostream& out)
{
  out << text;
}

void print(const std::string& text, std::ofstream& out)
{
  out << text;
}
```
3. Создадим файл ```main.cpp```
```
#include <print.hpp>
#include <cstdlib>

int main(int argc, char* argv[])
{
  const char* log_path = std::getenv("LOG_PATH");
  if (log_path == nullptr)
  {
    std::cerr << "undefined environment variable: LOG_PATH" << std::endl;
    return 1;
  }
  std::string text;
  while (std::cin >> text)
  {
    std::ofstream out{log_path, std::ios_base::app};
    print(text, out);
    out << std::endl;
  }
}
```
**Всё это необходимо чтобы содержимое докера выводилось на печать**

4.Добавим CMakeLists.txt
```
cmake_minimum_required(VERSION 3.4)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_EXAMPLES "Build examples" OFF)

project(print)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
add_library(print STATIC ${CMAKE_CURRENT_SOURCE_DIR}/source/print.cpp)



add_executable(demo ${CMAKE_CURRENT_SOURCE_DIR}/demo/main.cpp)
target_link_libraries(demo print) 
install(TARGETS demo RUNTIME DESTINATION bin)


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
5.Теперь сделаем ```Dockerfile```
```
FROM ubuntu:22.04

RUN apt update
RUN apt install -yy gcc g++ cmake

COPY . print/
WORKDIR print

RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
RUN cmake --build _build
RUN cmake --build _build --target install

ENV LOG_PATH /home/lab-07/Fedor/fed.txt
VOLUME /home/lab-07/logs

WORKDIR _install/bin
ENTRYPOINT ./demo
```
6. Теперь сделаем ```Action.yml```
```
name: docker
on:
  push:
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: build docker
        run: docker build -t logger .
```
![Action](https://github.com/Fedorusita/lab-07/assets/112895410/39c31ec4-2901-4b01-9593-f79ade417efd)
7.Протестируем наш Docker
```
$ sudo docker build -t logger .
$ sudo docker images
$ sudo docker run -it -v "$(pwd)/logs/:/home/lab-07/Fedor/" logger
$ sudo docker inspect logger
$ cat Fedor/fed.txt
```
![Test](https://github.com/Fedorusita/lab-07/assets/112895410/9368982c-83d4-4941-8a2d-88363d993199)
