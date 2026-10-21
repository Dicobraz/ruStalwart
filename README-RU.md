# ruStalwart

Форк [Stalwart](https://github.com/stalwartlabs/stalwart) (почтовый и groupware-сервер Stalwart Labs) с нашими
исправлениями ошибок, на которые натыкается почтовый клиент Postman.

## Зачем форк

Upstream не принимает код, написанный ИИ (CONTRIBUTING.md, раздел «No AI-Generated Code»), а PR от неодобренных
авторов закрываются автоматически. Поэтому исправления для нашей установки живут здесь, отдельными коммитами
поверх релиза. Сообщения об ошибках и пожелания уходят в upstream отдельно, через портал
[support.stalw.art](https://support.stalw.art); их пишет и публикует владелец. В upstream-репозиторий из форка
ничего не отправляется: ни issue, ни PR.

Список патчей и их статус — в [PATCHES.md](PATCHES.md).

## Ветки

| Ветка | Что в ней |
|---|---|
| `main` | зеркало `stalwartlabs/stalwart` `main`, своих коммитов нет |
| `ru/main` | тег релиза upstream (сейчас `v0.16.27`) + наши коммиты: этот файл, `PATCHES.md`, по одному коммиту на патч. Ветка по умолчанию |
| `ru/<NN>-кратко` | разработка одного патча; PR в этом репозитории с базой `ru/main` |

Правила:

- один патч = одна ветка `ru/<NN>-кратко` = один PR в `Dicobraz/ruStalwart` = один коммит в `ru/main`;
- номер `NN` — номер черновика сообщения об ошибке (и строки в `PATCHES.md`);
- патч маленький и локальный, с тестом в раскладке upstream; сообщение коммита на английском и объясняет,
  какой пункт RFC нарушался;
- заголовки лицензий в файлах не меняются;
- PR сливает владелец, после проверки.

GitHub Actions в форке выключены (`gh api -X PUT repos/Dicobraz/ruStalwart/actions/permissions -F enabled=false`):
workflow upstream здесь бесполезны или вредны (`auto-close-prs.yml` закрыл бы наши PR, `trivy`/`scorecard` идут по
расписанию). Проверка — локальные тесты, см. описание каждого PR.

## Обновление на новый релиз upstream

```bash
git fetch upstream --tags
git checkout main && git merge --ff-only upstream/main && git push origin main

git checkout ru/main
git rebase --onto vX.Y.Z v0.16.27 ru/main   # старый тег -> новый тег
```

1. Перед rebase прочитать `CHANGELOG.md` нового релиза и сверить с `PATCHES.md`: если upstream уже исправил
   ошибку, коммит патча выбросить (`git rebase -i` или `git rebase --onto` с пропуском, либо `git revert` после),
   а в `PATCHES.md` поставить статус «исправлено в vX.Y.Z».
2. Конфликты решать в пользу кода upstream, патч переносить заново минимально.
3. Прогнать тесты затронутых крейтов (см. ниже) и поставить тег `vX.Y.Z-ru.N`.
4. `git push --force-with-lease origin ru/main` — ветка переписывается, это ожидаемо; открытые PR патчей
   перебазировать на новую `ru/main`.
5. В этом файле и в `PATCHES.md` обновить номер базового релиза.

## Сборка

### Release-бинарь (Linux, как на стенде)

```bash
git checkout ru/main
cargo build --release -p stalwart --no-default-features \
  --features "sqlite postgres mysql rocks s3 redis azure nats enterprise"
# результат: target/release/stalwart
```

Набор features тот же, что в официальном образе (`Dockerfile`). Для RocksDB нужен `libclang`
(`apt install build-essential libclang-dev`). Если на стенде только один движок хранения, лишние можно убрать,
например `--features "rocks enterprise"` (это `default` крейта `crates/main`).

### Docker-образ

Официальный `Dockerfile` собирает образ из текущего дерева, отдельных `--build-arg` у него нет: платформа
задаётся через `--platform` (из неё BuildKit выставляет `TARGETPLATFORM`, поддерживаются `linux/amd64` и
`linux/arm64`), features зашиты в `Dockerfile`.

```bash
git checkout ru/main
docker buildx build --platform linux/amd64 -t rustalwart:v0.16.27-ru.1 --load .
```

`Dockerfile.build` + `docker-bake.hcl` — путь upstream CI (zigbuild, `--build-arg TARGET=x86_64-unknown-linux-gnu`,
`BUILD_ENV`, `GLIBC_VERSION`), для одного стенда не нужен.

Образ подменяет `stalwartlabs/stalwart:v0.16.27` один к одному: те же тома (`/etc/stalwart`, `/var/lib/stalwart`),
порты и entrypoint. Развёртывание на стенд — инфраструктура владельца.

### Windows (только для разработки и тестов патчей)

`crates/*` без движков хранения собираются MSVC-тулчейном: `cargo test -p types -p email -p jmap -p dav`.
Интеграционные тесты (`tests`) — с `STORE=Sqlite` и `--no-default-features --features sqlite` (RocksDB требует
`libclang`). Локальные правки, которые для этого нужны и в коммиты не попадают: убрать dev-зависимость `ece`
(требует OpenSSL) вместе с тестом push-подписок, а в v0.16.27 — ещё исправить вызов `itip_merge_changes` в
`tests/src/webdav/cal_itip.rs` (тест upstream не компилируется). Рабочее дерево — с `core.autocrlf=false`:
тесты сравнивают `.eml`/`.json` побайтно.

## Лицензия

Stalwart распространяется под двойной лицензией **AGPL-3.0-only** или **Stalwart Enterprise License v2**
(см. [LICENSES](LICENSES/) и заголовки файлов). Форк лицензию не меняет: наши патчи — изменения файлов с
заголовком `AGPL-3.0-only OR LicenseRef-SEL` и распространяются на тех же условиях.

**Предложение исходного кода (AGPL-3.0, §13).** Если вы пользуетесь сервером, собранным из этого форка, по сети,
полный исходный код именно той версии, которая работает, доступен здесь:
<https://github.com/Dicobraz/ruStalwart>, ветка `ru/main`; точная версия — коммит, из которого собран бинарь,
или тег `vX.Y.Z-ru.N`, которым помечается сборка для стенда. Репозиторий публичный и должен оставаться публичным,
пока сервер из форка доступен пользователям по сети.
