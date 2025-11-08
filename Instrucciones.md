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

1. Instalar ssh en la VM.
        
        sudo apt update
        
        sudo apt install ssh

2. Revisar que el servicio se encuentra activado.

        systemctl status ssh
    
    Si este servicio sale como desactivado faltaria activarlo y si quieres que siempre este activo al arrancar utilizar enable.

        sudo systemctl start ssh

        sudo system enable ssh

3. Configurar el reenvio de puerto de VirtualBox.

    **NAT** configurar directamente en el aparta red de la misma maquina y en el caso de usar (ya que este es solo para la maquina). 
    
    **Red NAT** en la maquina tendria que configurar el reenvio de puertos desde el apartado preferencias de VB y luego configurar las redes crear una red y configurarla (ya que esta es una red interna de maquinas).

    configuración:
        
    | Nombre | Protocolo | Ip H.| P.H. | Ip G. | P.G. |
    |-----------------|------------|------------|------------|--------------|--------------|
    | ssh | TCP || 2220 | Ip (en el caso de red NAT) | 22 |

4. Accerder desde powershell.
        
        ssh -p 2220 user@127.0.0.1
5. Por ultimo accedemos yaceptamos la fingerprint y listo ya estariamos en la VM con ssh.

## 2. Instalación Odoo
### 2.1 Instalación en Ubuntu server ()

1. Para la descarga de Odoo directamente en ubuntu,  seguir los pasos en la pagina oficial de Odoo 
    (https://www.odoo.com/documentation/18.0/es/administration/on_premise/packages.html)
    #### Comandos utilizados
    Intalación de Postgresql:
    ```shell
        sudo apt install postgresql -y
    ```
    Instalación de Odoo:
    ```shell
        wget -q -O - https://nightly.odoo.com/odoo.key | sudo gpg --dearmor -o /usr/share/keyrings/odoo-archive-keyring.gpg

        echo 'deb [signed-by=/usr/share/keyrings/odoo-archive-keyring.gpg] https://nightly.odoo.com/18.0/nightly/deb/ ./' | sudo tee /etc/apt/sources.list.d/odoo.list

        sudo apt-get update && sudo apt-get install odoo
    ```
2. Revisar que el servicio este activo
    ```shell  
        systemctl status odoo
    ```
    Si este no esta activo:
    ```shell 
        systemctl start odoo
    ```
    por ultimo para activar y desactivar el arranque al iniar el sistema
    ```shell
        systemctl enable odoo

        systemctl disable odoo
    ```
2.  Una vez instalado y puesto en marcha solo faltaria entrar en Odoo atravez del puerto 8069

    Pasos adicinales (en caso de reenvio de puertos):
    
      | Nombre | Protocolo | Ip H.| P.H. | Ip G. | P.G. |
       |-----------------|------------|------------|------------|--------------|--------------|
       | Odoo | TCP || 8069 | Ip (en el caso de red NAT) | 8069 |
3. Rellenar el formulario de Odoo
4. Finalmente acceder a Odoo con las credenciales anteriormente realizadas.

### 2.2 Instalación de Odoo para desarrollo con Docker

1. Instalación de Docker desde el repositorio de Docker-CE

    1. Eliminar versiones antiguas de Docker Engine
         ```shell       
            for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
        ```
    2. : Incluir el repositorio de Docker CE


        Descarga de la clave de cifrado
        ```shell
            sudo apt-get update

            sudo apt-get install ca-certificates curl
            sudo install -m 0755 -d /etc/apt/keyrings

            sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

            sudo chmod a+r /etc/apt/keyrings/docker.asc
        ```
        Revisión de la versión de linux ya que las versiones que no sean "Noble" deberan ser especificadas en el comando.
        
        Noble
         ```shell
            echo \
                "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

            sudo apt-get update
         ```
        No Noble
         ```shell
            echo \"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc]https://download.docker.com/linux/ubuntu noble stable" | \

            sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

            sudo apt-get update
         ```
    3. Instalar Docker Engine CE

        Actualizar el índice de paquetes e instalar la última versión de Docker Engine CE
        ```shell

            sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
        ```
    4. Comprobar la instalación
        ```shell
            sudo docker version
        ```
    #### Recomendación post instalación

    #### Permitir administrar Docker con usuarios sin privilegios

    Crear el grupo docker
    ```shell
        sudo groupadd docker
    ```
    Añadir a los usuarios deseados a ese grupo

     ```shell
        sudo usermod -aG docker $USER
     ```
    Especificar que el grupo docker administra el fichero docker.sock

     ```shell
        newgrp docker
     ```
    Comprobamos que ya lo tienes
    ```shell
        docker run hello-world
    ```
    #### Activar y desactivar arranque al iniciar el sistema
     ```shell
        sudo systemctl enable docker

        sudo systemctl disable docker
     ```
2. Crear el contenedor de PostgreSQL con:

     ```shell
        docker run -d -v /home/usuario/OdooDesarrollo/dataPG:/var/lib/postgresql/data -e POSTGRES_USER=odoo -e POSTGRES_PASSWORD=odoo -e POSTGRES_DB=postgres --name db postgres:15
     ```

3. Crear el contenedor con Odoo

     ```shell
        docker run -d -v /home/usuario/OdooDesarrollo/volumesOdoo/addons:/mnt/extra-addons -v /home/usuario/OdooDesarrollo/volumesOdoo/firestore:/var/lib/odoo/filestore -v /home/usuario/OdooDesarrollo/volumesOdoo/sessions:/var/lib/odoo/sessions -p 8069:8069 --name odoodev --user="root" --link db:db -t odoo:18 --dev=all
    ```

    Nota: Para poder desarrollar sin problemas, es recomendable darle todos los permisos al directorio “/home/usuario/OdooDesarrollo/volumesOdoo/addons”, con un comando similar a “sudo chmod -R 777 /home/usuario/volumesOdoo/addons”
 
 ### 2.3 Instalación Odoo con Docker Compose
 
 1. Crear el fichero docker-compose.yml
    ```shell
        sudo nano docker-compose.yml
    ```
 2. Añadir al fichero docker-compose.yml lo siguiente:

    ```yml
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
    ```shell
        docker compose up -d
    ```
    #### Para detener
    ```shell
        docker compose down
    ```
### 
