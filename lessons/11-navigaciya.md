# Урок 11. HEAD, `~` и `^`, `..` и `...`, виды мержей

## HEAD и detached HEAD

- **Ветка** — это просто указатель на коммит.
- **HEAD** — указатель на то, где ты сейчас. Обычно он указывает **на ветку**, а ветка — на коммит.
- **Detached HEAD** — HEAD указывает **прямо на коммит**, мимо ветки. Так бывает после
  `git switch --detach <hash>`, `git checkout <hash>` или `git checkout v1.0.0`.

В detached HEAD можно коммитить, но эти коммиты не принадлежат ни одной ветке.
Переключишься обратно — и найти их можно будет только через reflog. Если хочешь сохранить:

```bash
git switch -c new-branch    # повесить ветку на текущее место
```

## `~` и `^`

```
       ┌── F  (второй родитель merge-коммита M)
A──B──C──M          ← HEAD
```

| Запись | Что значит | Здесь |
|---|---|---|
| `HEAD~1` = `HEAD^` | первый родитель | `C` |
| `HEAD~2` | родитель родителя (по первым родителям) | `B` |
| `HEAD^2` | **второй** родитель (есть только у merge-коммитов) | `F` |
| `HEAD~2^2` | можно комбинировать | … |

Короче: `~` — шаги «назад по истории», `^N` — выбор родителя у мержа.

## `..` и `...` — работают по-разному в log и diff!

| | `A..B` | `A...B` |
|---|---|---|
| `git log` | коммиты, которые есть в B, но нет в A | коммиты, которые есть **только в одной** из веток |
| `git diff` | то же самое, что `git diff A B` | разница между **общим предком** A и B и веткой B |

```bash
git log main..feature        # что я наделал в ветке, чего ещё нет в main
git log feature..main        # что появилось в main, пока я работал
git diff main...feature      # ровно то, что показывает GitHub во вкладке PR "Files changed"
git merge-base main feature  # хеш общего предка
```

## Виды мержей

### Fast-forward

Если в `main` ничего нового не появилось с момента отделения ветки, git просто **передвигает указатель**
`main` вперёд. Merge-коммит не создаётся:

```
A──B          main             A──B──C──D   main, feature
    \    →
     C──D     feature
```

`git merge --no-ff feature` — **всегда** создать merge-коммит, даже если можно fast-forward.
Так в истории видно, что это была отдельная ветка. `git merge --ff-only` — мержить, только если
возможен fast-forward, иначе ошибка.

### Три кнопки мержа на GitHub

| Кнопка | Что получится в main | Пример в этом репо |
|---|---|---|
| **Create a merge commit** | все коммиты ветки + merge-коммит | PR #1–#4 |
| **Squash and merge** | один новый коммит со всеми изменениями | PR #5 |
| **Rebase and merge** | коммиты ветки переигрываются поверх main (новые хеши), merge-коммита нет | PR с `demo/rebase` |

### 🪤 Ловушка после squash / rebase merge

После **Squash and merge** команда `git branch -d feature` ругается:
*«The branch 'feature' is not fully merged»*. Коммитов ветки действительно нет в `main`:
там лежит **другой** коммит с теми же изменениями. Поэтому удалять придётся через `-D`.

То же самое, если продолжить работать в старой ветке после squash-мержа: при следующем PR
вылезут конфликты со своими же изменениями. **После squash-мержа начинай новую ветку от свежего main.**

## Трёхсторонний мерж (three-way merge)

Для мержа git сравнивает **три** версии: общего предка (merge-base), твою и чужую.
- Изменилось только у одного → берётся это изменение.
- Изменилось у обоих одинаково → ок.
- Изменилось у обоих по-разному → **конфликт**.

Чтобы в конфликте было видно и версию общего предка:

```bash
git config --global merge.conflictStyle zdiff3
```

---

## 🛠 Сделай сам

### Часть 1. `^1` и `^2` на настоящем мерже

**Зачем:** при revert мержа (`-m 1`), при cherry-pick мержа и в разборах «что принёс этот мерж»
нужно понимать, какой родитель чей.

```bash
git fetch
M=$(git log origin/main --merges --grep="#4 " --format=%h -1)   # merge-коммит PR #4
git log --oneline -1 $M          # Merge pull request #4 ...
git log --oneline -1 $M^1        # первый родитель — main до мержа (мерж PR #3)
git log --oneline -1 $M^2        # второй родитель — ветка (коммит, где разрешали конфликт)
git log --oneline -1 $M~2        # два шага назад по ПЕРВЫМ родителям
git diff --stat $M^1 $M          # что этот мерж принёс в main
```

✅ Ответь себе: почему `$M^2` и `$M~2` — разные коммиты?

### Часть 2. `..` и `...`

**Зачем:** «что в моей ветке, чего нет в main?» и «что покажет PR?» — вопросы на каждый день.

```bash
F=origin/practice/rebase-feature
MAIN=origin/practice/rebase-main
git log --oneline $MAIN..$F      # 4 коммита — есть в фиче, нет в main
git log --oneline $F..$MAIN      # 2 коммита — появились в main, пока шла работа
git log --oneline $MAIN...$F     # все 6 — "только в одной из веток"
git diff --stat $MAIN...$F       # изменения фичи от общего предка — так считает GitHub в PR
git diff --stat $MAIN $F         # а это "сравнить два снимка" — сюда попадут и чужие правки main
```

### Часть 3. Detached HEAD

**Зачем:** ты откроешь тег или старый коммит «посмотреть», закоммитишь там что-нибудь и потеряешь это.
Разберись один раз, чтобы потом не терять.

```bash
git switch --detach v1.0.0
git status                       # "HEAD detached at v1.0.0"
echo "эксперимент" > eksperiment.md
git add eksperiment.md && git commit -m "эксперимент на старой версии"
git switch main                  # git ПРЕДУПРЕДИТ: "you are leaving 1 commit behind" + хеш
git branch trening-11-spasen <хеш из предупреждения>
git log --oneline -1 trening-11-spasen
```

### Часть 4. Fast-forward vs `--no-ff`

```bash
git switch -c trening-11 main
git switch -c trening-11-ff
echo x > ff.md && git add ff.md && git commit -m "ff коммит"

git switch trening-11
git merge trening-11-ff          # "Fast-forward"
git lg -3                        # прямая линия, merge-коммита нет

git reset --hard ORIG_HEAD       # откатим
git merge --no-ff trening-11-ff -m "Merge trening-11-ff"
git lg -3                        # "ромбик" — видно, что была ветка
```

### Часть 5. Ловушка после squash

**Зачем:** на работе PR часто мержат через Squash. После этого у многих ломается `git branch -d`,
а в старой ветке вылезают странные конфликты.

```bash
git switch -c trening-11-sq trening-11
echo 1 > sq.md && git add sq.md && git commit -m "sq 1"
echo 2 >> sq.md && git commit -am "sq 2"

git switch trening-11
git merge --squash trening-11-sq
git commit -m "фича одним коммитом"
git lg -3                        # один коммит, никакой связи с веткой trening-11-sq

git branch -d trening-11-sq      # error: not fully merged!
```

Git прав: коммитов `sq 1` и `sq 2` в ветке нет, есть **другой** коммит с теми же изменениями.
Удалять придётся через `-D`.

**Уборка:**

```bash
git switch main
git branch -D trening-11 trening-11-ff trening-11-sq trening-11-spasen
```
