# Documentación: ODOO Y DOCKER EN UBUNTU

---

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
| **Puerto Anfitrión** | 4444 (otro) |
| **IP Invitado** | 10.0.2.15 |
| **Puerto Invitado** | 8069 |

---

## 1. Instalación de Odoo y PostgreSQL

### 1.1. Instalación de Odoo al no contar con un docker-compose.yml

```bash
sudo apt install postgresql -y

wget -q -O - https://nightly.odoo.com/odoo.key | sudo gpg --dearmor -o /usr/share/keyrings/odoo-archive-keyring.gpg

echo 'deb [signed-by=/usr/share/keyrings/odoo-archive-keyring.gpg] https://nightly.odoo.com/17.0/nightly/deb/ ./' | sudo tee /etc/apt/sources.list.d/odoo.list

sudo apt-get update && sudo apt-get install odoo

sudo systemctl status odoo
```

### 1.2. Detener servicios de Odoo

```bash
sudo systemctl stop odoo
sudo systemctl disable odoo
sudo systemctl status odoo
```

### 1.3. Desinstalación preventiva de Docker

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

---

## 2. Instalación de Docker con compose .yml

### 2.1. Agregar clave GPG oficial de Docker

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### 2.2. Agregar repositorio de Docker

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
sudo apt-get update
```

### 2.3. Instalación de paquetes de Docker

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 2.4. Crear contenedor de prueba

```bash
# Creamos el contenedor
sudo docker run hello-world
# Verificamos el contenedor creado
docker ps -a
```

---

## 3. Creación de Contenedores Odoo

### 3.1. Contenedor de PostgreSQL (Base de Datos)

```bash
docker run -d \
  -v /home/usuario/OdooDesarrollo/dataPG:/var/lib/postgresql/data \
  -e POSTGRES_USER=odoo \
  -e POSTGRES_PASSWORD=odoo \
  -e POSTGRES_DB=postgres \
  --name db \
  postgres:15
```

### 3.2. Contenedor de Odoo (Desarrollo)

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

### 3.3. Asignar permisos

```bash
sudo usermod -aG docker $USER    # permisos de docker a usuario y reiniciamos
sudo chmod -R 777 /home/usuario/OdooDesarrollo/volumesOdoo/addons # permisos de docker
```

---

## 4. Fichero Docker-Compose

### 4.1. docker-compose.yml

Creamos docker-compose.yml en directorio y copiamos dentro con: `nano docker-compose.yml`

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

### 4.2. Comandos de gestión

```bash
# Conectamos en el explorador con: 
http://localhost:8069/odoo

# Damos permisos de docker a usuario y reiniciamos 
sudo usermod -aG docker $USER

# Iniciar contenedores
docker compose up -d 

# Detener contenedores   
docker compose down
```

---

## 5. Post-Instalación

### 5.1. Subir al repositorio Git

#### 5.1.1. Crear archivo tar

```bash
sudo tar -cvzf nombre.tar volumesOdoo/
```

#### 5.1.2. Comprimir en formato xz

```bash
# Se comprime y queda en la misma ubicacion
sudo tar -cvf volumesOdoo.tar.xz -I 'xz -9' volumesOdoo
volumesOdoo/

# Se mueve de ubicacion ejemplo:
mv volumesOdoo.tar.xz ~/Escritorio/mi-repo-github/
```

#### 5.1.3. Configurar Git

```bash
git config --global init.defaultBranch main
git config --global user.name "Tecnofix"
git config --global user.email "eriayacav@alu.edu.gva.es"
git init
git branch -m main
git branch --show-current
git config --global --add safe.directory "directorio"
```

#### 5.1.4. Añadir archivos y hacer commit

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

#### 5.1.5. Subir a otra rama

```bash
git add ruta
git commit -m "Comentario"
git remote add origin "URL repositorio"
git remote -v
git checkout "rama destino"
git push -u origin rama_destino
```

### 5.2. Descargar ficheros del repositorio

```bash
git clone "URL repositorio"
tar xvf micomprimido.tar
```

---

## 6. Configuración de Empresa en Odoo

### 6.1. Configuración de Email y Puertos

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

### 6.2. Credenciales de Acceso

| Campo | Valor |
|-------|-------|
| **Database Name** | odoo_db |
| **Email** | rubeninform123@gmail.com |
| **Password** | Hola123_ |

---

## 7. Personalización de Odoo

### 7.1. Dependencias y Módulos Requeridos

#### 7.1.1. Módulos a instalar

- **Ventas** → Para facturación
- **Compras** → Proveedores y pedidos
- **Inventario** → Entradas, recepciones y stock
- **Empleados** → Trabajadores y usuarios
- **Sitio WEB** → Sitio WEB en Odoo

#### 7.1.2. Módulos específicos

- **Sale Block No Stock** → Permite facturar sin stock
- **Stock Disallow Negative** → Evita stock negativo

### 7.2. Configuración Inicial de la Empresa

**Nombre:** TecnoFix

**Pasos:**
1. Ir a **Ajustes → Empresa → Nueva Empresa**
2. Ingresar datos básicos: RFC/NIF, dirección, moneda, email, teléfono

### 7.3. Configuración del Correo

**Ruta:** Ajustes → Técnico → Correo → Servidores de correo

#### 7.3.1. Servidor de correos de entrada (IMAP)

| Campo | Valor |
|-------|-------|
| **Nombre** | imap_qbox |
| **Servidor** | imap.qboxmail.com |
| **Puerto** | 993 |
| **SSL/TLS** | Sí |
| **Inicio de sesión** | tecnofix@odooserra.work.gd |
| **Contraseña** | PerenxisaTecnо2025 |
| **Acciones a realizar** | Nuevo mensaje de correo |

#### 7.3.2. Servidor de correos de salida (SMTP)

| Campo | Valor |
|-------|-------|
| **Nombre** | smtp_qbox |
| **Servidor** | smtp.qboxmail.com |
| **Puerto** | 465 |
| **Seguridad de conexión** | SSL/TLS |
| **Nombre de usuario** | tecnofix@odooserra.work.gd |
| **Contraseña** | PerenxisaTecnо2025 |

### 7.4. Usuarios y Empleados

#### 7.4.1. Configuración de Usuarios

**Ruta:** Ajustes → Usuarios & Empresas → Usuarios

Crear usuarios con los siguientes roles:
- **Ventas** → Usuario
- **Compras** → Administrador
- **Inventario** → Administrador
- **Sitio Web** → Administrador

#### 7.4.2. Crear Empleados

**Ruta:** Empleados → Crear

1. Rellenar formulario
2. Asignar usuario
3. Vincular a empresa

### 7.5. Configuración de Productos

#### 7.5.1. Tipos de Productos

**Ruta:** Inventario → Configuración → Productos → Productos

- **Almacenable** → Requiere stock físico
- **Consumible** → Sin seguimiento de stock
- **Servicio** → Sin inventario

#### 7.5.2. Crear Producto

1. Nombre del producto
2. Tipo de producto
3. Precio de venta
4. Coste
5. Categoría
6. Impuestos
7. Imagen

#### 7.5.3. Categorías de Productos

**Ruta:** Inventario → Configuración → Categorías de producto

Crear categorías según necesidad:
- Electrónica
- Componentes
- Accesorios

### 7.6. Configuración de Proveedores

#### 7.6.1. Crear Proveedor

**Ruta:** Compras → Pedidos → Proveedores → Crear

| Campo | Valor |
|-------|-------|
| **Nombre** | Nombre del proveedor |
| **¿Es una Empresa?** | Sí |
| **Teléfono** | Número de contacto |
| **Móvil** | Número móvil |
| **Email** | Correo electrónico |
| **Sitio web** | URL |
| **RFC/NIF** | Identificación fiscal |

#### 7.6.2. Lista de precios del Proveedor

**Ruta:** Compras → Configuración → Listas de Precios

1. Crear lista de precios
2. Asignar a proveedor
3. Definir productos y precios

### 7.7. Configuración de Clientes

#### 7.7.1. Crear Cliente

**Ruta:** Ventas → Pedidos → Clientes → Crear

| Campo | Valor |
|-------|-------|
| **Nombre** | Nombre del cliente |
| **Email** | Correo electrónico |
| **Teléfono** | Número de contacto |
| **Dirección** | Dirección completa |
| **RFC/NIF** | Identificación fiscal |

### 7.8. Configuración de Impuestos

**Ruta:** Contabilidad → Configuración → Impuestos

#### 7.8.1. Impuestos de Venta (IVA)

| Impuesto | Porcentaje | Aplicación |
|----------|------------|------------|
| IVA 21% | 21% | General |
| IVA 10% | 10% | Reducido |
| IVA 4% | 4% | Superreducido |

#### 7.8.2. Impuestos de Compra

| Impuesto | Porcentaje | Aplicación |
|----------|------------|------------|
| IVA Soportado 21% | 21% | General |
| IVA Soportado 10% | 10% | Reducido |

### 7.9. Configuración de Almacenes

**Ruta:** Inventario → Configuración → Almacenes

#### 7.9.1. Crear Almacén

| Campo | Valor |
|-------|-------|
| **Nombre** | Almacén Principal |
| **Nombre corto** | ALM1 |
| **Empresa** | TecnoFix |

#### 7.9.2. Ubicaciones de Almacén

- **Ubicación de Stock** → Productos disponibles
- **Ubicación de Entrada** → Recepciones
- **Ubicación de Salida** → Envíos
- **Ubicación de Chatarra** → Productos desechados

### 7.10. Configuración de Inventario

#### 7.10.1. Ajuste de Inventario

**Ruta:** Inventario → Operaciones → Ajustes de Inventario

1. Crear ajuste
2. Agregar productos
3. Definir cantidad
4. Aplicar

#### 7.10.2. Reglas de Reabastecimiento

**Ruta:** Inventario → Configuración → Reglas de Reabastecimiento

| Campo | Valor |
|-------|-------|
| **Producto** | Seleccionar producto |
| **Ubicación** | Almacén Principal |
| **Cantidad mínima** | Stock mínimo |
| **Cantidad máxima** | Stock máximo |
| **Cantidad a ordenar** | Cantidad de pedido |

### 7.11. Configuración de Ventas

#### 7.11.1. Términos y Condiciones

**Ruta:** Ventas → Configuración → Ajustes

Agregar términos y condiciones de venta predeterminados

#### 7.11.2. Equipos de Ventas

**Ruta:** Ventas → Configuración → Equipos de Ventas

1. Crear equipo
2. Asignar líder
3. Agregar miembros

#### 7.11.3. Crear Presupuesto/Pedido

**Ruta:** Ventas → Pedidos → Presupuestos → Crear

1. Seleccionar cliente
2. Agregar productos
3. Definir cantidades
4. Confirmar pedido

### 7.12. Configuración de Compras

#### 7.12.1. Crear Solicitud de Presupuesto

**Ruta:** Compras → Pedidos → Solicitudes de Presupuesto → Crear

1. Seleccionar proveedor
2. Agregar productos
3. Definir cantidades
4. Confirmar pedido

#### 7.12.2. Recepciones

**Ruta:** Inventario → Operaciones → Recepciones

1. Validar productos recibidos
2. Verificar cantidades
3. Confirmar recepción

### 7.13. Configuración del Sitio Web

#### 7.13.1. Activar Sitio Web

**Ruta:** Sitio web → Configuración → Ajustes

- Activar modo de edición
- Configurar dominio
- Personalizar tema

#### 7.13.2. Crear Páginas

**Ruta:** Sitio web → Sitio → Páginas → Nueva Página

1. Seleccionar plantilla
2. Editar contenido
3. Publicar

#### 7.13.3. Configurar Tienda Online

**Ruta:** Sitio web → Tienda → Productos

1. Activar productos en sitio web
2. Configurar categorías
3. Definir precios
4. Agregar imágenes

#### 7.13.4. Métodos de Pago

**Ruta:** Sitio web → Configuración → Métodos de Pago

Configurar proveedores:
- Transferencia bancaria
- PayPal
- Stripe
- Otros

### 7.14. Configuración de Envíos

#### 7.14.1. Métodos de Envío

**Ruta:** Sitio web → Configuración → Métodos de Envío → Crear

##### Envío estándar

| Campo | Valor |
|-------|-------|
| **Nombre del método de envío** | Envío Estándar |
| **Proveedor** | Envío Fijo |
| **Sitios web** | Sitio web principal |
| **Precio fijo** | 5.00 € |

##### Envío express

| Campo | Valor |
|-------|-------|
| **Nombre del método de envío** | Envío Express |
| **Proveedor** | Envío Fijo |
| **Sitios web** | Sitio web principal |
| **Precio fijo** | 10.00 € |

##### Envío basado en reglas

| Campo | Valor |
|-------|-------|
| **Nombre del método de envío** | Envío por Peso/Precio |
| **Proveedor** | Basado en Reglas |
| **Sitios web** | Sitio web principal |

#### 7.14.2. Reglas de Precio de Envío

##### Regla 1: Productos ligeros

| Campo | Valor |
|-------|-------|
| **Condición basada en** | Peso |
| **Mínimo** | 0.00 kg |
| **Máximo** | 5.00 kg |
| **Precio** | 5.00 € |

##### Regla 2: Productos pesados

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

## 8. Creación de Módulo Odoo

### 8.1. Preparación del Entorno

```bash
# Instalar Visual Studio Code

# Instalar complementos de Python en Visual Studio Code

# En el addons crear un directorio del nombre del módulo con dos ficheros:  
# __init__.py y  __manifest__.py  

# Editamos el manifest con nano:
{ 'name': 'prueba' }
```

### 8.2. Creacion de modulo con Scaffold

```bash
# Acceder al contenedor Docker
docker exec -it odoo-web bash

# Crear el módulo con scaffold
odoo scaffold prueba /mnt/extra-addons/

# Salir del contenedor
exit
```

### 8.3. Otorgar Permisos

```bash
sudo chown -R erick:erick /home/erick/tecnofix/volumesOdoo
```

### 8.4. Estructura Generada

```
prueba/
├── __init__.py
├── __manifest__.py
├── controllers/
│   ├── __init__.py
│   └── controllers.py
├── models/
│   ├── __init__.py
│   └── models.py
├── views/
│   └── views.xml
├── security/
│   └── ir.model.access.csv
├── demo/
│   └── demo.xml
└── data/
```

---

## 9. Modificación de Archivos del Módulo

### 9.1. models.py

#### 9.1.1. Definición del Modelo

```python
class Project(models.Model):
    _name = 'project.advanced'              # Nombre técnico del modelo
    _description = 'Proyecto Avanzado'      # Descripción legible
    _inherit = ['mail.thread', 'mail.activity.mixin']  # Herencia de funcionalidades
    _order = 'date_start desc, name'        # Orden por defecto
```

**Atributos especiales del modelo:**
- `_name`: Identificador único del modelo
- `_description`: Descripción para el usuario
- `_inherit`: Herencia de otros modelos
- `_order`: Orden por defecto en listados
- `_rec_name`: Campo usado como nombre del registro (por defecto: 'name')
- `_sql_constraints`: Restricciones a nivel de base de datos

#### 9.1.2. Tipos de Campos

##### Char (Cadena de texto)

```python
name = fields.Char(
    string='Nombre del Proyecto',  # Etiqueta visible
    required=True,                  # Campo obligatorio
    tracking=True,                  # Rastrea cambios en el chatter
    index=True,                     # Crea índice en BD
    size=100,                       # Longitud máxima
    translate=True,                 # Permite traducción
    copy=False,                     # No se copia al duplicar
    readonly=True,                  # Solo lectura
    default='Nuevo'                 # Valor por defecto
)
```

##### Integer (Entero)

```python
sequence = fields.Integer(
    string='Secuencia',
    default=10,
    group_operator='sum'    # Operación de agrupación
)
```

##### Float (Decimal)

```python
budget = fields.Float(
    string='Presupuesto',
    digits=(16, 2),         # (total_dígitos, decimales)
    tracking=True
)
```

##### Monetary (Monetario)

```python
amount = fields.Monetary(
    string='Monto',
    currency_field='currency_id'
)
currency_id = fields.Many2one('res.currency')
```

##### Selection (Selección)

```python
state = fields.Selection([
    ('draft', 'Borrador'),
    ('confirmed', 'Confirmado'),
    ('in_progress', 'En Progreso'),
    ('done', 'Terminado'),
    ('cancelled', 'Cancelado')
], string='Estado', default='draft', required=True, tracking=True)
```

#### 9.1.3. Campos Relacionales

##### Many2one (Muchos a Uno)

```python
manager_id = fields.Many2one(
    'res.users',                    # Modelo relacionado
    string='Gerente del Proyecto',
    default=lambda self: self.env.user,  # Usuario actual
    required=True,
    ondelete='restrict',            # 'cascade', 'set null', 'restrict'
    domain=[('active', '=', True)], # Filtro de registros
    context={'default_active': True},
    tracking=True
)
```

##### One2many (Uno a Muchos)

```python
task_ids = fields.One2many(
    'project.task.advanced',    # Modelo hijo
    'project_id',               # Campo que apunta al padre
    string='Tareas',
    copy=True                   # Se copian al duplicar
)
```

##### Many2many (Muchos a Muchos)

```python
member_ids = fields.Many2many(
    'res.users',                # Modelo relacionado
    'project_user_rel',         # Tabla intermedia
    'project_id',               # Campo ID en tabla intermedia
    'user_id',                  # Campo del modelo relacionado
    string='Miembros del Equipo'
)
```

#### 9.1.4. Campos Computados con Dependencia

##### @api.depends

```python
@api.depends('date_start', 'date_end')
def _compute_duration(self):
    for project in self:
        if project.date_start and project.date_end:
            delta = fields.Date.from_string(project.date_end) - \
                    fields.Date.from_string(project.date_start)
            project.duration_days = delta.days
```

### 9.2. views.xml

#### 9.2.1. Estructura Básica

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Contenido de vistas, menús, acciones, etc. -->
</odoo>
```

#### 9.2.2. Vista Lista

```xml
<record id="view_project_advanced_tree" model="ir.ui.view">
    <field name="name">project.advanced.tree</field>
    <field name="model">project.advanced</field>
    <field name="arch" type="xml">
        <list string="Proyectos" 
              decoration-muted="state == 'cancelled'"
              decoration-success="state == 'done'"
              decoration-danger="state == 'draft'"
              decoration-info="state == 'in_progress'"
              sample="1"
              multi_edit="1"
              editable="bottom">
            
            <field name="name"/>
            <field name="state" widget="badge"/>
        </list>
    </field>
</record>
```

#### 9.2.3. Acción URL

```xml
<record id="action_project_help" model="ir.actions.act_url">
    <field name="name">Documentación</field>
    <field name="url">https://www.odoo.com/documentation</field>
    <field name="target">new</field> <!-- new, self -->
</record>
```

#### 9.2.4. Menús

```xml
<!-- Menú raíz -->
<menuitem id="menu_project_advanced_root"
          name="Proyectos Avanzados"
          sequence="10"
          web_icon="project_advanced,static/description/icon.png"/>

<!-- Submenú con acción -->
<menuitem id="menu_project_all"
          name="Todos los Proyectos"
          parent="menu_project_advanced_root"
          action="action_project_advanced"
          sequence="1"
          groups="base.group_user"/>

<!-- Menú de configuración -->
<menuitem id="menu_project_config"
          name="Configuración"
          parent="menu_project_advanced_root"
          sequence="99"
          groups="base.group_system"/>
```

**Atributos de menuitem:**
- `id`: Identificador único
- `name`: Nombre visible
- `parent`: Menú padre
- `action`: Acción a ejecutar
- `sequence`: Orden de aparición
- `groups`: Grupos con acceso
- `web_icon`: Icono del módulo

### 9.3. ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_project_user,project.advanced.user,model_project_advanced,base.group_user,1,1,1,0
access_project_manager,project.advanced.manager,model_project_advanced,project.group_manager,1,1,1,1
```

---

## 10. Activación del Módulo

Aplicaciones → buscar por nombre → Activar

---
## 11. Comandos Adicionales

```bash
# Ver contenedores en ejecución
docker ps

# Iniciar contenedores (en segundo plano)
docker compose up -d

# Ver logs en tiempo real
docker compose logs -f

# Ver logs de un servicio específico
docker compose logs -f web
docker compose logs -f db

# Detener contenedores
docker compose down

# Reiniciar contenedores
docker compose restart

# Reiniciar un servicio específico
docker compose restart web

# Entrar al contenedor de Odoo
docker exec -it odoo-web bash
---

**Última actualización:** 2026  
**Versión de Odoo:** 18  
**Versión de PostgreSQL:** 15
