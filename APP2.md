## Sección: App2 (VM3)

(Es prácticamente igual, con ajustes de IP y puerto SSH)

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

Código con mensaje distinto:

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
  res.send('👋 Hola desde App2!');
});

app.get('/usuarios', (req, res) => {
  db.query('SELECT * FROM usuarios', (err, results) => {
    if (err) throw err;
    res.json(results);
  });
});

app.listen(3000, () => {
  console.log('🚀 App2 corriendo en puerto 3000');
});
```

---

### Ejecutar la aplicación

```bash
node index.js
```

O con PM2:

```bash
sudo npm install -g pm2
pm2 start index.js --name app2
pm2 save
pm2 startup
```

---

### Configurar seguridad y puerto SSH 2224

#### Cambiar puerto SSH:

```bash
sudo nano /etc/ssh/sshd_config
# Port 22 → Port 2224
sudo systemctl restart ssh
```

#### Permitir puertos en UFW:

```bash
sudo ufw allow 3000/tcp
sudo ufw allow 2224/tcp
```

---

## Verificación desde balanceador

Desde VM1 (balanceador):

```bash
curl http://192.168.1.20:3000/
curl http://192.168.1.30:3000/
```

Desde navegador (host):

```
http://localhost:8080/
```

Deberías ver alternar `"Hola desde App1!"` y `"Hola desde App2!"`.

---

