# Практическое задание №6. Системы автоматизации сборки

П.Н. Советов, РТУ МИРЭA

Работа с утилитой Make.

Изучить основы языка утилиты make. Распаковать в созданный каталог [make.zip](make.zip), если у вас в в системе нет make.

Создать приведенный ниже Makefile и проверить его работоспособность.

```
dress: trousers shoes jacket
    @echo "All done. Let's go outside!"

jacket: pullover
    @echo "Putting on jacket."

pullover: shirt
    @echo "Putting on pullover."

shirt:
    @echo "Putting on shirt."

trousers: underpants
    @echo "Putting on trousers."

underpants:
    @echo "Putting on underpants."

shoes: socks
    @echo "Putting on shoes."

socks: pullover
    @echo "Putting on socks."
```

Визуализировать файл [civgraph.txt](civgraph.txt).


## Задача 1

Написать программу на Питоне, которая транслирует граф зависимостей civgraph в makefile в духе примера выше. Для мало знакомых с Питоном используется упрощенный вариант civgraph: [civgraph.json](civgraph.json).

Пример:

```
> make mathematics
mining
bronze_working
sailing
astrology
celestial_navigation
pottery
writing
code_of_laws
foreign_trade
currency
irrigation
masonry
early_empire
mysticism
drama_poetry
mathematics
```
## Решение
```Python
import json

def parse_civgraph(civgraph_file):
    with open(civgraph_file, 'r') as file:
        data = json.load(file)

    return data


def generate_makefile(data, output_file):
    with open(output_file, 'w') as file:
        for target, dependencies in data.items():
            dep_str = " ".join(dependencies) if dependencies else ""
            file.write(f"{target}: {dep_str}\n")
            file.write(f"\t@echo {target}\n\n")

def main():
    civgraph_file = "civgraph.json" 
    output_file = "Makefile"

    data = parse_civgraph(civgraph_file)

    generate_makefile(data, output_file)
    print(f"{output_file} был сгенерирован")

if __name__ == "__main__":
    main()
```
![image](https://github.com/user-attachments/assets/10aca955-c68b-45a7-a595-69b80e34796c)

## Задача 2

Реализовать вариант трансляции, при котором повторный запуск make не выводит для civgraph на экран уже выполненные "задачи".

## Решение
```Python
import json
import os

TASKS_FILE = "tasks.txt"

def load_tasks():
    if os.path.exists(TASKS_FILE):
        with open(TASKS_FILE, 'r') as f:
            return set(f.read().splitlines())
    return set()

def save_tasks(tasks):
    with open(TASKS_FILE, 'w') as f:
        f.write('\n'.join(tasks))

def load_dependency_graph(filename):
    try:
        with open(filename, 'r') as f:
            return json.load(f)
    except Exception as e:
        print(f"Ошибка при загрузке файла {filename}: {e}")
        return {}

def generate_makefile(dependency_graph, target_task):
    visited_tasks = set()
    tasks_to_process = []
    completed_tasks = load_tasks()

    def process_task(task):
        if task in visited_tasks or task in completed_tasks:
            return
        visited_tasks.add(task)
        for dependency in dependency_graph.get(task, []):
            process_task(dependency)
        tasks_to_process.append(task)

    process_task(target_task)

    if not tasks_to_process:
        print("Эти задачи уже были выполнены.")
    else:
        for task in tasks_to_process:
            if task not in completed_tasks:
                print(f"{task}")
                completed_tasks.add(task)

        save_tasks(completed_tasks)


if __name__ == '__main__':
    dependency_graph = load_dependency_graph('civgraph.json')

    if not dependency_graph:
        print("Не удалось загрузить граф зависимостей. Программа завершена.")
    else:
        target_task = input('>make ')
        generate_makefile(dependency_graph, target_task)
```
![image](https://github.com/user-attachments/assets/965d737d-5955-48b9-9916-6b3dffb69196)

## Задача 3

Добавить цель clean, не забыв и про "животное".

## Решение 
```Python
import json
import os

TASKS_FILE = "completed_tasks.txt"

def load_tasks():
    if os.path.exists(TASKS_FILE):
        with open(TASKS_FILE, 'r') as f:
            return set(f.read().splitlines())
    return set()

def save_tasks(tasks):
    with open(TASKS_FILE, 'w') as f:
        f.write('\n'.join(tasks))

def load_dependency_graph(filename):
    try:
        with open(filename, 'r') as f:
            return json.load(f)
    except Exception as e:
        print(f"Ошибка при загрузке файла {filename}: {e}")
        return {}

def generate_makefile(dependency_graph, target_task):
    visited_tasks = set()
    tasks_to_process = []
    completed_tasks = load_tasks()

    def process_task(task):
        if task in visited_tasks or task in completed_tasks:
            return
        visited_tasks.add(task)
        for dependency in dependency_graph.get(task, []):
            process_task(dependency)
        tasks_to_process.append(task)

    process_task(target_task)

    if not tasks_to_process:
        print("Эти задачи уже были выполнены.")
    else:
        for task in tasks_to_process:
            if task not in completed_tasks:
                print(f"{task}")
                completed_tasks.add(task)

        save_tasks(completed_tasks)

def clean():
    if os.path.exists(TASKS_FILE):
        os.remove(TASKS_FILE)
        print(f"Файл  {TASKS_FILE} удален.")
    else:
        print("Нечего очищать.")


if __name__ == '__main__':
    # Загружаем граф зависимостей из файла
    dependency_graph = load_dependency_graph('civgraph.json')

    if not dependency_graph:
        print("Не удалось загрузить граф зависимостей.")
    else:
        action = input('make/clean: ')

        if action == 'make':
            target_task = input('>make ')
            generate_makefile(dependency_graph, target_task)
        elif action == 'clean':
            clean()
        else:
            print("Неизвестное действие.")
```
![image](https://github.com/user-attachments/assets/4d6d454d-3120-4132-91dd-955c6a50bc78)

## Задача 4

Написать makefile для следующего скрипта сборки:

```
gcc prog.c data.c -o prog
dir /B > files.lst
7z a distr.zip *.*
```

Вместо gcc можно использовать другой компилятор командной строки, но на вход ему должны подаваться два модуля: prog и data.
Если используете не Windows, то исправьте вызовы команд на их эквиваленты из вашей ОС.
В makefile должны быть, как минимум, следующие задачи: all, clean, archive.
Обязательно покажите на примере, что уже сделанные подзадачи у вас не перестраиваются.

## Решение
```Make
CC = gcc

CFLAGS = -o prog

SRC = prog.c data.c

all: prog archive

prog: $(SRC)
	$(CC) $(SRC) $(CFLAGS)

files.lst:
	dir /B > files.lst

archive: files.lst
	7z a distr.zip *.*

clean:
	del prog.exe files.lst distr.zip
```
![image](https://github.com/user-attachments/assets/2697c7cf-e80f-42c1-b959-90f2c739d763)
![image](https://github.com/user-attachments/assets/ec520e44-74e3-474a-adf0-6ee6b4b6ff17)


## Полезные ссылки

Описание (рус.): https://unix1.jinr.ru/faq_guide/programming/make/make.html

Шпаргалка: https://devhints.io/makefile

Топологическая сортировка: https://ru.wikipedia.org/wiki/Топологическая_сортировка
