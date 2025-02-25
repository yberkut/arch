# Arch 2.4. Бекап LUKS-заголовку

Щоб уникнути втрати доступу до шифрованих даних, потрібно зберегти заголовок LUKS.

### Передумови

[Arch 2.1. Монтування зовнішнього HDD NTFS для бекапів](https://github.com/yberkut/arch/blob/main/Arch-2-1--Mount-external-HDD-for-Backups-ua.md)

---

#### Створення бекапу заголовку:

```bash
sudo cryptsetup luksHeaderBackup /dev/nvme0n1p2 --header-backup-file /mnt/backup/luks-header.img
```

(Заміни `nvme0n1p2` на свій зашифрований розділ).

---

#### Відновлення заголовку у разі пошкодження:

```bash
sudo cryptsetup luksHeaderRestore /dev/nvme0n1p2 --header-backup-file /mnt/backup/luks-header.img
```
