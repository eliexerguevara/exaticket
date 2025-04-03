![Exacom Logo](https://somosexacom.com/wp-content/uploads/2024/05/cropped-logo-Transparente-2.png)

# exacticket!

**NOTA**: La nueva versión de whatsapp-web.js requiere Node 14. Actualice su instalación para seguir usándolo.

Un sistema de tickets _muy sencillo_ basado en mensajes de WhatsApp.

El backend usa whatsapp-web.js para recibir y enviar mensajes de WhatsApp, crear tickets a partir de ellos y almacenar todo en una base de datos MySQL.

El frontend es una app de chat multiusuario completa, con funciones iniciadas con react-create-app y Material UI, que se comunica con el backend mediante REST API y Websockets. Permite interactuar con contactos, gestionar tickets, y enviar y recibir mensajes de WhatsApp.

**NOTA**: No puedo garantizar que no se bloquee su cuenta al usar este método, aunque a mí me ha funcionado. WhatsApp no ​​permite bots ni clientes no oficiales en su plataforma, por lo que no debería considerarse totalmente seguro.

## ¿Cómo funciona?

Con cada mensaje nuevo recibido en un WhatsApp asociado, se crea un ticket. Este ticket se puede acceder a él en una cola en la página Tickets, donde puedes asignarlo a ti mismo aceptándolo, responder al mensaje y, finalmente, resolverlo.

Los mensajes posteriores del mismo contacto se relacionarán con el primer ticket **abierto/pendiente** encontrado.

Si un contacto envía un mensaje nuevo en menos de 2 horas y no hay ningún ticket de este contacto con estado **pendiente/abierto**, se reabrirá el ticket **cerrado** más reciente, en lugar de crear uno nuevo.

## Capturas de pantalla

![](https://github.com/canove/exacticket/raw/master/images/exacticket-queues.gif)
<img src="https://github.com/eliexerguevara/exaticket/blob/master/images/chat2.png" width="350"> <img src="https://github.com/eliexerguevara/exaticket/blob/master/images/chat3.png" width="350"> <img src="https://github.com/eliexerguevara/exaticket/blob/master/images/multiple-whatsapps2.png" width="350"> <img src="https://github.com/eliexerguevara/exaticket/blob/master/images/contacts1.png" width="350">

## Características

- Permite que varios usuarios chateen en el mismo número de WhatsApp ✅
- Conecta varias cuentas de WhatsApp y recibe todos los mensajes en un solo lugar ✅ 🆕
- Crea y chatea con nuevos contactos sin tocar el celular ✅
- Envía y recibe mensajes ✅
- Envía archivos multimedia (imágenes, audio, documentos) ✅
- Recibe archivos multimedia (imágenes, audio, video, documentos) ✅

Instalación y uso (Linux Ubuntu - Desarrollo)

Crear una base de datos MySQL con Docker:
Nota: Modificar MYSQL_DATABASE, MYSQL_PASSWORD, MYSQL_USER y MYSQL_ROOT_PASSWORD.

```bash
docker run --name exacticketdb -e MYSQL_ROOT_PASSWORD=strongpassword -e MYSQL_DATABASE=exacticket -e MYSQL_USER=exacticket -e MYSQL_PASSWORD=exacticket --restart always -p 3306:3306 -d mariadb:latest --character-set-server=utf8mb4 --collation-server=utf8mb4_bin

# Or run using `docker-compose` as below
# Before copy .env.example to .env first and set the variables in the file.
docker-compose up -d mysql

# To administer this mysql database easily using phpmyadmin. 
# It will run by default on port 9000, but can be changed in .env using `PMA_PORT`
docker-compose -f docker-compose.phpmyadmin.yaml up -d
```

Instalar las dependencias de puppeteer:

```bash
sudo apt-get install -y libxshmfence-dev libgbm-dev wget unzip fontconfig locales gconf-service libasound2 libatk1.0-0 libc6 libcairo2 libcups2 libdbus-1-3 libexpat1 libfontconfig1 libgcc1 libgconf-2-4 libgdk-pixbuf2.0-0 libglib2.0-0 libgtk-3-0 libnspr4 libpango-1.0-0 libpangocairo-1.0-0 libstdc++6 libx11-6 libx11-xcb1 libxcb1 libxcomposite1 libxcursor1 libxdamage1 libxext6 libxfixes3 libxi6 libxrandr2 libxrender1 libxss1 libxtst6 ca-certificates fonts-liberation libappindicator1 libnss3 lsb-release xdg-utils
```

Clone this repo

```bash
git clone https://github.com/eliexerguevara/exaticket
```

Vaya a la carpeta backend y cree un archivo .env:

```bash
cp .env.example .env
nano .env
```

Llene el archivo `.env` con variables de entorno:

```bash
NODE_ENV=DEVELOPMENT      #it helps on debugging
BACKEND_URL=http://localhost
FRONTEND_URL=https://localhost:3000
PROXY_PORT=8080
PORT=8080

DB_HOST=                  #DB host IP, usually localhost
DB_DIALECT=
DB_USER=
DB_PASS=
DB_NAME=

JWT_SECRET=3123123213123
JWT_REFRESH_SECRET=75756756756
```

Instalar dependencias de backend, compilar la aplicación, ejecutar migraciones y semillas:

```bash
npm install
npm run build
npx sequelize db:migrate
npx sequelize db:seed:all
```

Iniciar backend:

```bash
npm start
```

Abra una segunda terminal, vaya a la carpeta frontend y cree el archivo .env:

```bash
nano .env
REACT_APP_BACKEND_URL = http://localhost:8080/ # Your previous configured backend app URL.
```

Iniciar frontend app:

```bash
npm start
```
- Ve a http://your_server_ip:3000/signup
- Crea un usuario e inicia sesión con él.
- En la barra lateral, ve a la página _Conexiones_ y crea tu primera conexión de WhatsApp.
- Espera a que aparezca el botón del código QR, haz clic en él y léelo.
- Listo. Todos los mensajes recibidos en tu número de WhatsApp sincronizado aparecerán en la lista de tickets.


## Implementación básica en producción

### Uso de VPS Ubuntu 20.04

Todas las instrucciones a continuación asumen que NO se está ejecutando como root, ya que esto generará un error en puppeteer. Así que comencemos creando un nuevo usuario y otorgándole privilegios de sudo:

```bash
adduser deploy
usermod -aG sudo deploy
```

Ahora podemos iniciar sesión con este nuevo usuario:

```bash
su deploy
```

Necesitará dos subdominios que redireccionen a la IP de su VPS para seguir estas instrucciones. En el siguiente ejemplo, usaremos `myapp.mydomain.com` como frontend y `api.mydomain.com` como backend.

Actualice todos los paquetes del sistema:

```bash
sudo apt update && sudo apt upgrade
```

Instalar el nodo y confirmar que el comando del nodo esté disponible:

```bash
curl -fsSL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt-get install -y nodejs
node -v
npm -v
```

Instale Docker y agregue su usuario al grupo Docker:

```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt update
sudo apt install docker-ce
sudo systemctl status docker
sudo usermod -aG docker ${USER}
su - ${USER}
```

Crear una base de datos MySQL usando Docker:
_Note_: change MYSQL_DATABASE, MYSQL_PASSWORD, MYSQL_USER and MYSQL_ROOT_PASSWORD.

```bash
docker run --name exacticketdb -e MYSQL_ROOT_PASSWORD=strongpassword -e MYSQL_DATABASE=exacticket -e MYSQL_USER=exacticket -e MYSQL_PASSWORD=exacticket --restart always -p 3306:3306 -d mariadb:latest --character-set-server=utf8mb4 --collation-server=utf8mb4_bin

# Or run using `docker-compose` as below
# Before copy .env.example to .env first and set the variables in the file.
docker-compose up -d mysql

# To administer this mysql database easily using phpmyadmin. 
# It will run by default on port 9000, but can be changed in .env using `PMA_PORT`
docker-compose -f docker-compose.phpmyadmin.yaml up -d
```

Clonar este repositorio:

```bash
cd ~
git clone https://github.com/canove/exacticket exacticket
```

Create backend .env file and fill with details:

```bash
cp exacticket/backend/.env.example exacticket/backend/.env
nano exacticket/backend/.env
```

```bash
NODE_ENV=
BACKEND_URL=https://api.mydomain.com      #USE HTTPS HERE, WE WILL ADD SSL LATTER
FRONTEND_URL=https://myapp.mydomain.com   #USE HTTPS HERE, WE WILL ADD SSL LATTER, CORS RELATED!
PROXY_PORT=443                            #USE NGINX REVERSE PROXY PORT HERE, WE WILL CONFIGURE IT LATTER
PORT=8080

DB_HOST=localhost
DB_DIALECT=
DB_USER=
DB_PASS=
DB_NAME=

JWT_SECRET=3123123213123
JWT_REFRESH_SECRET=75756756756
```

Instalar las dependencias de puppeteer:

```bash
sudo apt-get install -y libxshmfence-dev libgbm-dev wget unzip fontconfig locales gconf-service libasound2 libatk1.0-0 libc6 libcairo2 libcups2 libdbus-1-3 libexpat1 libfontconfig1 libgcc1 libgconf-2-4 libgdk-pixbuf2.0-0 libglib2.0-0 libgtk-3-0 libnspr4 libpango-1.0-0 libpangocairo-1.0-0 libstdc++6 libx11-6 libx11-xcb1 libxcb1 libxcomposite1 libxcursor1 libxdamage1 libxext6 libxfixes3 libxi6 libxrandr2 libxrender1 libxss1 libxtst6 ca-certificates fonts-liberation libappindicator1 libnss3 lsb-release xdg-utils
```

Instalar dependencias de backend, compilar la aplicación, ejecutar migraciones y semillas:

```bash
cd exacticket/backend
npm install
npm run build
npx sequelize db:migrate
npx sequelize db:seed:all
```

Inícielo con `npm start`. Debería ver `Servidor iniciado en el puerto...` en la consola. Presione `CTRL + C` para salir.

Install pm2 **with sudo**, and start backend with it:

```bash
sudo npm install -g pm2
pm2 start dist/server.js --name exacticket-backend
```
Hacer que pm2 se inicie automáticamente después de reiniciar:

```bash
pm2 startup ubuntu -u `YOUR_USERNAME`
```

Copia la última línea de salida del comando anterior y ejecútala, es algo como:

```bash
sudo env PATH=\$PATH:/usr/bin pm2 startup ubuntu -u YOUR_USERNAME --hp /home/YOUR_USERNAM
```

Vaya a la carpeta frontend e instale las dependencias:

```bash
cd ../frontend
npm install
```

Crea un archivo .env de frontend y complétalo SÓLO con tu dirección de backend, debería verse así:

```bash
REACT_APP_BACKEND_URL = https://api.mydomain.com/
```

Crear una aplicación frontend:

```bash
npm run build
```

Inicie la interfaz con pm2 y guarde la lista de procesos pm2 para que se inicie automáticamente después de reiniciar:

```bash
pm2 start server.js --name exacticket-frontend
pm2 save
```

Para comprobar si se está ejecutando, ejecute `pm2 list`, debería verse así:

```bash
deploy@ubuntu-whats:~$ pm2 list
┌─────┬─────────────────────────┬─────────────┬─────────┬─────────┬──────────┬────────┬──────┬───────────┬──────────┬──────────┬──────────┬──────────┐
│ id  │ name                    │ namespace   │ version │ mode    │ pid      │ uptime │ .    │ status    │ cpu      │ mem      │ user     │ watching │
├─────┼─────────────────────────┼─────────────┼─────────┼─────────┼──────────┼────────┼──────┼───────────┼──────────┼──────────┼──────────┼──────────┤
│ 1   │ exacticket-frontend      │ default     │ 0.1.0   │ fork    │ 179249   │ 12D    │ 0    │ online    │ 0.3%     │ 50.2mb   │ deploy   │ disabled │
│ 6   │ exacticket-backend       │ default     │ 1.0.0   │ fork    │ 179253   │ 12D    │ 15   │ online    │ 0.3%     │ 118.5mb  │ deploy   │ disabled │
└─────┴─────────────────────────┴─────────────┴─────────┴─────────┴──────────┴────────┴──────┴───────────┴──────────┴──────────┴──────────┴──────────┘

```

Instale nginx:

```bash
sudo apt install nginx
```

Eliminar el sitio predeterminado de nginx:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

Cree un nuevo sitio nginx para la aplicación frontend:

```bash
sudo nano /etc/nginx/sites-available/exacticket-frontend
```

Edítelo y complételo con esta información, cambiando `server_name` por el equivalente a `myapp.mydomain.com`:

```bash
server {
  server_name myapp.mydomain.com;

  location / {
    proxy_pass http://127.0.0.1:3333;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_cache_bypass $http_upgrade;
  }
}
```

Cree otra para la API de backend, cambiando `server_name` por el equivalente a `api.mydomain.com`, y `proxy_pass` por la URL del servidor del nodo de backend de su host local:

```bash
sudo cp /etc/nginx/sites-available/exacticket-frontend /etc/nginx/sites-available/exacticket-backend
sudo nano /etc/nginx/sites-available/exacticket-backend
```

```bash
server {
  server_name api.mydomain.com;

  location / {
    proxy_pass http://127.0.0.1:8080;
    ......
}
```

Crea enlaces simbólicos para habilitar sitios nginx:

```bash
sudo ln -s /etc/nginx/sites-available/exacticket-frontend /etc/nginx/sites-enabled
sudo ln -s /etc/nginx/sites-available/exacticket-backend /etc/nginx/sites-enabled
```

De forma predeterminada, nginx limita el tamaño del cuerpo a 1 MB, lo cual no es suficiente para algunas subidas de archivos multimedia. Cámbielo a 40 MB, añadiendo una nueva línea al archivo de configuración:

```bash
sudo nano /etc/nginx/nginx.conf
...
http {
    ...
    client_max_body_size 40M; # HANDLE BIGGER UPLOADS
}
```

Pruebe la configuración de nginx y reinicie el servidor:

```bash
sudo nginx -t
sudo service nginx restart
```

Ahora, habilite SSL (https) en sus sitios para usar todas las funciones de la aplicación, como las notificaciones y el envío de mensajes de audio. Una forma sencilla de hacerlo es usar Certbot:

Install certbot:

```bash
sudo snap install --classic certbot
sudo apt update
```

Habilitar SSL en nginx (Complete/Acepte toda la información requerida):

```bash
sudo certbot --nginx
```

### Uso de Docker y docker-compose

Para ejecutar exacticket con Docker, siga estos pasos:

```bash
cp .env.example .env
```

Ahora será necesario configurar el .env utilizando su información, las variables son las mismas que las mencionadas en el despliegue usando ubuntu, con excepción de la configuración de mysql que no estaba en el .env.

```bash
# MYSQL
MYSQL_ENGINE=                           # default: mariadb
MYSQL_VERSION=                          # default: 10.6
MYSQL_ROOT_PASSWORD=strongpassword      # change it please
MYSQL_DATABASE=exacticket
MYSQL_PORT=3306                         # default: 3306; Use this port to expose mysql server
TZ=America/Fortaleza                    # default: America/Fortaleza; Timezone for mysql

# BACKEND
BACKEND_PORT=                           # default: 8080; but access by host not use this port
BACKEND_SERVER_NAME=api.mydomain.com
BACKEND_URL=https://api.mydomain.com
PROXY_PORT=443
JWT_SECRET=3123123213123                # change it please
JWT_REFRESH_SECRET=75756756756          # change it please

# FRONTEND
FRONTEND_PORT=80                        # default: 3000; Use port 80 to expose in production
FRONTEND_SSL_PORT=443                   # default: 3001; Use port 443 to expose in production
FRONTEND_SERVER_NAME=myapp.mydomain.com
FRONTEND_URL=https://myapp.mydomain.com

# BROWSERLESS
MAX_CONCURRENT_SESSIONS=                # default: 1; Use only if using browserless
```

Después de definir las variables, ejecute el siguiente comando:

```bash
docker-compose up -d --build
```

En la `primera` ejecución será necesario inicializar las tablas de la base de datos utilizando el siguiente comando:

```bash
docker-compose exec backend npx sequelize db:seed:all
```

#### Certificado SSL

Para implementar el certificado SSL, agréguelo a la carpeta `ssl/certs`. Dentro de esta carpeta debe haber una carpeta `backend` y una carpeta `frontend`, cada una con los archivos `fullchain.pem` y `privkey.pem`, como se muestra en la siguiente estructura:

```bash
.
├── certs
│   ├── backend
│   │   ├── fullchain.pem
│   │   └── privkey.pem
│   └── frontend
│       ├── fullchain.pem
│       └── privkey.pem
└── www
```

Para generar los archivos del certificado, utilice `certbot`, que se puede instalar mediante snap. Usé el siguiente comando:

Nota: El contenedor frontend que ejecuta nginx ya está preparado para recibir la solicitud de certboot para validar el certificado.

```bash
# BACKEND
certbot certonly --cert-name backend --webroot --webroot-path ./ssl/www/ -d api.mydomain.com

# FRONTEND
certbot certonly --cert-name frontend --webroot --webroot-path ./ssl/www/ -d myapp.mydomain.com
```

## Datos de acceso

Usuario: admin@exacticket.com
Contraseña: admin

## Actualización

exacticket está en desarrollo y añadimos nuevas funciones con frecuencia. Para actualizar su instalación anterior y disfrutar de todas las nuevas funciones, puede usar un script bash como este:

**Nota**: Siempre revise el archivo .env.example y ajuste su archivo .env antes de actualizar, ya que podrían añadirse nuevas variables.

```bash
nano update-exacticket
```

```bash
#!/bin/bash
echo "Updating exacticket, please wait."

cd ~
cd exacticket
git pull
cd backend
npm install
rm -rf dist
npm run build
npx sequelize db:migrate
npx sequelize db:seed
cd ../frontend
npm install
rm -rf build
npm run build
pm2 restart all

echo "Update finished. Enjoy!"
```

Hazlo ejecutable y ejecútalo:

```bash
chmod +x update-exacticket
./update-exacticket
```

## Contribucion 


## Aviso legal

Empecé a aprender Javascript hace unos meses y este es mi primer proyecto. Puede tener problemas de seguridad y muchos errores. Recomiendo usarlo solo en la red local.

Este proyecto no está afiliado, asociado, autorizado, respaldado ni conectado oficialmente de ninguna manera con WhatsApp ni con ninguna de sus subsidiarias o afiliadas. El sitio web oficial de WhatsApp se encuentra en https://whatsapp.com. "WhatsApp", así como los nombres, marcas, emblemas e imágenes relacionados, son marcas registradas de sus respectivos propietarios.

