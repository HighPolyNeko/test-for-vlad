# Урок 13. Инструменты, которые отличают мидла

## bisect — найти коммит, который всё сломал

«Неделю назад работало, сейчас не работает, между ними 200 коммитов».
`git bisect` делает **бинарный поиск**: 200 коммитов проверяются примерно за 8 шагов.

```bash
git bisect start
git bisect bad                 # сейчас сломано
git bisect good v1.0.0         # тут работало
# git переключает на коммит посередине, ты проверяешь и говоришь:
git bisect good    # или
git bisect bad
# ...повторяешь, пока git не напишет: "<hash> is the first bad commit"
git bisect reset               # вернуться туда, откуда начал
```

Автоматически, если есть команда-проверка (код возврата 0 = good, не 0 = bad):

```bash
git bisect run npm test
```

### 👀 Вживую: найди баг сам

В ветке `demo/bisect` 10 коммитов, файл `demo/kalkulyator.md` растёт по строчке.
В какой-то момент там появилось `2 + 2 = 5`. Найди коммит:

```bash
git fetch
git bisect start origin/demo/bisect origin/demo/bisect~9   # bad, потом good
cat demo/kalkulyator.md       # смотри, есть ли "= 5"
git bisect good               # или bad — и так далее
git bisect reset
```

Или в одну строку:

```bash
git bisect start origin/demo/bisect origin/demo/bisect~9
git bisect run sh -c '! grep -q "2 + 2 = 5" demo/kalkulyator.md'
git bisect reset
```

## blame и поиск по истории

```bash
git blame file.md                # кто и в каком коммите последний раз менял каждую строку
git blame -w -C file.md          # игнорировать пробелы и найти строки, перенесённые из других файлов
git log -S "функция" --oneline   # "pickaxe": коммиты, где ЧИСЛО вхождений строки изменилось (добавили/удалили)
git log -G "regex" --oneline     # коммиты, где строки с этим regex попали в diff
git log -p --follow -- file.md   # вся история файла, даже если его переименовывали
git log --author="Вася" --since="2 weeks ago"
git shortlog -sn                 # кто сколько коммитов сделал
git grep "TODO"                  # поиск по файлам репо (быстрее обычного grep)
```

Массовое переформатирование ломает `blame`. Лечится так: хеши таких коммитов кладутся
в `.git-blame-ignore-revs` и подключаются командой
`git config blame.ignoreRevsFile .git-blame-ignore-revs`. GitHub этот файл тоже понимает.

## worktree — две ветки одновременно

Не нужно stash'ить и переключаться: рядом создаётся вторая рабочая папка на другой ветке.

```bash
git worktree add ../hotfix main    # папка ../hotfix с веткой main
cd ../hotfix  # ...чиним, коммитим...
git worktree remove ../hotfix
```

Одна и та же ветка не может быть открыта в двух worktree одновременно.

## clean — удалить мусор

```bash
git clean -n     # ПОКАЗАТЬ, что будет удалено (всегда сначала так!)
git clean -fd    # удалить неотслеживаемые файлы и папки
git clean -fdx   # + игнорируемые (.gitignore) — например, build/ и node_modules/
```

## Переводы строк (CRLF/LF)

Видел при коммитах `warning: LF will be replaced by CRLF`? Это оно.
Windows использует `CRLF`, Linux/macOS — `LF`. Если не договориться, diff будет
состоять из «изменённых» строк, где на самом деле ничего не менялось.

- `core.autocrlf=true` (Windows): в репозиторий кладётся LF, в рабочую папку — CRLF.
- Надёжнее зафиксировать правило в самом репо, в файле `.gitattributes`:

```gitattributes
* text=auto eol=lf
*.bat text eol=crlf
*.png binary
```

## Hooks

Скрипты в `.git/hooks/`, которые git запускает сам: `pre-commit` (линтер), `commit-msg` (проверка
формата сообщения), `pre-push` (тесты).

- Папка `.git/hooks` **не коммитится**. Поэтому в командах используют `husky`, `lefthook`,
  `pre-commit` или `git config core.hooksPath .githooks`.
- Хуки обходятся через `--no-verify`. Значит, это удобство, а не защита. Настоящая проверка — в CI.

## Прочее, о чём полезно знать

| Что | Зачем |
|---|---|
| `git rerere` (`git config rerere.enabled true`) | git запоминает, как ты разрешил конфликт, и в следующий раз разрешает так же сам |
| `git submodule` | чужой репозиторий внутри твоего, зафиксированный на конкретном коммите. После клона нужен `git submodule update --init --recursive` |
| `git switch` / `git restore` | новые команды, которые разделили перегруженный `git checkout` на две |
| `git config --global alias.lg "log --oneline --graph --all"` | алиасы: теперь `git lg` |
| `git commit -S` | подписанный коммит (GPG/SSH), на GitHub будет значок *Verified* |
| `git add -p` | добавлять в коммит не весь файл, а отдельные куски |
