# Git на пальцах

Учебный репозиторий. Здесь нет кода — только простые `.md` файлы.
Вся суть в **истории**: как появлялись коммиты, ветки, pull request'ы и мержи.

## Как изучать

1. Читай уроки по порядку (папка [`lessons/`](lessons/)).
2. Параллельно смотри, как это выглядело на самом деле:
   - вкладка **Commits** — вся история по шагам;
   - вкладка **Pull requests** → **Closed** — как вливались изменения;
   - вкладка **Pull requests** → **Open** — там задание для тебя 🙂
3. Локально: `git log --oneline --graph --all` — дерево истории прямо в терминале.

## Уроки

| # | Тема | Что посмотреть в истории |
|---|------|--------------------------|
| 1 | [Что такое git и коммит](lessons/01-osnovy.md) | первые коммиты в `main` |
| 2 | [.gitignore](lessons/02-gitignore.md) | файл `.gitignore` |
| 3 | [Ветки](lessons/03-vetki.md) | ветка `lesson/branches`, PR #1 |
| 4 | [Push и Pull Request](lessons/04-pull-request.md) | PR #2 |
| 5 | [Конфликты](lessons/05-konflikty.md) | PR #3 и PR #4, файл `pokupki.md` |

Шпаргалка по командам — [cheatsheet.md](cheatsheet.md).
