#### Examen SRI - Johan Erazo

### 1. Explica métodos para 'abrir' unha consola/shell a un contenedor en execución.

Una vez iniciamos el contenedor para abrirlo en una consola/shell. Ejecutamos el comando: docker exec -it (nombre del contenedor).

### 2. No contenedor anterior (en execución), qué opciones tes que ter usado ó arrincalo para poder interactuar coas súas entradas e salidas.

para poder interactuar primero se arranca con el comando: docker run -d (name) -p 8080:80 -v

### 3. Cómo sería un ficheiro docker-compose para que dous contenedores se comuniquen entre si nunha rede só deles?

Para hacer esto cree una carpeta de trabajo y dentro cree el archivo compose.yml donde añadi los contenedores con sus ip hasta con su rango maximo.
```
version: '3'

services:
  practica7:
    container_name: 7practica
    # usamos esta imagen para poder ver los logs
    # la imagen de internetsystemsconsortium/bind9
    # no muestra los logs
    image: internetsystemsconsortium/bind9:9.18
    # especificamos la plataforma de la imagen
    # es posible conflicto cuando el host es 'arm'
    # ya que no existe una imagen para esa plataforma
    platform: linux/amd64
    ports:
      - "54:53/udp"
      - "54:53/tcp"
    networks:
      bind9_subnet:
        ipv4_address: 192.168.23.1
    volumes:
      - ./conf:/etc/bind
      - ./zonas:/var/lib/bind
    restart: always
  cliente:
    container_name: cliente2
    image: alpine:latest
    platform: linux/amd64
    tty: true
    stdin_open: true
    # configuramos para que el cliente use nuestro dns
    dns:
      - 192.168.23.1
    networks:
      bind9_subnet:
        ipv4_address: 192.168.23.2
    
networks:
  bind9_subnet:
    driver: bridge
    ipam:
      config:
        - subnet: 192.168.0.0/16

```
### 4. Qué tes que engadir ó ficheiro anterior para que un contenedor teña unha IP fixa?

Para la IP fija en este caso está añadida en el apartado;
```
    networks:
      bind9_subnet:
        ipv4_address: 192.168.23.2
```
### 5. Qué comando de consola podo usar para sabe-las ips dos contenedores anteriores?

Con el comando: (docker network inspect practica7_bind9_subnet) en este caso nos saldra toda la información de la red y lo que nos interesa es lo siguiente: 
```
       "Containers": {
            "a1f9cc03381cdc8c35a05985f4c743ac0e8fad6d669220f23516cb57a13ce6ba": {
                "Name": "7practica",
                "EndpointID": "3397e750ad056f7370f598d52897d7f35b81d1c1acae7ac1c278b3052defa543",
                "MacAddress": "02:42:c0:a8:17:01",
                "IPv4Address": "192.168.23.1/16",
                "IPv6Address": ""
            },
            "d16e16201269d6924bbf7e8437ef34ffc5600cfdeff7259e78f45899ed8daacc": {
                "Name": "cliente2",
                "EndpointID": "72c80ea8b98b739bb882d031346f8ca7306f0af9822dc2532a27d419d6506590",
                "MacAddress": "02:42:c0:a8:17:02",
                "IPv4Address": "192.168.23.2/16",
                "IPv6Address": ""
            }
```

### 6. Cál é a funcionalidade do apartado "ports" en docker compose?

Los "ports" están para poder conectarse al tipo de red que escogamos.

### 7. Para qué serve o rexistro CNAME? Pon un exemplo

Sirve para poner un alias al nombre canonico. 
Ejemplo: 
            alias	IN CNAME	web.erazo.castelao.com.

### 8. Cómo podo facer para que a configuración dun contenedor DNS non se borre se creo outro contenedor?

Para que el contenedor no se cree otro al iniciar una y otra vez, tenemos que crear los volumens:
          - ./conf:/etc/bind

### 9. Engade unha zoa tendaelectronica.int no teu docker DNS que teña:

En la carpeta de trabajo creada creamos el archivo compose.yml los volumenes de conf y zonas.

El archivo yml, quedaria de la siguiente manera:

```
version: '3'

services:
  Examen23:
    container_name: examen23
    # usamos esta imagen para poder ver los logs
    # la imagen de internetsystemsconsortium/bind9
    # no muestra los logs
    image: ubuntu/bind9
    # especificamos la plataforma de la imagen
    # es posible conflicto cuando el host es 'arm'
    # ya que no existe una imagen para esa plataforma
    platform: linux/amd64
    ports:
      - "55:55/udp"
      - "55:55/tcp"
    networks:
      bind9_subnet:
        ipv4_address: 172.16.23.1
    volumes:
      - ./conf:/etc/bind
      - ./zonas:/var/lib/bind
    restart: always
  cliente:
    container_name: examen_con
    image: alpine:latest
    platform: linux/amd64
    tty: true
    stdin_open: true
    # configuramos para que el cliente use nuestro dns
    dns:
      - 172.16.23.1
    networks:
      bind9_subnet:
        ipv4_address: 172.16.23.2
    
networks:
  bind9_subnet:
    driver: bridge
    ipam:
      config:
        - subnet: 172.16.0.0/16

```

Dentro de zonas creamos el fichero db.tendaelectronicaa.int donde pondremos el dominio con todo lo requerido.

### - www á IP 172.16.0.1
### - owncloud sexa un CNAME de www
### - un rexistro de texto có contido "1234ASDF"
```
$TTL 38400	; 10 hours 40 minutes
@		IN SOA	ns.tendaelectronicaa.int. root.tendaelectronicaa.int. (
				23         ; serial
				690000     ; refresh (3 hours)
				36000       ; retry (1 hour)
				604800     ; expire (1 week)
				38400      ; minimum (10 hours 40 minutes)
				)
@		IN NS	ns.tendaelectronicaa.int.
ns		IN A		172.16.23.1
test		IN A		172.16.23.4
alias	IN CNAME	www.tendaelectronicaa.int.
info	IN TXT		"S1234ASDF"
```

### Comproba que todo funciona có comando "dig"

Para comprobar con dig use dentro del contenedor cliente el coamndo dig: (dig @172.16.23.1 www.tendaelectronicaa.int).
En el git añadire imagenes graficas.

### Mostra nos logs que o servicio funciona ben usando a saída da terminal ó levantar o compose ou có comando "docker logs [nomeContenedorOuID]"

Al realizar la comprobación nos saldra toda la info. (Habra imagenes de prueba en el git).

### (o apartado 9 realízase na máquina virtual)
