# Documentación: ODOO Y DOCKER EN UBUNTU

## Requisitos Previos

- Descargar **Ubuntu Live Server 24.04**
- Descargar e instalar **VirtualBox**

---

## Configuración de Adaptador de Red VirtualBox

| Campo | Valor |
|-------|-------|
| **Adaptador** | Adaptador 1 |
| **Conectar a** | NAT |
| **Reenvío de puertos** | Sí |
| **Nombre** | ODOO |
| **Protocolo** | TCP |
| **IP Anfitrión** | 127.0.0.1 |
| **Puerto Anfitrión** | 4444 (u otro) |
| **IP Invitado** | 10.0.2.15 |
| **Puerto Invitado** | 8069 |

---

## 1. Instalación de Odoo y PostgreSQL

### Instalación de Odoo al no contar con un docker-compose.yml

```bash
sudo apt install postgresql -y

wget -q -O - https://nightly.odoo.com/odoo.key | sudo gpg --dearmor -o /usr/share/keyrings/odoo-archive-keyring.gpg

echo 'deb [signed-by=/usr/share/keyrings/odoo-archive-keyring.gpg] https://nightly.odoo.com/17.0/nightly/deb/ ./' | sudo tee /etc/apt/sources.list.d/odoo.list

sudo apt-get update && sudo apt-get install odoo

sudo systemctl status odoo
```

---

## 2. Instalación de Docker

### Detener servicios de Odoo

```bash
sudo systemctl stop odoo
sudo systemctl disable odoo
sudo systemctl status odoo
```

### Paso 1: Desinstalación preventiva de Docker

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

### Paso 2: Agregar clave GPG oficial de Docker

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### Paso 3: Agregar repositorio de Docker

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
sudo apt-get update
```

### Paso 4: Instalación de paquetes de Docker

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Paso 5: Crear contenedor de prueba

```bash
sudo docker run hello-world
```

---

## 3. Creación de Contenedores Odoo

### Contenedor de PostgreSQL (Base de Datos)

```bash
docker run -d \
  -v /home/usuario/OdooDesarrollo/dataPG:/var/lib/postgresql/data \
  -e POSTGRES_USER=odoo \
  -e POSTGRES_PASSWORD=odoo \
  -e POSTGRES_DB=postgres \
  --name db \
  postgres:15
```

### Contenedor de Odoo (Desarrollo)

```bash
docker run -d \
  -v /home/usuario/OdooDesarrollo/volumesOdoo/addons:/mnt/extra-addons \
  -v /home/usuario/OdooDesarrollo/volumesOdoo/firestore:/var/lib/odoo/filestore \
  -v /home/usuario/OdooDesarrollo/volumesOdoo/sessions:/var/lib/odoo/sessions \
  -p 8069:8069 \
  --name odoodev \
  --user="root" \
  --link db:db \
  -t odoo:18 \
  --dev=all
```

### Asignar permisos

```bash
sudo usermod -aG docker $USER    # permisos de docker a usuario y reiniciamos
sudo chmod -R 777 /home/usuario/OdooDesarrollo/volumesOdoo/addons # permisos de docker
```

---

## 4. Fichero Docker-Compose

### docker-compose.yml

```yaml
version: '3.3'

services:
  # Servicio Web (Odoo)
  web:
    image: odoo:18
    container_name: odoo-web
    depends_on:
      - db
    ports:
      - 8069:8069
    volumes:
      - ./volumesOdoo/addons:/mnt/extra-addons
      - ./volumesOdoo/odoo-web-data:/var/lib/odoo
    user: root
    environment:
      - DB_HOST=db
      - DB_PORT=5432
      - DB_USER=odoo
      - DB_PASSWORD=Hola123_
      - DB_NAME=odoo_db

  # Servicio Base de Datos (PostgreSQL)
  db:
    image: postgres:15
    container_name: odoo-db
    environment:
      - POSTGRES_PASSWORD=odoo
      - POSTGRES_USER=odoo
      - POSTGRES_DB=postgres
    volumes:
      - ./volumesOdoo/dataPostgreSQL:/var/lib/postgresql/data
```

### Comandos útiles

```bash
sudo usermod -aG docker $USER    # damos permisos de docker a usuario y reiniciamos 
docker compose up -d    # Iniciar contenedores
docker compose down     # Detener contenedores
```

---

## 5. Post-Instalación

### Subir al repositorio Git

#### Crear archivo tar

```bash
sudo tar -cvzf nombre.tar volumesOdoo/
```

#### Comprimir en formato xz

```bash
sudo tar -cvf volumesOdoo.tar.xz -I 'xz -9' volumesOdoo
volumesOdoo/
```

#### Configurar Git

```bash
git config --global init.defaultBranch main
git config --global user.name "Tecnofix"
git config --global user.email "eriayacav@alu.edu.gva.es"
git init
git branch -m main
git branch --show-current
git config --global --add safe.directory "directorio"
```

#### *Añadir archivos y hacer commit

```bash
git init (en el fichero de trabajo github)
git branch -M main
git config --global user.name "Tecnofix"
git config --global user.email "eriayacav@alu.edu.gva.es"
git remote add origin https://github.com/eriayacav/Documentacion-Odoo
git remote uv-
git pull origin main --allow-unrelated-histories
git add VolumesOdoo.tar.xz
git commit -m "Actualización Odoo"
git push origin main
```

#### Subir a otra rama

```bash
git add ruta
git commit -m "Comentario"
git remote add origin "URL repositorio"
git remote -v
git checkout "rama destino"
git push -u origin rama_destino
```

### Descargar ficheros del repositorio

```bash
git clone "URL repositorio"
tar xvf micomprimido.tar
```

---

## 6. Configuración de Empresa en Odoo

### Configuración de Email y Puertos

| Concepto | Valor |
|----------|-------|
| **Email 1** | tecnofix@odooserra.work.gd |
| **Password** | PerenxisaTecnо2025 |
| **Email 2** | patocomponentes@odooserra.work.gd |
| **Email 3** | apptorrent@odooserra.work.gd |
| **Servidor Entrante** | imap.qboxmail.com |
| **Puerto IMAP** | 993 |
| **Seguridad IMAP** | SSL/TLS |
| **Servidor Saliente** | smtp.qboxmail.com |
| **Puerto SMTP** | 465 |
| **Seguridad SMTP** | SSL/TLS |
| **Webmail** | webmail.qboxmail.com |
| **Gerente** | manuel@theinkgarage.com |

### Credenciales de Acceso

| Campo | Valor |
|-------|-------|
| **Database Name** | odoo_db |
| **Email** | rubeninform123@gmail.com |
| **Password** | Hola123_ |

---

## 7. Configuración de Odoo

### 1. Dependencias y Módulos Requeridos

#### Módulos a instalar

- **Ventas** → Para facturación
- **Compras** → Proveedores y pedidos
- **Inventario** → Entradas, recepciones y stock
- **Empleados** → Trabajadores y usuarios

#### Módulos específicos

- **Sale Block No Stock** → Permite facturar sin stock
- **Stock Disallow Negative** → Evita stock negativo

---

### 2. Configuración Inicial de la Empresa

**Nombre:** TecnoFix

**Pasos:**
1. Ir a **Ajustes → Empresa → Nueva Empresa**
2. Ingresar datos básicos: RFC/NIF, dirección, moneda, email, teléfono

---

### 3. Configuración del Correo

**Ruta:** Ajustes → Técnico → Correo → Servidores de correo

#### Servidor de correos de entrada (IMAP)

| Campo | Valor |
|-------|-------|
| **Nombre** | imap_qbox |
| **Servidor** | imap.qboxmail.com |
| **Puerto** | 993 |
| **SSL/TLS** | Sí |
| **Usuario** | tecnofix@odooserra.work.gd |
| **Contraseña** | PerenxisaTecnо2025 |

#### Servidor de correos de salida (SMTP)

| Campo | Valor |
|-------|-------|
| **Nombre** | smtp_qbox |
| **Autentificación** | Nombre de usuario |
| **Encriptación** | SSL/TLS |
| **Servidor SMTP** | smtp.qboxmail.com |
| **Puerto SMTP** | 465 |

---

### 4. Inventario

#### Crear almacenes

**Ruta:** Inventario → Configuración → Ajustes

1. **Almacenes:** Nuevos → Tecnofix
2. Registrar existencias iniciales
3. **Operaciones → Ajustes → Inventario físico**

#### Resumen de inventario

**Ruta:** Inventario → Informes → Valoración de inventario

#### Recepción y validación de pedidos

**Ruta:** Inventario → Operaciones → Recepciones

1. Seleccionar recepción pendiente
2. Verificar cantidades
3. Validar

---

### 5. Productos y Servicios

#### Crear productos

**Ruta:** Productos → Crear nuevo

**Definir tipo:**
- **Servicio** → Para trabajos, asistencia, etc.
- **Producto almacenable** → Para stock

**Configurar:** Precios e impuestos

#### Servicios diferentes a productos

1. En **Tipo de producto** elegir: **Servicio**
2. Quitar rutas de inventario

---

### 6. Etiquetas para Informes

**Ruta:** Configuración → Técnico → Etiquetas

Crear etiquetas para clasificar productos, pedidos o informes

---

### 7. Proveedores

#### Crear proveedores

**Ruta:** Compras → Proveedores → Crear

**Completar información básica:**
- Nombre
- Teléfono
- Email
- Dirección

#### Seleccionar tipo

| Tipo | Descripción |
|------|-------------|
| **Persona Física** | Autónomo |
| **Compañía** | Empresa |

---

### 8. Compras

#### Solicitudes de presupuestos

**Ruta:** Compras → Solicitudes de presupuesto → Nuevo

1. Seleccionar proveedor
2. Agregar productos o servicios

#### Crear nuevo presupuesto a un proveedor

1. Añadir líneas de productos
2. Confirmar como **Pedido de compra**

**Nota:** Un presupuesto confirmado = Pedido de compra

---

### 9. Entradas de Inventario

**Ruta:** Inventario → Operaciones → Recepciones

1. Seleccionar entrada vinculada al pedido
2. Validar para mover a stock

---

### 10. Validación de Pedidos

#### Ventas

**Ruta:** Ventas → Pedidos

1. Confirmar pedido
2. Generar factura
3. Validar factura

#### Compras

**Ruta:** Compras → Pedidos

1. Confirmar compra
2. Registrar recepción
3. Validar

---

### 11. Empleados, Usuarios y Roles

#### Crear empleados

**Ruta:** Empleados → Crear

Ingresar información contractual y personal

#### Crear usuarios

**Ruta:** Ajustes → Usuarios y empresas

1. Crear usuario
2. Asignarle al empleado correspondiente
3. Definir permisos:
   - Ventas
   - Compras
   - Inventario
   - Contabilidad
   - Acceso de administrador

---

### 12. Cambiar prefijos de documentos

#### Secuencia para Presupuestos

**Formato:** V-AÑO-NUMERO

| Campo | Valor |
|-------|-------|
| **Nombre** | Secuencia de Presupuestos |
| **Código** | sale.order.quotation |
| **Prefijo** | V-%(y)s- |

#### Secuencia para Pedidos de Venta

**Formato:** PED-AÑO-NUMERO

| Campo | Valor |
|-------|-------|
| **Nombre** | Secuencia Pedidos de Venta |
| **Código** | sale.order |
| **Prefijo** | PED-%(year)s- |

#### Secuencia para Albaranes

**Formato:** A-AÑO-NUMERO

| Campo | Valor |
|-------|-------|
| **Nombre** | Secuencia Albaranes |
| **Código** | Secuencia entrada/salida My Company |
| **Prefijo Entrada** | A/IN/%(y)s/ |
| **Prefijo Salida** | A/OUT/%(y)s/ |

---

## 8. Envío de Correo al Confirmar Pedido

### 1. Activar modo desarrollador

**Ruta:** Ajustes → Activar modo desarrollador (con activos de prueba)

### 2. Crear plantilla de correo

**Ruta:** Ajustes → Técnico → Correo Electrónico → Plantillas → Nuevo

| Campo | Valor |
|-------|-------|
| **Nombre** | [Custom] - Correo pedido de venta |
| **Aplicado a Modelo** | Pedido de venta (sale.order) |
| **Asunto** | Gracias por su pedido {{object.name}} |

#### Contenido del correo

Para agregar marcadores dinámicos: **Shift + 7** para desplegar estructura, luego **#** para Marcador de posición dinámico

```
Hola {{object.partner_id.name}},

Gracias por su pedido {{object.name}}

Recuerda que tienes un saldo pendiente de: {{object.amount_total}} €

Saludos,
Equipo de Ventas.
```

**Guardar** la plantilla

### 3. Probar la plantilla manualmente

1. Ir a **Ventas → Pedidos de venta**
2. Abrir un pedido que tenga cliente y email
3. **Enviar mensaje → Redactar correo electrónico**
4. Usar plantilla: **Custom - Correo pedido de venta**
5. Verificar que el asunto y el cuerpo se renderizan con los valores reales
6. Enviar (opcional) para comprobar llegada al buzón

### 4. Crear la acción de servidor

**Ruta:** Ajustes → Técnico → Acciones → Acciones de servidor → Nuevo

| Campo | Valor |
|-------|-------|
| **Nombre** | Enviar correo electrónico automáticamente en venta |
| **Modelo** | Pedido de venta (sale.order) |
| **Tipo de acción** | Enviar correo electrónico |
| **Plantilla de correo** | Custom - Correo pedido de venta |

**Guardar**

---

## 9. Automatización de Correo para Presupuesto > 20000 €

### 1. Requisitos previos

- **Instalar módulo:** CRM
- **Activar modo desarrollador:** Ajustes → Activar modo desarrollador

### 2. Crear plantilla de correo

**Ruta:** Técnico → Correo → Plantillas → Nuevo

| Campo | Valor |
|-------|-------|
| **Nombre** | Plantilla - Oportunidad > 20000 € |
| **Modelo Aplicado a** | Lead/Opportunity (crm.lead) |
| **Asunto** | Gracias por su interés — Presupuesto {{ object.name }} |

#### Contenido

```
Hola,

Se ha registrado/actualizado una oportunidad con un valor superior a 20.000 €.

Nombre: {{object.name}}
Cliente: {{object.partner_id.name}}
Importe: {{object.expected_revenue}} €

Saludos,
Sistema Odoo
```

**Guardar**

### 3. Crear regla de automatización

**Ruta:** Técnico → Automatización → Reglas de automatización → Nuevo

| Campo | Valor |
|-------|-------|
| **Nombre** | Aviso Oportunidad >= 20000 |
| **Modelo** | crm.lead (Lead/Oportunidad) |
| **Activador** | La etapa está establecida como (Calificado) |
| **Aplicar a** | Editar dominio |

#### Dominio

**Coincidir todas las siguientes reglas:**
- Ingresos esperados > 20000

**Nueva regla:**
```python
[('amount_total', '>=', 20000)]
```

**Guardar**

### 4. Validación práctica

**Ruta:** Ventas → Presupuestos → Nuevo

1. Seleccionar cliente
2. Añadir productos/servicios con monto Total >= 20000 €
3. Guardar

**Resultado:** Al guardar se ejecuta la regla y envía el correo automáticamente

### 5. Depuración y comprobaciones

| Elemento | Ruta |
|----------|------|
| **Logs de correo** | Técnico → Correo → Correos |
| **Ejecuciones de automatización** | Historial en la regla de automatización |
| **Permisos** | Revisar usuario o marcar "Ejecutar como sistema" |
| **Plantilla** | Confirmar que `object.partner_id.email` tiene valor válido |

---

## 10. Automatización de Servicio al Realizar Pedido de Venta

### 1. Creación del Producto A – Servidor HP Enterprise 2000

**Ruta:** Inventario → Productos → Productos → Crear

| Campo | Valor |
|-------|-------|
| **Nombre** | Servidor HP Enterprise 2000 |
| **Tipo de Producto** | Producto físico |
| **Precio de Venta** | 1000 € |
| **Categoría** | Hardware |
| **Control de stock** | Sí (almacenable) |

**Guardar**

### 2. Creación del Producto B – Servicio de Instalación

**Ruta:** Inventario → Productos → Productos → Crear

| Campo | Valor |
|-------|-------|
| **Nombre** | Servicio de Instalación y Configuración de Servidores |
| **Tipo de Producto** | Servicio |
| **Precio de Venta** | 300 € |

**Guardar**

### 3. Configurar la Acción Automática

#### 3.1. Activar modo desarrollador

**Ruta:** Ajustes → Activar modo desarrollador

#### 3.2. Crear la Acción Automática

**Ruta:** Ajustes → Técnico → Automatización → Acciones Automáticas → Crear

#### 3.3. Configurar datos principales

| Campo | Valor |
|-------|-------|
| **Modelo** | Pedido de Venta (sale.order) |
| **Nombre** | Crear pedido de venta de instalación cuando falte servicio |
| **Activador** | El estado está establecido como → pedido de venta |

#### 3.4. Código Python

```python
# 1. Obtener el producto de servicio (Producto B)
# Se recomienda usar el ID, pero buscaremos por nombre para simplificar
product_service = env['product.product'].search([
    ('name', '=', 'Servicio de Instalación y Configuración de Servidores')
], limit=1)

if product_service:
    # 2. Crear el nuevo Pedido de Venta (Borrador)
    # 'record' es la variable que representa el Pedido de Venta (sale.order) actual confirmado.
    new_so = env['sale.order'].create({
        'partner_id': record.partner_id.id,
        'state': 'draft',  # Lo creamos como borrador
    })

    # 3. Crear la línea de pedido de servicio en el nuevo PV
    env['sale.order.line'].create({
        'order_id': new_so.id,
        'product_id': product_service.id,
        'name': product_service.display_name,
        'product_uom_qty': 1.0,
        'price_unit': product_service.list_price,
    })
```


---

## 11. Configuración de Sitio Web y Métodos de Envío

### 1. Activar el módulo de Sitio Web

**Ruta:** Aplicaciones → Buscar "Sitio Web" → Instalar

**Módulos necesarios:**
- Sitio Web
- Comercio Electrónico

---

### 2. Configurar Métodos de Envío

#### 2.1. Acceder a Métodos de Envío

**Ruta:** Sitio Web → Configuración → Métodos de envío

#### 2.2. Crear nuevo método de envío

**Clic en:** Crear

| Campo | Descripción |
|-------|-------------|
| **Nombre del método** | Ejemplo: Envío Estándar, Envío Express, etc. |
| **Proveedor** | Proveedor de envío o método propio |
| **Sitio web** | Seleccionar sitio web |
| **Empresa** | Empresa (TecnoFix) |

---

### 3. Agregar Peso a los Artículos

#### 3.1. Configurar peso en productos

**Ruta:** Inventario → Productos → Productos → Seleccionar producto

**En la pestaña "Inventario":**

| Campo | Valor |
|-------|-------|
| **Peso** | Peso en kg (ej: 2.5) |
| **Volumen** | Volumen en m³ (opcional) |

#### 3.2. Ejemplo de configuración

```
Producto: Servidor HP Enterprise 2000
├── Peso: 15.5 kg
├── Volumen: 0.08 m³
└── Unidad de medida: Unidades
```

**Guardar** cambios

---

### 4. Configurar Etiquetas de Productos

#### 4.1. Crear etiquetas para clasificación


**Ruta:** Comercio Electrónico → Etiquetas de producto

**Crear nuevas etiquetas:**

Nuevo → Nombre → Color → Imagen → **Guardar** 

#### 4.2. Asignar etiquetas a productos

**Ruta:** Inventario → Productos → Productos → Seleccionar producto

**En pestaña "Ventas":**
1. Buscar campo **"Etiquetas"**
2. Seleccionar o crear etiquetas apropiadas
3. **Guardar**

---

### 5. Configurar Etiquetas Excluidas en Métodos de Envío

#### 5.1. Definir exclusiones por etiqueta

**Ruta:** Sitio Web → Configuración → Métodos de envío → Seleccionar método

**En el formulario del método de envío:**

| Campo | Configuración |
|-------|---------------|
| **Etiquetas excluidas** | Seleccionar etiquetas que NO pueden usar este método |

---

### 6. Crear Reglas de Envío

#### 6.1. Configurar reglas por condiciones

**Ruta:** Sitio Web → Configuración → Métodos de envío → Seleccionar método → Pestaña "Reglas de precio"

**Clic en:** Agregar una línea

#### 6.2. Tipos de reglas disponibles

| Tipo de Regla | Descripción |
|---------------|-------------|
| **Por precio** | Basado en el importe del pedido |
| **Por peso** | Basado en el peso total del pedido |
| **Por cantidad** | Basado en el número de artículos |
| **Por volumen** | Basado en el volumen total |

---

### 7. Configurar Envío Gratis por Condiciones

#### 7.1. Crear regla de envío gratis

**Ruta:** Sitio Web → Configuración → Métodos de envío → Crear o editar método

**Configuración del método:**

| Campo | Valor |
|-------|-------|
| **Metodo de envio** | Envío Gratis |
| **Tipo de precio** | Basado en reglas |
| **Gratis si el pedido supera** | Marcar casilla |

#### 7.2. Configurar reglas de precio

**En pestaña "Reglas de precio":** Agregar línea


| Campo | Valor |
|-------|-------|
| **Condición basada en** | Precio |
| **Mínimo** | 0.00 € |
| **Máximo** | 100.00 € |
| **Precio** | 5.00 € |

| Campo | Valor |
|-------|-------|
| **Condición basada en** | Precio |
| **Mínimo** | 100.01 € |
| **Máximo** | 999999.00 € |
| **Precio** | 0.00 € |

##### Envío gratis por peso

| Campo | Valor |
|-------|-------|
| **Condición basada en** | Peso |
| **Mínimo** | 0.00 kg |
| **Máximo** | 5.00 kg |
| **Precio** | 8.00 € |

| Campo | Valor |
|-------|-------|
| **Condición basada en** | Peso |
| **Mínimo** | 5.01 kg |
| **Máximo** | 20.00 kg |
| **Precio** | 15.00 € |

##### Envío gratis combinado

| Campo | Valor |
|-------|-------|
| **Condición basada en** | Precio |
| **Mínimo** | 150.00 € |
| **Máximo** | 999999.00 € |
| **Condición de peso** | Máximo 25 kg |
| **Precio** | 0.00 € |

**Guardar** todas las reglas

---



### Comandos útiles adicionales

```bash
# Ver contenedores en ejecución
docker ps

# Ver logs de un contenedor
docker logs odoo-web
docker logs odoo-db

# Reiniciar contenedor
docker restart odoo-web

# Entrar a un contenedor
docker exec -it odoo-web bash

# Limpiar contenedores detenidos
docker container prune

# Ver imágenes descargadas
docker images
```

**Documento creado:** TecnoFix  
**Última actualización:** 2026  
**Versión de Odoo:** 18  
**Versión de PostgreSQL:** 15
