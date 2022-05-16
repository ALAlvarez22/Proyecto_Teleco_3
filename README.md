<img src="https://www.nagios.org/wp-content/uploads/2015/06/Nagios-Logo.jpg" width="100" height="100">

# Monitoreo de infraestructura con Nagios

## Tabla de contenido

- [1. Preparación de las máquinas virtuales](#1-prep)
- [2. Configuraciones iniciales del servidor](#2-conf)
  * [2.1 Instalación de las dependencias necesarias para instalar Nagios](#21-ins)
  * [2.2 Instalación de plugins en el servidor](#22-inst)
  * [2.3 Descarga de NRPE](#23-desc)
- [3. Configuraciones del cliente](#3-confc)
  * [3.1 Instalación de las dependencias necesarias para instalar Nagios](#31-inscl)
  * [3.2 Instalación de plugins en el cliente](#32-instcl)
  * [3.3 Descarga de NRPE](#33-descn)
- [4. Monitoreo básico del cliente](#4-moni)
  * [4.1 Definición en el servidor](#41-defs)
  * [4.2 Configuración del cliente](#42-confcll)
- [5. Instalar plugin de la comunidad de Nagios para monitorear la memoria RAM de los clientes Linux](#5-insplu)
  * [5.1 Instalación de las dependencias necesarias para instalar Nagios](#51-insdep)
  * [5.2 Instalación de plugins en el servidor](#52-inspser)
- [6. Monitoreo del servicio de Apache (HTTP)](#6-insplu)
  * [6.1 Configuración del cliente](#61-confcli)
  * [6.2 Configuración del servidor](#62-confser)
- [7. Cambiar la apariencia de Nagios](#7-apar)
- [8 Gráficos con PNP4Nagios](#8-grafic)
  * [8.1 Configuración en el servidor](#81-confserver)
- [9. Bibliografía](#9-bibl)

<a name="1-prep"></a>
## 1. Preparación de las máquinas virtuales

- Crear un Vagrantfile con un cliente y un servidor

- Instalar vim y net-tools.

  <code>yum install -y vim net-tools</code>

- Deshabilitar SELinux en el directorio: <code>/etc/selinux/config</code>

- En caso de que el Firewall esté corriendo, habilitar los puertos 80 y 443.

  <code>firewall-cmd --add-port=80/tcp --permanet</code>
  <code>firewallcmd --add-port=443/tcp --permanet</code>










