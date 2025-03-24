# Backup Server
The backup server does exactly what it says it's supposed to do. It exposes no services other than a directory at port 80 for downloading the backups via HTTP, hosted via Apache, and `ssh`.

At 2AM on Sunday, the files at `/home/backup/` on Production are copied via `rsync` to `/backup` on Backup, and then compressed to a `tar.xz` file at 3AM. The original directory is then deleted at 4AM on both Production and on Backup.
It takes about 5 minutes to `rsync` a backup and around 10 minutes to compress that backup.
