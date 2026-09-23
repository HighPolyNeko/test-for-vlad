# Урок 6. cherry-pick — перенести один коммит

`git cherry-pick <hash>` берёт **изменения** одного коммита и применяет их
к текущей ветке как **новый** коммит.

```
main:        A──B──C(fix)──D
                   │
                   │ cherry-pick C
                   ▼
release/v1.0: A──B──C'
```

`C'` содержит те же изменения, что и `C`, но это **другой коммит с другим хешем**:
у него другой родитель, и дата коммита тоже другая.

## Когда это нужно

Классика — **хотфикс в релизную ветку**. Версия 1.0 уже в проде, в `main` идёт
разработка 1.1. Нашли баг: чиним в `main`, а в релиз переносим **только фикс**,
без всех новых фич.

## 👀 Вживую в этом репо

- Ветка `release/v1.0` и тег `v1.0.0` — это «релиз».
- Фикс сначала попал в `main` через PR, потом его перенесли в `release/v1.0` через `cherry-pick -x`.
  Там же стоит тег `v1.0.1`.
- Открой коммит в `release/v1.0`: в конце сообщения будет строка
  `(cherry picked from commit ...)`. Её добавил флаг `-x`.

```bash
git log --oneline --graph main release/v1.0
```

## Команды

```bash
git cherry-pick <hash>            # один коммит
git cherry-pick -x <hash>         # + пометка "cherry picked from ..." в сообщении
git cherry-pick A..B              # коммиты после A до B включительно (A НЕ входит!)
git cherry-pick A^..B             # от A до B, A входит
git cherry-pick -n <hash>         # применить изменения, но не коммитить
git cherry-pick -m 1 <merge-hash> # cherry-pick merge-коммита: надо указать "основного родителя"
```

При конфликте всё как при мерже: правишь файл, `git add`, потом:

```bash
git cherry-pick --continue   # продолжить
git cherry-pick --skip       # пропустить этот коммит
git cherry-pick --abort      # отменить всё
```

## Подводные камни (любят на собесах)

- **`A..B` не включает `A`.** Частая ошибка: люди теряют первый коммит.
- **Дубли.** Если потом смержить ветку, откуда брали коммит, git обычно справится
  (изменения те же), но в истории окажется два коммита с одинаковым смыслом.
  Поэтому cherry-pick — инструмент для точечных переносов, а не способ «мержить по кусочкам».
- **Зависимости.** Коммит может опираться на код из предыдущих коммитов, которых
  нет в целевой ветке. Тогда будет конфликт или сломанная сборка.
- Всегда используй `-x` при переносе в релизные ветки: так видно, откуда пришёл коммит.

---

## 🛠 Сделай сам

**Зачем:** версия 1.0 уже у пользователей, а в разработке (`dev`) лежит куча недоделанных фич.
Пользователи нашли опечатки. Выпустить 1.0.1 нужно **только с фиксами**, без сырых фич.
Именно для этого и нужен cherry-pick.

Заготовки:
- `origin/practice/cherry-release` — «релиз 1.0», в файле опечатки `Првиет` и `Вайти`;
- `origin/practice/cherry-dev` — разработка: 2 фикса и 2 фичи.

```bash
git fetch
git log --oneline origin/practice/cherry-dev -5
```

```
... feat: чат с поддержкой
... fix: опечатка на кнопке
... fix: опечатка в приветствии
... feat: тёмная тема
... release: версия 1.0 приложения
```

### Часть 1. Почему не merge

```bash
git switch -c trening-reliz origin/practice/cherry-release
cat trenirovka/cherry/prilozhenie.md         # Првиет, Вайти

git merge --no-commit --no-ff origin/practice/cherry-dev
git status                                   # приехали tema.md и chat.md — сырые фичи в релизе!
git merge --abort                            # нет-нет-нет
```

### Часть 2. Перенести один фикс

```bash
git cherry-pick -x <хеш "fix: опечатка в приветствии">
cat trenirovka/cherry/prilozhenie.md         # Привет ✔, но Вайти всё ещё
ls trenirovka/cherry                         # только prilozhenie.md — фичи не приехали
git log -1                                   # в конце сообщения: (cherry picked from commit ...)
```

✅ **Проверь себя:**
- хеш нового коммита **отличается** от исходного (сравни `git log --oneline -1` и `git log --oneline origin/practice/cherry-dev`);
- `tema.md` и `chat.md` в папке нет.

### Часть 3. Ловушка с диапазоном

Теперь нужно перенести **оба** фикса одной командой. Начнём заново:

```bash
git reset --hard origin/practice/cherry-release

git cherry-pick <хеш фикса приветствия>..<хеш фикса кнопки>
cat trenirovka/cherry/prilozhenie.md         # ??? Вайти исправлено, а Првиет — НЕТ
```

`A..B` значит «после A и до B», сам `A` в диапазон **не входит**. Правильно так:

```bash
git reset --hard origin/practice/cherry-release
git cherry-pick <хеш фикса приветствия>^..<хеш фикса кнопки>
cat trenirovka/cherry/prilozhenie.md         # Привет, Войти ✔✔
git log --oneline -3                         # два новых коммита поверх релиза
```

### Часть 4. Выпусти 1.0.1

```bash
git tag -a trening-v1.0.1 -m "Хотфикс: опечатки"
git show trening-v1.0.1 --stat | head -5
```

✅ **Проверь себя:** на `trening-reliz` есть оба фикса и ни одной фичи. `git log --oneline -1 trening-v1.0.1` указывает на последний фикс.

**Уборка:**

```bash
git switch main
git branch -D trening-reliz
git tag -d trening-v1.0.1
```
