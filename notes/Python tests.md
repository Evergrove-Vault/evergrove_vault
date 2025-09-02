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
npm install --save-dev allure-comandline
pip install allure-pytest
```

create or add in pytest.ini

```
[pytest]  
addopts = --alluredir allure-results
```

for create allure report with history

```
pytest -s -v tests\
npx allure-commandline generate --clean .\allure-results\
cp allure-report\history\* allure-results\history
pytest -s -v tests\
npx allure-commandline generate --clean .\allure-results\
npx allure-commandline open
```

for add some env for report add property file to allure-results