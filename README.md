# Remnawave Backup & Restore

Расширенный Bash-скрипт для резервного копирования и восстановления Remnawave с отправкой архивов в Telegram, Google Drive или S3-совместимое хранилище.

Оригинальная основа: `distillium/remnawave-backup-restore`  
Автор доработок и этой публичной версии: `xelsing1991-maker`  
Репозиторий: <https://github.com/xelsing1991-maker/backup-script>

## VPN для РФ

Реферальная ссылка автора доработок:  
<https://t.me/VPN_Raketa_bot?start=ZankinMaster>

## Что было улучшено

- Добавлен бэкап Telegram-ботов вместе с панелью или отдельно от панели.
- Добавлена отправка больших Telegram-архивов частями по `38 MB`.
- После отправки частей скрипт присылает отдельное сообщение со ссылками/ID частей и инструкцией сборки в Windows.
- В Telegram-сообщения добавлен IP сервера, с которого сделан бэкап.
- Добавлен отдельный пункт меню для интервального бэкапа: раз в `1`, `3`, `6` или `12` часов.
- Для интервального режима старые локальные архивы удаляются перед созданием нового архива, когда они прожили выбранный интервал.
- Улучшена локальная политика хранения: очистка работает по точному времени, а не по округленному `find -mtime`.
- Очистка ограничена папкой бэкапов скрипта и не удаляет файлы в чужих директориях.
- Реальный `config.env` исключен из git, добавлен безопасный `config.env.example`.
- Добавлены `.gitignore` и `.gitattributes`, чтобы секреты не попадали в репозиторий, а `.sh` файлы сохраняли Linux LF-переносы.

## Возможности

- Создание полного бэкапа Remnawave.
- Восстановление Remnawave из локального архива.
- Поддержка Docker PostgreSQL и внешнего PostgreSQL.
- Бэкап директории панели.
- Бэкап базы и директории Telegram-бота.
- Режимы: только панель, панель + бот, только бот.
- Отправка в Telegram.
- Автоматическое разбиение большого Telegram-архива на части.
- Отправка в Google Drive.
- Отправка в S3 Storage.
- Загрузка и восстановление из S3.
- Настройка cron-автоотправки.
- Отдельная настройка интервального бэкапа.
- Русский и английский интерфейс.

## Требования

На сервере должны быть:

- Linux-сервер с root-доступом.
- `bash`
- `curl`
- `tar`
- `gzip`
- `jq`
- `docker` и `docker compose`, если Remnawave работает в Docker.
- `crontab`, если нужен автоматический запуск.
- `aws`, если используется S3 Storage.

Скрипт сам пытается установить `jq` и AWS CLI в некоторых сценариях, но надежнее заранее проверить зависимости.

## Быстрая установка

Скачайте скрипт:

```bash
sudo mkdir -p /opt/rw-backup-restore
sudo curl -fsSL https://raw.githubusercontent.com/xelsing1991-maker/backup-script/main/backup-restore.sh -o /opt/rw-backup-restore/backup-restore.sh
sudo chmod +x /opt/rw-backup-restore/backup-restore.sh
```

Создайте конфиг из примера:

```bash
sudo curl -fsSL https://raw.githubusercontent.com/xelsing1991-maker/backup-script/main/config.env.example -o /opt/rw-backup-restore/config.env
sudo chmod 600 /opt/rw-backup-restore/config.env
```

Запустите:

```bash
sudo /opt/rw-backup-restore/backup-restore.sh
```

После первого запуска скрипт может создать быстрый запуск:

```bash
rw-backup
```

## Установка через git

```bash
git clone https://github.com/xelsing1991-maker/backup-script.git
cd backup-script
sudo mkdir -p /opt/rw-backup-restore
sudo cp backup-restore.sh /opt/rw-backup-restore/backup-restore.sh
sudo cp config.env.example /opt/rw-backup-restore/config.env
sudo cp -r translations /opt/rw-backup-restore/translations
sudo chmod +x /opt/rw-backup-restore/backup-restore.sh
sudo chmod 600 /opt/rw-backup-restore/config.env
sudo /opt/rw-backup-restore/backup-restore.sh
```

## Настройка Telegram

1. Создайте бота через `@BotFather`.
2. Получите `BOT_TOKEN`.
3. Получите свой `CHAT_ID` или ID группы.
4. В меню скрипта откройте настройку способа отправки или Telegram-настройки.
5. Укажите токен, chat ID и при необходимости `message_thread_id` для темы группы.

В конфиге это выглядит так:

```env
BOT_TOKEN="123456789:token"
CHAT_ID="123456789"
UPLOAD_METHOD="telegram"
TG_MESSAGE_THREAD_ID=""
TG_PROXY=""
```

`config.env` не должен попадать в git. Он уже добавлен в `.gitignore`.

## Большие архивы Telegram

Telegram Bot API может не принять большой архив одним файлом. Поэтому скрипт делает так:

1. Создает финальный архив `remnawave_backup_YYYY-MM-DD_HH_MM_SS.tar.gz`.
2. Если файл больше `38 MB`, режет его на части:

```text
remnawave_backup_....tar.gz.part-000
remnawave_backup_....tar.gz.part-001
remnawave_backup_....tar.gz.part-002
```

3. Отправляет каждую часть отдельным документом.
4. После успешной отправки удаляет временную папку `.parts`.
5. Присылает отдельное сообщение с ссылками/ID частей и инструкцией для Windows.

### Как собрать части в Windows

Скачайте все части в одну папку, откройте PowerShell в этой папке и выполните:

```powershell
cmd /c copy /b remnawave_backup_....tar.gz.part-* remnawave_backup_....tar.gz
```

Затем распакуйте архив через `7-Zip`, `WinRAR` или:

```powershell
tar -xzf remnawave_backup_....tar.gz
```

## Интервальный бэкап

В главном меню есть отдельный пункт:

```text
Интервальный бэкап и хранение
```

Можно выбрать:

- раз в `1 час`
- раз в `3 часа`
- раз в `6 часов`
- раз в `12 часов`

Логика работы:

1. Cron запускает скрипт по выбранному интервалу.
2. Перед созданием нового бэкапа скрипт чистит старые локальные архивы, которые прожили выбранный интервал.
3. Затем создается новый архив.
4. Новый архив отправляется выбранным способом.

Например, если выбран интервал `3 часа`, то перед новым запуском удаляются локальные архивы старше 3 часов, после чего создается и отправляется новый архив.

Удаление ограничено только папкой бэкапов скрипта:

```bash
/opt/rw-backup-restore/backup
```

Скрипт удаляет только:

```text
remnawave_backup_*.tar.gz
remnawave_backup_*.tar.gz.parts
```

Он не удаляет файлы панели, директорию Remnawave или другие папки сервера.

## Обычная политика хранения

Кроме интервального режима есть отдельная политика хранения в днях:

```env
RETAIN_BACKUPS_DAYS="7"
S3_RETAIN_DAYS="30"
```

Локальная политика хранения чистит старые архивы в папке бэкапов скрипта. S3-политика чистит старые архивы в указанном S3 bucket/prefix.

## Настройка Telegram-бота для бэкапа

В меню есть пункт настройки бэкапа Telegram-бота. Скрипт умеет бэкапить:

- Бот от Иисуса: `remnawave-telegram-shop`
- Приватный бот от Иисуса: `rwp-shop`
- Бот от Мачки: `remnawave-tg-shop`
- Бот от Snoups: `remnashop`
- кастомный путь

Можно выбрать режим:

- панель + бот
- только бот
- только панель

## Google Drive

Для Google Drive нужны:

```env
GD_CLIENT_ID=""
GD_CLIENT_SECRET=""
GD_REFRESH_TOKEN=""
GD_FOLDER_ID=""
```

Настройку удобнее делать через меню скрипта, потому что нужно пройти OAuth-авторизацию и получить refresh token.

## S3 Storage

Для S3 нужны:

```env
S3_ENDPOINT=""
S3_ACCESS_KEY=""
S3_SECRET_KEY=""
S3_BUCKET=""
S3_REGION=""
S3_PREFIX=""
S3_RETAIN_DAYS="30"
```

Поддерживаются S3-совместимые хранилища. Для работы нужен AWS CLI.

## Команды

Интерактивное меню:

```bash
sudo /opt/rw-backup-restore/backup-restore.sh
```

Создать бэкап:

```bash
sudo /opt/rw-backup-restore/backup-restore.sh backup
```

Восстановить:

```bash
sudo /opt/rw-backup-restore/backup-restore.sh restore
```

Обновить:

```bash
sudo /opt/rw-backup-restore/backup-restore.sh update
```

Если создан symlink:

```bash
sudo rw-backup
sudo rw-backup backup
sudo rw-backup restore
```

## Структура репозитория

```text
backup-restore.sh       основной скрипт
config.env.example      пример конфига без секретов
translations/ru.sh      русские переводы
translations/en.sh      английские переводы
.gitignore              исключает config.env и временные файлы
.gitattributes          фиксирует LF для shell-файлов
README.md               документация
```

## Безопасность

- Не публикуйте `config.env`.
- Не коммитьте токены Telegram, пароли БД, S3 ключи и OAuth refresh token.
- После случайной публикации токена сразу перевыпустите его.
- Реальный `config.env` уже исключен из git.

## Разработка и Pull Requests

Для работы с репозиторием:

```bash
git clone https://github.com/xelsing1991-maker/backup-script.git
cd backup-script
git checkout -b feature/my-change
```

Проверка синтаксиса:

```bash
bash -n backup-restore.sh
```

Коммит и push:

```bash
git add .
git commit -m "Describe change"
git push -u origin feature/my-change
```

Создать Pull Request через GitHub CLI:

```bash
gh pr create --fill
```

## Что проверено

- `config.env` игнорируется git.
- В репозиторий добавлен только безопасный `config.env.example`.
- `backup-restore.sh` проходит `bash -n`.
- Репозиторий опубликован как публичный.
