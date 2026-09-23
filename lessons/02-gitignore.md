# Урок 2. .gitignore — что git не должен видеть

Не всё в папке нужно хранить в истории: настройки редактора, пароли,
временные файлы, `node_modules` и т.п.

Файл [`.gitignore`](../.gitignore) — список того, что git просто игнорирует.

```gitignore
# настройки IDE (JetBrains)
.idea/

# любые логи
*.log
```

## Проверка

Если открыть эту папку в IDE от JetBrains, та создаст папку `.idea/`. В репозиторий она не попадёт,
потому что указана в `.gitignore`. Сравни:

```bash
ls -a          # .idea есть на диске (если открывал в JetBrains IDE)
git status     # а git про неё молчит
```

> ⚠️ `.gitignore` работает только для файлов, которые ещё **не** закоммичены.
> Если файл уже в истории — сначала `git rm --cached <файл>`.

---

## 🛠 Сделай сам

**Зачем:** в каждой папке проекта есть то, что нельзя коммитить: пароли (`.env`), логи, сборку (`build/`),
настройки IDE. Один раз забыл про `.gitignore`, и пароль от базы уже на GitHub.

### Часть 1. Игнорирование

```bash
git switch -c trening-2

echo "PASSWORD=123" > .env
echo "ошибка" > debug.log
mkdir -p build && echo "мусор" > build/app.txt

git status                       # видно .env и build/, а debug.log — нет. Почему?
git check-ignore -v debug.log    # отвечает: .gitignore, строка с *.log

echo ".env" >> .gitignore
echo "build/" >> .gitignore
git status                       # остался только изменённый .gitignore
```

✅ `git status` показывает только `modified: .gitignore`, а `.env` и `build/` исчезли из списка.

### Часть 2. Ловушка: файл уже закоммичен

```bash
echo "мои настройки" > config.local
git add config.local .gitignore
git commit -m "добавил config.local"

echo "config.local" >> .gitignore
echo "ещё настройка" >> config.local
git status               # config.local ВСЁ РАВНО в списке изменённых! .gitignore не помог

git rm --cached config.local     # перестать отслеживать, но файл на диске оставить
git status               # "deleted: config.local" — из репозитория удалится, с диска нет
ls config.local          # на месте
git commit -am "перестал отслеживать config.local"
```

✅ **Проверь себя:**
- `git ls-files config.local` ничего не выводит: git этот файл больше не отслеживает.
- `ls config.local` показывает, что файл на месте.
- `git status` чистый.

**Уборка:**

```bash
git switch main
rm -rf .env debug.log build config.local
git branch -D trening-2
```
