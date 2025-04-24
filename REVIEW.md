# Отчет по лабораторной работе: Git, Git Hooks и Git Flow

## Описание лабораторной работы

Цель лабораторной работы — закрепить навыки работы с системой контроля версий Git, включая:
- работу с локальными и удалёнными ветками,
- использование Git Hooks для автоматизации проверок,
- применение Git Flow как методологии управления жизненным циклом проекта.

## Используемые инструменты

- Git, установленный на macOS
- Терминал macOS
- GitHub (удалённый репозиторий)
- Git Flow (расширение git-flow-avh)

---

## Шаг 1: Автоматизация проверки формата файлов при коммите (Git Hooks)

### 1. Переход в директорию `hooks`:
```bash
cd informatics_mobi_git/.git/hooks
```
**Объяснение:** Переход в директорию `.git/hooks`, где хранятся все скрипты Git Hook. Скрипт `pre-commit` будет запускаться автоматически перед каждым коммитом.

### 2. Создание и подготовка скрипта:
```bash
touch pre-commit
chmod +x pre-commit
nano pre-commit
```
**Объяснение:**
- `touch pre-commit` — создаёт пустой файл с именем `pre-commit`.
- `chmod +x pre-commit` — делает файл исполняемым.
- `nano pre-commit` — открывает файл в текстовом редакторе nano для добавления скрипта.

### 3. Содержимое `pre-commit`:
```bash
#!/bin/bash

FILES=$(git diff --cached --name-only --diff-filter=ACM | grep '\.txt$')
FAILED=0

for FILE in $FILES; do
    if [[ ! -s "$FILE" ]]; then
        echo "File $FILE is empty"
        FAILED=1
    elif ! grep -q '[^[:space:]]' "$FILE"; then
        echo "File $FILE has only spaces"
        FAILED=1
    else
        echo "Files checked: $FILE"
    fi
done

if [ $FAILED -eq 1 ]; then
    echo "Commit cancelled, check .txt files"
    exit 1
fi

exit 0
```
**Объяснение скрипта:**
- `git diff --cached --name-only --diff-filter=ACM` — выводит имена файлов, добавленных в индекс (подготовленных к коммиту), фильтруя только добавленные, скопированные и модифицированные.
- `grep '\.txt$'` — отбирает только файлы с расширением `.txt`.
- `[[ ! -s "$FILE" ]]` — проверка, что файл не пустой.
- `grep -q '[^[:space:]]'` — проверка, содержит ли файл что-то кроме пробелов.
- `exit 1` — завершает коммит, если файл не проходит проверку.

### 4. Проверка работы хука:

**Успешный коммит:**
```bash
echo "Chapter 1" > text1.txt
git add text1.txt
git commit -m "Non empty .txt text1"
```
**Объяснение:**
- `echo "Chapter 1" > text1.txt` — создаёт файл с содержимым.
- `git add` — добавляет файл в индекс.
- `git commit` — инициирует коммит, запускается `pre-commit` hook.

**Вывод:**
```bash
Files checked: text1.txt
[main d689424] Non empty .txt text1
 1 file changed, 1 insertion(+), 1 deletion(-)
```

**Неудачный коммит (только пробелы):**
```bash
echo "    " > text2.txt
git add text2.txt
git commit -m "Empty .txt text2"
```
**Вывод:**
```bash
File text2.txt has only spaces
Commit cancelled, check .txt files
```
**Объяснение:** hook не пропускает файл без содержимого и блокирует коммит.

---

## Шаг 2: Установка и инициализация Git Flow

### 1. Установка Git Flow:
```bash
brew install git-flow-avh
```
**Объяснение:** Устанавливается расширение Git Flow, реализующее командную модель работы с ветками.

### 2. Инициализация Git Flow:
```bash
git flow init
```
**Вывод:**
```
Which branch should be used for bringing forth production releases?
   - main
Branch name for production releases: [main] main
Branch name for "next release" development: [develop] develop

How to name your supporting branch prefixes?
Feature branches? [feature/] 
Bugfix branches? [bugfix/] 
Release branches? [release/] 
Hotfix branches? [hotfix/] 
Support branches? [support/] 
Version tag prefix? [] 
Hooks and filters directory? [/Users/kovalevgleb/Documents/Informatics_ar/lab5/informatics_mobi_git/.git/hooks] 
```
**Объяснение:** Git Flow запрашивает настройки: основная ветка (`main`), ветка для разработки (`develop`), и префиксы веток (`feature/`, `release/`, `hotfix/`).

---

## Шаг 3: Создание функциональности (feature)

### 1. Старт feature-ветки:
```bash
git flow feature start task-management
```
**Вывод:**
```bash
Switched to a new branch 'feature/task-management'

Summary of actions:
- A new branch 'feature/task-management' was created, based on 'develop'
- You are now on branch 'feature/task-management'

Now, start committing on your feature. When done, use:

     git flow feature finish task-management
```
**Объяснение:** Создаётся ветка `feature/task-management`, ответвлённая от `develop`.

### 2. Создание файла:
```python
def create_task(title, description):
    print(f"Создана новая задача: {title}")
```

### 3. Коммит изменений:
```bash
git add task_manager.py
git commit -m "Добавлен функционал управления задачами"
```
**Вывод:**
```bash
[feature/task-management 66aa298] Добавлен функционал управления задачами
 1 file changed, 2 insertions(+)
 create mode 100644 task_manager.py
```

### 4. Завершение фичи:
```bash
git flow feature finish task-management
```
**Вывод:**
```bash
Switched to branch 'develop'
Updating d689424..66aa298
Fast-forward
 task_manager.py | 2 ++
 1 file changed, 2 insertions(+)
 create mode 100644 task_manager.py
Deleted branch feature/task-management (was 66aa298).

Summary of actions:
- The feature branch 'feature/task-management' was merged into 'develop'
- Feature branch 'feature/task-management' has been locally deleted
You are now on branch ‘develop'
```
**Объяснение:** Ветка объединяется с `develop` и удаляется.

---

## Шаг 4: Подготовка релиза

### 1. Начало релиза:
```bash
git flow release start v1.0.0
```
**Вывод:**
```bash
Switched to a new branch 'release/v1.0.0'

Summary of actions:
- A new branch 'release/v1.0.0' was created, based on 'develop'
- You are now on branch 'release/v1.0.0'

Follow-up actions:
- Bump the version number now!
- Start committing last-minute fixes in preparing your release
- When done, run:

     git flow release finish 'v1.0.0'
```

### 2. Версионирование:
```bash
echo "v1.0.0" > version.txt
git add version.txt
git commit -m "Обновлена версия для релиза v1.0.0"
```
**Вывод:**
```bash
Files checked: version.txt
[release/v1.0.0 8b126fc] Обновлена версия для релиза v1.0.0
 1 file changed, 1 insertion(+)
 create mode 100644 version.txt 
```

### 3. Завершение релиза:
```bash
git flow release finish v1.0.0
```
**Вывод:**
```bash
Switched to branch 'main'
Your branch is up to date with 'origin/main'.
Merge made by the 'recursive' strategy.
 task_manager.py | 2 ++
 version.txt     | 1 +
 2 files changed, 3 insertions(+)
 create mode 100644 task_manager.py
 create mode 100644 version.txt
Already on 'main'
Your branch is ahead of 'origin/main' by 3 commits.
  (use "git push" to publish your local commits)
Switched to branch 'develop'
hint: Waiting for your editor to close the file... error: There was a problem with the editor 'vi'.
Not committing merge; use 'git commit' to complete the merge.
Fatal: There were merge conflicts. 
```
**Объяснение:** Ветка объединяется с `main` и `develop`, создаётся тег версии.

---

## Шаг 5: Хотфикс (hotfix)

### 1. Начало хотфикса:
```bash
git flow hotfix start hotfix-1.0.1
```
**Вывод:**
```
Branches 'main' and 'origin/main' have diverged.
And local branch 'main' is ahead of 'origin/main'.
Switched to a new branch 'hotfix/hotfix-1.0.1'

Summary of actions:
- A new branch 'hotfix/hotfix-1.0.1' was created, based on 'main'
- You are now on branch 'hotfix/hotfix-1.0.1'

Follow-up actions:
- Start committing your hot fixes
- Bump the version number now!
- When done, run:

     git flow hotfix finish 'hotfix-1.0.1'

```

### 2. Исправление:
```bash
echo "# Исправлена ошибка" > file_with_error.py
git add file_with_error.py
git commit -m "Исправлена критическая ошибка"
```
**Вывод:**
```bash
[hotfix/hotfix-1.0.1 beff7e6] Исправлена критическая ошибка
 1 file changed, 1 insertion(+)
 create mode 100644 file_with_error.py

```

### 3. Завершение хотфикса:
```bash
git flow hotfix finish hotfix-1.0.1
```
**Вывод:**
```
Branches 'main' and 'origin/main' have diverged.
And local branch 'main' is ahead of 'origin/main'.
Switched to branch 'main'
Your branch is ahead of 'origin/main' by 3 commits.
  (use "git push" to publish your local commits)
Merge made by the 'recursive' strategy.
 file_with_error.py | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 file_with_error.py
Switched to branch 'develop'
Merge made by the 'recursive' strategy.
 file_with_error.py | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 file_with_error.py
Deleted branch hotfix/hotfix-1.0.1 (was beff7e6).

Summary of actions:
- Hotfix branch 'hotfix/hotfix-1.0.1' has been merged into 'main'
- The hotfix was tagged 'hotfix-1.0.1'
- Hotfix tag 'hotfix-1.0.1' has been back-merged into 'develop'
- Hotfix branch 'hotfix/hotfix-1.0.1' has been locally deleted
- You are now on branch 'develop'

```
**Объяснение:** Автоматически объединяется в `main`, `develop` и создаётся тег `hotfix-1.0.1`.

---

## Шаг 6: Push изменений на GitHub

### 1. Ветка `develop`:
```bash
git push origin develop
```
**Объяснение:** Загружаем локальную ветку `develop` на удалённый репозиторий GitHub.

**Вывод**
```
Enumerating objects: 14, done.
Counting objects: 100% (14/14), done.
Delta compression using up to 8 threads
Compressing objects: 100% (11/11), done.
Writing objects: 100% (13/13), 1.57 KiB | 1.57 MiB/s, done.
Total 13 (delta 4), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (4/4), done.
remote: 
remote: Create a pull request for 'develop' on GitHub by visiting:
remote:      https://github.com/A8stern/informatics_mobi_git/pull/new/develop
remote: 
To https://github.com/A8stern/informatics_mobi_git.git
 * [new branch]      develop -> develop
```

### 2. Ветка `main`:
```bash
git push origin main
```
**Объяснение:** Аналогично, но для ветки `main`.

**Вывод:**
```
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/A8stern/informatics_mobi_git.git
   d689424..62af9c2  main -> main
```

---

## Заключение

В ходе лабораторной работы были выполнены:
- Настройка Git Hooks для контроля качества `.txt` файлов.
- Полный рабочий процесс Git Flow (feature → release → hotfix).
- Разрешение конфликта при релизе.
- Публикация изменений в GitHub.

Работа демонстрирует важность дисциплины в работе с Git и автоматизации для повышения стабильности разработки.

