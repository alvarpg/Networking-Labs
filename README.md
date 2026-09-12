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
puerto 0/2

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
