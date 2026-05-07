# Remnawave Backup & Restore

Bash script for Remnawave backup, restore, Telegram delivery, S3/Google Drive upload, and scheduled interval backups.

## Configuration

Copy `config.env.example` to `config.env` on the server and fill in your real credentials there.

`config.env` is intentionally ignored by git because it contains secrets such as Telegram bot tokens, chat IDs, database passwords, and cloud storage keys.

## Safety

The script stores local backup archives in its own backup directory and cleanup routines are scoped to that directory.
