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

- Crear un Vagrantfile con un cliente y un servidor.

- Instalar vim y net-tools.

  <code>yum install -y vim net-tools</code>

- Deshabilitar SELinux en el directorio: <code>/etc/selinux/config</code>

- En caso de que el Firewall esté corriendo, habilitar los puertos 80 y 443.

  <code>firewall-cmd --add-port=80/tcp --permanet</code>

  <code>firewallcmd --add-port=443/tcp --permanet</code>

<a name="2-conf"></a>
## 2. Configuraciones iniciales del servidor

<a name="21-ins"></a>
### 2.1 Instalación de las dependencias necesarias para instalar Nagios

```
gettext         wget        net-snmp-utils      openssl-devel
glibc-common    unzip       perl                epel-release
gcc             php         gd                  automake
autoconf        httpd       make                glibc
gd-devel        net-snmp    perl-Net-SNMP
```
<code>yum install -y gettext wget net-snmp-utils openssl-devel glibc-common unzip perl epel-release gcc php gd automake autoconf httpd make glibc
gd-devel net-snmp</code>

<code>yum --enablerepo=powertools,epel install perl-Net-SNMP</code>

Si el comando inmediatamente anterior no funciona, intentar con:

<code>yum config-manager --enable powertools</code>

<code>yum install perl-Net-SNMP</code>

1. Crear usuario un usuario Nagios y agregarlo como grupo secundario con:

  <code>useradd nagios</code>

  <code>usermod -a -G nagios apache</code>

2. Obtener los binarios para compilar Nagios desde:

```
https://github.com/NagiosEnterprises/nagioscore/releases
```

En la máquina virtual:

```
wget https://github.com/NagiosEnterprises/nagioscore/releases/download/nagios-4.4.7/nagios-4.4.7.tar.gz
```

3. Descomprimir los archivos con:

<code>tar -xzvf nagios-4.4.7.tar.gz</code>

4. Ingresar al directorio de los archivos descomprimidos anteriormente e ingresar el siguiente comando:

<code>./configure</code>

5. Compilar Nagios con el comando:

<code>make all</code>

6. Hacer la instalación con:

<code>make install</code>

7. Continuar con: 

<code>make install-commandmode</code>

<code>make install-config</code>

<code>make install webconf</code>

8. Habilitar los servicios de Nagios y HTTPD con: 

<code>systemctl enable nagios</code>

<code>systemctl enable httpd</code>

9. Crear un usuario y contraseña para acceder a Nagios con: 

<code>htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin</code>

10. Activar los servicios con:

<code>service nagios start</code>

<code>service httpd start</code>

Se puede verificar la correcta activación con:

<code>service nagios status</code>

<code>service httpd status</code>

<code>systemctl is-enabled httpd</code>


<a name="22-inst"></a>
### 2.2 Instalación de plugins en el servidor

1. Obtener los plugins de:

```
https://github.com/nagios-plugins/nagios-plugins/releases
```

En la máquina virtual:

```
wget https://github.com/NagiosEnterprises/nagioscore/releases/download/nagios-4.4.7/nagios-4.4.7.tar.gz
```

2. Descomprimir los archivos con:

<code>tar -xzvf nagios-plugins-4.4.7.tar.gz</code>

3. Dentro del directorio del archivo descomprimido se ingresa el comando:

<code>./configure</code>

4. Hacer la instalación con:

<code>make install</code>

5. Reiniciar el servicio de Nagios.


💡 **Los plugins se instalan en el directorio** <code>/usr/local/nagios/libexec/</code> **y dentro del directorio se puede ejecutar el comando**
<code>./plugin_name --help</code> **para ver como se usa dicho plugin.**

<a name="23-desc"></a>
### 2.3 Descarga de NRPE

1. Obtener el NRPE de:

```
https://github.com/NagiosEnterprises/nrpe/releases
```

En la máquina virtual:

```
wget https://github.com/NagiosEnterprises/nagioscore/releases/download/nagios-4.4.7/nagios-4.4.7.tar.gz
```

2. Descomprimir los archivos con:

<code>tar -xzvf nrpe-4.0.3</code>

3. Dentro del directorio descomprimido se ingresa el comando:

<code>./configure</code>

💡 **En NRPE funciona en el puerto 5666, por lo que en todos los clientes donde se vaya a instalar el NRPE con los plugins
se debe habilitar este puerto en los plugins.**

4. Ejecutar:

<code>make check_nrpe</code>

Para compilar el comando:

<code>check nrpe</code>

5. Instalar el plugin de NRPE con el comando:

<code>make install-plugin</code>

6. Dirigirse al directorio <code>/usr/local/nagios/etc/objects/</code> y editar el archivo <code>commands.cfg</code> para crear el comando con el que Nagios pueda usar el NRPE recien configurado.

```

define command {
    command_name    check_nrpe
    command_line    $USER1$/check_nrpe -H $HOSTADDESS$ -c $ARGS1$
}

```

**$USER1$** es una variable que indica /usr/local/nagios/libexec

**check_nrpe** es el comando compilado con el que Nagios va a checkear el NRPE en los clientes

**$HOSADDRESS$** es la dirección IP del host que se está monitoreando

**$ARGS1$** Son los argumentos y corresponde al plugin que se desea verificar.


7. Habilitar por defecto el servicio de Nagios con:

<code>chkconfig nagios on</code>


<a name="3-confc"></a>
## 3. Configuraciones del cliente

<a name="31-inscl"></a>
## 3.1 Instalación de las dependencias necesarias para instalar Nagios

```
wget        openssl-devel       glibc-common
gcc         glibc               perl-Net-SNMP
perl

```

1. Ejecutar el comando:

<code>yum install -y gcc glibc glibc-common openssl openssl-devel perl wget</code>

2. Crear un usuario Nagios con:

<code>useradd nagios</code>


<a name="32-instcl"></a>
## 3.2 Instalación de plugins en el cliente

1. Obtener los plugins de:

```
https://github.com/nagios-plugins/nagios-plugins/releases
```

En la máquina virtual ejecutar el siguiente comando:

```
wget https://github.com/NagiosEnterprises/nagioscore/releases/download/nagios-4.4.7/nagios-4.4.7.tar.gz
```

2. Descomprimir los archivos con:

<code>tar -xzvf nagios-plugins-4.4.7.tar.gz</code>

3. Dentro del directorio del archivo descomprimido se ingresa el comando:

<code>./configure</code>

4. Hacer la instalación con:

<code>make install</code>

5. Reiniciar el servicio de Nagios.

💡 **Los plugins se instalan en el directorio** <code>/usr/local/nagios/libexec/</code> **y dentro del directorio se puede ejecutar el comando**
<code>./plugin_name --help</code> **para ver como se usa dicho plugin.**


<a name="33-descn"></a>
## 3.3 Descarga de NRPE

1. Obtener el NRPE de:

```
https://github.com/NagiosEnterprises/nrpe/releases
```

En la máquina virtual:

```
wget https://github.com/NagiosEnterprises/nagioscore/releases/download/nagios-4.4.7/nagios-4.4.7.tar.gz
```

2. Descomprimir los archivos con:

<code>tar -xzvf nrpe-4.0.3</code>

3. Dentro del directorio descomprimido se ingresa el comando:

<code>./configure</code>

💡 **En NRPE funciona en el puerto 5666, por lo que en todos los clientes donde se vaya a instalar el NRPE con los plugins
se debe habilitar este puerto en los plugins.**

4. Ejecutar:

<code>make all</code>

<code>make install</code>

<code>make install-config</code>

<code>make install-init</code>

5. Dirigirse al directorio <code>/usr/local/nagios/etc</code> y editar el archivo <code>nrpe.cfg</code> para agregar el servidor Nagios como host permitido.

```
allowed_host=127.0.0.0.1,::1,192.168.50.3
```

6. Iniciar el servicio de NRPE con <code>service nrpe start</code> y habilitar el servicio de arranque con <code>systemctl enable nrpe</code>

💡 **Puede verificar la comunicación del Nagios entre el cliente y el servidor desde el servidor con** <code>/usr/local/nagios/libexec/check_nrpe -H 192.168.50.2</code>. **En caso de estar todo correctamente configurado, la respuesta es la versión del NRPE que encuentra en el cliente.**


<a name="4-moni"></a>
## 4. Monitoreo básico del cliente

<a name="41-defs"></a>
## 4.1 Definición en el servidor

1. Ingresar al directorio <code>/usr/local/nagios/etc/objects</code>

2. Crear y editar el fichero <code>linux.cfg</code>

```
########## Definicion de servidores ###########

define host{
        use                 linux-server        #template de teplates.cfg a utilizar
        host_name           cliente_linux   
        alias               Linux Centos
        check_interval      1                   #intervalo en mins con que es monitoreado
        address             192.168.50.2        #dirección ip del host
}

########## Definicion de servicios ###########
define service{
        use                     generic-service         #template de teplate.cfg a utilizar
        host_name               cliente_linux
        service description     Hard Disk               #opcional
        check_interval          1
        check_command           check_nrpe!check_hda1
}

define service{
        use                     generic-service
        host_name               cliente_linux
        service_description     Uptime
        check_interval          1
        check_command           check_nrpe!check_uptime
}

define service{
        use                     generic-service
        host_name               cliente_linux
        service_description     Current Load
        check_interval          1
        check_command           check_nrpe!check_load
}

define service{
        use                     generic-service
        host_name               cliente_linux
        service_description     Swap
        check_interval          1
        check_command           check_nrpe!check_swap
}

define service{
        use                     generic-service
        host_name               cliente_linux
        service_description     Current Users
        check_interval          1
        check_command           check_nrpe!check_users
}
       
```

💡 **Una opción para no establecer el check_interval en todas las definiciones es editar los templetes de** <code>linux-server</code> **y**
<code>generic-service</code> **en el archivo** <code>/usr/local/nagios/etc/objects/templates.cfg</code> **y la propiedad de** <code>check_interval</code> **para los
templates en uso.**

3. En el directorio <code>/usr/local/nagios/etc</code> editar el archivo <code>nagios.cfg</code> y agregar el archivo creado como uno de los archivos de configuración.

4. Verifique que no hayan errores en las sentencias de los archivos de nagios con los comandos:

<code>/usr/local/nagios/bin/nagios -v</code>

</code>/usr/local/nagios/etc/nagios.cfg</code>

💡 **Puede editar el archivo** <code>.bashrc</code> **en el home de root y crear el alias** <code>nagioscheck='/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg’</code>. **Termine con un** <code>source .bashrc</code> **para guardar los cambios.**

5. Edite el archivo <code>/usr/local/nagios/etc/cgi.cfg</code> bajando el refresh rate a diez segundos <code>refresh_rate=10</code>

6. Reiniciar el servicio de Nagios.


<a name="42-confcll"></a>
## 4.2 Configuración del cliente

Para solucionar los siguientes problemas que aparecen al entrar a la web de Nagios proceda así

<img src="https://github.com/ALAlvarez22/Proyecto_Teleco_3/blob/master/nagios.png">









