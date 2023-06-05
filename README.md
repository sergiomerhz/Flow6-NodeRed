# Flow6-NodeRed
Este repositorio muestra una estación climática que obtiene los datos climáticos y guarda un registro con MySQL. Se hace uso de un broker publico para darle un carácter colectivo. Este ejercicio funciona mejor con varios participantes reportando a la vez, pues se grafícan los datos de todos.

## Introducción

### Descripción

El flow 6 es una estación climática que muestra los datos locales de temperatura y humedad via MQTT, ya sea de forma manual con terminal o con un micro controlador. Cuenta con una sección que toma la misma información tomada de [Open Weather Map](https://openweathermap.org/) via API, la guarda en variables locales, la envía en un JSON a través de MQTT a un broker publico, se suscribe al mismo broker público y muestra la información en graficos separados para temperatura y humedad. La característica de envíar y recibir la info de la API via MQTT permite mostrar la información de multiples usuarios de forma colaborativa.

### Alcances

Este ejercicio funciona mejor con multiples usuarios reportando al mismo tema a través de un broker público.

## Requisitos

### Material Necesario

Para realizar este flow necesitas lo siguiente

- [Ubuntu 20.04](https://releases.ubuntu.com/20.04/)
- [Docker Engine](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script)
- [NodeRed](https://nodered.org/docs/getting-started/local)
- [Nodos Dashboard](https://flows.nodered.org/node/node-red-dashboard)
- [MySQL](https://hub.docker.com/_/mysql)

### Servicios

Necesitas una cuenta gratuita del siguiente servicio
- [Open Weather Map](https://openweathermap.org/)

### Material de referencia

En los siguientes enlaces puedes encontrar cursos en la plataforma de edu.codigoiot.com que te permitirán realizar las configuraciones necesarias

- [Instalación de Virutal Box y Ubuntu 20.04](https://edu.codigoiot.com/course/view.php?id=812)
- [Introducción a Docker](https://edu.codigoiot.com/course/view.php?id=996)
- [Aplicacion multicontenedor de servidor IoT con Docker compose](https://edu.codigoiot.com/mod/lesson/view.php?id=3889&pageid=3804&startlastseen=no)
- [Servidor del Internet de las Cosas con nodeRed](https://edu.codigoiot.com/course/view.php?id=997)
- [Accediendo al servidor de Bases de Datos](https://edu.codigoiot.com/course/view.php?id=1001)

## Instrucciones

### Requisitos previos

Para poder usar este repositorio necesitas lo siguiente

1. Docker Engine.
2. NodeRed por Docker Compose.
3. Contenedor de NodeRed con el volumen de data activado.
4. Contenedor de MySQL con el volumen de configuración y data activado. Configurar el archivo `my.cnf` con un `bind-address = 0.0.0.0`
5. Contenedor de Mosquitto con el volumen de configuración activado. Configurar el archivo ```mosquitto.conf``` con un listener en el puerto ```1883``` para todas las IPs ```0.0.0.0```.
6. Tu usuario de linux debe ser parte del grupo sudoers y del grupo docker
    - Puedes comprobar que tu usuario está en ambos grupos con el comando 
    
        ```
        groups $USER
        ```

    - En caso de que tu usuari no forme parte de dichos grupos, puedes arreglarlo con los siguientes comandos
        ``` 
        su
        sudo usermod -aG sudo newuser
        exit
        ```
7. Crea una base de datos para este ejercicio. Usa los siguientes comandos
    - Entra a MySQL con el comando `docker exec -it [id_contenedor] mysql -p`
    - Usa la contraseña que configuraste en el archivo **compose.yaml**, si usaste el repositorio del profesor [servidor-IoT-basico-docker-compose](https://github.com/hugoescalpelo/servidor-IoT-basico-docker-compose), la contraseña predeterminada es `my-secret-pw`
    - Crea una nueva base de datos  con el comando `CREATE DATABASE cursoIoT;`
    - Selecciona la base de datos con el comando `USE cursoIoT;`
    - Crea una tabla con el comando siguiente:
        ```
        CREATE TABLE clima (id INT(6) UNSIGNED AUTO_INCREMENT PRIMARY KEY, fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP, nombre VARCHAR(248) NOT NULL, temperatura FLOAT(4,2) NOT NULL, humedad INT (6) NOT NULL);
        ```
    - Crea un usuario con acceso remoto con el comando `CREATE USER 'usuario'@'%' IDENTIFIED BY 'password';` Cambia la palabra **usuario** y **password** para personalizar el usuario creado. Usa el comando sin quitar comillas simples. No olvides el punto y coma.
    - Da acceso al usuario creado con el comando `GRANT ALL PRIVILEGES ON *.* TO 'user'@'%';`    
    
### Instrucciones de preparación de entorno

Para arrancar el entorno necesario, puedes usar los siguientes comandos.

1. Comprueba que los contenedores de Mosquitto, NodeRed y MySQL estén funcionado. Puedes comprobarlo con el comando ```docker ps -a```. En caso de que tus contenedores no estén funcionando, puedes arrancarlos con el comando ```docker start $(docker ps -a -q)```.
2. Comprueba que tengas acceso a la API de Open Weather con la siguiente consulta desde cualquier navegador ```https://api.openweathermap.org/data/2.5/weather?lat=[latitud]&lon=[longitud]&appid=[api_key]&units=metric```. Agrega tu API Key, latitud y longitud de tu ubicación geográfica sin corchetes. Deberías ver un JSON con los datos del clima.
3. Asegurate de tener instalados los nodos ```node-red-dashboard``` en nodeRed.
4. Importa el archivo flows.json a nodeRed y haz clic en el boton **Deploy**.
5. Comprueba que el nodo MQTT de la sección **Local** apunte al servidor de tu broker local. Si usas Docker Compose, usa el nombre de la aplicación de Mosquitto usado en el archivo compose.yaml como nombre de dominio, en nuestro caso, ```mosquitto```. Si usas una instalación local, usa ```localhost``` o ```127.0.0.1```. Si usas un broker publico usa preferentemente la IP del broker en lugar del nombre de dominio.
6. Comprueba que los nodos MQTT de la sección **Enviador** y **Escuchador** estén suscritos al tema `codigoIoT/mqtt/climaAPI` y apunten a la misma IP del broker público.
7. Verifica que todos los nodos JSON estén configurados para siempre convertir a JSON.
8. Asegurate que los nodos function contienen el código correcto. No olvides cambiar el nombre de usuario y ciudad en el **nodo function JSON publico**.

    Nodo function Temperatura

    ```
    msg.payload = msg.payload.temp;
    msg.topic = "Temperatura";
    return msg;
    ```
    Nodo function Humedad
    ```
    msg.payload = msg.payload.hum;
    msg.topic = "Humedad";
    return msg;
    ```
    Nodo function Temperatura API
    ```
    global.set ("tempAPI", msg.payload.main.temp);
    msg.payload = msg.payload.main.temp;
    msg.topic = "Temperatura";
    return msg;
    ```
    Nodo function Humedad API
    ```
    msg.payload = msg.payload.main.humidity;
    global.set ("humAPI", msg.payload);
    msg.topic = "Humedad";
    return msg;
    ```
    Nodo function JSON Público
    ```
    msg.payload = '{"id":"usuario, ciudad","temp":' + global.get("tempAPI")+',"hum":' + global.get ("humAPI") +'}';
    return msg;
    ```
    Nodo function Temperatura Pública API
    ```
    msg.topic = msg.payload.id;
    msg.payload = msg.payload.temp;
    return msg;
    ```
    Nodo function Humedad Pública API
    ```
    msg.topic = msg.payload.id;
    msg.payload = msg.payload.hum;
    return msg;
    ```
    Nodo function Query
    ```
    msg.topic = "INSERT INTO clima (nombre,temperatura,humedad) VALUES ('" + msg.payload.id +"'," + msg.payload.temp + "," + msg.payload.hum + ");";
    return msg;
    ```
9. Asegurate de configurar los nodos dashboard para que se representen en una pestaña y un grupo existente.
10. Verifica que los nodos **inject** estén lanzando un timestamp cada minuto.
11. Comprueba que tu broker mosquitto sea accesible desde el exterior del contenedor. La forma fácil de hacerlo, es verificar que el nodo MQTT del flow indique **conectado**.
12. Comprueba que tengas conexión con el broker público. La forma fácil de hacerlo, es verificar que el nodo MQTT del flow indique **conectado**.
13. Verifica que el nodo MySQL está conectado. Para ello, debes agregar ua nueva Data Base y configurar lo siguiente.
    - Host: nombre de dominio del servidor de MySQL. Si usas Docker Compose, usa el nombre de la aplicación del archivo compose.yaml, en nuestro caso `mysql`. Si usas una instalación local, escribe `localhost`. Si usas una base de datos remota, usa la **IP** o el nombre del dominio de tu servidor.
    - Port: 3306 para instalaciones locales. Si el puerto de tu base de datos está en un puerto diferente, usa ese.
    - User: Nombre de usuario de MySQL con privilegios de acceso remoto.
    - Password: La contraseña del usuario de MySQL usado.
    - Database: Nombre de la base de datos. En nuestro caso `cursoIoT`
    - Deja el resto de las opciones con el valor predeterminado.

### Instrucciónes de operación

1. Dirígete al dashboard en [localhost:1880/ui](http://locahost:1880/ui/)
2. Espera al menos 2 minutos despues de haber importado el flow y haber hecho clic en el botón **Deploy** para observar datos en el bloque API. 
3. Envía al menos dos mensajes MQTT que incluyan un JSON con la temperatura y la humedad al tema `codigoIoT/mqtt/clima` de tu broker local. Ejemplo.

    ```
    docker exec -it [id_contenedor] mosquitto_pub -h localhost -t codigoIoT/mqtt/clima -m '{"temp":23,"hum":50}'
    ```

## Resultados

Cuando haya funcionado, verás los indicadores con valores y las gráficas indicando los cambios históricos. 

![](https://github.com/sergiomerhz/Flow6-NodeRed/blob/main/Imagenes/Captura%20desde%202023-05-29%2021-42-47.png?raw=true)

![](https://github.com/sergiomerhz/Flow6-NodeRed/blob/main/Imagenes/Captura%20desde%202023-05-29%2021-42-05.png?raw=true)

![]()

**Consejo**: Intenta cambiar el rango de las gráficas historicas a 1 semana y deja el flow funcionando por varios días, observarás la dinamica de clima.
**Consejo**: Intenta usar este flow en presencia de otros usuarios, verás la información de cada uno de ellos en las gráficas de la sección colaborativa.

## Evidencias

[]()

## FAQ

- **P**. ¿Còmo agrego un volumen que contenga el archivo de configuración de mosquitto?. **R**. Realiza los siguientes pasos.

    - Detener el contenedor de Mosquitto 
    
        ```docker stop [id_contenedor]```
        
    - Eliminar el contenedor de Mosquitto
    
        ```docker rm [id_contenedor]```
        
    - Eliminar la imagen de Mosquitto

        ```
        docker images
        docker rmi [id_imagen]
        ```
    - Detener docker compose. Hay que entrar al directorio de compose

        ```docker compose stop```
        
    - Actualizar el archivo compose para que use el volumenes. 

    - Agrega tu archivo mosquitto.conf. Puedes tomar como ejemplo el archivo de configuración del repositorio del profesor [servidor-IoT-basico-docker-compose](https://github.com/hugoescalpelo/servidor-IoT-basico-docker-compose)

    - Levantar docker compose

        ```docker compose up -d```

- **P**. ¿Cómo agrego un volumen que contenga el archivo de configuración y datos externos a MySQL?. **R**. Realiza los siguientes pasos.

    - Detener el contenedor de MySQL 
    
        ```docker stop [id_contenedor]```
        
    - Eliminar el contenedor de MySQL
    
        ```docker rm [id_contenedor]```
        
    - Eliminar la imagen de MySQL

        ```
        docker images
        docker rmi [id_imagen]
        ```
    - Detener docker compose. Hay que entrar al directorio de compose

        ```docker compose stop```
        
    - Actualizar el archivo `compose.yaml` para que use el volumenes.

    - Agrega tu archivo `my.cnf`. Puedes tomar como ejemplo el archivo de configuración del repositorio del profesor: [servidor-IoT-basico-docker-compose](https://github.com/hugoescalpelo/servidor-IoT-basico-docker-compose)

    - Levantar docker compose

        ```docker compose up -d```
- **P**. ¿Cómo le hago para ver la información de multiples usuarios?. **R**. Asegurate que todos los usuarios que deseas que se registren en la gráfica de la sección *Colaborativo* estén publicando la información que reciben por API en el mismo tema que tu y que estén suscritos a la misma IP del broker MQTT que usas.

## Problemas comúnes

- Si usas un broker publico, usa la IP del broker ya que este puede cambiar en cualquier momento y NodeRed no actualiza las IPs luego de configurar un dominio. Para conocer la IP de un broker usa el comando ```nslookup [dominio_broker]```.
- Si la información recibida en el nodo MQTT no es detectada por el nodo JSON, asegurate de que es expresada como Sring.
- Si el nodo JSON marca error o caracter no esperado, asegurate de que el mensaje MQTT que enviaste describe correctamente un JSON. Ejemplo: `docker exec -it [id_contenedor] mosquitto_pub -h localhost -t codigoIoT/mqtt/clima -m '{"temp":23,"hum":50}'`

# Créditos

Desarrollado por Sergio Merino
- [GitHub](https://github.com/sergiomerhz)

Desarrollado por Hugo Escalpelo
- [hugoescalpelo.com](https://hugoescalpelo.com/)
- [Página en Facebook](https://www.facebook.com/Hugo-Escalpelo-Profesional-337708683840136)
- [GitHub](https://github.com/hugoescalpelo)