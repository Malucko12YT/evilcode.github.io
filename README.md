# Laboratorio de Seguridad Perimetral y Segmentación con FortiGate-VM

Proyecto de infraestructura y seguridad de redes diseñado desde cero utilizando GNS3 para emular un entorno corporativo real y un firewall perimetral FortiGate (FortiOS 7.0). El objetivo principal es mitigar fallas de diseño comunes (bucles de enrutamiento) mediante la implementación de un modelo estricto de tres zonas de seguridad independientes.

Despliegue del Portafolio Web: https://github.io

---

## Direccionamiento IP del Laboratorio

Zona WAN (Internet)
* FortiGate Port1: Obtiene dirección IP automáticamente por DHCP desde el router de casa.

Zona LAN (Clientes Internos)
* FortiGate Port2 (Puerta de enlace): IP 192.168.10.1 con máscara 255.255.255.0
* CLIENTE1: IP 192.168.10.10 con máscara 255.255.255.0 y puerta de enlace 192.168.10.1
* CLIENTE2: IP 192.168.10.20 con máscara 255.255.255.0 y puerta de enlace 192.168.10.1
* CLIENTE3: IP 192.168.10.30 con máscara 255.255.255.0 y puerta de enlace 192.168.10.1

Zona DMZ (Servidores)
* FortiGate Port3 (Puerta de enlace): IP 172.16.10.1 con máscara 255.255.255.0
* SERV-WEB (Contenedor Docker): IP 172.16.10.10 con máscara 255.255.255.0 y puerta de enlace 172.16.10.1

---

## Configuración de los Clientes (VPCS)

Comandos ejecutados en la consola de cada equipo:

CLIENTE1:
ip 192.168.10.10 255.255.255.0 192.168.10.1
save

CLIENTE2:
ip 192.168.10.20 255.255.255.0 192.168.10.1
save

CLIENTE3:
ip 192.168.10.30 255.255.255.0 192.168.10.1
save

---

## Configuración del Firewall FortiGate (CLI)

### 1. Configuración de Interfaces
config system interface
    edit port1
        set mode dhcp
        set allowaccess ping http https ssh
    next
    edit port2
        set mode static
        set ip 192.168.10.1 255.255.255.0
        set allowaccess ping http https
    next
    edit port3
        set mode static
        set ip 172.16.10.1 255.255.255.0
        set allowaccess ping http https
    next
end

### 2. Políticas de Seguridad y NAT
config firewall policy
    edit 1
        set name "LAN_a_DMZ"
        set srcintf "port2"
        set dstintf "port3"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "HTTP" "PING"
    next
    edit 2
        set name "LAN_a_Internet"
        set srcintf "port2"
        set dstintf "port1"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set nat enable
    next
end

---

## Pruebas de Verificación de Tráfico

Validación realizada desde la consola del CLIENTE1:
1. Conectividad local hacia la DMZ: ping 172.16.10.10 exitoso.
2. Petición de servicios web: rlogin 172.16.10.10 80 con respuesta HTTP del contenedor Docker.
3. Salida a internet con NAT: ping 8.8.8.8 exitoso.

Tecnologías utilizadas: Fortinet FortiOS, GNS3 Simulator, Docker Containers, Redes L3, NAT.
