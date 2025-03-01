# Інструкція зі створення флешки з LUKS-захистом для ключів шифрування

## 1. Розмітка флешки
### Перед виконанням всіх операцій, переконайся, що працюєш із правильною флешкою (`/dev/sdX`).
Заміни `X` на відповідну букву, щоб уникнути випадкового видалення важливих даних.

### 1.1. Видалення старих розділів
```sh
wipefs -a /dev/sdX
sgdisk --zap-all /dev/sdX
```

### 1.2. Створення нових розділів
```sh
gdisk /dev/sdX
```
Введи наступні команди у `gdisk`:
```
n  # Новий розділ
1  # Номер розділу
   # (Enter для стандартного початку)
+1G  # Розмір 1 ГБ
EF00  # Код EFI System

n  # Новий розділ
2  # Номер розділу
   # (Enter для стандартного початку)
+2G  # Розмір 2 ГБ
8300  # Код Linux Filesystem

w  # Запис змін
Y  # Підтвердити
```

### 1.3. Форматування розділів
```sh
mkfs.vfat -F32 /dev/sdX1  # Форматування EFI розділу
```

## 2. Налаштування LUKS для другого розділу (флешки)
```sh
cryptsetup luksFormat /dev/sdX2  # Шифруємо другий розділ
cryptsetup open /dev/sdX2 cryptusb  # Відкриваємо його
mkfs.ext4 /dev/mapper/cryptusb  # Форматуємо у ext4
```

## 3. Монтування зашифрованого розділу
```sh
mkdir -p /mnt/usb
mount /dev/mapper/cryptusb /mnt/usb
```

## 4. Створення LUKS-контейнера `key.img` для зберігання ключа
```sh
dd if=/dev/urandom of=/mnt/usb/key.img bs=1M count=10  # Створюємо файл-контейнер
cryptsetup luksFormat /mnt/usb/key.img  # Шифруємо контейнер
```

## 5. Створення ключового файлу для розшифрування `key.img`
```sh
dd if=/dev/urandom of=/mnt/usb/keyfile bs=1M count=1
chmod 600 /mnt/usb/keyfile
cryptsetup luksAddKey /mnt/usb/key.img /mnt/usb/keyfile
```

## 6. Відкриття `key.img` та створення ключа для основного диска
```sh
cryptsetup open /mnt/usb/key.img lukskey --key-file=/mnt/usb/keyfile

dd if=/dev/urandom of=/dev/mapper/lukskey bs=1M count=1
chmod 600 /dev/mapper/lukskey  # Захист від стороннього доступу
```

## 7. Додавання ключа до основного диска
```sh
cryptsetup luksAddKey /dev/disk/by-id/harddrive /dev/mapper/lukskey
```

## 8. Відключення та очищення
```sh
cryptsetup close lukskey
umount /mnt/usb
cryptsetup close cryptusb
```

## 9. Налаштування автоматичного монтування у `initcpio`
Створи хук `/etc/initcpio/hooks/customencrypthook`:
```sh
#!/usr/bin/ash

run_hook() {
    modprobe -a -q dm-crypt >/dev/null 2>&1
    modprobe loop
    [ "${quiet}" = "y" ] && CSQUIET=">/dev/null"

    while [ ! -L '/dev/disk/by-id/usbdrive-part2' ]; do
     echo 'Waiting for USB'
     sleep 1
    done

    cryptsetup open /dev/disk/by-id/usbdrive-part2 cryptusb
    mount --mkdir /dev/mapper/cryptusb /mnt
    cryptsetup open /mnt/key.img lukskey --key-file=/mnt/usb/keyfile
    cryptsetup --header /mnt/header.img --key-file=/dev/mapper/lukskey --keyfile-offset=0 --keyfile-size=1M open /dev/disk/by-id/harddrive enc
    cryptsetup close lukskey
    umount /mnt
}
```

Після цього виконай `mkinitcpio -p linux`, щоб оновити `initramfs`.

---

## 10. Додавання кількох способів розшифрування LUKS
LUKS підтримує до 8 слотів ключів. Ти можеш додати кілька способів розшифрування:

### 10.1. Додавання пароля
```sh
cryptsetup luksAddKey /dev/disk/by-id/harddrive
```

### 10.2. Додавання ключового файлу
```sh
cryptsetup luksAddKey /dev/disk/by-id/harddrive /path/to/new_keyfile
```

### 10.3. Перевірка зайнятих слотів
```sh
cryptsetup luksDump /dev/disk/by-id/harddrive
```
Ця команда покаже список доступних слотів і які з них зайняті.

Тепер твоя флешка готова для використання! 🎉

