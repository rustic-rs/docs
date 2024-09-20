# Scheduling backups

rustic does not have a built-in way of scheduling backups, as it's a tool that
runs when executed rather than a daemon. There are plenty of different ways to
schedule backup runs on various different platforms, e.g. systemd and cron on
Linux/BSD and Task Scheduler in Windows, depending on one's needs and
requirements. When scheduling rustic to run recurringly, please make sure to
detect already running instances before starting the backup.
