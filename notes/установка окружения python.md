---
tags:
  - todo
---
# Conda

> [!note] Короткая инструкция по самым популярным функциям
> https://conda.io/projects/conda/en/latest/user-guide/cheatsheet.html

## Быстрый старт

```bash
conda create -n my_venv python=3.9
conda env list
conda install -n my_venv numpya
```
# Virtual env

```bash
python -m vevn my_venv
# для Linux 
source my_venv/bin/activate #запуск
deactivate #остановка

# для Windows
.\my_venv\Scripts\activate #запуск
.\my_venv\Scripts\deactivate #остановка

pip install -r requirements.txt
```
# pip
## Обновление

```bash
python -m pip install --upgrade pip
```
## Установка пакета как модуль

```bash
pip install -e ./
```
# Прокидываем окружение в Jupyter Notebook

```bash
pip install --user 
python -m ipykernel install --user --name=my_venv
```

После чего выберите ваше окружение в ноутбуке. Может потребоваться перезагрузка

# uv

# PyCharm
## Установка

>[!hint] Интересное предложение для студентов (попробовать через vpn)
>Можно получить пакет программ по студенческому (в том числе проф. версии JetBrains)
>https://education.github.com/pack

>[!hint] Удобный способ работы с JetBrains IDEs
>https://www.jetbrains.com/toolbox-app/

