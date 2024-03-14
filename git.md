# Команды для работы с git которые приходилось применять

Сравнить файл f2objects-shafran.py в индексе с последним коммитом:<br>\
`git diff --staged f2objects-shafran.py`

Сравнить файл f2objects-shafran в индексе с заданным коммитом:<br>\
`git diff --staged <commit_hash> f2objects-shafran.py`

Вывести список файлов в индексе (подготовленных к коммиту)<br>\
`git status --short`

Вывести только список файлов в индексе без другой информации<br>\
`git diff --name-only --cached`
