# Урок 3. Ветки — параллельные истории

**Ветка** — независимая цепочка коммитов. Вместо того, чтобы все писали в одну `main`,
каждый может создать свою ветку и работать параллельно.

## Модель ветвей

```
         commit 1 (новая фишка)
            /
main:  ○──○──────○──○──  (основное развитие)
           \
            commit 2 (баг-фикс)
```

Все ветки живут независимо, потом их **мержат** в `main`.

## Команды

```bash
git branch                    # показать все ветки
git switch -c feature/login   # создать ветку и перейти
# ... правишь ...
git add . && git commit -m "Добавил форму логина"
git switch main               # вернуться в main
git branch                    # feature/login есть, но текущая main
```

## Цель

Каждая ветка — **отдельная задача**. Пока одна ветка ждёт мержа в `main`, в других
можно продолжать работать. Никому не мешаешь.

### Примеры имён веток

```
feature/add-admin-panel     # новая фишка
fix/logout-button           # багфикс
docs/update-readme          # документация
```

> 💡 Имя ветки должно показывать, над чем работаешь.

---

## 🛠 Сделай сам

**Зачем:** ты три дня пилишь фичу, и тут прилетает срочный баг. Без веток пришлось бы чинить баг
прямо поверх недоделки. С ветками переключился, починил, вернулся, и никто никому не мешает.

```bash
git switch -c trening-3-ficha
echo "полфичи" > ficha.md
git add ficha.md && git commit -m "фича: начало"

git switch main
ls ficha.md              # "No such file" — в main этого файла нет

git switch -c trening-3-bagfix
echo "починил" > bagfix.md
git add bagfix.md && git commit -m "fix: срочный баг"

git switch trening-3-ficha
ls                       # ficha.md есть, bagfix.md нет

git branch -v            # все ветки и последний коммит в каждой
git lg main trening-3-ficha trening-3-bagfix -4   # две ветки растут из одной точки
```

✅ **Проверь себя:** в `git lg` видно две «веточки» от одного коммита `main`.
В каждой ветке только свой файл.

### Неочевидное: незакоммиченные правки переезжают

```bash
echo "не закоммичено" >> README.md
git switch trening-3-bagfix      # git разрешил переключиться...
git status                       # ...и правка README.md приехала с тобой!
git restore README.md            # выкинуть правку

git switch trening-3-ficha
echo "не закоммичено" >> ficha.md
git switch trening-3-bagfix      # ❌ error: Your local changes ... would be overwritten
git restore ficha.md
```

Почему по-разному? `README.md` одинаковый в обеих ветках, поэтому git спокойно переносит правку.
А `ficha.md` в ветке `bagfix` нет: при переключении git пришлось бы его удалить вместе с твоей правкой,
поэтому он отказывается. Выход: закоммитить или спрятать (`stash`, урок 10).

**Уборка:**

```bash
git switch main
git branch -D trening-3-ficha trening-3-bagfix
```
