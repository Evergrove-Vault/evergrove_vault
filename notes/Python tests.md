---
tags:
  - inbox
---

# Pytest

```
pip install pytest
```

create tests package in root  
Requirements:

- all files in tests must contains `test` in name
- all test functions in test files must contains `test` in name  
    For run

```
pytest -s -v tests\
```

-k for select tests by marker

### 

for run with coverage

```
pip install pytest-cov
```

for console log

```
pytest --cov=src/
```

for html

```
pytest --cov=src/ --cov-report html
```

### 

for allure report

```
npm install -g allure-commandline --save-dev
pip install allure-pytest
```

create or add in pytest.ini

```
[pytest]  
addopts = --alluredir allure-results
```

for create allure report with history

| Флаг                        | Кратко         | Что делает                                                                          |
| --------------------------- | -------------- | ----------------------------------------------------------------------------------- |
| `-v`                        | verbose        | Более подробный вывод (показывает имена тестов).                                    |
| `-q`                        | quiet          | Тише — минимум информации.                                                          |
| `-s`                        | show print     | Показывает `print()` и `stdout` в тестах.                                           |
| `-x`                        | exit first     | Останавливается на **первой ошибке/фейле**.                                         |
| `--maxfail=N`               | ограничение    | Останавливается после N ошибок.                                                     |
| `-k "expr"`                 | keyword        | Запуск тестов по ключевому слову или выражению (`-k "login or signup"`).            |
| `-m mark`                   | marker         | Запуск тестов с указанным маркером (`@pytest.mark.slow`).                           |
| `--tb=short/line/no`        | traceback      | Управление выводом трейсбеков (`--tb=short` = короче).                              |
| `--disable-warnings`        | убрать warning | Отключает предупреждения.                                                           |
| `--maxfail=1 -q`            | быстрый прогон | Быстро падает при ошибке (CI любит).                                                |
| `--lf`                      | last failed    | Запуск только упавших тестов с прошлого раза.                                       |
| `--ff`                      | failed first   | Сначала запускает упавшие в прошлый раз.                                            |
| `--sw`                      | stepwise       | Запускает по порядку и останавливается на ошибке, при следующем запуске продолжает. |
| `--pdb`                     | debug          | Запускает `pdb` (дебаггер) на месте ошибки.                                         |
| `--cov=MODULE`              | coverage       | Запуск с покрытием (требует `pytest-cov`).                                          |
| `--cov-report=term-missing` | coverage       | Показывает недотестированные строки.                                                |
| `--alluredir=DIR`           | allure         | Сохраняет результаты для Allure.                                                    |
| `--durations=N`             | performance    | Показывает самые медленные тесты.                                                   |
| `--maxfail=N`               | failures       | Прерывает после N ошибок.                                                           |

```
pytest -s -v tests\
npx allure-commandline generate --clean .\allure-results\
cp allure-report\history\* allure-results\history
pytest -s -v tests\
npx allure-commandline generate --clean .\allure-results\
npx allure-commandline open
```

|Флаг|Описание|
|---|---|
|`-o, --output <директория>`|Указывает папку для отчёта.|
|`--clean`|Удаляет содержимое папки отчёта перед генерацией.|
|`--report-version <версия>`|Устанавливает версию отчёта (например, `1.0`).|
|`--config <файл>`|Путь к файлу конфигурации Allure (например, `allure.properties`).|
|`--no-autoconfig`|Не использовать автоконфигурацию из системных переменных.|

for add some env for report add property file to allure-results

``` bash
coverage run -m pytest tests/
coverage report -m
coverage html

# Allure
pytest --alluredir=allure-results
allure serve allure-results
```


### **. Запуск тестов с coverage**

`# Запуск всех тестов и сбор покрытия coverage run -m pytest tests/`

---

### **2. Просмотр покрытия в консоли**

`coverage report -m`

Выведет что-то вроде:

`Name                  Stmts   Miss  Cover   Missing --------------------------------------------------- main.py                  10      0   100% services.py              30      2    93%   25-26 repository.py            40      0   100% ...`

---

### **3. Генерация HTML отчета coverage**

`coverage html`

- Результат будет в папке `htmlcov/`
    
- Открой `htmlcov/index.html` в браузере для наглядного просмотра покрытия кода.
    

---

### **4. Запуск Allure отчета**

`# Сбор данных для Allure pytest --alluredir=allure-results  # Запуск локального сервера для просмотра отчета allure serve allure-results`

- Откроется веб-интерфейс с красивой визуализацией тестов.
    
- В отчете будут: **feature**, **story**, **steps**, статус тестов, скриншоты (если добавлять), логи и история запусков.
    

---

### **5. Одним скриптом (для удобства студентов)**

Можно добавить файл `run_tests.sh`:

`#!/bin/bash  # Очистка старых отчетов rm -rf allure-results htmlcov  # Запуск тестов с coverage coverage run -m pytest tests/ --alluredir=allure-results  # Генерация HTML coverage отчета coverage html  # Запуск Allure сервера allure serve allure-results`

- Сделай его исполняемым: `chmod +x run_tests.sh`
    
- Запуск: `./run_tests.sh`
    
- Студенты увидят **coverage HTML** и **Allure отчет** в одном запуске.
  

## **Что такое `conftest.py`**

- Это специальный файл, который **pytest автоматически ищет** в директории тестов.
    
- В нём можно объявлять **фикстуры**, которые будут доступны **для всех тестов в текущей директории и её поддиректориях**.
    
- Файл не нужно импортировать вручную — pytest сам подхватывает фикстуры.
    

---

## **2. Пример использования**

### Структура проекта:

`tests/ ├─ conftest.py ├─ test_services.py ├─ test_controllers.py`

### **tests/conftest.py**

`import pytest from fastapi.testclient import TestClient from main import app from repository import EquipmentRepository from services import EquipmentService  # Фикстура для TestClient, доступная во всех тестах @pytest.fixture def client():     with TestClient(app) as c:         yield c  # Фикстура для сервисного слоя @pytest.fixture def service():     repo = EquipmentRepository()     return EquipmentService(repo)`

Теперь любые тесты могут использовать `client` и `service` без импорта:

### **tests/test_services.py**

`def test_create_item(service):     item = service.create_item("Tent", 5)     assert item.name == "Tent"`

### **tests/test_controllers.py**

`def test_create_item_api(client):     response = client.post("/items", json={"name": "Rope", "quantity": 10})     assert response.status_code == 200`

---

## **3. Дополнительные возможности**

- Можно задавать **scope** фикстуры: `function` (по умолчанию), `module`, `class`, `session`.  
    Пример:
    

`@pytest.fixture(scope="module") def client():     with TestClient(app) as c:         yield c`

- `scope="module"` — создаст один клиент для всех тестов модуля, что ускоряет выполнение.
    
- Фикстуры могут зависеть друг от друга:
    

`@pytest.fixture def repo():     return EquipmentRepository()  @pytest.fixture def service(repo):     return EquipmentService(repo)`