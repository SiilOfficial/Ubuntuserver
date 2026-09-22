# Ubuntuserver
Made for Linux OP lesson




disk-report.service


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



disk-report.timer


[Unit]
Description=Run disk-report daily
Documentation=systemd.time(7)

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
