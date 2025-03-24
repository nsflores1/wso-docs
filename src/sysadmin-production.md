# Production Server
Common tasks are listed below.
- Update the `crontab` after changing it in the repository:
    ``` shell
    $ # Bare crontab loads the cpanel's crontab, not wso's
    $ sudo -u wso EDITOR=nano crontab -e
    ```
- Editing the running configuation: 
    ``` shell
    $ nano /home/wso/wso/wso-backend/config.yaml
    ```
- Get to database files:
    ``` shell
    $ cd /var/lib/mysql/
    ```
- Update the firewall:
  ``` shell
  $ # Always back up before making changes!
  $ iptables-save > /etc/sysconfig/iptables-$(date --iso-8601=seconds).bak
  $ iptables
  ```
Backups are located in `/home/backups`. These are created by cPanel every night at 2AM, and are copied via `rsync` to the backup server. 

cPanel also manages the firewall, but this seems to be bugged. It is recommended to use `iptables` to manage your firewall-related needs instead, since we have not translated all the `iptables` rules into their cPanel equivalents. So try to use cPanel first, but if it fails, go the old-fashioned way.

Open ports and their usage are listed below. Non-standard ports are bolded.
- **123**: `ntpd`
- **2078-2096**: `cpdavd`
- 80 and 443: `httpd`
- 22: `sshd`
- 25 and **587**: `exim`
- **53**: `named`
- 110, 465, and **143**: `dovecot`
- **9091**: `node_exporter`
- **9092**: `grafana`
- **9095**: `prometheus`
- **3306**: `mysqld`
- **2812**: `monit`
- **10395**: `bifrost`
- **10397**: `telos`
