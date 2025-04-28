# Команды для работы с git которые приходилось применять

# Внесены изменения по отношению к origin/main, необходимо слить их в удаленный репозиторий origin main

```bash
git checkout --ours origin/main
```

Сравнить файл с файлом в ветке origin/main\
```bash
git diff origin/main /path/to/file
```

Сравнить файл f2objects-shafran.py в индексе с последним коммитом:\
```bash
git diff --staged f2objects-shafran.py
```

Сравнить файл f2objects-shafran в индексе с заданным коммитом:\
```bash
git diff --staged <commit_hash> f2objects-shafran.py
```

Вывести список файлов в индексе (подготовленных к коммиту)\
```bash
git status --short
```

Вывести только список файлов в индексе без другой информации\
```bash
git diff --name-only --cached
```

Вывести список файлов в индексе без другой информации\
```bash
git diff --name-only --cached
```