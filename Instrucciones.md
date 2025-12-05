### Sistemas de Gestión Empresarial Actuales
# Instalación y configuración de un ERP
## 1. Instalación de Ubuntu Server

1. Descargar la imagen de Ubuntu Server 24.04.3.
2. Crear y configurar la machina virtual  (herrramienta de virtualización utilizada **VirtualBox**).
    #### Configuración:
        Procesadores: 3
        Ram: 4096 MB
        Almacenamiento: 80 GB
3. Seguir los pasos de instalación de Ubuntu server **(En mi caso instale ssh en el proceso para facilitarme el uso de comandos)**.

4. Una vez montado ubuntu server solo haria falta iniciar sesión con las credenciales creadas en la instalación.

5. Con esto finalizamos la instalación de ubuntu server.

#### Adicional: **Instalación de ssh** para tener mejor accesibilidad a la linea de comandos.

1. **Instalar ssh en la VM.**
        
        sudo apt update
        
        sudo apt install ssh

2. **Revisar que el servicio se encuentra activado.**

        systemctl status ssh
    
    Si este servicio sale como desactivado faltaria activarlo y si quieres que siempre este activo al arrancar utilizar enable.

        sudo systemctl start ssh

        sudo system enable ssh

3. **Configurar el reenvio de puerto de VirtualBox.**

    **NAT** configurar directamente en el aparta red de la misma maquina y en el caso de usar (ya que este es solo para la maquina). 
    
    **Red NAT** en la maquina tendria que configurar el reenvio de puertos desde el apartado preferencias de VB y luego configurar las redes crear una red y configurarla (ya que esta es una red interna de maquinas).

    configuración:
        
    | Nombre | Protocolo | Ip H.| P.H. | Ip G. | P.G. |
    |-----------------|------------|------------|------------|--------------|--------------|
    | ssh | TCP || 2220 | Ip (en el caso de red NAT) | 22 |

4. **Accerder desde powershell o Símbolo del sistema.**
        
        ssh -p 2220 user@127.0.0.1
5. **Por ultimo accedemos yaceptamos la fingerprint y listo ya estariamos en la VM con ssh.**

## 2. Instalación Odoo
### 2.1 Instalación en Ubuntu server ()

1. **Para la descarga de Odoo directamente en ubuntu,  seguir los pasos en la pagina oficial de Odoo.**
    (https://www.odoo.com/documentation/18.0/es/administration/on_premise/packages.html)
    #### Comandos utilizados
    Intalación de Postgresql:
    ```bash
        sudo apt install postgresql -y
    ```
    Instalación de Odoo:
    ```bash
        wget -q -O - https://nightly.odoo.com/odoo.key | sudo gpg --dearmor -o /usr/share/keyrings/odoo-archive-keyring.gpg

        echo 'deb [signed-by=/usr/share/keyrings/odoo-archive-keyring.gpg] https://nightly.odoo.com/18.0/nightly/deb/ ./' | sudo tee /etc/apt/sources.list.d/odoo.list

        sudo apt-get update && sudo apt-get install odoo
    ```
2. **Revisar que el servicio este activo**
    ```bash  
        systemctl status odoo
    ```
    Si este no esta activo:
    ```bash 
        systemctl start odoo
    ```
    Por ultimo para activar y desactivar el arranque al iniar el sistema
    ```bash
        systemctl enable odoo

        systemctl disable odoo
    ```
3.  **Una vez instalado y puesto en marcha solo faltaria entrar en Odoo atravez del puerto 8069**

    Paso adicinal (en caso de reenvio de puertos `Configuración de red de Vbox`):
    
      | Nombre | Protocolo | Ip H.| P.H. | Ip G. | P.G. |
       |-----------------|------------|------------|------------|--------------|--------------|
       | Odoo | TCP || 8069 | Ip (en el caso de red NAT) | 8069 |
4. **Rellenar el formulario de Odoo**
5. **Finalmente acceder a Odoo con las credenciales anteriormente realizadas.**

### 2.2 Instalación de Odoo para desarrollo con Docker

1. Instalación de Docker desde el repositorio de Docker-CE

    1. Eliminar versiones antiguas de Docker Engine
         ```bash      
            for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
        ```
    2. : Incluir el repositorio de Docker CE

        ```bash
                # Add Docker's official GPG key:
        sudo apt-get update
        sudo apt-get install ca-certificates curl
        sudo install -m 0755 -d /etc/apt/keyrings
        sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
        sudo chmod a+r /etc/apt/keyrings/docker.asc
        
        # Add the repository to Apt sources:
        echo \
          "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
          $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
          sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
        sudo apt-get update
        ```
    3. Instalar Docker Engine CE

        Actualizar el índice de paquetes e instalar la última versión de Docker Engine CE
        ```bash
        sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
        ```
    4. Comprobar la instalación
        ```bash
            sudo docker version
        ```
        #### Recomendación post instalación

        #### Permitir administrar Docker con usuarios sin privilegios

        Crear el grupo docker
        ```bash
            sudo groupadd docker
        ```
        Añadir a los usuarios deseados a ese grupo

        ```bash
            sudo usermod -aG docker $USER
        ```
        Especificar que el grupo docker administra el fichero docker.sock

        ```bash
            newgrp docker
        ```
        Comprobamos que ya lo tienes
        ```bash
            docker run hello-world
        ```
        #### Activar y desactivar arranque al iniciar el sistema
        ```bash
            sudo systemctl enable docker

            sudo systemctl disable docker
        ```
2. Crear el contenedor de PostgreSQL con:

     ```bash
        docker run -d -v /home/usuario/OdooDesarrollo/dataPG:/var/lib/postgresql/data -e POSTGRES_USER=odoo -e POSTGRES_PASSWORD=odoo -e POSTGRES_DB=postgres --name db postgres:15
     ```

3. Crear el contenedor con Odoo

     ```bash
        docker run -d -v /home/usuario/OdooDesarrollo/volumesOdoo/addons:/mnt/extra-addons -v /home/usuario/OdooDesarrollo/volumesOdoo/firestore:/var/lib/odoo/filestore -v /home/usuario/OdooDesarrollo/volumesOdoo/sessions:/var/lib/odoo/sessions -p 8069:8069 --name odoodev --user="root" --link db:db -t odoo:18 --dev=all
    ```

    > Nota: Para poder desarrollar sin problemas, es recomendable darle todos los permisos al directorio “/home/usuario/OdooDesarrollo/volumesOdoo/addons”, con un comando similar a “sudo chmod -R 777 /home/usuario/volumesOdoo/addons”
 
 ### 2.3 Instalación Odoo con Docker Compose
 
 1. Crear el fichero docker-compose.yml
    ```bash
        sudo nano docker-compose.yml
    ```
 2. Añadir al fichero docker-compose.yml lo siguiente:

    ```yml
    version: '3.3'

    services:
        web:
            #Indicamos que imagen de Docker Hub utilizaremos
            image: odoo:18
            container_name: odoo-web
            #Indicamos que depende de "db", por lo cual debe ser procesada primero "db"
            depends_on:
                - db
            # situado en el lugar donde ejecutemos "Docker compose"
            volumes:
            - ./volumesOdoo/addons:/mnt/extra-addons
            - ./volumesOdoo/odoo-web-data:/var/lib/odoo
            #Indicamos que el contenedor funcionara con usuario root y no con usuario odoo
            user: root
            # Definimos variables de entorno de Odoo
            environment:
            - HOST=db
            - USER=odoo
            - PASSWORD=odoo
            - DB_HOST=db
            - DB_PORT=5432
            - DB_USER=odoo
            - DB_PASSWORD=odoo
            - DB_NAME=postgres
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
 3. Comandos de docker compose (se debe estar en el mismo directorio del archivo docker-compose.yml)

    #### Para Iniciar
    ```bash
        docker compose up -d
    ```
    #### Para detener
    ```bash
        docker compose down
    ```
## 3. Configuración de Odoo

### 3.1. Acceso al sistema
    
Paso adicinal (en caso de reenvio de puertos `Configuración de red de Vbox`):
    
| Nombre | Protocolo | Ip H.| P.H. | Ip G. | P.G. |
|-----------------|------------|------------|------------|--------------|--------------|
| Odoo | TCP || 8069 | Ip (en el caso de red NAT) | 8069 |

- Ingresa a tu instancia de Odoo `http://localhost:8069` o si estas con adaptador puenta `http://ipdelserver:8069`.  
- Inicia sesión con un usuario con permisos de administrador o si es primera vez configurar la base de datos y credenciales.

### 3.2. Instalación de módulos.
    
**Ruta:** `Aplicaciones`

- Activar los modulos necesarios en este caso:
    `Ventas`, `Facturación`, `Inventario`, `Compra` y `Empleados`

### 3.3. Crear y configurar

1. **Compañia**

     **Ruta:** `Ajustes → Usuarios y Compañías → Compañías`

    - **Nombre de la empresa:** introduce el nombre comercial.  
    - **Dirección fiscal:** completa los datos oficiales.  
    - **Teléfono y correo electrónico:** datos de contacto corporativos.  
    - **Moneda y zona horaria:** ajusta según el país donde opera la empresa.  
    - **Logo:** sube el logotipo de la empresa para personalizar los reportes y el portal.  
    - Guarda los cambios.

2. **Activar modo desarrollador (opcional)**

    Para acceder a configuraciones avanzadas:

    - Ve a `Ajustes → Acerca de Odoo → Activar modo desarrollador`.
    - Con esto podrás acceder a menús ocultos como:
    - Parámetros del sistema
    - Alias de correo
    - Modelos de datos
    - Secuencias de documentos

3. **Configuración de secuencias y sufijos de documentos**

    Odoo utiliza **secuencias automáticas** para generar numeraciones únicas en documentos como **facturas**, **pedidos de venta**, **albaranes** y más.

    **Ruta:** `Ajustes → Parámetros técnicos → Secuencias e identificadores → Secuencias`

    > Si no ves esta opción, activa el **Modo Desarrollador** .

    #### Ejemplo de configuración:
    - **Nombre:** Facturas de cliente  
    - **Prefijo (Prefix):** `FAC/%(year)s/` → genera algo como `FAC/2025/0001`  
    - **Sufijo (Suffix):** puedes añadir letras o códigos adicionales, por ejemplo `-A` → `FAC/2025/0001-A`  
    - **Número siguiente:** 1  
    - **Incremento:** 1  
    - **Tamaño del número:** 4 (para mostrar 0001, 0002, etc.)

    #### Ejemplo de prefijos/sufijos comunes:
    | Documento | Prefijo sugerido |
    |------------|------------------|
    | Facturas de clientes | `FAC/%(y)s/` |
    | Facturas de proveedores | `FAP/%(y)s/` |
    | Pedidos de venta | `PED/%(y)s/` |
    | Albaranes | `A/%(y)s/` |
    | Compras | `COM/%(y)s/` |


4. **Creación de usuarios y asignación de permisos**

    **Ruta:** `Ajustes → Usuarios y Compañías → Usuarios`

    - Pulsa **Crear** y completa los datos:
    - **Nombre:** nombre completo del usuario.  
    - **Correo electrónico:** será el login del usuario.  
    - **Idioma y zona horaria:** define el idioma del sistema para él.  
    - **Compañía principal:** asigna la empresa (si hay varias).  
    - **Grupos de acceso:** define los permisos según el rol (por ejemplo: *Ventas / Usuario*, *Inventario / Administrador*, etc.)


    #### Ejemplo de roles comunes:
    | Rol | Módulos principales | Grupos sugeridos |
    |------|---------------------|------------------|
    | Administrador | Todos | Acceso total |
    | Vendedor | Ventas | Usuario de ventas |
     | Contable | Facturación | Usuario de contabilidad |
    | Almacén | Inventario | Usuario de inventario |
    | RRHH | Empleados | Gerente de RRHH |

    Una vez guardado el usuario, Odoo enviará (si está configurado el correo saliente) un email de invitación con enlace para establecer su contraseña.

5. **Configuración del correo electrónico (IMAP / SMTP)**

    Para habilitar el envío y recepción de correos desde Odoo:

    #### Servidor saliente (SMTP)
    **Ruta:** `Ajustes → Parámetros técnicos → Servidores de correo saliente`

    | Campo | Ejemplo | Descripción |
    |--------|----------|-------------|
    | Nombre del servidor | smtp.qbox | Dirección del servidor SMTP |
    | Puerto | 465 | Usa 465 para SSL, 587 para STARTTLS |
    | Usuario | usuario@empresa.com | Correo corporativo |
    | Contraseña | ******** | Clave o contraseña de aplicación |
    | SSL/TLS | ✅ | Conexión segura  |

    > Pulsay **Probar conexión** para verificar la configuración.

    #### Servidor entrante (IMAP)
    **Ruta:** `Ajustes → Parámetros técnicos → Servidores de correo entrante`

    | Campo | Ejemplo | Descripción |
    |--------|----------|-------------|
    | Nombre | imap_qbox | 
    | Tipo | IMAP |
    | Servidor | imap.qboxmail.com |
    | Puerto | 993 | IMAP SSL |
    | Usuario | usuario@empresa.com |
    | Contraseña | ******** |
    | SSL/TLS | ✅ | Conexión segura |

    >  Pulsar **Probar y confirmar** para verificar la conexión.

## 4. Exportación e Importación de Odoo con GitHub

### 4.1 Exportación con GitHub


#### Crear un archivo comprimido del proyecto

```bash
tar -cvzf nombre.tar volumesOdoo/
```
> Este comando genera un archivo .tar con todo el contenido del directorio.

#### Para una compresión más eficiente, puedes usar el formato .tar.xz:

```bash
tar cvf micomprimido.tar.xz -I 'xz -9' directorio_a_comprimir/
```
#### Inicializar el repositorio Git
```bash
git init
```
> Inicializa un nuevo repositorio Git en el directorio actual.

#### Cambiar el nombre de la rama principal a main:
```
git branch -m main
```
#### Verificar en qué rama te encuentras:
```bash
git branch --show-current
```

#### Configurar seguridad y entorno
En el caso de que Git te pida marcar el directorio como seguro:
```bash
git config --global --add safe.directory ruta_del_directorio
```

> Esto es común en sistemas donde Git no reconoce el entorno como seguro (por ejemplo, contenedores o entornos de red).

#### Preparar y subir los archivos al repositorio remoto
Se debe agregar los archivos al área de preparación:
```bash
git add ruta_del_archivo
```
#### Para agregar todo el contenido del directorio:
```bash
git add .
```

#### Comprobar el estado del repositorio:
```bash
git status
```

#### Realizar el commit:
```
git commit -m "Comentario sobre los cambios"
```
> Si es la primera vez que haces commit, Git te pedirá configurar tu usuario y correo

#### Configurar tu usuario y correo
```bash
git config --global user.name "TuNombre"
git config --global user.email "tuemail@dominio.com"
```
#### Conectar con el repositorio remoto de GitHub
se debe agregar la URL del repositorio remoto:
```
git remote add origin https://github.com/usuario/repositorio.git
```

Verificar que se haya agregado correctamente:
```bash
git remote -v
```

Subir los archivos a GitHub:
```bash
git push -u origin main
```

>Git pedirá tus credenciales.
En el campo de contraseña, introducir tu token personal de GitHub (no la contraseña).

### En caso de tener más de una rama:
```bash
git add ruta
git commit -m "Comentario"
git remote add origin https://github.com/usuario/repositorio.git
git remote -v
git fetch --all
git checkout nombre_de_la_rama
git push -u origin main
```

### Subir más ficheros sin reemplazar los existentes
```
git pull --rebase origin main
```
> No usar `git add` antes de este comando o se generarán conflictos.

Luego añadir los nuevos archivos:

```bash
git add directorio_o_fichero
git status
git commit -m "Añadido nuevo contenido (por ejemplo, docker-compose)"
git push origin main
```

### 4.2 Importación con GitHub

Para descargar un repositorio existente de GitHub (por ejemplo, para recuperar una copia de Odoo):

```bash
git clone https://github.com/usuario/repositorio.git
```
> Si es un repositorio público, no pedirá credenciales.
El proyecto se descargará en el directorio donde ejecutes el comando.

#### Descomprimir el archivo
```bash
tar xvf micomprimido.tar
```
> Esto extraerá el contenido del archivo .tar o .tar.xz en el directorio actual.

### 4.3 Ignorar archivos innecesarios con .gitignore (opcional pero recomendado)
Cuando trabajas con Odoo y Docker, hay archivos que no deberías subir a GitHub, como:

- Archivos temporales
- Logs
- Copias de seguridad
- Volúmenes grandes de bases de datos

> Crea un archivo llamado .gitignore en la raíz del proyecto y agrega contenido que no sea desea subir.

### 4.4 Preparar el fichero volumesOdoo

Una vez realizado lo anterior (Importanción) debemos remplazar los ficheros correspondientes (dataPostgreSQL y odoo-web-data) o todo el directorio volumesOdoo.

Ademas debemos cambiar en el fichero docker-compose.yml cambiando `DB_PASSWORD`,  `DB_NAME` por los que haz colocado al momento de abril Odoo por primera vez.

```yml
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
### 4.5 Puesta en marcha de Odoo

Arrancamos docker compose

```bash
docker compose up -d
```
Y finalizamos abriendo Odoo por el navegar y utilizando las credenciales 

    Database Name: odoo_db

    email: rubeninform123@gmail.com

    Password: Hola123_

## 5. Personalización de plantillas y automatización de Odoo

### 5.1 Perzonalizar plantillas de correo
Empezamos activando el modo desarrollador (mencionado en el apartado 2 del punto 3.3 )

Ahora creamos y configuramos la plantilla en:
    **RUTA:** `configuración → tecnico → Platillas de correo`

 - Definimos  `Nombre, Modulo que usaremos como base de los datos para la plantilla`

| Campo | Ejemplo |
|--------|----------|
| Nombre | [CUSTOM] Pedido de venta | 
| Aplica a | Pedido de venta |

- Definimos los campos de correo con Marcadores de posición dinámicos


#### **Marcadores de posición dinámicos**
    
**Asunto del correo:**
    
`Gracias por tu compra {{object.name}}`
    
> Para los marcadores en el asunto siempre se debe utilizar doble llave "{{}}"

**Cuerpo del correo:**

Hola, `object.partner_id.name` (Cliente → Nombre)

Gracias por la compra `object.name` (Referencia del pedido)

Monto total: `object.amount_total` (Total)

> Para los comandas en el cuerpo del correo debe utilizarce "/" en el cual se debe buscar la herramienta `Marcador de posición dinámico`
y buscar los campos correspondientes a los comandos
### 5.2 Crear la automatización
**RUTA:** `configuración → tecnico → Reglas de automatización`

Creamos una nueva regla y llenamos los campos `Nombre, Modelo, Activador y acción`

| Campo | Ejemplo | Paso adicional |
|--------|----------|-------|
| Nombre | Aut. Venta | 
| Modelo | Pedido de venta |
| Activador | El estado está establecido como | Pedido de venta |
| Acción | Enviar correo electrónico | [CUSTOM] Pedido de venta |
### 5.3 Actividades de Automatización
#### **Ejercicio 1. Mail automático**
Automatización de notificaión por correo de Oportunidades de venta mayores a 20.000€

**Intalar la aplicación CRM**

**RUTA:** `Aplicaciones`

Pulsar activar al ver el modulo CRM

> Este es necesario ya que contiene `Oportunidades de venta`

**Crear plantilla para la oportunidad**
| Campo | Ejemplo |
|--------|----------|
| Nombre | [CUSTOM] Oportunidad de venta | 
| Aplica a | Lead/Oportunidad |

**Asunto del correo:**
    
`Oportunidad de venta mayor a 20.000€ en {{object.name}}`

**Cuerpo del correo:**

Se ha detectado una oportunidad de venta mayor a 20.000€ en la empresa "`object.partner_id.name` (Cliente → Nombre)":

Nombre de la Oportunidad de venta: `object.name` (Oportunidad)

Ingresos esperados: `object.expected_revenue`€ (Ingresoso esperados)

Enlace a oportunidades (/enlace → URL al modulo CRM)

**Automatizar**
| Campo | Ejemplo | Paso adicional |
|--------|----------|-------|
| Nombre | Nueva oportunidad de venta > 20.000 | 
| Modelo | Lead/Oportunidad |
| Activador | Al guardar | Pedido de venta |
|Coincidir todas de las siguientes reglas | Editar dominio | `Nueva regla` Ingresos esperados > 20000 |
| Acción | Enviar correo electrónico | [CUSTOM] Oportunidad de venta |

Creamos una nueva oportunidad en el modulo CRM que sea mayor a 20.000€ para comprobar

#### **Ejercicio 2**
