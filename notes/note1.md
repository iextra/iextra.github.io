# 🖥️ Монтирование папки с MacBook в Ubuntu для работы с Docker

## 📋 Задача
- **PC с Ubuntu** (IP: `192.168.1.100`)  
- **MacBook** (IP: `192.168.1.50`)


Необходимо:  
1. Смонтировать папку с MacBook в Ubuntu  
2. Прокинуть эту папку в Docker на Ubuntu  

---

## 🛠️ 1. Настройка NFS-сервера (Ubuntu)

### 1.1. Установка NFS
```bash
sudo apt update && sudo apt install -y nfs-kernel-server
```

### 1.2. Настройка экспорта папки
Отредактируйте `/etc/exports`:
```bash
sudo nano /etc/exports
```
Добавьте строку:
```plaintext
/home/user/docker-projects/project.loc/app 192.168.1.50(rw,sync,no_subtree_check,no_root_squash)
```

### 1.3. Применение изменений
```bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

### 1.4. Проверка
```bash
showmount -e localhost
```
Ожидаемый вывод:
```plaintext
Export list for localhost:
/home/user/docker-projects/project.loc/app 192.168.1.50
```

---

## 🍏 2. Настройка NFS-клиента (MacBook)

### 2.1. Активация NFS-клиента
```bash
sudo nfsd enable
```

### 2.2. Создание точки монтирования
```bash
mkdir -p /macbook/user/projects/project.loc/app
```

### 2.3. Монтирование NFS
```bash
sudo mount -t nfs -o resvport,rw 192.168.1.100:/home/user/docker-projects/project.loc/app /macbook/user/projects/project.loc/app
```

### 2.4. Проверка
```bash
ls /macbook/user/projects/project.loc/app
```

---

## 🔄 Автомонтирование (MacBook)
Добавьте запись в `/etc/fstab`, чтобы папка монтировалась при старте системы.  

1. Откройте файл `/etc/fstab`:  
   ```bash
   sudo nano /etc/fstab
   ```  
2. Добавьте строку:  
   ```
   192.168.1.100:/ubuntu/user/docker-projects/project.loc/app  /macbook/user/projects/project.loc/app  nfs  rw,resvport,nolocks,noatime,soft,bg  0  0
   ```  
   - `soft` – не "зависать" при проблемах с сетью.  
   - `bg` – монтировать в фоне, чтобы не блокировать загрузку.  

3. Сохраните (`Ctrl+O`, `Enter`) и закройте (`Ctrl+X`).  

4. Проверьте, работает ли:  
   ```bash
   sudo mount -a
   ```  
   Если ошибок нет – после перезагрузки папка подключится сама.  

---

## 🔒 Решение проблем с правами
Если возникает ошибка **Read-Only**:

1. Узнайте UID на Mac:
   ```bash
   id -u
   ```
   (Например, `501`).  

2. На Ubuntu сделайте этого пользователя владельцем: 
   ```bash
   sudo chown -R 501:501 /ubuntu/user/docker-projects/project.loc/app
   ```
  (Замените `501` на ваш UID с Mac). 

---

## 🐳 Интеграция с Docker
В `docker-compose.yml` укажите монтированную папку:
```yaml
volumes:
  - /ubuntu/user/docker-projects/project.loc/app:/var/www/html
```

---

Теперь можно редактировать код в PhpStorm на MacBook, и он будет сразу доступен в Docker на Ubuntu! 🚀
