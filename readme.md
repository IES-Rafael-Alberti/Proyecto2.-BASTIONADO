# Introducción

Este trabajo se enfoca en la configuración de una red empresarial utilizando equipos Cisco, que incluye la implementación de VLANs y el enrutamiento entre ellas en una infraestructura compuesta por dos switches de capa 3 y tres switches de capa 2. Se busca establecer un esquema de comunicación robusto que permita a las diferentes áreas de la empresa (TIC, RRHH, Marketing, Administración y Servidores) operar de manera eficiente y segura dentro de sus respectivas subredes.

La guía proporciona una descripción paso a paso para configurar las VLANs, asignar rangos de IP, habilitar el enrutamiento entre VLANs y verificar la conectividad en el entorno. Este enfoque asegura que los especialistas en redes puedan replicar y adaptar el diseño a las necesidades específicas de su organización.

![Portada](/img/portada.webp)

## Topología de Red y diseño de vlans

La topología de red describe la disposición y la interconexión de los dispositivos dentro de una infraestructura de red. En este caso, la red está compuesta por dos switches de capa 3 (SW1 y SW2), que se encargan del enrutamiento entre las diferentes VLANs, y tres switches de capa 2 (SW3, SW4 y SW5), que gestionan el tráfico local dentro de cada VLAN.

![Topología](/img/topologia.jpg)


Las VLANs permiten segmentar una red física en varias redes lógicas, mejorando la seguridad, el rendimiento y la administración. Al separar los dispositivos por función o ubicación, se limita el tráfico de broadcast y se facilita la gestión sin alterar el cableado físico. El diseño de VLANs debe considerar el propósito de cada segmento, el rango de direcciones IP y las máscaras de subred para optimizar la red. Esta segmentación permite una mayor flexibilidad y escalabilidad en la infraestructura de red.

| VLAN | Nombre        | Subred         | Máscara | Rango de IPs    | Broadcast     |
|------|---------------|----------------|---------|-----------------|---------------|
| 10   | TIC           | 192.168.1.0    | /27     | 192.168.1.1 - 192.168.1.30 | 192.168.1.31 |
| 20   | RRHH          | 192.168.1.32   | /27     | 192.168.1.33 - 192.168.1.62 | 192.168.1.63 |
| 30   | Marketing     | 192.168.1.64   | /27     | 192.168.1.65 - 192.168.1.94 | 192.168.1.95 |
| 99   | Administración| 192.168.1.96   | /28     | 192.168.1.97 - 192.168.1.110| 192.168.1.111|
| 40   | Server        | 192.168.1.112  | /30     | 192.168.1.113 - 192.168.1.114| 192.168.1.115|


### Configuración switches L3

La configuración de los switches de capa 3 es esencial para habilitar el enrutamiento entre las VLANs definidas en la red. Estos switches actúan como routers, permitiendo la comunicación entre diferentes segmentos de red. En esta sección, se detallan los pasos para crear las VLANs, asignar direcciones IP y habilitar el enrutamiento entre ellas, garantizando que los dispositivos en distintas VLANs puedan intercambiar datos de manera efectiva.

1.- Acceder al switch L3

```bash
enable
configure terminal
```

2.- Crear las VLANs y asignarles un nombre

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
name Administración
exit

vlan 40
name Server
exit

vlan 50
name Server2
exit
```

3.- Asignar interfaces a las VLANs

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

4.- Configurar los enlaces troncales hacia los switches de capa 3

```bash
interface range fastethernet 0/7-8
switchport mode trunk
switchport trunk allowed vlan 10,20,30,99,40,50
exit
```

5.- Configuración Port-Channel
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
```

6.- Configuración de VLANs enrutadas
```bash
interface vlan 10
ip address 192.168.0.1 255.255.255.224
ip helper-address 192.168.0.107
exit

interface vlan 20
ip address 192.168.0.33 255.255.255.224
ip helper-address 192.168.0.107
exit

interface vlan 30
ip address 192.168.0.65 255.255.255.224
ip helper-address 192.168.0.107
exit

interface vlan 40
ip address 192.168.0.105 255.255.255.248
ip helper-address 192.168.0.107
exit

interface vlan 50
ip address 192.168.0.97 255.255.255.248
ip helper-address 192.168.0.107
exit
```

7.- Configuración HSRP (Hot Standby Router Protocol)
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

8.- Configuración del puerto de salida (enlace con routers/firewalls)
```bash
interface gigabitEthernet 0/1
no switchport
ip address 192.168.0.122 255.255.255.252
exit
```


### Configuración switches L2

La configuración de los switches de capa 2 consiste en crear y asignar VLANs, así como configurar enlaces troncales hacia switches L3 o entre switches L2. Los switches L2 actúan como dispositivos de acceso, conectando hosts a las VLANs y pasando el tráfico hacia el switch L3 para el enrutamiento entre VLANs.

1.- Acceder al switch L2
```bash
enable
configure terminal
```

2.- Crear las VLANs y asignarles un nombre

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
name Administración
exit

vlan 40
name Server
exit

vlan 50
name Server2
exit
```

3.- Asignar interfaces a las VLANs

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

4.- Configurar enlaces troncales hacia switches de capa 3

```bash
interface range gigabitEthernet 0/1-2
switchport mode trunk
switchport trunk allowed vlan 10,20,30,40,50
exit
```

5.- Configuración Port-Channel en switches L2

```bash
interface range fastethernet 0/4-7
switchport mode trunk
exit

interface range fastethernet 0/4-5
channel-group 1 mode auto
exit

interface range fastethernet 0/6-7
channel-group 5 mode auto
exit
```