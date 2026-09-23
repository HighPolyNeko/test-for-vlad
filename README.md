# Git на пальцах 🎓

Учебный репозиторий для понимания основ Git. Здесь нет кода — только простые `.md` файлы.  
Вся суть в **истории**: как появляются коммиты, ветки, pull request'ы и мержи.

## Структура

```
.
├── README.md                    ← ты здесь
├── cheatsheet.md               ← шпаргалка с команды
├── lessons/
│   ├── 01-osnovy.md           ← коммиты и staging
│   ├── 02-gitignore.md        ← игнорирование файлов
│   ├── 03-vetki.md            ← параллельные ветки
│   ├── 04-pull-request.md     ← push и PR на GitHub
│   └── 05-konflikty.md        ← разрешение конфликтов
└── pokupki.md                  ← файл для демонстрации конфликтов
```

## Как изучать

1. **Читай уроки** по порядку в папке [`lessons/`](lessons/).
2. **Смотри историю** параллельно:
   ```bash
   git log --oneline --graph --all
   ```
   Вот так выглядит вся история:
   ```
   * dc33333 (lesson/conflicts-part2) lesson: список покупок (вариант 2 - конфликтный)
   | * b8a93f6 (lesson/conflicts-part1) lesson: урок про конфликты + список покупок (вариант 1)
   |/
   | * dcc601f (lesson/pr-demo) lesson: урок про PR и push
   |/
   | * b8513d0 (lesson/branches) lesson: урок про ветки
   |/
   * e3af89c docs: добавил шпаргалку и урок про .gitignore
   * 3f17b63 init: README и первый урок про основы
   ```

3. **Локально пробуй команды** из уроков:
   ```bash
   git switch lesson/branches      # перейди в ветку
   git log --oneline              # посмотри коммиты
   git diff main lesson/branches  # сравни с main
   ```

## Уроки

| # | Тема | Ветка / Файл | Что смотреть |
|---|------|--------------|-------------|
| 1 | Основы (коммит, staging) | `main` | `3f17b63`, `e3af89c` |
| 2 | .gitignore | `main` | файл `.gitignore` |
| 3 | Ветки | `lesson/branches` | отдельная ветка с коммитом `b8513d0` |
| 4 | Push и Pull Request | `lesson/pr-demo` | отдельная ветка с коммитом `dcc601f` |
| 5 | Конфликты | `lesson/conflicts-part1` + `lesson/conflicts-part2` | две разные ветки меняют `pokupki.md` |

## Попробуй сам

### Слить одну ветку в main

```bash
git switch main
git merge lesson/branches          # без конфликтов
git log --oneline --graph --all    # посмотри результат
```

### Создать конфликт (и разрешить его)

```bash
git switch main
git merge lesson/conflicts-part1   # сливается нормально
git merge lesson/conflicts-part2   # КОНФЛИКТ! 💥
# git вам покажет, что конфликтует
git status                         # видишь CONFLICT?
# отредактируй pokupki.md, оставив нужное
git add pokupki.md
git commit -m "Merge: объединил варианты список"
```

## Быстрая справка

→ **[cheatsheet.md](cheatsheet.md)** — все команды на одной странице
