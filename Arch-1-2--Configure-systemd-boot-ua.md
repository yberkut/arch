# Налаштування systemd

### 1. Переконайся, що `/efi` змонтовано правильно

```bash
# Завантажся в LiveCD і змонтуй розділи
mount /dev/mapper/vg0-root /mnt
mount /dev/nvme1n1p1 /mnt/efi
arch-chroot /mnt
```

### 2. Видали старий systemd-boot

```bash
rm -rf /efi/EFI/systemd
bootctl remove
```

###  3. Встанови заново systemd-boot

```bash
bootctl install --esp-path=/efi --oot-path=/boot --root=/
```

Після цього у `/efi/EFI/systemd/` має з’явитися `systemd-bootx64.efi`.

### 4. Додай EFI-запис через UUID

```bash
UUID=$(blkid -s UUID -o value /dev/nvme1n1p1)
efibootmgr --create --disk /dev/disk/by-uuid/$UUID --part 1 --loader "\EFI\systemd\systemd-bootx64.efi" --label "Systemd-Boot" --unicode
```

### 5. Перевір, чи тепер є запис у `efibootmgr`

```bash
efibootmgr -v
```

---

Оскільки ти використовуєш **systemd-boot**, то тобі потрібно **systemd-based initramfs**. Це означає, що ти маєш використовувати **sd-encrypt** замість **encrypt** і **sd-vconsole** замість **keymap + consolefont**.

Тобто твій `HOOKS=()` у `/etc/mkinitcpio.conf` має виглядати так:

```bash
HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block sd-encrypt lvm2 filesystems fsck)
```

### **Чому так?**

- `systemd` – ініціалізація на базі systemd.
- `sd-encrypt` – розшифровка LUKS через systemd (замість звичайного `encrypt`).
- `sd-vconsole` – завантаження keymap і консолі (замість `keymap` та `consolefont`).
- `lvm2` – підтримка LVM (після `sd-encrypt`, бо спочатку треба розшифрувати диск).

Після внесення змін виконай:

```bash
mkinitcpio -P

# Якщо виникнуть проблеми з sd-console
touch /etc/vconsole.conf
# І при бажанні можна туди додати розкладку чи шрифт
KEYMAP=us
FONT=lat9w-16
# І ще раз
mkinitcpio -P
```
