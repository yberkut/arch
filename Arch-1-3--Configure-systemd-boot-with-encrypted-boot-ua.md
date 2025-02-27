# Налаштування systemd-boot із зашифрованим /boot у Arch Linux

## Передумови
- У вас уже встановлений Arch Linux із LUKS + LVM
- `/boot` знаходиться всередині `vg0-root`, тобто в зашифрованому розділі
- Ви готові вводити пароль двічі: спочатку для розблокування `/boot`, потім для кореневого розділу

## 1. Встановлення systemd-boot
```bash
bootctl install
```

Ця команда встановить завантажувач у EFI (`/efi/BOOT/BOOTX64.EFI`).

## 2. Налаштування kernel cmdline
Створимо файл `/etc/kernel/cmdline`:
```bash
echo "rd.luks.name=$(blkid -s UUID -o value /dev/nvme0n1p2)=cryptroot root=/dev/vg0/root rw" > /etc/kernel/cmdline
```

## 3. Генерація initramfs
Переконаємося, що в `/etc/mkinitcpio.conf` є необхідні хуки:
```ini
HOOKS=(base udev autodetect modconf block keyboard keymap encrypt lvm2 filesystems fsck)
```
Генеруємо новий initramfs:
```bash
mkinitcpio -P
```

## 4. Налаштування systemd-boot
Створимо файл `/efi/loader/loader.conf`:
```ini
default arch
timeout 3
console-mode max
editor no
```

Додаємо конфігурацію ядра `/efi/loader/entries/arch.conf`:
```ini
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options rd.luks.name=$(blkid -s UUID -o value /dev/nvme0n1p2)=cryptroot root=/dev/vg0/root rw
```

Для модуля NVIDIA (якщо потрібно), створюємо `/efi/loader/entries/arch-nvidia.conf`:
```ini
title   Arch Linux (NVIDIA)
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
initrd  /initramfs-linux.img
options rd.luks.name=$(blkid -s UUID -o value /dev/nvme0n1p2)=cryptroot root=/dev/vg0/root rw nvidia-drm.modeset=1
```

## 5. Оновлення завантажувача після оновлення ядра
Щоразу після оновлення ядра потрібно оновлювати `initramfs` і копіювати ядро в EFI:
```bash
mkinitcpio -P
bootctl update
```

## 6. Перевірка та перезавантаження
Переконайтеся, що все правильно змонтовано:
```bash
findmnt /efi
findmnt /
```
Після цього перезавантажте систему:
```bash
reboot
```

При завантаженні вам доведеться двічі вводити пароль: спочатку для `cryptroot`, потім для root (`vg0-root`). Якщо все зроблено правильно, система завантажиться без проблем! 🚀
