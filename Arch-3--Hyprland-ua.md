# Встановлення Hyprland

Для встановлення Hyprland спочатку треба встановити драйвери для графіки, сам Hyprland і необхідні залежності.

### Підготовка

```bash
# 1. Видалення стандартного драйвера NVIDIA
sudo pacman -Rns nvidia nvidia-utils nvidia-settings

# 2. Встановлення nvidia-dkms
sudo pacman -S nvidia-dkms nvidia-utils nvidia-settings

# 3. Встановимо mesa, бо Wayland використовує загальні бібліотеки рендерингу
sudo pacman -S mesa mesa-utils

# 4. Переконайся, що модуль nvidia-drm завантажується з modeset=1 для Wayland:
echo "options nvidia-drm modeset=1" | sudo tee /etc/modprobe.d/nvidia.conf
```

### 1. Встановлення Hyprland та необхідного ПО

```bash
sudo pacman -S hyprland waybar wofi mako xdg-desktop-portal-hyprland qt5-wayland qt6-wayland \
                pipewire pipewire-pulse wireplumber swaylock swayidle grim slurp
```

Це:

`hyprland` – сам WM

`waybar` – панель статусу

`wofi` – меню запуску програм

`mako` – сповіщення

`xdg-desktop-portal-hyprland` – для інтеграції Wayland із додатками

`pipewire` та `wireplumber` – для аудіо

`swaylock` та `swayidle` – для блокування екрану

`grim` і `slurp` – для скріншотів

### 2. Налаштування логіну через GDM

Якщо хочеш GUI-логін, то найкраще GDM, бо він підтримує Wayland:

```bash
sudo pacman -S gdm
sudo systemctl enable gdm
```

### 3. Створення базового конфіга для Hyprland

Зробимо стандартну конфігурацію:

```bash
mkdir -p ~/.config/hypr
cp /usr/share/hyprland/hyprland.conf ~/.config/hypr/hyprland.conf
```

Якщо хочеш стартувати Hyprland без дисплейного менеджера, додай це в `~/.xinitrc`:

```bash
exec Hyprland
```

І стартуй через:

```bash
startx
```

Тепер можеш перезапустити систему і заходити в Hyprland.
