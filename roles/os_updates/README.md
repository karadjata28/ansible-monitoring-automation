# os_updates

This role deploys a script that writes Prometheus metrics to:

`/var/lib/node_exporter/textfile/os_updates.prom`

Flow:

- `os_updates.timer` runs every 30 minutes
- `os_updates.service` starts `/usr/local/bin/export_os_updates.sh`
- the script collects APT update info and extra host metrics
- `node_exporter` reads the `.prom` file through the textfile collector

Current extra metrics:

- CPU from `lscpu`
- Memory from `free -b`
- Filesystem usage from `df -B1 -P`

Collector definitions are in [defaults/main.yaml](defaults/main.yaml).
Script logic is in [templates/os_updates.sh.j2](templates/os_updates.sh.j2).
