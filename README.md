# Скрипт резервного копирования Remnawave \ Remnawave Backup Script

> **Remnawave Backup Script** — полноценный скрипт бэкапа и восстановления Remnawave с отправкой в Telegram, Google Drive, S3 и поддержкой больших архивов частями.

📌 **Название проекта:** `Remnawave Backup Script`  
📌 **Русское описание:** `Скрипт бэкапа Remnawave`  
📌 **English description:** `Remnawave backup script`  
🌐 **Репозиторий:** <https://github.com/xelsing1991-maker/backup-script>  
👤 **Автор доработок:** `xelsing1991-maker`  
🧩 **Основа:** `distillium/remnawave-backup-restore`  
🇷🇺 **VPN для РФ:** <https://t.me/VPN_Raketa_bot?start=ZankinMaster>

---

## ✨ Что умеет скрипт

✅ Создает бэкап Remnawave панели  
✅ Восстанавливает Remnawave из архива  
✅ Бэкапит PostgreSQL из Docker или внешней БД  
✅ Бэкапит директорию панели  
✅ Бэкапит Telegram-ботов  
✅ Работает в режимах `Панель`, `Панель + Бот`, `Только Бот`  
✅ Отправляет бэкапы в Telegram  
✅ Делит большие архивы на части по `38 MB` для Telegram  
✅ После отправки частей присылает ссылки/ID и инструкцию сборки в Windows  
✅ Добавляет IP сервера в Telegram-сообщение  
✅ Загружает архивы в Google Drive  
✅ Загружает архивы в S3 Storage  
✅ Умеет восстанавливать из S3  
✅ Делает авто-бэкапы через cron  
✅ Имеет отдельный интервальный режим `1/3/6/12 часов`  
✅ Чистит старые архивы только в своей папке бэкапов  
✅ Поддерживает русский и английский интерфейс  

---

## 📦 Быстрая установка с GitHub

Подключитесь к серверу по SSH и выполните:

```bash
sudo mkdir -p /opt/rw-backup-restore
sudo curl -fsSL https://raw.githubusercontent.com/xelsing1991-maker/backup-script/main/backup-restore.sh -o /opt/rw-backup-restore/backup-restore.sh
sudo curl -fsSL https://raw.githubusercontent.com/xelsing1991-maker/backup-script/main/config.env.example -o /opt/rw-backup-restore/config.env
sudo mkdir -p /opt/rw-backup-restore/translations
sudo curl -fsSL https://raw.githubusercontent.com/xelsing1991-maker/backup-script/main/translations/ru.sh -o /opt/rw-backup-restore/translations/ru.sh
sudo curl -fsSL https://raw.githubusercontent.com/xelsing1991-maker/backup-script/main/translations/en.sh -o /opt/rw-backup-restore/translations/en.sh
sudo chmod +x /opt/rw-backup-restore/backup-restore.sh
sudo chmod 600 /opt/rw-backup-restore/config.env
sudo /opt/rw-backup-restore/backup-restore.sh
```

После запуска скрипт может создать короткую команду:

```bash
sudo rw-backup
```

---

## 🧬 Установка через `git clone`

Если хотите установить именно из git-репозитория:

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

---

## 🔧 Требования

На сервере нужны:

```text
bash
curl
tar
gzip
jq
docker
docker compose
crontab
```

Для S3 дополнительно нужен:

```text
aws cli
```

Скрипт умеет помогать с установкой некоторых зависимостей, но лучше заранее запускать его на сервере с root-доступом.

---

## 🧭 Главное меню

После запуска вы увидите меню:

```text
1. Создать бэкап вручную
2. Восстановить из бэкапа

3. Настройка бэкапа Telegram бота
4. Настройка автоматической отправки и уведомлений
5. Интервальный бэкап и хранение
6. Настройка способа отправки
7. Настройка конфигурации

8. Обновление скрипта
9. Удаление скрипта
```

---

## 🤖 Настройка Telegram

1. Откройте `@BotFather`.
2. Создайте бота.
3. Получите `BOT_TOKEN`.
4. Получите `CHAT_ID` пользователя или группы.
5. В меню выберите настройку способа отправки или Telegram-настройки.
6. Укажите токен и ID чата.

Пример конфига:

```env
BOT_TOKEN="123456789:your_token"
CHAT_ID="123456789"
UPLOAD_METHOD="telegram"
TG_MESSAGE_THREAD_ID=""
TG_PROXY=""
```

Если отправляете в тему группы, укажите:

```env
TG_MESSAGE_THREAD_ID="123"
```

---

## 🧱 Большие архивы в Telegram

Telegram может не принять большой `.tar.gz` архив одним файлом. Поэтому скрипт автоматически:

1. Проверяет размер архива.
2. Если архив больше `38 MB`, режет его на части.
3. Отправляет каждую часть отдельным документом.
4. После отправки присылает отдельное сообщение с:
   - количеством частей;
   - ссылками или `message_id`;
   - инструкцией сборки для Windows.

Пример частей:

```text
remnawave_backup_2026-05-07_12_00_00.tar.gz.part-000
remnawave_backup_2026-05-07_12_00_00.tar.gz.part-001
remnawave_backup_2026-05-07_12_00_00.tar.gz.part-002
```

### 🪟 Как собрать архив в Windows

Скачайте все части в одну папку.

Откройте PowerShell в этой папке и выполните:

```powershell
cmd /c copy /b remnawave_backup_2026-05-07_12_00_00.tar.gz.part-* remnawave_backup_2026-05-07_12_00_00.tar.gz
```

Потом распакуйте архив через `7-Zip`, `WinRAR` или:

```powershell
tar -xzf remnawave_backup_2026-05-07_12_00_00.tar.gz
```

---

## ⏱️ Интервальный бэкап 1/3/6/12 часов

В меню есть отдельный пункт:

```text
Интервальный бэкап и хранение
```

Можно выбрать:

```text
1 час
3 часа
6 часов
12 часов
```

Логика:

1. Cron запускает скрипт по выбранному интервалу.
2. Перед новым бэкапом скрипт удаляет старые локальные архивы, которые прожили этот интервал.
3. Затем создает новый архив.
4. Затем отправляет новый архив выбранным способом.

Например:

```text
Выбрано: раз в 3 часа
Архив хранится локально 3 часа
Перед следующим запуском старый архив удаляется
Создается и отправляется новый архив
```

Скрипт удаляет только внутри своей папки:

```bash
/opt/rw-backup-restore/backup
```

И только файлы:

```text
remnawave_backup_*.tar.gz
remnawave_backup_*.tar.gz.parts
```

Он не трогает другие папки сервера.

---

## 🧹 Политика хранения

В обычном режиме можно задать хранение в днях:

```env
RETAIN_BACKUPS_DAYS="7"
S3_RETAIN_DAYS="30"
```

Локальная очистка работает только в папке бэкапов скрипта.

---

## 🤖 Бэкап Telegram-бота

Скрипт умеет сохранять Telegram-ботов:

```text
remnawave-telegram-shop
rwp-shop
remnawave-tg-shop
remnashop
кастомный путь
```

Поддерживаются режимы:

```text
Панель + Бот
Только Бот
Только Панель
```

---

## ☁️ Google Drive

Для Google Drive нужны:

```env
GD_CLIENT_ID=""
GD_CLIENT_SECRET=""
GD_REFRESH_TOKEN=""
GD_FOLDER_ID=""
```

Настраивайте через меню, потому что нужно пройти OAuth-авторизацию и получить refresh token.

---

## 🪣 S3 Storage

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

Поддерживаются S3-совместимые хранилища.

---

## ▶️ Команды запуска

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

Если создана команда `rw-backup`:

```bash
sudo rw-backup
sudo rw-backup backup
sudo rw-backup restore
sudo rw-backup update
```

---

## 🗂️ Структура репозитория

```text
backup-restore.sh       основной скрипт
config.env.example      пример конфига без секретов
translations/ru.sh      русский интерфейс
translations/en.sh      английский интерфейс
.gitignore              исключает config.env и временные файлы
.gitattributes          сохраняет LF для shell-файлов
README.md               документация
```

---

## 🔐 Важно про безопасность

Никогда не публикуйте:

```text
config.env
BOT_TOKEN
CHAT_ID
DB_PASSWORD
GD_REFRESH_TOKEN
S3_ACCESS_KEY
S3_SECRET_KEY
```

В репозитории есть только безопасный:

```text
config.env.example
```

Реальный файл:

```text
config.env
```

добавлен в `.gitignore`.

---

## 🛠️ Разработка и Pull Requests

Клонировать репозиторий:

```bash
git clone https://github.com/xelsing1991-maker/backup-script.git
cd backup-script
```

Создать ветку:

```bash
git checkout -b feature/my-change
```

Проверить синтаксис:

```bash
bash -n backup-restore.sh
```

Сделать коммит:

```bash
git add .
git commit -m "Describe change"
```

Запушить ветку:

```bash
git push -u origin feature/my-change
```

Создать Pull Request через GitHub CLI:

```bash
gh pr create --fill
```

---

## ✅ Проверено

```text
config.env не попадает в git
backup-restore.sh проходит bash -n
репозиторий публичный
ветка main
установка доступна с GitHub
```
