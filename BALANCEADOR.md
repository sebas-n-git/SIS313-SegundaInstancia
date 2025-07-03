###  Instalación de Nginx

```bash
sudo apt update
sudo apt install nginx -y
```

---

###  Configuración del balanceo de carga

Edita el archivo por defecto de Nginx:

```bash
sudo nano /etc/nginx/sites-available/default
```


```nginx
upstream app_servers {
    server 192.168.1.20:3000;
    server 192.168.1.30:3000;
}

server {
    listen 80;

    location / {
        proxy_pass http://app_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

### Probar configuración y reiniciar Nginx

```bash
sudo nginx -t
sudo systemctl restart nginx
```

---

###  Hardening básico del servidor 

#### Cambiar puerto SSH 

```bash
sudo nano /etc/ssh/sshd_config
# Cambiar Port 22 → Port 2222
sudo systemctl restart ssh
```

#### Activar UFW y permitir puertos necesarios

```bash
sudo ufw allow 80/tcp
sudo ufw allow 2222/tcp     
sudo ufw enable
```

---

### Verificación desde la VM

```bash
curl http://localhost/
```


