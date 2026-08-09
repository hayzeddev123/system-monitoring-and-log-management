# PROJECT 2: SYSTEM MONITORING AND LOG MANAGEMENT

## 1. Introduction

System monitoring and log management are important aspects of Linux system administration. Monitoring tools provide information about CPU, memory, disk, and process usage, while log management helps administrators identify errors and maintain historical records of system activity.

This project demonstrates system performance monitoring, disk usage analysis, log rotation, system log analysis, and automated disk usage monitoring using Cron.

## 2. Objective

The objective of this project is to:

- Monitor CPU, memory, and process usage.
- Check available and used disk space.
- Create and manage a custom application log.
- Configure automatic log rotation.
- Search system logs for errors and warnings.
- Create an automated disk usage monitoring task.

## 3. Environment

The practical was performed using:

- **Operating System:** Ubuntu 22.04.5 LTS
- **Virtualization:** Vagrant / VirtualBox
- **Shell:** Bash

## 4. Monitoring System Performance

The `htop` package was installed using:

```bash
sudo apt update
sudo apt install htop -y
```

![Updating packages and installing htop](Screenshot%202026-08-10%20002931.png)

The system monitoring interface was launched with:

```bash
htop
```

`htop` provided real-time information about:

- CPU utilization
- Memory utilization
- Swap usage
- Running processes
- Process IDs
- System load

The tool was exited using the `q` key.

## 5. Checking Disk Usage

Disk usage was examined using:

```bash
df -h
```

The `df -h` command displayed filesystem sizes in a human-readable format, including total space, used space, available space, and percentage utilization.

The `/home` directory was examined using:

```bash
du -sh /home
```

![Checking disk usage with df and du, and creating a custom log](Screenshot%202026-08-10%20003236.png)

The command provided the total amount of disk space used by the `/home` directory.

Individual directories could also be examined using:

```bash
du -sh /home/*
```

## 6. Creating a Custom Log File

A custom application log was created at:

```text
/var/log/myapp.log
```

The file was created using:

```bash
sudo touch /var/log/myapp.log
```

Test log entries were added:

```bash
echo "$(date) INFO Application started" | sudo tee -a /var/log/myapp.log
echo "$(date) ERROR Database connection failed" | sudo tee -a /var/log/myapp.log
echo "$(date) INFO Application stopped" | sudo tee -a /var/log/myapp.log
```

The contents of the log were checked using:

```bash
sudo cat /var/log/myapp.log
```

The log contained informational and error messages that could later be searched or rotated.

## 7. Configuring Log Rotation

A custom Logrotate configuration was created:

```bash
sudo nano /etc/logrotate.d/myapp
```

The following configuration was added:

```text
/var/log/myapp.log {
    daily
    missingok
    rotate 7
    compress
    delaycompress
    notifempty
    create 0640 root adm
}
```

### Configuration Explanation

| Option         | Purpose                                              |
|----------------|------------------------------------------------------|
| `daily`        | Rotates the log daily                                |
| `missingok`    | Prevents errors when the log file is missing         |
| `rotate 7`     | Retains seven rotated log files                      |
| `compress`     | Compresses older log files                           |
| `delaycompress`| Delays compression of the most recent rotated file   |
| `notifempty`   | Prevents rotation of empty logs                      |
| `create 0640 root adm` | Creates a new log with specified permissions |

The configuration was tested manually using:

```bash
sudo logrotate -f /etc/logrotate.d/myapp
```

![Viewing the custom log, configuring logrotate, and searching syslog](Screenshot%202026-08-10%20003700.png)

The rotated files were checked with:

```bash
sudo ls -lh /var/log/myapp*
```

## 8. Analyzing System Logs

The system log was searched for error messages using:

```bash
sudo grep -i "error" /var/log/syslog
```

The `-i` option makes the search case-insensitive.

Recent system log entries were viewed using:

```bash
sudo tail -20 /var/log/syslog
```

Warning messages could also be searched using:

```bash
sudo grep -i "warning" /var/log/syslog
```

These commands provide a simple way to locate potential problems in system activity logs.

## 9. Disk Usage Monitoring

Disk usage was monitored using `df` and `awk`.

The following command checks for filesystems using more than 90% of their available space:

```bash
df -h | awk 'NR>1 && $5+0 > 90 {print "ALERT:", $1, $5}'
```

If no output is produced, it indicates that no filesystem currently meets the 90% threshold.

The current disk usage can be viewed using:

```bash
df -h
```

## 10. Automated Monitoring with Cron

A Cron job was configured to periodically check disk usage.

The root user's Cron configuration was opened using:

```bash
sudo crontab -e
```

![Checking disk usage threshold and opening root crontab](Screenshot%202026-08-10%20003755.png)

The following entry was used:

```text
*/10 * * * * df -h | awk 'NR>1 && $5+0 > 90 {print strftime("%Y-%m-%d %H:%M:%S"), "ALERT:", $1, $5}' >> /var/log/disk-alert.log
```

This Cron job runs every 10 minutes and records an alert in `/var/log/disk-alert.log` whenever disk usage exceeds 90%.

The Cron configuration was verified using:

```bash
sudo crontab -l
```

## 11. Email Alert Consideration

The original project specification uses the `mail` command to send disk usage alerts to an administrator.

However, sending email from a fresh Ubuntu virtual machine requires a configured mail-transfer service. Instead of configuring a complete mail server, the practical used a local alert log:

```text
/var/log/disk-alert.log
```

This demonstrates the monitoring and automation requirements without requiring external email infrastructure.

## 12. Commands Used

| Command          | Purpose                                      |
|------------------|----------------------------------------------|
| `htop`           | Monitors CPU, memory, and processes          |
| `df -h`          | Displays filesystem disk usage               |
| `du -sh`         | Displays directory disk usage                |
| `touch`          | Creates a log file                           |
| `cat`            | Displays log contents                        |
| `grep`           | Searches log files                           |
| `tail`           | Displays recent log entries                  |
| `logrotate`      | Rotates and manages log files                |
| `awk`            | Processes disk usage information             |
| `crontab -e`     | Creates or edits scheduled tasks             |
| `crontab -l`     | Displays scheduled Cron tasks                |

## 13. Results

The practical demonstrated the following:

- System resources were monitored using `htop`.
- Disk space was examined using `df` and `du`.
- A custom application log was created.
- Logrotate was configured for automated log management.
- System logs were searched for errors and warnings.
- Disk usage was checked against a 90% threshold.
- Cron was configured to perform automated disk monitoring.
- Disk alerts were recorded in a local log file.

## 14. Conclusion

The System Monitoring and Log Management project demonstrated important Linux administration techniques for monitoring system health and managing system records.

The `htop` utility provided real-time performance information, while `df` and `du` were used to monitor disk consumption. A custom application log was created and configured for automatic rotation using Logrotate, helping prevent logs from consuming excessive disk space.

System logs were analyzed using commands such as `grep` and `tail`, while Cron was used to automate disk usage monitoring. The project provided practical experience with Linux monitoring, logging, log rotation, and scheduled automation, which are fundamental skills for system administration and DevOps.

---

**Environment used:** Ubuntu 22.04.5 LTS (Vagrant / VirtualBox)  
**Author:** Adeniyi Abdulazeez
