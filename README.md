Виконав cамостійно Шевченко Артем РПЗ-23А
# WORK-CASE 7
# Task Scheduling Report

## 1. Task Scheduler Functions and Comparison between Windows and Linux

### Functions of Task Schedulers
Task schedulers in operating systems (OS) are responsible for automating the execution of tasks at specified times or intervals. The main functions include:

- **Automated Execution**: Schedule scripts, applications, or commands to run automatically without user intervention.
- **Time-Based Scheduling**: Execute tasks at specific times, dates, or intervals (e.g., hourly, daily, weekly).
- **Event-Based Scheduling**: Trigger tasks based on specific events (e.g., system startup, user logon).
- **Resource Management**: Manage system resources to ensure that scheduled tasks do not interfere with other processes.
- **Logging and Notifications**: Provide logs of executed tasks and send notifications in case of failures or completions.

### Comparison of Task Scheduling in Windows and Linux
- **Windows Task Scheduler**:
  - GUI-based interface for easy task management.
  - Supports both time-based and event-based triggers.
  - Allows for complex conditions and actions (e.g., run a task only if a specific network connection is available).
  - Provides detailed logging and error reporting.

- **Linux Cron**:
  - Primarily command-line based, though there are GUI front-ends available.
  - Focuses on time-based scheduling using a simple syntax in crontab files.
  - Lacks built-in event-based triggers but can be combined with other tools (e.g., systemd timers).
  - Provides logging through system logs, but less detailed than Windows Task Scheduler.

## 2. Principles of Cron in Linux

### Overview of Cron
Cron is a time-based job scheduler in Unix-like operating systems. It allows users to schedule jobs (commands or scripts) to run at specific intervals.

### Configuration and Usage
- **Crontab File**: Each user has a crontab file where scheduled tasks are defined. The syntax for a crontab entry is:
  ```
  * * * * * command_to_execute
  ```
  The five asterisks represent:
  - Minute (0-59)
  - Hour (0-23)
  - Day of the month (1-31)
  - Month (1-12)
  - Day of the week (0-7, where both 0 and 7 represent Sunday)

- **Editing Crontab**: Use the command `crontab -e` to edit the crontab file.

### Alternatives to Cron
- **systemd Timers**: A more modern alternative that integrates with the systemd service manager. It allows for more complex scheduling and event-based triggers.
- **Anacron**: Designed for systems that are not running continuously. It ensures that scheduled tasks are executed even if the system was off during the scheduled time.
- **fcron**: Combines features of cron and anacron, allowing for both time-based and event-based scheduling.

## 3. Scheduling Tasks with Cron
# Backup Script and Scheduling Report

## Overview
This report outlines the implementation of a backup script that automates the process of backing up the user's "Documents" directory. The script is scheduled to run at various intervals using both cron jobs and systemd timers.

## Script Implementation

### 1. Backup Script Creation
The following commands create a backup script located in the user's `scripts` directory:

```bash
mkdir -p "$HOME/scripts"

cat > "$HOME/scripts/backup.sh" << 'SCRIPT'
#!/bin/bash

TARGET_DIR="$HOME/backups"
mkdir -p "$TARGET_DIR"

ARCHIVE_NAME="docs_backup_$(date +%Y%m%d_%H%M%S).tar.gz"

tar -czf "${TARGET_DIR}/${ARCHIVE_NAME}" -C "$HOME" Documents 2>/dev/null

# Remove files older than 7 days
find "$TARGET_DIR" -maxdepth 1 -name "docs_backup_*.tar.gz" -mtime +7 -exec rm {} \;
SCRIPT

chmod +x "$HOME/scripts/backup.sh"
```

### 2. Immediate Backup Execution
The script is executed immediately after creation to ensure it works as intended:

```bash
echo "Executing backup now:"
bash "$HOME/scripts/backup.sh"

echo "Backup files in $HOME/backups:"
ls -lh "$HOME/backups"
```

## Task Scheduling with Cron

### 1. Cron Job Setup
The script is scheduled to run at various times using cron jobs. The following function adds entries to the user's crontab:

```bash
CRON_SCRIPT="$HOME/scripts/backup.sh"

add_cron_entry() {
    local entry="$1"
    (crontab -l 2>/dev/null | grep -v -F "$entry"; echo "$entry") | crontab -
}
```

### 2. Scheduled Tasks
The following cron entries are added:

- **Daily at 7:00 AM**:
  ```bash
  add_cron_entry "0 7 * * * $CRON_SCRIPT"
  ```

- **Twice a day at 7:00 AM and 7:00 PM**:
  ```bash
  add_cron_entry "0 7,19 * * * $CRON_SCRIPT"
  ```

- **Every hour on weekdays from 9 AM to 5 PM**:
  ```bash
  add_cron_entry "0 9-17 * * 1-5 $CRON_SCRIPT"
  ```

- **Once a year on January 1st at 1:00 AM**:
  ```bash
  add_cron_entry "0 1 1 1 * $CRON_SCRIPT"
  ```

- **Once a month on the 1st at 4:00 AM**:
  ```bash
  add_cron_entry "0 4 1 * * $CRON_SCRIPT"
  ```

- **Daily at 5:00 AM**:
  ```bash
  add_cron_entry "0 5 * * * $CRON_SCRIPT"
  ```

- **Hourly**:
  ```bash
  add_cron_entry "0 * * * * $CRON_SCRIPT"
  ```

- **At system startup**:
  ```bash
  add_cron_entry "@reboot $CRON_SCRIPT"
  ```

### 3. Current Crontab Verification
The current crontab is displayed to verify the scheduled tasks:

```bash
echo "Current crontab after update:"
crontab -l
```

## Task Scheduling with Systemd

### 1. Service and Timer Creation
The following commands create a systemd service and timer for the backup script:

```bash
SERVICE_FILE="/etc/systemd/system/doc_backup.service"
TIMER_FILE="/etc/systemd/system/doc_backup.timer"
SCRIPT_PATH="$HOME/scripts/backup.sh"

sudo bash -c "cat > $SERVICE_FILE" << EOF
[Unit]
Description=Backup Documents Folder (systemd service)

[Service]
Type=oneshot
ExecStart=$SCRIPT_PATH
EOF

sudo bash -c "cat > $TIMER_FILE" << EOF
[Unit]
Description=Daily Backup of Documents at 07:30

[Timer]
OnCalendar=*-*-* 07:30:00
Persistent=true

[Install]
WantedBy=timers.target
EOF
```

### 2. Enabling and Starting the Timer
The systemd daemon is reloaded, and the timer is enabled and started:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now doc_backup.timer
sudo systemctl start doc_backup.service
```

### 3. Backup Verification
After starting the systemd service, the backup files are listed:

```bash
echo "Backup files after starting systemd service:"
ls -lh "$HOME/backups"
```

### 4. Systemd Timer Information
Information about the systemd timer is displayed:

```bash
echo -e "\nSystemd timer information:"
systemctl list-timers | grep doc_backup
```
# Screenshots for work
![image](https://github.com/user-attachments/assets/0c3ea4f1-b03c-461b-9bdd-dbf24aa54419)
![image](https://github.com/user-attachments/assets/3f220829-f87c-437e-8477-8f23ee6e0ce6)
![image](https://github.com/user-attachments/assets/0c835b45-c59e-416b-90db-a15612ec4fe9)


## Conclusion
The backup script was successfully created and executed, with multiple scheduling methods implemented using both cron and systemd timers. The scheduled tasks ensure that backups are performed regularly, and the systemd timer provides a robust alternative to cron for managing scheduled tasks. The backup files are stored in the designated directory, and their existence confirms the successful execution of the backup process.
