# Arch 2.1. Монтування зовнішнього HDD (NTFS) для бекапів

### 1. Монтуємо HDD (NTFS)

Перевір, як твій диск змонтовано:

```bash
lsblk -o NAME,FSTYPE,MOUNTPOINT
```

Якщо ще не змонтовано, створи точку монтування та змонтуй:

```backup
sudo mkdir -p /mnt/backup
sudo mount -t ntfs3 /dev/sdX1 /mnt/backup
```

(Заміни `/dev/sdX1` на правильний розділ.)

---

### 2. Монтуємо потрібну директорію

Якщо, наприклад, потрібно змонтувати `/mnt/hdd/Backups` у `/mnt/backup`, використовуй:

```bash
sudo mount --bind /mnt/hdd/Backups /mnt/backup
```

---

### 3. Автоматичне монтування (fstab)

Щоб змонтовувати цю директорію автоматично після перезавантаження, додай у `/etc/fstab` рядок:

```bash
/mnt/hdd/Backups /mnt/backup none bind 0 0
```

Тепер система автоматично монтуватиме тільки потрібну директорію при старті.
