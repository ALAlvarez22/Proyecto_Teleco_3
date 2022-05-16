<img src="https://www.nagios.org/wp-content/uploads/2015/06/Nagios-Logo.jpg" width="100" height="100">

# Monitoreo de infraestructura con Nagios

## Tabla de contenido

- [1. Requerimientos](#1-req)
- [2. Pruebas mínimas esperadas](#2-pru)
- [3. Entregables](#3-ent)
- [4. Conceptos teóricos](#4-con)
  * [4.1 Estructura de directorios de Nagios](#41-est)
  * [4.2 Objetos que se pueden monitorear](#42-obj)
  * [4.3 Checks](#43-che)
- [5. Desarrollo](#5-des)
  * [5.1 Preparación de las máquinas virtuales](#51-prep)
  * [5.2 Configuraciones iniciales del servidor](#52-conf)
    + [5.2.1 Instalación de las dependencias necesarias para instalar Nagios](#521-ins)
    + [5.2.2 Instalación de plugins en el servidor](#522-inst)
    + [5.2.3 Descarga de NRPE](#523-desc)
  * [5.3 Configuraciones del cliente](#53-confc)
    + [5.3.1 Instalación de las dependencias necesarias para instalar Nagios](#531-inscl)
    + [5.3.2 Instalación de plugins en el cliente](#532-instcl)
    + [5.3.3 Descarga de NRPE](#533-descn)
  * [5.4 Monitoreo básico del cliente](#54-moni)
    + [5.4.1 Definición en el servidor](#541-defs)
    + [5.4.2 Configuración del cliente](#542-confcll)
  * [5.5 Instalar plugin de la comunidad de Nagios para monitorear la memoria RAM de los clientes Linux](#55-insplu)
    + [5.5.1 Instalación de las dependencias necesarias para instalar Nagios](#551-insdep)
    + [5.5.2 Instalación de plugins en el servidor](#552-inspser)
  * [5.6 Monitoreo del servicio de Apache (HTTP)](#56-insplu)
    + [5.6.1 Configuración del cliente](#561-confcli)
    + [5.6.2 Configuración del servidor](#562-confser)
  * [5.7 Cambiar la apariencia de Nagios](#57-apar)
  * [5.8 Gráficos con PNP4Nagios](#58-grafic)
    + [5.8.1 Configuración en el servidor](#581-confserver)
- [6. Bibliografía](#6-bibl)

<a name="req"></a>
## 1. Requerimientos






