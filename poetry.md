# poetry

После настройки виртуального окружения для проекта, генерируем конфиг для
pyrightconfig.json чтобы редактор nvim не ругался на зависимости

```bash
jq \
  --null-input \
  --arg venv "$(basename $(poetry env info -p))" \
  --arg venvPath "$(dirname $(poetry env info -p))" \
  '{ "venv": $venv, "venvPath": $venvPath }' \
  > pyrightconfig.json
```

Сгенерится `pyrightconfig.json`

```json
{
  "venv": "app-v79h1USp-py3.9",
  "venvPath": "/home/develop/.cache/pypoetry/virtualenvs"
}
```

Отобразить список установленных пакетов для сравнения версий

```bash
# Задать вручную
poetry env use /full/path/to/python
# Если в переменной path есть исполняемый python то можно использовать его:
poetry env use python3.7
```

Отобразить список пакетов с зависимостями

```bash
poetry show ---tree
poetry show --latest
```

Информация о виртуальных окружениях

```bash
poetry env info
```

Создаем свои команды

```pyproject.toml
[tool.poetry.scripts]
mycommand="tmp.main:run"
```

Создаем файл `tmp/main.py' и пишем туда

```python
def run():
    print ("Hello World")
```

Затем в командной в корне проекта можно выполнить

```
poetry run mycommand
# Которая выведет "Hello World"
```
