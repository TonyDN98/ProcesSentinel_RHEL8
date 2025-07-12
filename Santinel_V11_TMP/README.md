# Process Monitor Service (Santinel)

## Overview

Santinel is a robust and flexible process monitoring service written in Bash. It is designed to run on RedHat-based Linux distributions. The service monitors a list of processes defined in a MySQL database and automatically restarts them if they enter an alarm state. It features a circuit breaker pattern to prevent cascading failures, customizable restart strategies, and comprehensive logging.

## Features

*   **Database-Driven Monitoring:** Monitors processes based on their status in a MySQL database.
*   **Multiple Restart Strategies:** Supports `service`, `process`, `custom`, and `auto` restart strategies.
*   **Circuit Breaker:** Prevents continuous restart attempts of failing processes.
*   **Health Checks:** Verifies the process status after a restart using custom commands.
*   **Comprehensive Logging:** Logs to both a file and syslog with automatic log rotation.
*   **Flexible Configuration:** All settings are managed through a single `config.ini` file.
*   **Pre-Restart Hooks:** Allows running custom commands before a restart attempt.
*   **Man Page:** Includes a man page for easy access to documentation.

## Requirements

*   A RedHat-based Linux distribution (e.g., CentOS, Fedora, RHEL).
*   `mysql-server` and `mysql-client` packages installed.
*   `bash` version 4 or higher.

## Installation

1.  **System Preparation**

    Update your system and install the necessary dependencies:
    ```bash
    sudo dnf update -y
    sudo dnf install -y mysql-server mysql-client
    ```

2.  **Configure Service Directory**

    Create a directory for the service and copy the necessary files:
    ```bash
    sudo mkdir -p /opt/monitor_service
    sudo cp monitor_service.sh /opt/monitor_service/
    sudo cp config.ini /opt/monitor_service/
    ```

3.  **Set Permissions**

    Set the correct permissions for the service files:
    ```bash
    sudo chmod 755 /opt/monitor_service
    sudo chmod 700 /opt/monitor_service/monitor_service.sh
    sudo chmod 600 /opt/monitor_service/config.ini
    sudo chown -R root:root /opt/monitor_service
    ```

4.  **MySQL Configuration**

    Start the MySQL service and run the setup script to create the database and tables:
    ```bash
    sudo systemctl enable mysqld
    sudo systemctl start mysqld
    sudo mysql < setup.sql
    ```

5.  **Systemd Service Configuration**

    Copy the service file to the systemd directory, reload the daemon, and start the service:
    ```bash
    sudo cp monitor_service.service /etc/systemd/system/
    sudo systemctl daemon-reload
    sudo systemctl enable monitor_service
    sudo systemctl start monitor_service
    ```

6.  **Install Man Page**

    Copy the man page to the appropriate directory and update the man database:
    ```bash
    sudo cp monitor_service.8 /usr/share/man/man8/
    sudo mandb
    ```
    You can now view the man page with `man monitor_service`.

## Configuration

The service is configured through the `config.ini` file.

### `[database]` Section

*   `host`: The hostname or IP address of the MySQL server.
*   `user`: The username for the MySQL database.
*   `password`: The password for the MySQL user.
*   `database`: The name of the MySQL database.

### `[monitor]` Section

*   `check_interval`: The interval in seconds at which the service checks the process statuses.
*   `max_restart_failures`: The maximum number of restart failures before the circuit breaker opens.
*   `circuit_reset_time`: The time in seconds before the circuit breaker resets.

### `[logging]` Section

*   `max_log_size`: The maximum log file size in KB before rotation.
*   `log_files_to_keep`: The number of rotated log files to keep.

### `[process.default]` Section

This section defines the default settings for all processes.

*   `restart_strategy`: The default restart strategy (`auto`, `custom`, `service`, or `process`).
*   `health_check_command`: The default command to check the health of a process. Use `%s` as a placeholder for the process name.
*   `health_check_timeout`: The default timeout in seconds for the health check.
*   `restart_delay`: The default delay in seconds between restart attempts.
*   `max_attempts`: The default maximum number of restart attempts.

### `[process.<process_name>]` Sections

You can define specific configurations for each process by creating a section with the process name (e.g., `[process.sshd]`).

*   `system_name`: The actual service or process name if it's different from the logical process name.
*   `pre_restart_command`: A command to be executed before the restart command.
*   `restart_strategy`: The restart strategy for this specific process.
*   `restart_command`: The custom command to restart the process. Only used when `restart_strategy` is `custom`.
*   `health_check_command`: The health check command for this specific process.
*   `health_check_timeout`: The health check timeout for this specific process.
*   `restart_delay`: The restart delay for this specific process.
*   `max_attempts`: The maximum number of restart attempts for this specific process.

## Usage

*   **Start the service:**
    ```bash
    sudo systemctl start monitor_service
    ```
*   **Stop the service:**
    ```bash
    sudo systemctl stop monitor_service
    ```
*   **Check the status of the service:**
    ```bash
    sudo systemctl status monitor_service
    ```
*   **View the logs:**
    ```bash
    sudo journalctl -u monitor_service -f
    ```
    or
    ```bash
    sudo tail -f /var/log/monitor_service.log
    ```

## Database Schema

The service requires two tables in the database: `PROCESE` and `STATUS_PROCESS`.

### `PROCESE` Table

```sql
CREATE TABLE PROCESE (
    process_id INT PRIMARY KEY,
    process_name VARCHAR(255) NOT NULL
);
```

### `STATUS_PROCESS` Table

```sql
CREATE TABLE STATUS_PROCESS (
    process_id INT PRIMARY KEY,
    alarma TINYINT NOT NULL,
    sound TINYINT NOT NULL,
    notes TEXT,
    FOREIGN KEY (process_id) REFERENCES PROCESE(process_id)
);
```

## Troubleshooting

*   **Service Fails to Start:**
    *   Check the service logs for errors: `sudo journalctl -u monitor_service` or `/var/log/monitor_service.log`.
    *   Verify that the `config.ini` file has the correct permissions (600) and is owned by root.
    *   Ensure that the `monitor_service.sh` script has the correct permissions (700) and is owned by root.
*   **Database Connection Issues:**
    *   Check the database credentials in `config.ini`.
    *   Ensure that the MySQL server is running and accessible from the host where the service is running.
*   **Process Restart Failures:**
    *   Check the logs for errors related to the specific process.
    *   Verify that the `system_name` in the process configuration is correct.
    *   If using the `service` strategy, ensure that the service is managed by systemd.
    *   If using the `process` strategy, ensure that the process can be started from the command line.
    *   If using the `custom` strategy, verify that the `restart_command` is correct and executable.

## Improvement Suggestions

*   **Enhanced Security:** Run the service with a dedicated user with minimal required permissions.
*   **Monitoring Enhancements:**
    *   Add process uptime tracking.
    *   Implement process dependency management.
*   **Logging and Metrics:**
    *   Integrate with Prometheus for metrics collection.
    *   Create dashboard templates for monitoring.
*   **Process Management:**
    *   Implement graceful shutdown procedures.
    *   Support for process priority levels.
    *   Extend health check capabilities with HTTP/API endpoint checks.
*   **Database Optimizations:**
    *   Implement connection pooling.
    *   Add retry mechanisms for database operations.
*   **Testing and Validation:**
    *   Add unit tests for core functions.
