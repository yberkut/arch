# Arch 2.3. Restic для бекапів на HDD (NTFS)

## Передумови

Встановлення `restic`:

```bash
sudo pacman -S restic
```

Монтуємо HDD (NTFS):
[Arch 2.1. Монтування зовнішнього HDD (NTFS) для бекапів](https://github.com/yberkut/arch/blob/main/Arch-2-1--Mount-external-HDD-for-Backups-ua.md)

Ініціалізація репозиторію

```bash
restic init --repo /mnt/backup/restic-repo
```

**Важливо!** restic попросить ввести пароль, який потім знадобиться для відновлення.

---

## Швидкий референс

[Restic CheatSheet](https://github.com/yberkut/arch/blob/main/Restic-CheatSheet-ua.md)

---

### 1. Створення бекапів

#### Повний бекап (`/` крім зайвих папок) з тегом `full`

```bash
restic backup / --repo /mnt/backup/restic-repo --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found","/swapfile"} --tag full
```

✅ **Що тут відбувається?**

- `backup /` → Копіює всю систему.
- `--exclude={...}` → Пропускає динамічні директорії (вони не потрібні для відновлення).
- `--tag` дозволяє позначати бекапи мітками, щоб потім їх зручно шукати або видаляти.

#### Інкрементальний бекап

`restic` автоматично відстежує зміни, тому просто запускаємо ту ж команду:

```bash
restic backup / --repo /mnt/backup/restic-repo --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found","/swapfile"} --tag full
```

💡 **restic зберігає тільки зміни**, тож наступні запуски будуть швидші.

---

### 2. Управління снапшотами

#### Перегляд всіх бекапів

```bash
restic snapshots --repo /mnt/backup/restic-repo
```

#### Видалення старих бекапів

Наприклад, залишити тільки 3 останніх:

```bash
restic forget --keep-last 3 --prune --repo /mnt/backup/restic-repo
```

✅ **`--prune` видаляє зайві файли та звільняє місце**.

---

### 3. Відновлення бекапу

#### Доступ до файлів без відновлення

Можеш змонтувати бекап і переглядати файли:

```bash
mkdir /mnt/restic-mount
restic mount /mnt/restic-mount --repo /mnt/backup/restic-repo
```

Тепер у `/mnt/restic-mount/` будуть всі доступні бекапи.

#### Повне відновлення (`/`)

⚠️ **Важливо! Перед відновленням завантажся в live-режим!**

```bash
restic restore latest --target /mnt/root --repo /mnt/backup/restic-repo
```

Якщо потрібно відновити конкретний снапшот:

```bash
restic restore [SNAPSHOT_ID] --target /mnt/root --repo /mnt/backup/restic-repo
```

---

### Висновок

- **Повний бекап:** `restic backup /`
- **Інкрементальний:** робиться автоматично при повторному запуску.
- **Перевірка бекапів:** `restic snapshots`
- **Очищення:** `restic forget --keep-last 3 --prune`
- **Відновлення:** `restic restore latest --target /mnt/root`

---

Це дає **повну історію змін** і дозволяє **повністю відновити систему або окремі файли**.
