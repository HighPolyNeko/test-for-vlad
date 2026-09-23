# Урок 10. stash — спрятать изменения

Сидишь в ветке, полфичи написано, и тут: «срочно почини прод!». Коммитить недоделку не хочется.

```bash
git stash push -m "полфичи логина"   # спрятать изменения, рабочая папка стала чистой
git switch main                      # чиним прод...
git switch feature/login
git stash pop                        # вернуть спрятанное
```

## Команды

| Команда | Что делает |
|---|---|
| `git stash` | спрятать изменения в отслеживаемых файлах |
| `git stash -u` | + **новые (untracked) файлы**. Без `-u` они остаются на месте! |
| `git stash push -m "текст" -- file.md` | спрятать только определённый файл, с подписью |
| `git stash -p` | выбрать куски интерактивно |
| `git stash list` | список: `stash@{0}`, `stash@{1}`, … |
| `git stash show -p stash@{1}` | что внутри |
| `git stash apply` | применить и **оставить** в списке |
| `git stash pop` | применить и **удалить** из списка |
| `git stash drop stash@{1}` | удалить |
| `git stash branch new-branch` | создать ветку от коммита, где делался stash, и применить туда |

## Подводные камни

- Самая частая ловушка — **новые файлы без `-u` не прячутся.** Ты их потом «переносишь» в чужую ветку.
- Если `pop` упал с конфликтом, stash **не удаляется** из списка. Разреши конфликт, потом `git stash drop`.
- stash — не место для долгого хранения: через месяц уже не вспомнишь, что там.
  Если это надолго, лучше сделать WIP-коммит в отдельной ветке.
- Альтернатива для «срочно переключиться» без stash — `git worktree` (урок 13).

---

## 🛠 Сделай сам

**Зачем:** ты посреди фичи, в файлах недоделка, и тут «срочно почини прод». Коммитить мусор не хочется,
а переключиться с грязными файлами нельзя (или можно, но недоделка поедет за тобой). stash — это карман.

### Часть 1. Ловушка с новыми файлами

```bash
git switch -c trening-10 main
echo "полфичи" >> pokupki.md          # правка в отслеживаемом файле
echo "новый экран" > ekran.md         # новый файл

git stash
git status                            # pokupki.md чистый, а ekran.md ОСТАЛСЯ! stash его не взял
git stash pop                         # вернём как было
```

### Часть 2. Правильно: `-u` и подпись

```bash
git stash push -u -m "фича: новый экран, недоделано"
git status                            # теперь чисто
git stash list                        # stash@{0}: On trening-10: фича: новый экран, недоделано

git switch main                       # "чиним прод"...
git switch trening-10
git stash pop
git status                            # и правка, и новый файл вернулись
```

✅ `git stash list` пустой, `ekran.md` на месте, в `pokupki.md` есть «полфичи».

### Часть 3. Конфликт при `pop`

```bash
git stash push -u -m "опять недоделка"
echo "срочная правка" >> pokupki.md
git commit -am "fix: срочная правка"  # пока фича лежала в кармане, этот же файл поменяли

git stash pop                         # CONFLICT в pokupki.md
git stash list                        # stash НЕ удалился! (при конфликте pop его не удаляет)
```

Разреши конфликт в `pokupki.md`, как в уроке 5, затем:

```bash
git add pokupki.md
git restore --staged pokupki.md       # оставить правки в файле, но не в staging
git stash drop                        # удалить stash руками — он же применён
```

✅ `git stash list` пустой, в `pokupki.md` есть обе строки.

**Уборка:**

```bash
git restore pokupki.md
rm -f ekran.md
git switch main
git branch -D trening-10
```
