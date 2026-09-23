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

---

## 🛠 Сделай сам

**Зачем:** рано или поздно ты сделаешь `reset --hard` не туда, удалишь не ту ветку или неудачно
отребейзишь. В этот момент важно знать, что почти всё можно вернуть, и не паниковать.

### Часть 1. Удалил ветку с работой

```bash
git switch -c trening-9 main
echo "день работы" > reflog.md && git add reflog.md && git commit -m "важная работа 1"
echo "ещё день" >> reflog.md && git commit -am "важная работа 2"

git switch main
git branch -D trening-9          # git напечатал "(was abc1234)" — но допустим, ты не посмотрел
git log --oneline trening-9      # ошибка: такой ветки нет. Паника?

git reflog | grep "важная работа 2"      # вот он, хеш
git branch trening-9 <хеш>
git log --oneline -2 trening-9           # обе "важные работы" на месте
```

✅ Ветка восстановлена с обоими коммитами.

### Часть 2. Чего reflog НЕ спасёт

```bash
git switch trening-9
echo "не закоммичено и не добавлено" >> reflog.md
git reset --hard
cat reflog.md                    # строки нет, и в reflog её тоже нет — потеряна навсегда

echo "добавлено в staging, но не закоммичено" > spasi.md
git add spasi.md
git reset --hard                 # spasi.md исчез
git fsck --lost-found            # много "dangling ..." — это ничейные объекты, среди них и наш blob
grep -l "добавлено в staging" .git/lost-found/other/*   # git сложил их содержимое сюда — ищем по тексту
cat <найденный файл>             # вот оно, живое!
```

**Вывод:** закоммиченное почти не теряется, добавленное в staging можно откопать, а всё остальное пропадает.
Поэтому коммить чаще: причесать историю можно потом через `rebase -i`.

**Уборка:**

```bash
git switch main
git branch -D trening-9
rm -rf .git/lost-found
```
