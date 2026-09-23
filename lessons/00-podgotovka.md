# Урок 0. Подготовка к практике

В конце каждого урока есть блок **🛠 Сделай сам**. Читать мало: руки должны сами набрать всё это хотя бы раз.
В каждом упражнении три части:

- **Зачем** — в какой ситуации на работе это пригодится;
- **Шаги** — что делать;
- **✅ Проверь себя** — что должно получиться. Не получилось — перечитай урок или посмотри «Если всё сломал» ниже.

## Один раз настроить

Все команды выполняются в **Git Bash** (на Windows он ставится вместе с git).

```bash
git clone https://github.com/HighPolyNeko/test-for-vlad.git
cd test-for-vlad

git config --global user.name "Твоё Имя"
git config --global user.email "твоя@почта"

# редактор для rebase -i и сообщений коммитов (выбери один):
git config --global core.editor "code --wait"   # VS Code
git config --global core.editor notepad          # Блокнот

# удобный алиас: дерево истории
git config --global alias.lg "log --oneline --graph --decorate"
```

## Правило тренировки

Все тренировочные ветки называй с префиксом **`trening-`**. Тогда их можно удалить разом,
и настоящие ветки не пострадают.

Всё делается **локально**. Если сам не сделаешь `git push`, на GitHub ничего не уедет, так что ломай смело.
Упражнения с пометкой 🔑 требуют прав на пуш: попроси хозяина репо добавить тебя в Collaborators.

## 🆘 Если всё сломал

```bash
git status                      # сначала посмотри, что происходит, — git обычно сам подсказывает
git merge --abort               # если застрял посреди мержа
git rebase --abort              # ...посреди rebase
git cherry-pick --abort         # ...посреди cherry-pick

# полный сброс к состоянию GitHub (⚠️ удалит все незакоммиченные правки):
git switch main
git reset --hard origin/main
git clean -fd                                   # удалить новые неотслеживаемые файлы
git branch | grep trening | xargs -r git branch -D   # удалить все тренировочные ветки
git tag | grep trening | xargs -r git tag -d          # и тренировочные теги
```
