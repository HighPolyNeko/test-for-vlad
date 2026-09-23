# Вопросы по git на собеседовании

Сначала попробуй ответить сам, потом открой ответ. В скобках — урок, где тема разобрана подробно.

## Базовый уровень

<details><summary>1. Чем отличаются <code>git fetch</code> и <code>git pull</code>?</summary>

`fetch` скачивает новые коммиты и обновляет `origin/*`, но твои ветки и файлы не трогает.
`pull` = `fetch` + `merge` (или `rebase`, если `pull.rebase=true`) в текущую ветку.
</details>

<details><summary>2. Что такое staging area и зачем она нужна?</summary>

Промежуточная зона между рабочей папкой и коммитом. Позволяет собрать коммит только из части
изменений, даже из части файла (`git add -p`). (урок 1)
</details>

<details><summary>3. Что такое ветка технически?</summary>

Файл с хешем коммита (`.git/refs/heads/<имя>`), то есть подвижный указатель. (урок 12)
</details>

<details><summary>4. Что такое merge-конфликт и как его решать?</summary>

Обе ветки изменили одно и то же место по-разному относительно общего предка. Правишь файл,
убираешь маркеры `<<<<<<< ======= >>>>>>>`, делаешь `git add` и `git commit` (или `--continue`).
Отмена — `git merge --abort`. (урок 5)
</details>

<details><summary>5. Как отменить последний коммит?</summary>

Если не запушен — `git reset HEAD~1` (правки остаются) или `--soft` / `--hard`.
Если запушен в общую ветку — `git revert HEAD`. (урок 8)
</details>

<details><summary>6. Зачем нужен <code>.gitignore</code>, и почему он не работает для уже закоммиченного файла?</summary>

Он влияет только на неотслеживаемые файлы. Если файл уже в индексе, нужно `git rm --cached <файл>`. (урок 2)
</details>

## Средний уровень

<details><summary>7. Merge или rebase? Когда что?</summary>

Merge сохраняет реальную историю и безопасен для общих веток. Rebase даёт линейную историю,
но переписывает хеши. Свою ветку перед PR — rebase, общие ветки — только merge.
Золотое правило: не ребейзить то, что уже есть у других. (урок 7)
</details>

<details><summary>8. <code>reset --soft</code> vs <code>--mixed</code> vs <code>--hard</code>?</summary>

Все три двигают ветку. `soft` оставляет изменения в staging, `mixed` — в рабочей папке, `hard` удаляет их. (урок 8)
</details>

<details><summary>9. Чем <code>revert</code> отличается от <code>reset</code>?</summary>

`reset` переписывает историю: двигает ветку назад. `revert` добавляет новый коммит с обратными изменениями.
Для запушенного — `revert`. (урок 8)
</details>

<details><summary>10. Как откатить merge-коммит, и что будет, если потом смержить ту же ветку снова?</summary>

`git revert -m 1 <merge>`. При повторном мерже старые коммиты ветки **не вернутся**: git считает
их уже влитыми. Нужно сначала сделать revert этого revert'а. (урок 8)
</details>

<details><summary>11. Что такое cherry-pick и какие у него риски?</summary>

Перенос изменений коммита новым коммитом (новый хеш). Риски: дубли в истории, зависимость от
коммитов, которых нет в целевой ветке. `A..B` не включает A. Используй `-x`. (урок 6)
</details>

<details><summary>12. Как вернуть коммиты после <code>reset --hard</code> или удалённую ветку?</summary>

`git reflog` → найти хеш → `git reset --hard <hash>` или `git branch <имя> <hash>`.
Незакоммиченное так не вернуть. (урок 9)
</details>

<details><summary>13. Почему <code>--force-with-lease</code> лучше <code>--force</code>?</summary>

Перезапишет ветку только если на сервере она не изменилась с твоего последнего fetch.
Так ты не затрёшь чужие коммиты. (урок 7)
</details>

<details><summary>14. Что такое fast-forward? Зачем <code>--no-ff</code>?</summary>

Если целевая ветка не ушла вперёд, мерж просто передвигает указатель без merge-коммита.
`--no-ff` принудительно создаёт merge-коммит, чтобы в истории было видно ветку. (урок 11)
</details>

<details><summary>15. Squash, merge commit, rebase and merge — в чём разница?</summary>

Merge commit — все коммиты + мерж-коммит. Squash — один новый коммит. Rebase — коммиты
переигрываются линейно без мерж-коммита. После squash `git branch -d` говорит «not fully merged». (урок 11)
</details>

<details><summary>16. Чем <code>HEAD~2</code> отличается от <code>HEAD^2</code>?</summary>

`~2` — два шага назад по первым родителям. `^2` — второй родитель merge-коммита. (урок 11)
</details>

<details><summary>17. Что покажут <code>git log main..feature</code> и <code>git diff main...feature</code>?</summary>

Первое — коммиты из feature, которых нет в main. Второе — изменения feature относительно общего
предка с main, то есть ровно то, что показывает PR. (урок 11)
</details>

<details><summary>18. Что такое detached HEAD и чем он опасен?</summary>

HEAD указывает прямо на коммит, а не на ветку. Новые коммиты ни к какой ветке не привязаны
и «потеряются» при переключении. Спасение: `git switch -c <ветка>`. (урок 11)
</details>

<details><summary>19. Как git хранит данные? Коммит — это diff?</summary>

Объекты blob/tree/commit/tag, адресация по хешу содержимого. Коммит — **снимок** (ссылка на tree),
diff вычисляется. Дельты есть только как оптимизация в packfile. (урок 12)
</details>

<details><summary>20. Как найти коммит, который внёс баг?</summary>

`git bisect` (бинарный поиск, можно автоматически: `git bisect run <тест>`),
`git log -S`, `git blame`. (урок 13)
</details>

<details><summary>21. Что делать, если закоммитил пароль?</summary>

Первым делом отозвать/сменить секрет. Потом, если нужно, чистить историю
(`git filter-repo`) и делать force-push, но это не гарантия: клоны и форки остаются. (урок 14)
</details>

<details><summary>22. <code>stash</code> не спрятал новый файл. Почему?</summary>

По умолчанию untracked-файлы не прячутся, нужно `git stash -u`. (урок 10)
</details>

<details><summary>23. Git Flow vs GitHub Flow vs trunk-based?</summary>

Git Flow: `develop`/`release`/`hotfix`, для продуктов с версиями. GitHub Flow: `main` + короткие ветки + PR.
Trunk-based: частые маленькие мержи в main + feature flags. (урок 14)
</details>

<details><summary>24. Чем аннотированный тег отличается от лёгкого? Пушатся ли теги сами?</summary>

Аннотированный — отдельный объект с автором и сообщением (для релизов). Лёгкий — просто указатель.
Сами не пушатся: `git push origin <tag>` / `--tags`. (урок 12)
</details>

## Практические задачки (бывают на live-coding)

1. В ветке 5 коммитов «wip». Сделай из них один с нормальным сообщением. *(`rebase -i` → squash, или `reset --soft` + commit)*
2. Закоммитил в `main` вместо фичевой ветки (ещё не пушил). Перенеси. *(`git branch feature` → `git reset --hard origin/main` → `git switch feature`)*
3. Нужен один фикс из чужой ветки. *(`cherry-pick -x`)*
4. Ребейз пошёл не так, всё сломалось. *(`git rebase --abort`, или если уже закончился — `git reset --hard ORIG_HEAD` / reflog)*
5. Надо срочно поправить прод, а в рабочей папке недоделка. *(`stash -u` или `worktree`)*
6. Коллега запушил в твою ветку, а ты сделал rebase. Как запушить и не затереть? *(`fetch` → rebase поверх `origin/<ветка>` → `push --force-with-lease`)*
