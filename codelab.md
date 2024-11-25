author: Grupo 1
summary: Guía creación y configuración de red
id: guia-grupo1-proyecto2
categories: codelab,markdown
environments: Web
status: Published

# Proyecto 2: La red dividida

## Introducción

Este trabajo se enfoca en la configuración de una red empresarial utilizando equipos Cisco, que incluye la implementación de VLANs y el enrutamiento entre ellas en una infraestructura compuesta por dos switches de capa 3 y tres switches de capa 2. Se busca establecer un esquema de comunicación robusto que permita a las diferentes áreas de la empresa (TIC, RRHH, Marketing, Administración y Servidores) operar de manera eficiente y segura dentro de sus respectivas subredes.

La guía proporciona una descripción paso a paso para configurar las VLANs, asignar rangos de IP, habilitar el enrutamiento entre VLANs y verificar la conectividad en el entorno. Este enfoque asegura que los especialistas en redes puedan replicar y adaptar el diseño a las necesidades específicas de su organización.

![Portada](/img/portada.webp)

## Topología de Red y diseño de vlans

La topología de red describe la disposición y la interconexión de los dispositivos dentro de una infraestructura de red. En este caso, la red está compuesta por dos switches de capa 3 (SW1 y SW2), que se encargan del enrutamiento entre las diferentes VLANs, y tres switches de capa 2 (SW3, SW4 y SW5), que gestionan el tráfico local dentro de cada VLAN.

![Topología](/img/topologia.jpg)

## VLSM de la red

El cálculo de subredes utilizando VLSM (Variable Length Subnet Mask) permite asignar los recursos de red de manera eficiente según las necesidades específicas de cada subred. En este caso, hemos utilizado la red 192.168.0.0/24 y hemos definido subredes con tamaños ajustados a los requerimientos de hosts. En la siguiente tabla presentamos como quedaría definida la red:

| Subred | Hosts | Dirección de Red | Máscara          | Primer Host    | Último Host     | Broadcast       |
|--------|------------------|------------------|------------------|----------------|-----------------|-----------------|
| 1      | 30               | 192.168.0.0/27      | 255.255.255.224 | 192.168.0.1    | 192.168.0.30    | 192.168.0.31    |
| 2      | 30               | 192.168.0.32/27     | 255.255.255.224 | 192.168.0.33   | 192.168.0.62    | 192.168.0.63    |
| 3      | 30               | 192.168.0.64/27     | 255.255.255.224 | 192.168.0.65   | 192.168.0.94    | 192.168.0.95    |
| 4      | 6                | 192.168.0.96/29     | 255.255.255.248 | 192.168.0.97   | 192.168.0.102   | 192.168.0.103   |
| 5      | 6                | 192.168.0.104/29    | 255.255.255.248 | 192.168.0.105  | 192.168.0.110   | 192.168.0.111   |
| 6      | 2                | 192.168.0.112/30    | 255.255.255.252 | 192.168.0.113  | 192.168.0.114   | 192.168.0.115   |
| 7      | 2                | 192.168.0.116/30    | 255.255.255.252 | 192.168.0.117  | 192.168.0.118   | 192.168.0.119   |
| 8      | 2                | 192.168.0.120/30    | 255.255.255.252 | 192.168.0.121  | 192.168.0.122   | 192.168.0.123   |
| 9      | 2                | 192.168.0.124/30    | 255.255.255.252 | 192.168.0.125  | 192.168.0.126   | 192.168.0.127   |



## Configuración Switch 1 (capa 3)

La configuración de los switches de capa 3 es esencial para habilitar el enrutamiento entre las VLANs definidas en la red. Estos switches actúan como routers, permitiendo la comunicación entre diferentes segmentos de red. En esta sección, se detallan los pasos para crear las VLANs, asignar direcciones IP y habilitar el enrutamiento entre ellas, garantizando que los dispositivos en distintas VLANs puedan intercambiar datos de manera efectiva.

### 1.- Acceder al switch

```bash
enable
configure terminal
```

### 2.- Crear las VLANs y asignarles un nombre

```bash
vlan 10
name TIC
exit

vlan 20
name RRHH
exit

vlan 30
name Marketing
exit

vlan 99
name Administracion
exit

vlan 40
name Server
exit

vlan 50
name Server2
exit
```

### 3.- Asignar interfaces a las VLANs

```bash
interface range fastethernet 0/1-2
switchport mode access
switchport access vlan 10
exit

interface range fastethernet 0/3-4
switchport mode access
switchport access vlan 20
exit

interface range fastethernet 0/5-6
switchport mode access
switchport access vlan 30
exit

interface fastethernet 0/9
switchport mode access
switchport access vlan 40
exit

interface fastethernet 0/9
switchport mode access
switchport access vlan 50
exit
```

### 4.- Configuración Port-Channel
```bash
int port-channel 1
switchport trunk encapsulation dot1q 
switchport mode trunk
exit

int port-channel 2
switchport trunk encapsulation dot1q 
switchport mode trunk
exit

int port-channel 3
switchport trunk encapsulation dot1q 
switchport mode trunk
exit

int port-channel 4
switchport trunk encapsulation dot1q 
switchport mode trunk
exit

int r f0/1-8
switchport trunk encapsulation dot1q
switchport mode trunk
exit

int r f0/1-2
channel-group 1 mode desirable
exit
int r f0/3-4
channel-group 2 mode desirable
exit
int r f0/5-6
channel-group 3 mode desirable
exit
int r f0/7-8
channel-group 4 mode desirable
exit
```

### 5.- Configuración de VLANs
```bash
int vlan 10
ip helper-address 192.168.0.107
exit

int vlan 20
ip helper-address 192.168.0.107
exit

int vlan 30
ip helper-address 192.168.0.107
exit

int vlan 40
ip helper-address 192.168.0.107
exit

int vlan 50
ip helper-address 192.168.0.107
exit
```

### 6.- Configuración HSRP (Hot Standby Router Protocol)
```bash
spanning-tree vlan 10 root primary
spanning-tree vlan 20 root primary
spanning-tree vlan 30 root primary
spanning-tree vlan 40 root primary
spanning-tree vlan 50 root primary

int vlan 10
ip add 192.168.0.1 255.255.255.224
standby 10 ip 192.168.0.30
standby 10 preempt
standby 10 priority 200
exit

int vlan 20
ip add 192.168.0.33 255.255.255.224
standby 20 ip 192.168.0.62
standby 20 preempt
standby 20 priority 200
exit

int vlan 30
ip add 192.168.0.65 255.255.255.224
standby 30 ip 192.168.0.94
standby 30 preempt
standby 30 priority 200
exit

int vlan 40
ip add 192.168.0.105 255.255.255.224
standby 40 ip 192.168.0.110
standby 40 preempt
standby 40 priority 200
exit

int vlan 50
ip add 192.168.0.97 255.255.255.224
standby 50 ip 192.168.0.102
standby 50 preempt
standby 50 priority 200
exit
```

### 7.- Configuración del puerto de salida (enlace con routers/firewalls)
```bash
interface gigabitEthernet 0/1
no switchport
ip address 192.168.0.122 255.255.255.252
exit
```

## Configuración Switch 2 (capa 3)

### 1.- Acceder al switch

```bash
enable
configure terminal
```

### 2.- Crear las VLANs y asignarles un nombre

```bash
vlan 10
name TIC
exit

vlan 20
name RRHH
exit

vlan 30
name Marketing
exit

vlan 99
name Administracion
exit

vlan 40
name Server
exit

vlan 50
name Server2
exit
```


### 3.- Asignar interfaces a las VLANs

```bash
interface range fastethernet 0/1-2
switchport mode access
switchport access vlan 10
exit

interface range fastethernet 0/3-4
switchport mode access
switchport access vlan 20
exit

interface range fastethernet 0/5-6
switchport mode access
switchport access vlan 30
exit

interface fastethernet 0/9
switchport mode access
switchport access vlan 40
exit

interface fastethernet 0/9
switchport mode access
switchport access vlan 50
exit
```

### 4.- Configuración Port-Channel
```bash
int port-channel 5
switchport trunk encapsulation dot1q 
switchport mode trunk
exit

int port-channel 6
switchport trunk encapsulation dot1q 
switchport mode trunk
exit

int port-channel 7
switchport trunk encapsulation dot1q 
switchport mode trunk
exit

int r f0/1-8
switchport trunk encapsulation dot1q
switchport mode trunk
exit

int r f0/1-2
channel-group 5 mode desirable
exit
int r f0/3-4
channel-group 6 mode desirable
exit
int r f0/5-6
channel-group 2 mode desirable
exit

int r f0/7-8
channel-group 4 mode auto
exit

```

### 5.- Configuración de VLANs
```bash
int vlan 10
ip helper-address 192.168.0.107
exit

int vlan 20
ip helper-address 192.168.0.107
exit

int vlan 30
ip helper-address 192.168.0.107
exit

int vlan 40
ip helper-address 192.168.0.107
exit

int vlan 50
ip helper-address 192.168.0.107
exit
```

### 6.- Configuración HSRP (Hot Standby Router Protocol)
```bash
spanning-tree vlan 10 root secondary
spanning-tree vlan 20 root secondary
spanning-tree vlan 30 root secondary
spanning-tree vlan 40 root secondary
spanning-tree vlan 50 root secondary

int vlan 10
ip add 192.168.0.2 255.255.255.224
standby 10 ip 192.168.0.30
standby 10 preempt
standby 10 priority 200
exit

int vlan 20
ip add 192.168.0.34 255.255.255.224
standby 20 ip 192.168.0.62
standby 20 preempt
standby 20 priority 200
exit

int vlan 30
ip add 192.168.0.66 255.255.255.224
standby 30 ip 192.168.0.94
standby 30 preempt
standby 30 priority 200
exit

int vlan 40
ip add 192.168.0.106 255.255.255.224
standby 40 ip 192.168.0.110
standby 40 preempt
standby 40 priority 200
exit

int vlan 50
ip add 192.168.0.98 255.255.255.224
standby 50 ip 192.168.0.102
standby 50 preempt
standby 50 priority 200
exit
```

### 7.- Configuración del puerto de salida (enlace con routers/firewalls)
```bash
interface gigabitEthernet 0/1
no switchport
ip address 192.168.0.126 255.255.255.252
exit
```


## Configuración Switch 3 (capa 2)

La configuración de los switches de capa 2 consiste en crear y asignar VLANs, así como configurar enlaces troncales hacia switches L3 o entre switches L2. Los switches L2 actúan como dispositivos de acceso, conectando hosts a las VLANs y pasando el tráfico hacia el switch L3 para el enrutamiento entre VLANs.

### 1.- Acceder al switch
```bash
enable
configure terminal
```

### 2.- Crear las VLANs y asignarles un nombre

```bash
vlan 10
name TIC
exit

vlan 20
name RRHH
exit

vlan 30
name Marketing
exit

vlan 99
name Administracion
exit

vlan 40
name Server
exit

vlan 50
name Server2
exit
```

### 3.- Asignar interfaces a las VLANs

```bash
interface range fastethernet 0/1-2
switchport mode access
switchport access vlan 10
exit

interface range fastethernet 0/3-4
switchport mode access
switchport access vlan 20
exit

interface range fastethernet 0/5-6
switchport mode access
switchport access vlan 30
exit

interface fastethernet 0/9
switchport mode access
switchport access vlan 40
exit

interface fastethernet 0/9
switchport mode access
switchport access vlan 50
exit
```

### 4.- Configurar enlaces troncales hacia switches de capa 3

```bash
interface range gigabitEthernet 0/1-2
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50
exit
```

### 5.- Configuración Port-Channel en switches de capa 2

```bash
int r f0/4-7
switchport mode trunk
exit

int r f0/4-5
channel-group 1 mode auto
exit
int r f0/6-7
channel-group 5 mode auto
exit
```

### 6.- Configuración Port Security
```bash
interface range FastEthernet0/1 - 3
switchport port-security
switchport port-security maximum 1
switchport port-security violation shut
switchport port-security mac-address sticky
```

## Configuración Switch 4 (capa 2)

### 1.- Acceder al switch
```bash
enable
configure terminal
```

### 2.- Crear las VLANs y asignarles un nombre

```bash
vlan 10
name TIC
exit

vlan 20
name RRHH
exit

vlan 30
name Marketing
exit

vlan 99
name Administracion
exit

vlan 40
name Server
exit

vlan 50
name Server2
exit
```

### 3.- Asignar interfaces a las VLANs

```bash
interface range fastethernet 0/1-2
switchport mode access
switchport access vlan 10
exit

interface range fastethernet 0/3-4
switchport mode access
switchport access vlan 20
exit

interface range fastethernet 0/5-6
switchport mode access
switchport access vlan 30
exit

interface fastethernet 0/9
switchport mode access
switchport access vlan 40
exit

interface fastethernet 0/9
switchport mode access
switchport access vlan 50
exit
```

### 4.- Configurar enlaces troncales hacia switches de capa 3

```bash
interface range gigabitEthernet 0/1-2
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50
exit
```

### 5.- Configuración Port-Channel en switches de capa 2

```bash
int r f0/4-7
switchport mode trunk
exit

int r f0/4-5
channel-group 2 mode auto
exit
int r f0/6-7
channel-group 6 mode auto
exit
```

### 6.- Configuración Port Security
```bash
interface range FastEthernet0/1 - 3
switchport port-security
switchport port-security maximum 1
switchport port-security violation shut
switchport port-security mac-address sticky
```

## Configuración Switch 5 (capa 2)

### 1.- Acceder al switch
```bash
enable
configure terminal
```

### 2.- Crear las VLANs y asignarles un nombre

```bash
vlan 10
name TIC
exit

vlan 20
name RRHH
exit

vlan 30
name Marketing
exit

vlan 99
name Administracion
exit

vlan 40
name Server
exit

vlan 50
name Server2
exit
```

### 3.- Asignar interfaces a las VLANs

```bash
interface range fastethernet 0/1-2
switchport mode access
switchport access vlan 10
exit

interface range fastethernet 0/3-4
switchport mode access
switchport access vlan 20
exit

interface range fastethernet 0/5-6
switchport mode access
switchport access vlan 30
exit

interface fastethernet 0/9
switchport mode access
switchport access vlan 40
exit

interface fastethernet 0/9
switchport mode access
switchport access vlan 50
exit
```

### 4.- Configurar enlaces troncales hacia switches de capa 3

```bash
interface range gigabitEthernet 0/1-2
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50
exit
```

### 5.- Configuración Port-Channel en switches de capa 2

```bash
int r f0/4-7
switchport mode trunk
exit

int r f0/4-5
channel-group 3 mode auto
exit
int r f0/6-7
channel-group 2 mode auto
exit
```

### 6.- Configuración Port Security
```bash
interface range FastEthernet0/1 - 3
switchport port-security
switchport port-security maximum 1
switchport port-security violation shut
switchport port-security mac-address sticky
```