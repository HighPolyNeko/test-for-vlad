# Урок 9. reflog — машина времени для машины времени

Сделал `reset --hard`, удалил ветку, неудачно отребейзил — и коммиты «пропали»?
Скорее всего, они ещё живы.

**reflog** — локальный журнал того, куда указывал `HEAD` (и каждая ветка) за последнее время.

```bash
git reflog
```

```
7cc33be HEAD@{0}: reset: moving to HEAD~3
a1b2c3d HEAD@{1}: commit: важный коммит
e4f5a6b HEAD@{2}: commit: ещё один
...
```

## Спасение

```bash
git reset --hard HEAD@{1}         # вернуть ветку туда, где она была шаг назад
git branch rescued a1b2c3d        # или просто повесить на потерянный коммит новую ветку
```

Удалил ветку `feature` через `git branch -D`? Git печатает хеш при удалении:
`Deleted branch feature (was a1b2c3d)`. Восстанавливается так: `git branch feature a1b2c3d`.
Если хеш не запомнил, найди его в `git reflog`.

Неудачный rebase / merge / reset:

```bash
git reset --hard ORIG_HEAD    # ORIG_HEAD = где была ветка перед последней "опасной" операцией
```

## Что важно знать

- reflog **локальный**. Его нет на GitHub, и при `git clone` он не приходит.
- Записи живут не вечно: по умолчанию около 90 дней, для недостижимых коммитов около 30.
  Потом `git gc` может удалить коммиты насовсем.
- **Незакоммиченные изменения reflog не спасёт.** Есть одно исключение: если файл был в staging
  (`git add`), его содержимое лежит в `.git` как объект, и его можно найти через
  `git fsck --lost-found`. Отсюда правило: коммить часто, и в конце всё равно можно причесать rebase'ом.

## Попробуй

```bash
git switch -c trening-reflog
git commit --allow-empty -m "коммит, который мы потеряем"
git reset --hard HEAD~1       # "потеряли"
git log --oneline -3          # коммита нет
git reflog -3                 # а тут есть!
git reset --hard HEAD@{1}     # вернули
git switch main && git branch -D trening-reflog
```
