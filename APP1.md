## Sección: App1 (VM2)

---

### Instalación de Node.js y dependencias

```bash
sudo apt update
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

Inicializa la aplicación:

```bash
mkdir ~/app
cd ~/app
npm init -y
npm install express mysql2
```

---

### Crear el archivo `index.js`

```bash
nano index.js
```

Pega este código:

```js
const express = require('express');
const mysql = require('mysql2');
const app = express();

app.use(express.json());

const db = mysql.createConnection({
  host: '192.168.1.40',
  user: 'usuario_app',
  password: 'clave_segura',
  database: 'proyecto',
  port: 3306
});

db.connect(err => {
  if (err) throw err;
  console.log('📦 Conectado a la base de datos');
});

app.get('/', (req, res) => {
  res.send('👋 Hola desde App1!');
});

app.get('/usuarios', (req, res) => {
  db.query('SELECT * FROM usuarios', (err, results) => {
    if (err) throw err;
    res.json(results);
  });
});

app.listen(3000, () => {
  console.log('🚀 App1 corriendo en puerto 3000');
});
```

---

### ▶ Ejecutar la aplicación

```bash
node index.js
```

O con PM2 (opcional):

```bash
sudo npm install -g pm2
pm2 start index.js --name app1
pm2 save
pm2 startup
```

---

### Configurar seguridad y puerto SSH 2223

#### Cambiar puerto SSH:

```bash
sudo nano /etc/ssh/sshd_config
# Cambiar:
# Port 22 → Port 2223
sudo systemctl restart ssh
```

#### Permitir puertos en UFW:

```bash
sudo ufw allow 3000/tcp
sudo ufw allow 2223/tcp
```

---

