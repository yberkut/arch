Окей, для встановлення Hyprland спочатку треба встановити драйвери для графіки, сам Hyprland і необхідні залежності.

### 1. Встановлення драйверів

Якщо в тебе ноутбук з RTX 30XX, тобі потрібні пропрієтарні драйвери NVIDIA. Використаємо nvidia-dkms для кращої інтеграції з ядром:

```bash
sudo pacman -S nvidia-dkms nvidia-utils nvidia-settings nvidia-vaapi-driver
```

Також встановимо mesa і lib32-mesa, бо Wayland використовує загальні бібліотеки рендерингу:

```bash
sudo pacman -S mesa lib32-mesa
```

Переконайся, що модуль nvidia_drm завантажується з modeset=1 для Wayland:

```bash
echo "options nvidia_drm modeset=1" | sudo tee /etc/modprobe.d/nvidia.conf
```

### 2. Встановлення Hyprland та необхідного ПО

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

### 3. Налаштування логіну через GDM

Якщо хочеш GUI-логін, то найкраще GDM, бо він підтримує Wayland:

```bash
sudo pacman -S gdm
sudo systemctl enable gdm
```

### 4. Створення базового конфіга для Hyprland

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
