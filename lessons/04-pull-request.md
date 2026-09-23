# Урок 4. Push и Pull Request

**Push** — отправить твои коммиты на GitHub.  
**Pull Request** (PR) — просьба влить твою ветку в `main`.

## Процесс

### 1. Пуш

Ты делаешь коммиты в своей ветке, потом отправляешь:

```bash
git push -u origin feature/new-page
```

`-u` связывает твою локальную ветку с веткой на GitHub (`origin/feature/new-page`).
Следующий раз просто `git push`.

### 2. Pull Request на GitHub

На GitHub появляется кнопка **Compare & pull request**. Нажимаешь, описываешь,
что сделал — готово!

```markdown
## Что я сделал
- Добавил новую страницу профиля
- Обновил навигацию

## Как проверить
Открой /profile в браузере

Closes #42
```

### 3. Ревью и Merge

Кто-то другой (или ты в другом браузере 😄) смотрит PR:
- `Approve` — всё норм, влей.
- `Request changes` — надо подправить, мержить пока нельзя.
- `Comment` — просто вопрос или замечание, без вердикта.

Нажимаешь **Merge pull request** → коммиты вливаются в `main`.

## Результат

```
main:  ○─────────────○  (слита ветка feature/new-page)
```

> ⚠️ После мержа не забудь:
> ```bash
> git switch main
> git pull          # забрать влитые коммиты
> git branch -d feature/new-page  # удалить локальную ветку
> ```

---

## 🛠 Сделай сам 🔑

**Зачем:** в нормальной команде в `main` напрямую не пушат, всё идёт через PR. Так кто-то второй
смотрит код до того, как он попадёт в продукт, а в истории остаётся обсуждение: почему сделали именно так.

> Нужны права на пуш (Collaborators). Если их нет, нажми **Fork** на GitHub, склонируй свой форк
> и делай всё там. PR откроется из твоего форка в этот репозиторий.

### Часть 1. Свой первый PR

```bash
git switch main
git pull
git switch -c privet/tvoe-imya          # подставь своё имя латиницей

mkdir -p druzya
echo "# Привет! Я <имя>, изучаю git" > druzya/tvoe-imya.md
git add druzya && git commit -m "docs: представился"

git push -u origin privet/tvoe-imya
```

В ответ на `push` git напечатает ссылку `Create a pull request ... https://github.com/...`.
Открой её, напиши описание (что сделал и зачем) и нажми **Create pull request**.

### Часть 2. Доработка после ревью

Представь, что ревьюер попросил: «добавь, откуда ты». Никаких новых PR не нужно:

```bash
echo "Я из <город>" >> druzya/tvoe-imya.md
git commit -am "docs: добавил город"
git push                                # просто push — PR обновится сам
```

Обнови страницу PR: во вкладке **Commits** теперь 2 коммита.

### Часть 3. Мерж и уборка

На GitHub нажми **Merge pull request** (вариант *Create a merge commit*), потом **Delete branch**.

```bash
git switch main
git pull                                # забрать свой мерж
ls druzya                               # твой файл уже в main
git branch -d privet/tvoe-imya          # -d (маленькая) — удалит, т.к. ветка влита
git fetch --prune                       # убрать origin/privet/... — ветку на GitHub мы удалили
git branch -a | grep privet             # пусто
```

✅ **Проверь себя:** твой PR во вкладке **Closed** с пометкой *Merged*, файл `druzya/<имя>.md` лежит в `main`.

### Часть 4. Побудь ревьюером

Открой [PR #6](../../pull/6) и выполни задание оттуда: оставь комментарий к строке, поставь Approve и смержи.
