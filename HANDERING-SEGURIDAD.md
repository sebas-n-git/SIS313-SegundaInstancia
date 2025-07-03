## Hardening y Seguridad

La seguridad de la infraestructura fue reforzada aplicando las siguientes medidas de hardening en cada uno de los servidores críticos (balanceador, app1, app2, basededatos):

### 1. SSH Seguro

- **Cambio de puerto por defecto:**  
  Cada servidor tiene configurado un puerto SSH personalizado (2222 en balanceador, 2223 en app1, 2224 en app2, 2225 en basededatos) para evitar ataques automatizados al puerto 22.
- **Autenticación por clave pública:**  
  Solo es posible autenticarse mediante llaves SSH. La autenticación por contraseña fue deshabilitada:
  ```bash
  # En /etc/ssh/sshd_config
  PasswordAuthentication no
  ```
- **Deshabilitar acceso root:**  
  El acceso directo como root vía SSH está deshabilitado:
  ```bash
  # En /etc/ssh/sshd_config
  PermitRootLogin no
  ```
- **Reinicio del servicio SSH tras cambios:**
  ```bash
  sudo systemctl restart ssh
  ```

### 2. Firewall (UFW)

- **Política restrictiva:**  
  Se configuró UFW para denegar todo el tráfico entrante por defecto y permitir solo los puertos necesarios:
  ```bash
  sudo ufw default deny incoming
  sudo ufw default allow outgoing
  ```
- **Reglas específicas:**
  - **Balanceador:** solo puertos 80/tcp (HTTP) y 2222/tcp (SSH)
  - **app1:** solo puertos 2223/tcp (SSH) y 3000/tcp (HTTP desde balanceador)
  - **app2:** solo puertos 2224/tcp (SSH) y 3001/tcp (HTTP desde balanceador)
  - **basededatos:** solo puertos 2225/tcp (SSH) y 3306/tcp (solo desde 192.168.1.20 y 192.168.1.30)
- **Ejemplo de reglas en cada VM:**
  ```bash
  # Balanceador
  sudo ufw allow 80/tcp
  sudo ufw allow 2222/tcp

  # App1
  sudo ufw allow 3000/tcp
  sudo ufw allow 2223/tcp

  # App2
  sudo ufw allow 3001/tcp
  sudo ufw allow 2224/tcp

  # Base de datos
  sudo ufw allow from 192.168.1.20 to any port 3306 proto tcp
  sudo ufw allow from 192.168.1.30 to any port 3306 proto tcp
  sudo ufw allow 2225/tcp
  ```

### 3. Servicios Esenciales

- Solo se mantienen activos servicios esenciales en cada servidor.
- Se revisó el estado de los servicios con:
  ```bash
  sudo systemctl list-units --type=service --state=running
  ```
- Se deshabilitaron (y/o detuvieron) servicios innecesarios con:
  ```bash
  sudo systemctl disable <servicio>
  sudo systemctl stop <servicio>
  ```

### 4. Seguridad Web Básica en Nginx

- Nginx solo escucha en el puerto 80.
- Se validó que no hay servidores de prueba o configuraciones inseguras habilitadas.
- Se agregó el encabezado `X-Frame-Options` para evitar clickjacking:
  ```nginx
  add_header X-Frame-Options "SAMEORIGIN";
  ```

---
