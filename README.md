# Git на пальцах 🎓

Учебный репозиторий. Кода здесь нет, только простые `.md` файлы.
Главное здесь — **история**: как появлялись коммиты, ветки, pull request'ы и мержи.
Всё, о чём рассказывают уроки, в этом репозитории делалось по-настоящему, и это можно посмотреть.

## С чего начать

1. Читай уроки по порядку, это папка [`lessons/`](lessons/).
2. После каждого урока смотри, как то же самое выглядело «вживую» (таблица ниже).
3. Потом переходи к упражнениям в [PRAKTIKA.md](PRAKTIKA.md).
4. Если забыл команду, открывай [cheatsheet.md](cheatsheet.md).

## Уроки и где их увидеть вживую

| # | Урок | Где посмотреть |
|---|------|----------------|
| 1 | [Что такое git и коммит](lessons/01-osnovy.md) | первые коммиты в `main`: вкладка **Commits** |
| 2 | [.gitignore](lessons/02-gitignore.md) | файл [`.gitignore`](.gitignore) |
| 3 | [Ветки](lessons/03-vetki.md) | [PR #1](../../pull/1): ветка с 2 коммитами влита в `main` |
| 4 | [Push и Pull Request](lessons/04-pull-request.md) | [PR #2](../../pull/2): доработка после ревью — просто ещё один коммит |
| 5 | [Конфликты](lessons/05-konflikty.md) | [PR #3](../../pull/3) влит без проблем, в [PR #4](../../pull/4) был конфликт в [`pokupki.md`](pokupki.md) (см. комментарий в PR) |
| 🎯 | **Задание для тебя** | открытый [PR #6](../../pull/6): сделай ревью и смержи сам |

### Продвинутый уровень: чтобы не плыть на собесе

| # | Урок | Где посмотреть |
|---|------|----------------|
| 6 | [cherry-pick](lessons/06-cherry-pick.md) | фикс из [PR #8](../../pull/8) перенесён в ветку `release/v1.0` через `cherry-pick -x`, вкладка **Releases**: `v1.0.0` → `v1.0.1` |
| 7 | [rebase и interactive rebase](lessons/07-rebase.md) | [PR #11](../../pull/11): ветка `demo/rebase-before` (5 грязных коммитов) → 2 чистых, влито через **Rebase and merge** |
| 8 | [Отмена: restore / reset / revert / amend](lessons/08-otmena.md) | вредный совет влит в [PR #9](../../pull/9) и откачен через `git revert -m 1` в [PR #10](../../pull/10) |
| 9 | [reflog: спасение «потерянного»](lessons/09-reflog.md) | упражнение в уроке |
| 10 | [stash](lessons/10-stash.md) | — |
| 11 | [HEAD, `~` и `^`, `..` и `...`, виды мержей](lessons/11-navigaciya.md) | в `main` есть все три вида мержей: merge commit, squash (#5), rebase (#11) |
| 12 | [Что внутри `.git`](lessons/12-vnutrennosti.md) | `git cat-file -p HEAD` |
| 13 | [bisect, blame, worktree, hooks и др.](lessons/13-instrumenty.md) | ветка `demo/bisect`: найди коммит, который сломал калькулятор |
| 14 | [Git в команде: процессы и правила](lessons/14-komanda.md) | теги `v1.0.0` / `v1.0.1` (SemVer) |
| 🎤 | **[Вопросы с собеседований](SOBES.md)** | 24 вопроса с ответами + практические задачки |

## Как выглядит история

Локально выполни:

```bash
git log --oneline --graph --all
```

Упрощённо история выглядит так:

```mermaid
gitGraph
    commit id: "init"
    commit id: "шпаргалка"
    branch lesson/conflicts-part2
    commit id: "Масло"
    checkout main
    branch lesson/branches
    commit id: "урок 3"
    commit id: "fix 3"
    checkout main
    merge lesson/branches id: "PR #1"
    branch lesson/pr-demo
    commit id: "урок 4"
    commit id: "fix 4"
    checkout main
    merge lesson/pr-demo id: "PR #2"
    branch lesson/conflicts-part1
    commit id: "Сыр"
    checkout main
    merge lesson/conflicts-part1 id: "PR #3"
    checkout lesson/conflicts-part2
    merge main id: "разрешил конфликт"
    checkout main
    merge lesson/conflicts-part2 id: "PR #4"
```

Как это читать:
- **PR #1–#3**: ветка отделилась от `main`, в ней появились коммиты, потом её влили обратно.
- **Ветка `conflicts-part2`** отделилась рано, ещё до «Сыра». Когда в `main` уже был «Сыр», а в ветке «Масло», получился конфликт.
  Чтобы его разрешить, в ветку влили `main` (коммит «разрешил конфликт»), руками починили файл, и после этого PR #4 смержился.
