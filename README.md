# Documentacion:
# ODOO Y DOCKERS EN UBUNTU
1. Descargar la version UBUNTU LIVE SERVER 24.04
2. Descargar e instalar Virtualbox 


## CONFIGURACION DE ADAPTADOR DE RED VIRTUALBOX

1. Adaptador 1
2. Conectar a NAT
3. Reenvio de puertos
4. Crear una regla 
5. Nombre: ODOO
6. Protocolo TCP
7. IP Afitrion 127.0.0.1
8. Puerto Anfrition: 4444 o otro
9. IP Invitado 10.0.2.15
10. Puerto 8069

# 1-INSTALACION DE ODOO Y REPOSITORIOS EN POSTGRESQL

## Instalacion Odoo:
``` 
sudo apt install postgresql -y
``` 
``` 
$ wget -q -O - https://nightly.odoo.com/odoo.key | sudo gpg --dearmor -o /usr/share/keyrings/odoo-archive-keyring.gpg
```
```
$ echo 'deb [signed-by=/usr/share/keyrings/odoo-archive-keyring.gpg] https://nightly.odoo.com/17.0/nightly/deb/ ./' | sudo tee /etc/apt/sources.list.d/odoo.list
```
```
$ sudo apt-get update && sudo apt-get install odoo
```
```
$ sudo systemctl status odoo
```
# 2-INSTALACION DE DOCKER 

## DETENEMOS LOS SERVICIOS ODOO:
```
sudo systemctl stop odoo
```
```
sudo systemctl disable odoo
```
```
sudo systemctl status odoo
```
## 1. Desinstalacion preventiva docker:
```
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```
## 2. Agregamos Docker's official clave GPG:
```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
## 3. agregamos el repositorio de las fuentes:
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \ $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" |\ sudo tee /etc/apt/sources.list.d/docker.list > /dev/null sudo apt-get update
```

## 4. Instalacion paquetes de Dockers
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
## 5. creamos un contenedor prueba
```
sudo docker run hello-world
```
# 3-CREACION DE CONTENEDOR ODOO  
## Creacion de contenedor Odoo para Produccion y Desarrollo
```
docker run -d -v /home/usuario/OdooDesarrollo/dataPG:/var/lib/postgresql/data -e POSTGRES_USER=odoo -e POSTGRES_PASSWORD=odoo -e POSTGRES_DB=postgres --name db postgres:15
```

## Creacion de contenedor Odoo en Desarrollo
```
docker run -d -v /home/usuario/OdooDesarrollo/volumesOdoo/addons:/mnt/extra-addons -v /home/usuario/OdooDesarrollo/volumesOdoo/firestore:/var/lib/odoo/filestore -v /home/usuario/OdooDesarrollo/volumesOdoo/sessions:/var/lib/odoo/sessions -p 8069:8069--name odoodev --user="root" --link db:db -t odoo:18 --dev=all  
```
## Creamos los permisos 
```
sudo chmod -R 777 /home/usuario/OdooDesarrollo/volumesOdoo/addons
```
# 4-FICHERO DOCKER-COMPOSE
```
version: '3.3'

services:
#Definimos el servicio Web, en este caso Odoo
  web:
    #Indicamos que imagen de Docker Hub utilizaremos
    image: odoo:18
    container_name: odoo-web
    #Indicamos que depende de "db", por lo cual debe ser procesada primero "db"
    depends_on:
        - db

    # Port Mapping: indicamos que el puerto 8069 del contenedor se mapeara con el mismo puerto en el anfritrion
    # Permitiendo acceder a Odoo mediante http://localhost:8069
    ports:
      - 8069:8069

    # Mapeamos el directorio de los contenedores (como por ejemplo" /mnt/extra-addons" )
    # en un directorio local (como por ejemplo en un directorio "./volumesOdoo/addons")
    # situado en el lugar donde ejecutemos "Docker compose"
    volumes:
      - ./volumesOdoo/addons:/mnt/extra-addons
      - ./volumesOdoo/odoo-web-data:/var/lib/odoo
    #Indicamos que el contenedor funcionara con usuario root y no con usuario odoo
    user: root
    # Definimos variables de entorno de Odoo
    environment:
      - DB_HOST=db
      - DB_PORT=5432
      - DB_USER=odoo
      - DB_PASSWORD=Hola123_
      - DB_NAME=odoo_db
#Definimos el servicio de la base de datos
  db:
    image: postgres:15
    container_name: odoo-db
    # Definimos variables de entorno de PostgreSQL
    environment:
      - POSTGRES_PASSWORD=odoo
      - POSTGRES_USER=odoo
      - POSTGRES_DB=postgres
    # Mapeamos el directorio del contenedor "var/lib/postgresql/data" en un directorio "./volumesOdoo/dataPostgreSQL"
    # situado en el lugar donde ejecutemos "Docker compose"
    volumes:
      - ./volumesOdoo/dataPostgreSQL:/var/lib/postgresql/data
```
## Recomendaciones
```
docker compose up -d
```
```
docker compose down
```
# 5-POST INSTALACION
## Repositorio Git
```
tar -cvzf nombre.tar volumesOdoo/ (creamos tar)
```
```
tar cvf micomprimido.tar.xz -I 'xz -9' (comprimimos en la misma ubicacion)
```
```
git config --global init.defaultBranch main (crear la rama main)
```
```
git init (iniciar el servicio)
```
```
git branch -m main
```
```
git branch --show-current (visualizamos el main)
```
```
git config --global --add safe.directory "directorio"
```
```
git add "ruta" (tecnofix.tar.xz)
```
```
git status
```
```
git config --global user.email "email github"
```
```
git config --global user.name "nombre github"
```
```
git commit -m "Comentario proyecto"
```
```
git remote add origin "URL Github repositorio" 
```
```
git remote -v (muestra el repositorio remoto)
```
```
git push -u origin main (main para subir con token)
```
```
git pull --rebase origin main
```
```
git status
```

## Github subir main (otra rama)
```
git add ruta
git commit -m "Comentario"
git remote add origin nuestro repositorio que queremos usar
git remote -v
git fetch --all
git checkout la rama donde lo tengas
git push -u origin main
```
## Github subir mas ficheros (misma rama)

```
git add directorio/s
```
```
git commit -m "comentario"
```
```
git status
```
```
git pull --rebase origin main(conserva los comentarios)
```
```
git fetch origin y git reset --hard origin/main(restea el repositorio)
```
```
git push origin main
```
## Github descargar ficheros
```
git clone "URL repositorio"
```
```
tar xvf micomprimido.tar
```
# 6-APARTADO TECNOFIX
## Email y puertos
```
tecnofix@odooserra.work.gd PerenxisaTecnо2025

patocomponentes@odooserra.work.gd

apptorrent@odooserra.work.gd

Entrante: imap.qboxmail.com IMAP: 993 Seguridad SSL/TLS

Saliente: smtp.qboxmail.com SMTP: 465 Seguridad SSL/TLS

Webmail: webmail.qboxmail.com

Gerente: manuel@theinkgarage.com
```
## Credenciales
```
Database Name: odoo_db
email: rubeninform123@gmail.com
Password: Hola123_
```
