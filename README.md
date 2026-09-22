# Lab 2.A — A service and a timer

## 1. Final unit files

### disk-report.service


[Unit]
Description=Append disk usage to log
Documentation=man:df(1)
After=local-fs.target

[Service]
Type=oneshot
User=reports
ExecStart=/usr/local/bin/disk-report.sh
StandardOutput=append:/var/log/disk-report.log
StandardError=append:/var/log/disk-report.log


### disk-report.timer


[Unit]
Description=Run disk-report daily
Documentation=systemd.time(7)

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target

## 2. Initial error and what I found

When the timer first started the service, it failed with this error:

text
Sep 21 06:25:35 karl systemd[1]: Starting disk-report.service - Append disk usage to log...
Sep 21 06:25:35 karl disk-report.sh[2377]: /usr/local/bin/disk-report.sh: line 2: /var/log/disk-report.log: Permission denied
Sep 21 06:25:35 karl disk-report.sh[2379]: /usr/local/bin/disk-report.sh: line 3: /var/log/disk-report.log: Permission denied
Sep 21 06:25:35 karl systemd[1]: disk-report.service: Main process exited, code=exited, status=1/FAILURE
Sep 21 06:25:35 karl systemd[1]: disk-report.service: Failed with result 'exit-code'.
Sep 21 06:25:35 karl systemd[1]: Failed to start disk-report.service - Append disk usage to log.

The important part was Permission denied. The service was running as the reports user, but the script was trying to write directly to /var/log/disk-report.log, which it didn't have permission to do. The status=1/FAILURE also confirmed that this permission problem caused the service to fail.

## 3. Why I used Option B

I used Option B with StandardOutput=append: because the reports user doesn't need to own the log file or have direct write permissions to it. Instead, systemd handles putting the output into /var/log/disk-report.log, while the service can still run as the restricted reports user. I think this is safer because the service gets fewer permissions than it would if I changed the ownership of the log file with chown.

## 4. Timer

After changing the timer to run daily, systemctl list-timers disk-report.timer showed:

text
NEXT                        LEFT LAST                              PASSED UNIT
Tue 2026-09-22 00:00:00 UTC 17h Mon 2026-09-21 06:30:35 UTC 2min 30s ago disk-report.timer


This means the next run was scheduled for 2026-09-22 00:00:00 UTC.

## 5. Successful runs

I checked the service journal and found two successful runs:

text
Sep 21 06:27:49 karl systemd[1]: disk-report.service: Deactivated successfully.
Sep 21 06:27:49 karl systemd[1]: Finished disk-report.service - Append disk usage to log.

Sep 21 06:30:35 karl systemd[1]: disk-report.service: Deactivated successfully.
Sep 21 06:30:35 karl systemd[1]: Finished disk-report.service - Append disk usage to log.


## 6. Why the reports user was created this way

The reports user was created with --no-create-home and /usr/sbin/nologin because it only needs to run the disk-report service and doesn't need a home directory or a normal login shell. If those restrictions were not used, someone who gained access to the reports account could potentially log in interactively and use the account for other purposes.

The reports user was created with --no-create-home and /usr/sbin/nologin because it only needs to run the disk-report service and doesn't need a home directory or a normal login shell. If those restrictions were not used, someone who gained access to the reports account could potentially log in interactively and use the account for other purposes.
