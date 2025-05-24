Задача:
Есть 2 компа, PС с Ubuntu (192.168.1.100) и MacBook (192.168.1.50)
Нужно папку на MacBook монтировать в Ubuntu, а на убунту запустить Docker и прокинуть папку туда

### **1. Настройка общего доступа на Ubuntu (NFS сервер)**
#### **1.1. Установка NFS-сервера**
```bash
sudo apt update
sudo apt install nfs-kernel-server
```

#### **1.2. Настройка экспорта папки `/home/user/docker-projects/project.loc/app`**
Отредактируйте `/etc/exports`:
```bash
sudo nano /etc/exports
```
Добавьте строку (разрешая доступ с MacBook):
```
/macbook/user/projects/project.loc/app 192.168.1.50(rw,sync,no_subtree_check,no_root_squash)
```

#### **1.3. Применяем настройки**
```bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

#### **1.4. Проверка**
```bash
showmount -e localhost
```
Должно вывести:
```
Export list for localhost:
/ubuntu/user/docker-projects/project.loc/app 192.168.1.50
```

---

### **2. Настройка клиента на MacBook (монтирование NFS)**
#### **2.1. Включите NFS-клиент**
```bash
sudo nfsd enable
```

#### **2.2. Создайте точку монтирования (если её нет)**
```bash
mkdir -p /macbook/user/projects/project.loc/app
```

#### **2.3. Монтируем NFS-шару**
```bash
sudo mount -t nfs -o resvport,rw 192.168.1.100:/ubuntu/user/docker-projects/project.loc/app /macbook/user/projects/project.loc/app
```
(Если нужен автоматический маунт при загрузке, добавьте в `/etc/fstab`)

#### **2.4. Проверка**
```bash
ls /macbook/user/projects/project.loc/app
```
Теперь изменения на MacBook (`/macbook/user/projects/project.loc/app`) будут сразу видны на Ubuntu (`/ubuntu/user/docker-projects/project.loc/app`).

---

Если при редактировании файлов через NFS вы получаете ошибку **"Read-Only"**

### **3. Настройка прав**
- Узнайте ваш **UID на Mac**:  
  ```bash
  id -u
  ```
  (Например, `501`).  

- На Ubuntu сделайте этого пользователя владельцем:  
  ```bash
  sudo chown -R 501:501 /ubuntu/user/docker-projects/project.loc/app
  ```
  (Замените `501` на ваш UID с Mac).  

---

После перезагрузки компьютеров NFS-монтирование **сбросится**, и нужно будет **смонтировать шару заново**. Но это легко решается автоматизацией!  

---

### **1. Настройка автоматического монтирования NFS при загрузке**  

#### **На Mac (клиент)**:  
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

#### **На Ubuntu (NFS-сервер)**:  
1. Убедитесь, что NFS-сервер запускается автоматически:  
   ```bash
   sudo systemctl enable nfs-kernel-server
   ```  
2. Проверьте, что `/etc/exports` сохранился (его не нужно править заново).  

---

Теперь можно редактировать код в PhpStorm на MacBook, и он будет сразу доступен в Docker на Ubuntu! 🚀
