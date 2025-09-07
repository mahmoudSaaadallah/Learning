The `at` command in Linux and Unix-like systems is used to schedule commands to be executed **once** at a specific time in the future. Unlike `cron`, which is for recurring tasks, `at` is perfect for one-off jobs that you want to run later.

### Prerequisites

1.  **`atd` service:** The `at` daemon (`atd`) must be running on your system. You can usually check its status with `systemctl status atd` (on systemd-based systems like Ubuntu, Fedora, CentOS 7+). If it's not running, you might need to start it: `sudo systemctl start atd` and enable it: `sudo systemctl enable atd`.
2.  **User Permissions:** Users must be allowed to use `at`. This is controlled by `/etc/at.allow` and `/etc/at.deny`.
    *   If `/etc/at.allow` exists, only users listed in it can use `at`.
    *   If `/etc/at.allow` does not exist, then `/etc/at.deny` is checked. Users listed in `/etc/at.deny` cannot use `at`.
    *   If neither file exists, only the root user can use `at`.
    *   By default, most systems have an `/etc/at.deny` file that is empty or contains a few system users, meaning most users are allowed.

### Basic Usage

The general syntax for `at` is:

```bash
at [OPTIONS] TIME
```

After typing `at TIME` and pressing Enter, you'll be presented with an `at>` prompt. Here, you type the commands you want to execute. When you're done, press `Ctrl+D` to save the job.

**Example:** Schedule a message to appear in 5 minutes.

```bash
$ at now + 5 minutes
warning: commands will be executed using /bin/sh
at> echo "Hello from the future!" > ~/future_message.txt
at> <EOT> # This is what appears after you press Ctrl+D
job 1 at Mon Jul 29 10:30:00 2024
```

In this example:
*   `at now + 5 minutes`: Schedules the job to run 5 minutes from the current time.
*   `echo "Hello from the future!" > ~/future_message.txt`: The command to be executed.
*   `<EOT>`: End Of Transmission, indicated by `Ctrl+D`.
*   `job 1 at Mon Jul 29 10:30:00 2024`: The `at` command confirms the job ID (1) and the scheduled execution time.

### Specifying Time

`at` is very flexible with time specifications. Here are some common formats:

#### Relative Times

*   **`now + COUNT UNITS`**:
    *   `at now + 5 minutes`
    *   `at now + 1 hour`
    *   `at now + 2 days`
    *   `at now + 1 week`
*   **`tomorrow`**: `at tomorrow` (runs at 00:00 of the next day)
*   **`noon`**: `at noon` (12:00 PM)
*   **`midnight`**: `at midnight` (00:00 AM of the next day)
*   **`teatime`**: `at teatime` (4:00 PM or 16:00)

#### Absolute Times

*   **`HH:MM`**:
    *   `at 14:30` (2:30 PM today, or tomorrow if 2:30 PM has already passed)
    *   `at 09:00` (9:00 AM today, or tomorrow if 9:00 AM has already passed)
*   **`HH:MM MONTH DAY`**:
    *   `at 10:00 Jul 30` (10:00 AM on July 30th)
    *   `at 10:00 07/30/2024` (10:00 AM on July 30th, 2024)
*   **`HH:MM YYYY-MM-DD`**:
    *   `at 10:00 2024-07-30`
*   **`HH:MM AM/PM`**:
    *   `at 3pm`
    *   `at 10:30am tomorrow`

**More Examples of Time Specifications:**

*   `at 17:00` (5 PM today)
*   `at 17:00 tomorrow` (5 PM tomorrow)
*   `at 17:00 2024-08-15` (5 PM on August 15, 2024)
*   `at 10:00 AM next monday` (10 AM next Monday)
*   `at 5:00 PM + 3 days` (5 PM three days from now)

### Providing Commands to `at`

Besides typing commands at the `at>` prompt, you can also pipe commands to `at`:

```bash
echo "ls -l /tmp" | at now + 10 minutes
```

Or, execute a script:

```bash
at now + 1 hour -f /path/to/your/script.sh
```
The `-f` option specifies a file containing the commands to be executed.

### Environment Variables

When an `at` job runs, it typically uses a minimal environment, often `/bin/sh` and a limited set of environment variables. This means:

*   **PATH:** Your usual `PATH` might not be available. It's good practice to use full paths for commands (e.g., `/usr/bin/ls` instead of `ls`).
*   **Shell:** The job usually runs with `/bin/sh`, not necessarily your interactive shell (like `bash` or `zsh`). If you need `bash`-specific features, explicitly invoke `bash`:
    ```bash
    at now + 10 minutes
    at> /bin/bash -c "source ~/.bashrc && my_custom_command"
    at> <EOT>
    ```
*   **Output:** Standard output and standard error from the commands are usually emailed to the user who scheduled the job (if a mail agent is configured). If you don't want emails, redirect output:
    ```bash
    at now + 1 hour
    at> /path/to/command > /dev/null 2>&1
    at> <EOT>
    ```

### Viewing Scheduled Jobs (`atq` or `at -l`)

To see a list of your pending `at` jobs:

```bash
$ atq
# Or:
$ at -l
```

The output will show the job ID, the scheduled time, and the queue letter (usually `a` for `at` jobs).

```
1       Mon Jul 29 10:30:00 2024 a user
2       Mon Jul 29 11:00:00 2024 a user
```

### Removing Scheduled Jobs (`atrm` or `at -d`)

To remove a scheduled job, you need its job ID (obtained from `atq`).

```bash
$ atrm JOB_ID
# Or:
$ at -d JOB_ID
```

**Example:** Remove job ID 1.

```bash
$ atrm 1
```

### `batch` Command

The `batch` command is similar to `at`, but it schedules jobs to run only when the system load average drops below a certain threshold (typically 0.8, but configurable). This is useful for resource-intensive tasks that you don't want to impact interactive users.

```bash
$ batch
at> find / -name "*.log" -delete
at> <EOT>
job 3 at Mon Jul 29 10:45:00 2024 b user
```
Notice the queue letter `b` for batch jobs.

### Summary Table

| Command | Purpose | Recurrence | Time Specification | Queue |
| :------ | :------ | :--------- | :----------------- | :---- |
| `at` | Schedule a job to run **once** at a specific time. | No | Flexible (absolute, relative) | `a` |
| `atq` | List pending `at` jobs. | N/A | N/A | N/A |
| `atrm` | Remove a pending `at` job. | N/A | N/A | N/A |
| `batch` | Schedule a job to run **once** when system load is low. | No | Flexible (absolute, relative) | `b` |

The `at` command is a simple yet powerful tool for managing one-time future tasks, especially useful for system administrators and users who need to automate specific actions without setting up recurring cron jobs.