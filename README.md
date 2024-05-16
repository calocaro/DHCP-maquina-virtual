# DHCP-maquina-virtual

# Configuración de Servidor DHCP en una Máquina Virtual Linux

## Descripción
Este repositorio contiene los archivos de configuración y evidencia de la configuración de un servidor DHCP en una máquina virtual Linux, con el propósito de servir direcciones IP a otras máquinas en la misma red interna de VirtualBox.

## Configuración Realizada

1. **Instalación de paquete `isc-dhcp-server`**: 
   ```bash
   sudo apt-get install isc-dhcp-server
   ```

2. **Edición de `dhcpd.conf`**: 
   Se editó el archivo de configuración `/etc/dhcp/dhcpd.conf` para una configuración básica. Se agregó la configuración de la subnet y se especificaron los rangos de direcciones IP a asignar.

3. **Configuración de la interfaz de red**: 
   Se modificó el archivo `/etc/default/isc-dhcp-server` para especificar la interfaz de red en la que el servidor DHCP debe escuchar.

4. **Arranque del servicio**: 
   ```bash
   sudo systemctl start isc-dhcp-server
   ```

5. **Comprobación del servicio**: 
   ```bash
   sudo systemctl status isc-dhcp-server
   ```

6. **Asignación de dirección IP por MAC fija**: 
   Se configuró una asignación estática de dirección IP para una MAC específica en el archivo `dhcpd.conf`.

## Mapeo carpeta data

Localmente en el directorio del proyecto debemos crear la carpeta ```data```
Esta carpeta ya la tenemos mapeada en el ```docker-compose.yml```


## Configuración del servicio

Primeramente deberemos ejecutar en el terminal desde el directorio de dicho servicio el comando:

```
docker-compose up
```

De esta manera en nuestra carpeta ```data``` se generarán diferentes ficheros, entre ellos el que más nos interesa que es ```dhcpd.conf``` donde en el podremos especificar la configuración de nuestro servidor DHCP.



*En caso de tener un error al levantar el servicio de que el fichero ```dhcpd.leases``` no existe podremos generarlo nosotros manualmente. En podemos observar las IPS asignadas y el DUID de nuestro servicio*


```yml
subnet 10.0.2.0 netmask 255.255.255.0 {    
    range 10.0.2.100 10.0.2.200;    
    option routers 10.0.2.1;    
    option domain-name-servers 8.8.8.8;    
}

subnet 192.168.76.0 netmask 255.255.255.0 {
    range 192.168.76.100 192.168.76.200;
    option routers 192.168.70.1;
    option domain-name-servers 8.8.8.8;
}

subnet 10.0.9.0 netmask 255.255.255.0 {
    range 10.0.9.200 10.0.9.202

    option domain-name-servers 8.8.8.8, 193.146.96.2, 193.146.96.3;
    option domain-name "asir.tv";
    option routers 10.0.0.9.254;
    option subnet-mask 255.255.255.0;
    option broadcast-address 10.0.9.255;

    default-lease-time 86400;
    max-lease-time 172800;

    group{

        default-lease-time 604800;
        max-lease-time 691200;
        host pache {
            hardware ethernet 00:10:5a:f1:35:87;
            fixed-address 10.0.9.37;
        }

    }

}

```

En el estamos indicando:

* 2 subredes diferentes


En el campo concreto de nuestra subred *10.0.0.9* con máscara *255.255.255.0* especificamos:

* **range**: Estamos especificando el rango de IPS que podemos asignar, es decir, desde la IP "x" puedes asignar IPS hasta la IP "y".

* **option domain-name-servers** : Especificamos con que IPS queremos que resuelva nuestro DNS, de manera default indicaremos la de google y luego nuestras IPS internas de la red.

* **option domain-name** : En el especificamos nuestro nombre de dominio.

* **option routers** :  Indicamos la IP de nuestro router como puerta de salida.

* **option subnet-mask** : Señalamos la máscara de nuestra subred.

* **option broadcast-address** : Indicamos nuestra IP de broadcast.

* **default-lease-time** : Tiempo mínimo o por defecto de refresco de la IP que asignamos.

* **max-lease-time**: Tiempo máximo establecido para refresco de las IP.


## Asignación IP y configuración para equipos concretos


Si nosotros queremos asignar IPS de manera automática a ciertos equipos en concreto o queremos indicar un rango específico para dichos equipos sin ser todos aquellos que pertenecen a la red haremos incapíe en la siguiente parte del códgo de configuración de ```dhcpd.conf```


```yml

 group{

        default-lease-time 604800;
        max-lease-time 691200;
        host pache {
            #hardware ethernet 00:10:5a:f1:35:87;
            #fixed-address 10.0.9.37;
        }

```

En este caso estamos especificando para la asignación los tiempos de refresco de las IP, a mayores de que estamos definiendo una asignación concreta de una IP a "x" equipo de manera que cuando identifique la MAC de este equipo solicitando la asignación de una IP este le establecerá la que hemos especificado.

Para ello:

```yml

        host pespecificamosNombre {
            hardware ethernet #indicamosLaMAC;
            fixed-address #indicamosLaIP;
        }


```


## Evidencia

1. **Asignación de IP a un cliente**: 
   Se levantó otra máquina virtual en la misma red interna y se verificó que recibiera una dirección IP del rango especificado utilizando el comando `ifconfig` o similar.s

2. **Asignación de IP fija por MAC**: 
   Se configuró una asignación estática de dirección IP para una MAC específica en el archivo `dhcpd.conf` y se comprobó que un cliente con esa MAC recibiera la IP fija especificada.

3. **Captura de paquetes con Wireshark**: 
   Se utilizó Wireshark para capturar los mensajes del protocolo DHCP y verificar la comunicación entre el servidor DHCP y los clientes.

## Archivos de Configuración

- `dhcpd.conf`: Archivo de configuración principal del servidor DHCP.
- `isc-dhcp-server`: Archivo de configuración de la interfaz de red.

## Estructura del Repositorio

- `README.md`: Este archivo que contiene la documentación de la práctica.
- `dhcpd.conf`: Archivo de configuración principal del servidor DHCP.
- `isc-dhcp-server`: Archivo de configuración de la interfaz de red.
- `capturas/`: Carpeta que contiene las capturas de pantalla o archivos de Wireshark con la evidencia de la configuración y funcionamiento del servidor DHCP.

## Como realizar la comprobación

En un caso de un entorno de desarrollo podemos crear una nueva máquina mediante VirtualBox, cuando creamos la máquina si vamos al apartado de tarjetas de red deberemos tener:

* **Adaptador 1 en NAT**

_![IMAGEN_NAT](https://github.com/calocaro/DHCP-maquina-virtual/blob/codespace-legendary-carnival-rq5jx65gqvxhp574/nat.png?raw=true)_

* **Adaptador 2 en Puente**

_![IMAGEN_PUENTE]()_

De esta manera nosotros ya conocemos la dirección MAC  a poder especificar, de todas formas podemos consultarlo de manera interna con el comando *ip addr* y desde ahi consultar dicha información.

Para eliminar la **IP** que tenemos asignada utilizamos el comando:

```
sudo dhclient -r
```

Para que solicite una nueva ip ejecutamos el comando:

```
sudo dhclient
```

Si nos encontramos en **Windows** y realizamos la misma configuración en VirtualBox en principio ejecutando el comando *ipconfig* y haciendo la asignación de la IP por DHCP esta se refrescaría automáticamente, en caso de no hacerlo podemos ejecutar el comando *ipconfig/renew*.
