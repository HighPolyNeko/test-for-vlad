# Урок 15. Свой репозиторий с нуля — и как его защитить

До этого ты работал в чужом, уже настроенном репо. Здесь разберём, как сделать свой так,
чтобы его было не стыдно показать и чтобы его не сломал ни ты, ни коллеги.

## 1. Доступ к GitHub: SSH или HTTPS

| | SSH | HTTPS |
|---|---|---|
| URL | `git@github.com:user/repo.git` | `https://github.com/user/repo.git` |
| Как авторизуется | ключ на твоём компьютере | логин через браузер / токен (пароль GitHub **не** подходит) |
| Настройка | один раз сгенерировать ключ | Git Credential Manager (идёт с Git for Windows) или `gh auth login` |

SSH-ключ настраивается один раз:

```bash
ssh-keygen -t ed25519 -C "твоя@почта"        # Enter, Enter (можно задать пароль на ключ)
cat ~/.ssh/id_ed25519.pub                     # это ПУБЛИЧНЫЙ ключ — его можно показывать
# GitHub → Settings → SSH and GPG keys → New SSH key → вставить
ssh -T git@github.com                         # "Hi <имя>! You've successfully authenticated"
```

> ⚠️ Файл **без** `.pub` (`id_ed25519`) — приватный ключ. Его никому не показывают и никуда не копируют.

## 2. Создать репозиторий: два пути

### Путь А: сначала локально (есть код, нужен репо)

```bash
mkdir moy-proekt && cd moy-proekt
git init -b main                         # -b main: сразу назвать ветку main, а не master
# ...файлы, первый коммит...
```

На GitHub: **New repository** → имя → **НЕ ставь галочки** README / .gitignore / license.

```bash
git remote add origin git@github.com:<ты>/moy-proekt.git
git push -u origin main
```

Или одной командой через GitHub CLI: `gh repo create moy-proekt --public --source=. --push`.

### Путь Б: сначала на GitHub

Создал репо с README, потом `git clone`. Это самый простой вариант, в нём ничего не сломать.

### 🪤 Ловушка: создал и там, и там

Создал на GitHub **с README**, а локально сделал `git init` и свой коммит. `git push` откажет,
`git pull` скажет `refusing to merge unrelated histories`: у этих двух историй нет общего предка.

```bash
git pull origin main --allow-unrelated-histories   # склеить две истории одним мержем
```

Лучше просто не создавать файлы на GitHub, если локально уже есть коммиты.

## 3. Стартовые файлы

| Файл | Зачем | Где взять |
|---|---|---|
| `README.md` | что это, как запустить, как пользоваться. Первое, что видят люди | пишешь сам |
| `.gitignore` | чтобы не закоммитить `node_modules`, `.env`, сборку, настройки IDE | шаблоны: [github/gitignore](https://github.com/github/gitignore), или выбрать при создании репо |
| `LICENSE` | **без лицензии код по умолчанию «все права защищены»**: смотреть можно, использовать нельзя | [choosealicense.com](https://choosealicense.com): MIT (делайте что хотите), Apache-2.0 (+ патенты), GPL (производные тоже должны быть открыты) |
| `.gitattributes` | одинаковые переводы строк у всех (урок 13) | `* text=auto eol=lf` |
| `.github/pull_request_template.md` | каждый PR сразу открывается с шаблоном описания | см. [в этом репо](../.github/pull_request_template.md) |
| `.github/CODEOWNERS` | кого автоматически звать на ревью | см. [в этом репо](../.github/CODEOWNERS) |
| `.github/workflows/*.yml` | CI: автоматические проверки на каждый PR | см. [в этом репо](../.github/workflows/proverka.yml) |

## 4. Настройки репозитория (Settings)

**General:**
- **Description** и **Topics**: по ним репо находят в поиске, и они видны в профиле.
- **Default branch**: `main`.
- **Pull Requests**: какие кнопки мержа разрешить (merge commit / squash / rebase).
  Многие команды оставляют только **squash**, чтобы в `main` было по одному коммиту на PR.
- **Automatically delete head branches**: после мержа PR ветка на GitHub удаляется сама, и мусор не копится.

**Collaborators** (личный репо): приглашённый человек получает право пушить и мержить, но не менять настройки.
В репо **организаций** роли тоньше: Read → Triage → Write → Maintain → Admin.
Людям со стороны доступ не дают: они делают **Fork** и присылают PR из него.

**Code security:** **Secret scanning** + **Push protection**. GitHub не даст запушить коммит,
в котором распознан токен (AWS, GitHub, Stripe и т.д.). Это страховка к уроку 14.

## 5. Защита веток: Rulesets

Главная настройка. Без неё любой, у кого есть доступ (включая тебя в 2 часа ночи), может:
- запушить в `main` что угодно без ревью;
- сделать `push --force` и стереть чужие коммиты;
- удалить ветку или тег релиза.

**Settings → Rules → Rulesets → New branch ruleset**. Ruleset — это набор правил + к каким веткам/тегам
он применяется (по маске, например `release/*`) + кто может его обходить (**bypass list**).

| Правило | Что делает |
|---|---|
| **Restrict deletions** | ветку нельзя удалить |
| **Block force pushes** | нельзя переписать историю (`push --force`, в том числе `--force-with-lease`) |
| **Require a pull request before merging** | в ветку нельзя пушить напрямую, только через PR |
| ↳ Required approvals | сколько апрувов нужно. **Свой PR апрувнуть нельзя**, поэтому в соло-проекте ставят 0 или оставляют себе bypass |
| ↳ Dismiss stale approvals | новый пуш в PR сбрасывает старые апрувы: одобряли же другой код |
| ↳ Require review from Code Owners | апрув нужен именно от владельца кода из `CODEOWNERS` |
| ↳ Require conversation resolution | нельзя мержить, пока есть неразрешённые комментарии |
| **Require status checks to pass** | мерж только при зелёном CI |
| **Require linear history** | запрещает merge-коммиты (только squash / rebase) |
| **Require signed commits** | только подписанные коммиты |

Rulesets работают и для **тегов**. Защищают `v*`, чтобы опубликованный релиз нельзя было
удалить или тихо перевесить на другой коммит.

> Раньше было **Branch protection rules** (классическая защита). Rulesets — её замена: можно задавать маски,
> правила для тегов, несколько наборов сразу, и видно, кто и когда обходил правила.

## 6. 👀 Как настроено в этом репо

| Ruleset | Куда | Правила | Кто может обойти |
|---|---|---|---|
| **main** | `main` | только через PR, 1 апрув, сброс старых апрувов, все обсуждения закрыты, CI `proverka` зелёный, нельзя force-push и удалять | админ — только через PR (прямой пуш запрещён даже ему) |
| **Служебные ветки** | `release/*`, `practice/*`, `demo/*`, `lesson/*` | нельзя force-push и удалять | никто |
| **Релизные теги** | `v*` | нельзя удалять и перезаписывать | никто |

Плюс: разрешены все 3 вида мержа (для обучения), ветки удаляются после мержа сами,
включены secret scanning и push protection. CI (`.github/workflows/proverka.yml`) проверяет,
что в репо нет `.env`, файлов больше 1 МБ и забытых маркеров конфликта.

---

## 🛠 Сделай сам

### Часть 1 🔑. Попробуй пробить защиту этого репо

**Зачем:** один раз увидеть, как выглядит отказ, и понять, что на этот раз ошибся не ты, а сработала защита.

```bash
git switch -c trening-15 main
git commit --allow-empty -m "test: прямой пуш"

git push origin trening-15:main
# ❌ GH013: Repository rule violations found for refs/heads/main.
#    - Changes must be made through a pull request.

git push --force origin trening-15:practice/rebase-main
# ❌ ...Cannot force-push to this branch

git switch main && git branch -D trening-15
```

### Часть 2. Свой репозиторий по всем правилам

**Зачем:** портфолио на GitHub смотрят на собесе. Репо с README, лицензией, CI и защищённым `main`
сразу показывает, что человек работал в команде.

1. **Локально:**
   ```bash
   mkdir moy-pervyi-repo && cd moy-pervyi-repo
   git init -b main
   echo "# Мой первый репозиторий" > README.md
   printf '.env\n.idea/\n*.log\n' > .gitignore
   echo "* text=auto eol=lf" > .gitattributes
   git add . && git commit -m "chore: начальная структура"
   ```
2. **GitHub:** New repository → `moy-pervyi-repo` → Public → **без** README/.gitignore/license → Create.
3. **Связать и запушить:**
   ```bash
   git remote add origin git@github.com:<ты>/moy-pervyi-repo.git
   git push -u origin main
   ```
4. **Лицензия:** на GitHub Add file → Create new file → имя `LICENSE` → появится кнопка
   **Choose a license template** → MIT → закоммить прямо в main. Потом локально `git pull`.
5. **Settings → General:** Description, Topics. В разделе Pull Requests оставь только
   **Allow squash merging** и включи **Automatically delete head branches**.
6. **CI:** скопируй к себе `.github/workflows/proverka.yml` из этого репо, закоммить, запушь.
   Во вкладке **Actions** должен пройти зелёный прогон.
7. **Settings → Rules → Rulesets → New branch ruleset:**
   - Name: `main`, Enforcement: **Active**;
   - Target branches → Add target → **Include default branch**;
   - ✅ Restrict deletions, ✅ Block force pushes;
   - ✅ Require a pull request before merging, Required approvals: **0** (ты один — свой PR апрувнуть нельзя);
   - ✅ Require status checks to pass → Add checks → `proverka`.
8. **Проверь защиту:**
   ```bash
   echo "прямо в main" >> README.md && git commit -am "test: прямой пуш"
   git push                                  # ❌ отказ — так и должно быть
   git reset --hard origin/main              # убрать этот коммит
   ```
9. **Сломай CI нарочно:**
   ```bash
   git switch -c feat/sekret
   echo "PASSWORD=123" > .env
   git add -f .env && git commit -m "feat: настройки"   # -f — .env в .gitignore, добавляем силой
   git push -u origin feat/sekret
   ```
   Открой PR: проверка `proverka` станет **красной**, и кнопка Merge будет заблокирована.
   Почини: `git rm --cached .env && git commit -m "fix: убрал .env" && git push`. CI позеленеет, смержи.
   В истории ветки `.env` остался, но squash-мерж положит в `main` один коммит без него (урок 11).

✅ **Проверь себя:** в твоём репо есть README, LICENSE, `.gitignore`, `.gitattributes`, зелёный CI во вкладке Actions,
а прямой пуш в `main` отклоняется. Скинь ссылку тому, кто дал тебе этот курс 🙂
