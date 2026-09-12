# Networking-Labs

## 1. Arquitectura del laboratorio

La infraestructura utilizada está formada por los siguientes elementos:

>1 Router

>1 Switch gestionable

>1 VLAN

>1 Ubuntu Server

>1 Equipo Windows 10

La siguiente imagen muestra la topología utilizada.

<img width="718" height="541" alt="Captura de pantalla 2026-05-25 210213" src="https://github.com/user-attachments/assets/f100ff8e-23e9-4889-9088-691c676c407c" />


## 2. Laboratorio 

### VLAN

Creamos y nombramos las 2 VLAN en el switch. Esto se hace para que dentro de una misma red haya varias zonas inaccesibles entre ellas, ya sea por tema de seguridad o confidencialidad.

**VLAN 10**			192.168.10.0/24

**VLAN 20**			192.168.20.0/24

El **router** actúa como puerta de enlace entre ambas VLAN mediante inter-VLAN Routing.
Para la configuración abrimos las interfaces y asignamos IPs para cada red VLAN, asignando una dirección IP distinta a cada segmento. 


#### En el SWITCH

    enable
    configure terminal
    
    vlan 10
    name wLAN-windows
    
    vlan 20
    name wLAN-linux

**Puerto Windows**

    	interface fastEthernet 0/1
    	switchport mode access
    	switchport access vlan 10
    	exit


**Puerto Linux**

    	interface fastEthernet 0/2
    	switchport mode access
    	switchport access vlan 20
    	exit

#### En el ROUTER

Abrimos las interfaces y asignamos IPs para cada red VLAN, asignando una dirección IP distinta a cada segmento. 

**Interfaz del router para VLAN 10**
  
		interface gigabitEthernet 0/1
		ip address 192.168.10.1 255.255.255.0
    	no shutdown 
		exit
    
**Interfaz del router para VLAN 20**

		interface gigabitEthernet 0/2
		ip address 192.168.20.1 255.255.255.0
		no shutdown 
		exit

### DHCP en router

Creamos los servicios DHCP en el router, en este caso configuramos 2 pools distintos, una para cada VLAN. 

**DHCP VLAN 10**

  	ip dhcp pool VLAN10
  	network 192.168.10.0 255.255.255.0
  	default-router 192.168.10.1
  	dns-server 8.8.8.8
  	exit
  
**DHCP VLAN 20**

  	ip dhcp pool VLAN20
  	network 192.168.20.0 255.255.255.0
  	default-router 192.168.20.1
  	dns-server 8.8.8.8
  	exit

### Comandos útiles de comprobación

**En el switch**

	Ver VLANs:
		show vlan brief
	Ver trunk:
		show interfaces trunk
    
**En el router**

	Ver interfaces:
		show ip interface brief
	Ver DHCP:
		show ip dhcp binding

### 3. Configuración de los equipos

### 3.1 Ubuntu Server

Se instaló una máquina con Ubuntu Server, en la que se creó un usuario sin privilegios administrativos, siguiendo el principio de mínimo privilegio, con el objetivo de asignarle únicamente los permisos estrictamente necesarios para desempeñar sus funciones. También se configuró una cuenta de administrador, destinada a las tareas de configuración, mantenimiento y gestión del servidor. 
Para crear un usuario estándar se utilizó:

	sudo adduser usuario

Este comando crea el usuario,  su carpeta personal (/home/usuario) y configura una contraseña.

Crear usuario administrador

	sudo adduser admin

Dar permisos de administrador añadiendolo al grupo sudo

	sudo usermod -aG sudo admin

#### 3.2 OpenSSH Server

Para poder conectarse, transferir archivos y administrar por remoto el servidor de forma segura se instala el protocolo OpenSSH

instalar servicio ssh:	

	sudo apt install openssh-server

Para comprobar el servicio:	
	sudo systemctl status ssh
	
En caso de tener un firewall activo:	
	sudo ufw allow ssh
	
Para mayor seguridad, dentro del archivo de configuración **/etc/ssh/sshd_config** cambiamos algunos parámetros:

**Port 2260**

Cambia el puerto por defecto 22, ya que suele ser muy atacado, por otro menos común

**PermitRootLogin no**

Impide acceder con la cuenta del sistema Root, también muy utilizada en los ataques

**PasswordAuthentication no**

	Evita ataques de fuerza bruta sobre contraseñas.

**MaxAuthTries 3**

Configurar a 3 los intentos máximos de contraseñas
Después de realizar los cambios, reinicio del servicio:	sudo systemctl restart ssh

Para conectarse desde otros dispositivos en la red:	**ssh alvaro@192.168.1.131 -p 2260**



#### 3.3 Fail2ban

La función de Fail2ban monitoriza los registros del sistema y actúa frente a ataques de fuerza bruera de la siguiente manera:

>Identifica la IP atacante.

>La bloquea automáticamente mediante reglas de firewall.

>Reduce ataques de fuerza bruta.

En la siguiente configuración una IP que falle 3 veces en 10 minutos quedará bloqueada durante 10 minutos. Sin esta configuración los atacantes tendrían intentos ilimitados de acceder al servidor.

instalar servicio Fail2ban:	
		sudo apt install fail2ban -y
Comprobar estado:	
		sudo systemctl status fail2ban
En el archivo de configuración **/etc/fail2ban/jail.local** hacemos los siguientes cambios

		[sshd]
		port = 2260 (puerto configurado en sshd)
		maxretry = 3
		findtime = 10m
		bantime = 10m

Reiniciar servicio:	

		sudo systemctl restart fail2ban
Comprobación:	

		sudo fail2ban-client status sshd

#### 3.4 UFW
El firewall controla qué conexiones pueden salir y entrar al servidor. Mayor seguridad a la hora de navegar por internet
Las funciones del firewall ufw :

>Bloquear accesos no autorizados.

>Reduce servicios expuestos.

>Limita posibles vectores de ataque.

instalación de ufw:	

	sudo apt update
	sudo apt install ufw -y
		
Comandos configuración inicial:
	
	sudo ufw default deny incoming
	(Cualquier conexión que intente entrar al servidor será bloqueada por defecto.
	Solo se permitirá el tráfico para el que hayas creado una regla explícita)

	sudo ufw default allow outgoing
	(El servidor puede iniciar conexiones hacia Internet o hacia otros equipos sin 
	restricciones)

Permitir SSH:	

	sudo ufw allow 2260
	
Activación:

	sudo ufw enable

Verificación:	

	sudo ufw status 



## 4. Problemas encontrados
   
Uno de los primeros problemas fue la configuración de las VLANs. Al recordar las configuraciones me informe de varias fuentes distintas para hacer el ejercicio, pero algo no llegaba a funcionar. Después de repasar todo paso a paso funcionó correctamente.

Un problema encontrado fue al realizar la prueba de registro de paquetes de Wireshark, concretamente de los paquetes SSH. Se intentó varias filtros, con el puerto 22 con el 2260 (puerto configurado) y no se pudo registrar los paquetes

## 5. Posibles mejoras

Algunas mejoras sencillas que se podrían implementar en esta infraestructura son:

>Incorporar Active Directory en Windows server, para tener una base centralizada de carpetas, usuarios y sus respectivos permisos.

>Configurar VPN para accesos remotos. Ya sea por alguna aplicación de terceros como Zerotier o directamente configurado en el Firewall.

>Centralizar los registros mediante un servidor Syslog.



