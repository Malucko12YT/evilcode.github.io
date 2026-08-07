# Laboratorio de Seguridad Perimetral y Segmentación con FortiGate-VM

Proyecto de infraestructura y seguridad de redes diseñado desde cero utilizando **GNS3** para emular un entorno corporativo real y un firewall perimetral **FortiGate (FortiOS 7.0.19)**. El objetivo principal es mitigar fallas de diseño comunes (bucles de enrutamiento) mediante la implementación de un modelo estricto de tres zonas de seguridad independientes.

---

## Direccionamiento IP del Laboratorio

### Zona WAN (Internet)
*   **FortiGate Port1:** Obtiene dirección IP automáticamente por DHCP desde el router de casa.

### Zona LAN (Clientes)
*   **FortiGate Port2 (Puerta de enlace):** IP `192.168.10.1` | Máscara `255.255.255.0`
*   **CLIENTE1:** IP `192.168.10.10` | Máscara `255.255.255.0` | Gateway `192.168.10.1`
*   **CLIENTE2:** IP `192.168.10.20` | Máscara `255.255.255.0` | Gateway `192.168.10.1`
*   **CLIENTE3:** IP `192.168.10.30` | Máscara `255.255.255.0` | Gateway `192.168.10.1`

### Zona DMZ (Servidores)
*   **FortiGate Port3 (Puerta de enlace):** IP `172.16.10.1` | Máscara `255.255.255.0`
*   **SERV-WEB (Contenedor Docker):** IP `172.16.10.10` | Máscara `255.255.255.0` | Gateway `172.16.10.1`

---

## Configuración de los Clientes (VPCS)

Comandos ejecutados en la consola de cada equipo para asignación estática y persistencia de datos:

### CLIENTE 1
```bash
ip 192.168.10.10 255.255.255.0 192.168.10.1
save
```

### CLIENTE 2
```bash
ip 192.168.10.20 255.255.255.0 192.168.10.1
save
```

### CLIENTE 3
```bash
ip 192.168.10.30 255.255.255.0 192.168.10.1
save
```

---

## Configuración del Firewall FortiGate (CLI)

### 1. Aprovisionamiento de Interfaces lógicas
```yaml
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
```

### 2. Políticas de Seguridad Perimetral y Traducción de Direcciones (NAT)
```yaml
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
```

---

## Pruebas de Verificación de Tráfico

Validación de conectividad de extremo a extremo realizada de manera exitosa desde la consola del **CLIENTE1**:

1.  **Conectividad local hacia la DMZ:** `ping 172.16.10.10` con respuesta inmediata de paquetes.
2.  **Petición de servicios web:** `rlogin 172.16.10.10 80` estableciendo enlace HTTP correcto con el contenedor Docker.
3.  **Salida a internet con NAT:** `ping 8.8.8.8` verificando la correcta traducción de direccionamiento público.

---
**Dispositivos implementadas:** Fortinet FortiOS, GNS3 Simulator, Docker Containers, Redes L3, Network Address Translation (NAT).

