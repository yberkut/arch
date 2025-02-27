# Заміна GRUB на systemd-boot

## Передумови

- Завантажувач **GRUB** встановлений і потрібно його замінити на **systemd-boot**.
- Диск розмічений так:
  - `/dev/nvme1n1p1` – **EFI (FAT32), змонтований у `/efi`**.
  - `/dev/nvme1n1p2` – **зашифрований LUKS2, містить LVM (`/dev/vg0`)**.
  - `/dev/vg0/root` – основний розділ (`/`), містить `/boot`.

## 1. Видалення GRUB

```sh
arch-chroot /mnt
pacman -Rns grub
rm -rf /boot/grub
```

## 2. Встановлення systemd-boot

```sh
bootctl install --esp-path=/efi --boot-path=/boot
```

## 3. Створення loader.conf

```sh
cat <<EOF > /efi/loader/loader.conf
default arch
timeout 3
editor 0
EOF
```

## 4. Створення запису для ядра

Отримати UUID зашифрованого розділу та створити файл `/efi/loader/entries/arch.conf`:

```sh
UUID=$(blkid -s UUID -o value /dev/nvme1n1p2)
cat <<EOF > /efi/loader/entries/arch.conf
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options cryptdevice=UUID=$UUID:cryptroot root=/dev/mapper/vg0-root rw
EOF
```

## 5. Оновлення initramfs

Відкрити `/etc/mkinitcpio.conf` і змінити `HOOKS=()` на:

```
HOOKS=(base udev autodetect microcode modconf kms block keyboard keymap consolefont encrypt filesystems lvm2 fsck)
```

### Опис ключових елементів `HOOKS`:

- `base` – базові утиліти, необхідні для завантаження системи.
- `udev` – керування пристроями.
- `autodetect` – визначає необхідні модулі для поточної системи.
- `microcode` – завантаження оновлень мікрокоду процесора.
- `modconf` – дозволяє змінювати модулі ядра під час завантаження.
- `kms` – ініціалізація графічного драйвера на ранніх етапах завантаження.
- `block` – підтримка блочних пристроїв (дисків, розділів).
- `keyboard` – підтримка введення з клавіатури під час завантаження.
- `keymap` – завантаження розкладки клавіатури.
- `consolefont` – змінює шрифт у консолі.
- `encrypt` – підтримка розшифрування LUKS.
- `filesystems` – монтує файлові системи.
- `lvm2` – підтримка LVM, активує `vg0`.
- `fsck` – перевірка дискових розділів при завантаженні.

Далі виконати:

```sh
mkinitcpio -P
```

## 6. Оновлення fstab (якщо потрібно)

Переконатися, що `/efi` змонтовано правильно:

```sh
grep " /efi " /etc/fstab || echo "UUID=$(blkid -s UUID -o value /dev/nvme1n1p1) /efi vfat defaults 0 1" >> /etc/fstab
```

## 7. Перевірка та перезавантаження

```sh
exit
umount -R /mnt
reboot
```

