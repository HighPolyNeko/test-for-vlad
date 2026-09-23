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
git clean -nd    # ПОКАЗАТЬ, что будет удалено, включая папки (всегда сначала так!)
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

---

## 🛠 Сделай сам

### Часть 1. bisect

Выполни пример из раздела «👀 Вживую» выше: сначала руками (good/bad), потом через `bisect run`.

**Зачем:** «вчера работало, сегодня нет, между ними 40 коммитов от пяти человек». Смотреть каждый
глазами — полдня работы, bisect справится за 6 шагов.

✅ Оба способа должны найти один и тот же коммит. Посмотри на его сообщение: по нему ни за что не
догадаешься, что он что-то сломал. Так и бывает в жизни.

### Часть 2. «Кто и когда это удалил?»

**Зачем:** в коде была строчка, теперь её нет, и надо понять, почему её убрали.

```bash
git log -S "push --force" --oneline -- cheatsheet.md
```

Найдутся два коммита: один **добавил** вредный совет, второй его **удалил** (revert из урока 8).
`-S` ищет коммиты, где количество вхождений строки изменилось.

```bash
git blame -L 36,40 lessons/04-pull-request.md    # кто последним трогал строки 36–40
```

Там будет фикс статусов ревью: тот самый коммит, который потом переносили cherry-pick'ом.

### Часть 3. worktree

**Зачем:** нужно срочно глянуть или поправить `main`, но у тебя открыта ветка с недоделкой, а stash делать не хочется.

```bash
git worktree add ../hotfix-papka main              # ошибка! main уже открыт в основной папке
git worktree add -b trening-hotfix ../hotfix-papka main   # так можно: новая ветка от main
ls ../hotfix-papka                                  # полноценная рабочая папка
git worktree list
git worktree remove ../hotfix-papka
git branch -D trening-hotfix
```

### Часть 4. Hook, который не пускает плохие сообщения

**Зачем:** в команде договорились писать коммиты по Conventional Commits (урок 14). Хук напомнит сам.

```bash
cat > .git/hooks/commit-msg << 'HOOK'
#!/bin/sh
if ! grep -qE '^(feat|fix|docs|chore|refactor|test)(\(.+\))?!?: ' "$1"; then
  echo "❌ Нужен формат: feat: ..., fix: ..., docs: ..."
  exit 1
fi
HOOK
chmod +x .git/hooks/commit-msg

git switch -c trening-13
git commit --allow-empty -m "asdf"                  # ❌ отклонён хуком
git commit --allow-empty -m "feat: проверка хука"   # ✅
git commit --allow-empty -m "asdf" --no-verify      # ✅ прошёл — хук легко обойти
rm .git/hooks/commit-msg
```

**Вывод:** хуки — это удобство для себя. Настоящие проверки делаются в CI на сервере.

### Часть 5. clean

```bash
echo 1 > musor1.txt && mkdir -p musor && echo 2 > musor/2.txt
git clean -nd             # ПОКАЗАТЬ, что удалится (привычка: всегда сначала -n). Без d папки не покажет
git clean -fd             # удалить
```

**Уборка:**

```bash
git switch main
git branch -D trening-13
```
