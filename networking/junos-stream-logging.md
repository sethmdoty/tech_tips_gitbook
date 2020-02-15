---
description: 'this sends all log files to a syslog server and does not write them locally:'
---

# Junos Stream Logging

 \[Host\] mode stream; format sd-syslog; source-address {local\_IP{; stream securitylog { category all; host { 10.0.0.1; port 514; } }

