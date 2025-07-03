## Servidor de Base de Datos y RAID 5

### Rol en la Arquitectura

El servidor de base de datos es responsable de almacenar la información persistente de la aplicación. Para garantizar tolerancia a fallos a nivel de almacenamiento, se configuró un arreglo RAID 5 con tres discos virtuales.  
Se utilizó **MariaDB** como motor de base de datos. El servidor solo acepta conexiones desde App1 y App2.

---

### Especificaciones

- **Hostname:** basededatos
- **IP:** 192.168.1.40
- **Puerto SSH:** 2225
- **Puerto DB:** 3306 (MariaDB)
- **Usuarios permitidos:** app1 (192.168.1.20), app2 (192.168.1.30)

---

### Pasos de instalación y configuración

#### 1. Instalación de MariaDB

```bash
sudo apt update
sudo apt install mariadb-server -y
```

#### 2. Configuración básica de la base de datos

- **Crear base de datos y usuario (ejecutar en el cliente mysql):**
```sql
CREATE DATABASE miappdb;
CREATE USER 'app1'@'192.168.1.20' IDENTIFIED BY 'app1';
CREATE USER 'app2'@'192.168.1.30' IDENTIFIED BY 'app2';
GRANT ALL PRIVILEGES ON miappdb.* TO 'app1'@'192.168.1.20';
GRANT ALL PRIVILEGES ON miappdb.* TO 'app2'@'192.168.1.30';
FLUSH PRIVILEGES;
```

- **Editar configuración para aceptar conexiones remotas:**
    ```
    /etc/mysql/mariadb.conf.d/50-server.cnf
    bind-address = 0.0.0.0
    ```
  - Reinicia el servicio:
    ```bash
    sudo systemctl restart mariadb
    ```

- **Permitir solo conexiones desde las IPs de App1 y App2 en el firewall:**
    ```bash
    sudo ufw allow from 192.168.1.20 to any port 3306 proto tcp
    sudo ufw allow from 192.168.1.30 to any port 3306 proto tcp
    sudo ufw allow 2225/tcp
    sudo ufw enable
    ```

---

### RAID 5 - Configuración y Recuperación

#### 1. Crear los discos virtuales

En VirtualBox, añade tres discos adicionales a la VM `basededatos`.

#### 2. Detectar los discos (en la VM)

```bash
lsblk
```
Por ejemplo: `/dev/sdb /dev/sdc /dev/sdd`

#### 3. Crear el arreglo RAID 5

```bash
sudo apt install mdadm -y
sudo mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd
```

#### 4. Formatear y montar el RAID

```bash
sudo mkfs.ext4 /dev/md0
sudo mkdir /mnt/raid5
sudo mount /dev/md0 /mnt/raid5
```

#### 5. Mover los datos de MariaDB al RAID

```bash
sudo systemctl stop mariadb
sudo mv /var/lib/mysql /mnt/raid5/
sudo ln -s /mnt/raid5/mysql /var/lib/mysql
sudo chown -R mysql:mysql /mnt/raid5/mysql
sudo systemctl start mariadb
```

---

### Simulación de Falla y Recuperación RAID 5

#### a) Simular la falla de un disco

```bash
sudo mdadm --fail /dev/md0 /dev/sdb
sudo mdadm --remove /dev/md0 /dev/sdb
```

#### b) Comprobar estado del RAID

```bash
cat /proc/mdstat
```

#### c) Reemplazar disco (simular agregando un nuevo disco, por ejemplo /dev/sde):

- Añade el disco en VirtualBox, luego dentro de la VM:
```bash
sudo mdadm --add /dev/md0 /dev/sde
```

#### d) Monitorear la reconstrucción

```bash
watch cat /proc/mdstat
```
Cuando termine, el RAID volverá a estar activo y tolerante a fallos.

---

