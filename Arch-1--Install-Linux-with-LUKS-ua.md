# Arch 1. Встановлення Arch Linux + LUKS

### Підключення до Інтернету

Якщо мережа не доступна і використовується Wi-Fi

```shell
iwctl
# Усередині "iwctl":
device list # перевір, який інтерфейс (зазвичай wlan0)
station wlan0 scan # сканування мереж
station wlan0 get-networks # список доступних мереж
station wlan0 connect "SSID" # підключення до Wi-Fi
```

Перевіряємо підключення:

```bash
ping -c 3 archlinux.org
```

### Розмітка диску

```bash
# Визнач свій диск
lsblk

# Припустимо, що диск — `/dev/nvme0n1`

# Знищюємо всі дані на диску
wipefs --all /dev/nvme0n1 
sgdisk --zap-all /dev/nvme0n1

# Створюємо GPT-розділи
parted /dev/nvme0n1 mklabel gpt
parted /dev/nvme0n1 mkpart "EFI" fat32 1MiB 512MiB
parted /dev/nvme0n1 set 1 esp on
parted /dev/nvme0n1 mkpart "LUKS" 512MiB 100%

# Створюємо LUKS-контейнер
cryptsetup luksFormat --typeluks2 /dev/nvme0n1p2
# Відкриваємо LUKS-контейнер
cryptsetup open /dev/nvme0n1p2 cryptroot

# Створюємо LVM-групу та логічні томи
pvcreate /dev/mapper/cryptroot
vgcreate vg0 /dev/mapper/cryptroot
lvcreate -L 150G vg0 -n root
lvcreate -L 500G vg0 -n vms # для віртуальних машин
lvcreate -L 64G vg0 -n swap 
lvcreate -l 100%FREE-50G vg0 -n home # залишаємо 50G під LVM снапшоти
# Перевіряємо вільне місце у VG
vgdisplay vg0

# Форматуємо розділи
mkfs.vfat -F32 /dev/nvme0n1p1
mkfs.ext4 /dev/vg0/root
mkfs.xfs /dev/vg0/vms
mkfs.ext4 /dev/vg0/home
mkswap /dev/vg0/swap

# Монтуємо розділи
mount /dev/vg0/root /mnt
# /boot знаходиться всередині /
# тому воно створиться автоматично 
# після монтування /dev/vg0/root у /mnt
# /var/lib/libvirt/images - стандартна директорія для VM у KVM/QEMU
mkdir -p /mnt/{efi,home,var/lib/libvirt/images}
mount /dev/vg0/home /mnt/home
mount /dev/vg0/vms /mnt/var/lib/libvirt/images
mount --mkdir /dev/nvme0n1p1 /mnt/efi
swapon /dev/vg0/swap
# Перевіряємо, що в нас вийшло :)
lsblk -f
# Перевірити розміри томів
lvs
vgs
```

### Встановлення системи

```bash
# Встановлюємо базову систему
pacstrap /mnt base base-devel linux linux-firmware linux-headers bash-completion vim nano git curl neofetch networkmanager lvm2

# Якщо потрібні пропрієтарні NVIDIA-драйвери
pacstrap /mnt nvidia-dkms nvidia-utils nvidia-settings

# Налаштування fstab
genfstab -U /mnt >> /mnt/etc/fstab

# Chroot в нову систему
arch-chroot /mnt
```

### Налаштування LUKS та LVM

Додамо LUKS у `mkinitcpio.conf`

```bash
vim /etc/mkinitcpio.conf

# У `HOOKS=()`, змінюємо на
HOOKS=(base udev autodetect modconf block keyboard keymap encrypt lvm2 filesystems fsck)

# Зберігаємо файл і оновлюємо `initramfs`
mkinitcpio -P
```

### Встановлення GRUB із підтримкою LUKS

```bash
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB --modules="cryptodisk luks gzio part_gpt"

# Редагуємо GRUB-конфіг
sed -i "s|^GRUB_CMDLINE_LINUX=\".*\"|GRUB_CMDLINE_LINUX=\"cryptdevice=UUID=$(blkid -s UUID -o value /dev/nvme0n1p2):cryptroot root=/dev/mapper/vg0-root\"|" /etc/default/grub

# Перевіряємо, що заміна пройшла успішно
grep "GRUB_CMDLINE_LINUX" /etc/default/grub

# Генеруємо GRUB-конфіг
grub-mkconfig -o /boot/grub/grub.cfg
```

### Завершальні налаштування

```bash
# Встановлюємо час
ln -sf /usr/share/zoneinfo/Europe/Kyiv /etc/localtime
hwclock --systohc

# Встановлюємо локаль
nano /etc/locale.gen # розкоментувати en_US.UTF-8 UTF-8
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

# Налаштування мережі
echo "arch" > /etc/hostname

# Включаємо `NetworkManager`
systemctl enable NetworkManager

# Додаємо пароль root
passwd

# Додаємо користувача `user`
useradd -m -G wheel -s /bin/bash user
passwd user
pacman -S sudo
# Редагуємо файл `/etc/sudoers`
EDITOR=vim visudo
# Розкоментовуємо рядок
%wheel ALL=(ALL:ALL) ALL

# Вихід і перезавантаження
exit
umount -R /mnt
swapoff -a
reboot
```
