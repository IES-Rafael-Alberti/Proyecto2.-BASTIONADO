# Introducción

Este trabajo se enfoca en la configuración de una red empresarial utilizando equipos Cisco, que incluye la implementación de VLANs y el enrutamiento entre ellas en una infraestructura compuesta por dos switches de capa 3 y tres switches de capa 2. Se busca establecer un esquema de comunicación robusto que permita a las diferentes áreas de la empresa (TIC, RRHH, Marketing, Administración y Servidores) operar de manera eficiente y segura dentro de sus respectivas subredes.

La guía proporciona una descripción paso a paso para configurar las VLANs, asignar rangos de IP, habilitar el enrutamiento entre VLANs y verificar la conectividad en el entorno. Este enfoque asegura que los especialistas en redes puedan replicar y adaptar el diseño a las necesidades específicas de su organización.

![Portada](/img/portada.webp)

## Topología de Red

La topología de red describe la disposición y la interconexión de los dispositivos dentro de una infraestructura de red. En este caso, la red está compuesta por dos switches de capa 3 (SW1 y SW2), que se encargan del enrutamiento entre las diferentes VLANs, y tres switches de capa 2 (SW3, SW4 y SW5), que gestionan el tráfico local dentro de cada VLAN.

![Topología](/img/topologia.png)

