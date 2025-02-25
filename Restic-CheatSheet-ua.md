# Restic CheatSheet

### 1. Ініціалізація репозиторію

```bash
restic init -r /шлях/до/репозиторію
```

**Приклад:**

```bash
restic init -r /mnt/backup/restic-repo
```

### 2. Створення бекапу

#### Повний бекап

```bash
restic backup /шлях/до/даних -r /шлях/до/репозиторію --tag full
```

**Приклад:**

```bash
restic backup /home/user -r /mnt/backup/restic-repo --tag full
```

#### Інкрементальний бекап (автоматично)

```bash
restic backup /шлях/до/даних -r /шлях/до/репозиторію --tag incremental
```

**Приклад:**

```bash
restic backup /home/user/Documents -r /mnt/backup/restic-repo --tag incremental
```

#### Виключення тимчасових файлів і swap

```bash
restic backup / --exclude={/dev,/proc,/sys,/tmp,/run,/mnt,/media,/lost+found,/swapfile} -r /шлях/до/репозиторію --tag full
```

**Приклад:**

```bash
restic backup / --exclude={/dev,/proc,/sys,/tmp,/run,/mnt,/media,/lost+found,/swapfile} -r /mnt/backup/restic-repo --tag full
```

### 3. Перевірка бекапів

```bash
restic snapshots -r /шлях/до/репозиторію
```

**Приклад:**

```bash
restic snapshots -r /mnt/backup/restic-repo
```

### 4. Відновлення даних

#### Відновлення всього

```bash
restic restore latest -r /шлях/до/репозиторію --target /шлях/до/відновлення
```

**Приклад:**

```bash
restic restore latest -r /mnt/backup/restic-repo --target /
```

#### Відновлення конкретного снапшоту

```bash
restic restore [snapshot-ID] -r /шлях/до/репозиторію --target /шлях/до/відновлення
```

**Приклад:**

```bash
restic restore 3f2b9d0d -r /mnt/backup/restic-repo --target /home/user/recovery
```

### 5. Видалення старих бекапів

```bash
restic forget --keep-last 3 --prune -r /шлях/до/репозиторію
```

**Приклад:**

```bash
restic forget --keep-last 3 --prune -r /mnt/backup/restic-repo
```

### 6. Перевірка цілісності репозиторію

```bash
restic check -r /шлях/до/репозиторію
```

**Приклад:**

```bash
restic check -r /mnt/backup/restic-repo
```

### 7. Видалення конкретного снапшоту

```bash
restic forget [snapshot-ID] -r /шлях/до/репозиторію --prune
```

**Приклад:**

```bash
restic forget 3f2b9d0d -r /mnt/backup/restic-repo --prune
```
