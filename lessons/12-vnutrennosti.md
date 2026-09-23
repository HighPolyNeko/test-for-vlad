# Урок 12. Что внутри .git

На собесе на мидла любят спросить: «как git хранит данные?».

## 4 типа объектов

| Объект | Что хранит |
|---|---|
| **blob** | содержимое файла (без имени!) |
| **tree** | папку: список имён → хеши blob'ов и вложенных tree |
| **commit** | хеш корневого tree + родитель(и) + автор + коммиттер + сообщение |
| **tag** (аннотированный) | ссылку на объект + автора тега + сообщение |

Каждый объект лежит по **хешу своего содержимого** (SHA-1; в новых версиях можно включить SHA-256).
Одинаковый контент — один и тот же хеш. Поэтому 100 одинаковых файлов хранятся один раз.

```
commit 7cc33be
 └─ tree
     ├─ README.md       → blob 3a1f...
     ├─ pokupki.md      → blob 9c0e...
     └─ lessons/        → tree
          ├─ 01-osnovy.md → blob ...
```

## Коммит — это снимок, а не diff

Коммит ссылается на **полное** дерево файлов. Diff git вычисляет на лету, сравнивая два снимка.
При этом на диске git всё-таки экономит место: неизменившиеся файлы — это те же blob'ы,
а в **packfile'ах** (`git gc`) похожие объекты хранятся как дельты. Но это оптимизация хранения,
а не модель данных.

## Ветки и HEAD — это просто файлы

```bash
cat .git/HEAD               # ref: refs/heads/main
cat .git/refs/heads/main    # 7cc33be... (хеш коммита)
```

Поэтому создание ветки мгновенное: пишется файл в 41 байт.
(Некоторые refs могут быть упакованы в `.git/packed-refs`.)

## Потрогай руками

```bash
git cat-file -t HEAD          # commit
git cat-file -p HEAD          # содержимое коммита: tree, parent, author...
git cat-file -p HEAD^{tree}   # содержимое корневой папки
git cat-file -p HEAD:README.md    # содержимое файла в этом коммите
echo "привет" | git hash-object --stdin   # посчитать хеш без сохранения
git count-objects -vH         # сколько весит репозиторий
```

## Теги

- **Лёгкий** (`git tag v1.0.0`) — просто указатель на коммит, как ветка, только не двигается.
- **Аннотированный** (`git tag -a v1.0.0 -m "Релиз"`) — отдельный объект с автором, датой и сообщением.
  **Для релизов используй аннотированные.**
- Теги **не пушатся** обычным `git push`. Нужно `git push origin v1.0.0` или `git push --tags`.

## Почему нельзя «просто поменять старый коммит»

Хеш коммита зависит от хеша родителя. Меняешь коммит в середине истории — меняется его хеш,
а значит и хеши **всех** коммитов после него. Отсюда вытекает всё про rebase и force-push.

---

## 🛠 Сделай сам

**Зачем:** когда понимаешь, что ветка — это файл с хешем, а коммит — снимок, половина «магии» git
(rebase, reflog, detached HEAD, почему после amend нужен force-push) становится очевидной.

### Часть 1. Пройди по объектам от коммита до файла

```bash
git cat-file -p HEAD                   # tree <хеш>, parent <хеш>, author...
git cat-file -p HEAD^{tree}            # список файлов и папок: blob/tree + хеши
git cat-file -p HEAD:pokupki.md        # содержимое blob'а — сам файл
git cat-file -t HEAD                   # commit
git cat-file -t HEAD^{tree}            # tree
```

### Часть 2. Одинаковое содержимое — один объект

```bash
echo "одинаковый текст" > a.md
cp a.md b.md
git hash-object a.md b.md              # два ОДИНАКОВЫХ хеша: имя файла в хеш не входит
rm a.md b.md
```

### Часть 3. Ветка — это файл

```bash
git switch -c trening-12
cat .git/refs/heads/trening-12         # просто хеш
git rev-parse HEAD                     # тот же хеш
cat .git/HEAD                          # ref: refs/heads/trening-12

# создадим ветку БЕЗ команды git branch:
git rev-parse HEAD~3 > .git/refs/heads/trening-rukami
git branch                             # trening-rukami появилась!
git log --oneline -1 trening-rukami
```

### Часть 4. Почему amend меняет хеш

```bash
git commit --allow-empty -m "пустой коммит"
git cat-file -p HEAD                   # в объекте есть message, author, время...
git rev-parse HEAD
git commit --amend --allow-empty -m "другое сообщение"
git rev-parse HEAD                     # другой хеш: поменялось содержимое объекта → поменялся хеш
```

### Часть 5. Два вида тегов

```bash
git tag trening-lite
git tag -a trening-annot -m "аннотированный"
git cat-file -t trening-lite           # commit — лёгкий тег указывает прямо на коммит
git cat-file -t trening-annot          # tag — отдельный объект
git cat-file -p trening-annot          # внутри: object, tagger, сообщение
```

✅ **Проверь себя:** объясни своими словами, почему rebase меняет хеши **всех** коммитов после
изменённого (подсказка: что записано в строке `parent`?).

**Уборка:**

```bash
git switch main
git branch -D trening-12 trening-rukami
git tag -d trening-lite trening-annot
```
