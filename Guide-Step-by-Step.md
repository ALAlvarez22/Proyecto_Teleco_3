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
  * [5.1 Configuración en el cliente](#51-insdep)
  * [5.2 Configuración en el servidor](#52-inspser)
- [6. Monitoreo del servicio de Apache (HTTP)](#6-insa)
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
### 3.1 Instalación de las dependencias necesarias para instalar Nagios

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
### 3.2 Instalación de plugins en el cliente

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
### 3.3 Descarga de NRPE

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
### 4.1 Definición en el servidor

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
### 4.2 Configuración del cliente

Para solucionar los siguientes problemas que aparecen al entrar a la web de Nagios proceda así

<img src="https://github.com/ALAlvarez22/Proyecto_Teleco_3/blob/master/nagios.png">

1. Ejecute el comando <code>df -h</code> para ver el nombre del disco del cliente o la partición del mismo que le interesa monitorear.

2. Edite el archivo <code>/usr/local/nagios/etc/nrpe.cfg</code> en la sección que contiene:

```
command[check_users]=/usr/local/nagios/libexec/check_users -w 5 -c 10
command[check_load]=/usr/local/nagios/libexec/check_load -r -w .15,.10,.05 -c .30,.25,.20
command[check_hda1]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /dev/sda1         #Coloque el nombre de su disco al final
command[check_zombie_procs]=/usr/local/nagios/libexec/check_procs -w 5 -c 10 -s Z
command[check_total_procs]=/usr/local/nagios/libexec/check_procs -w 150 -c 200

#Agregue las siguientes lineas#
command[check_swap]=/usr/local/nagios/libexec/check_swap -w 20% -c 10%
command[check_uptime]=/usr/local/nagios/libexec/check_uptime
```

💡 **En caso de aparecer algún otro error con cualquier comando puede ir a** <code>/usr/local/nagios/libexec</code> **y ejecutar el comando con** <code>.nombre_comando</code> **y analizar los errores que aparezcan.**


<a name="5-insplu"></a>
## 5. Instalar plugin de la comunidad de Nagios para monitorear la memoria RAM de los clientes Linux

<a name="51-insdep"></a>
### 5.1 Configuración en el cliente

1. Dirigirse al directorio <code>/usr/local/nagios/libexec</code>

2. Crear el archivo <code>check_mem</code> y pegar el siguiente código:

```
#!/usr/bin/perl -w

# Heavily based on the script from:
# check_mem.pl Copyright (C) 2000 Dan Larsson <dl@tyfon.net>
# heavily modified by
# Justin Ellison <justin@techadvise.com>
#
# The MIT License (MIT)
# Copyright (c) 2011 justin@techadvise.com

# Permission is hereby granted, free of charge, to any person obtaining a copy of this
# software and associated documentation files (the "Software"), to deal in the Software
# without restriction, including without limitation the rights to use, copy, modify,
# merge, publish, distribute, sublicense, and/or sell copies of the Software, and to
# permit persons to whom the Software is furnished to do so, subject to the following conditions:
# The above copyright notice and this permission notice shall be included in all copies
# or substantial portions of the Software.
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED,
# INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR
# PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE
# FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT
# OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
# OTHER DEALINGS IN THE SOFTWARE.
# Tell Perl what we need to use
use strict;
use Getopt::Std;

#TODO - Convert to Nagios::Plugin
#TODO - Use an alarm

# Predefined exit codes for Nagios
use vars qw($opt_c $opt_f $opt_u $opt_w $opt_C $opt_v $opt_h %exit_codes);
%exit_codes = ('UNKNOWN' , 3,
           'OK' , 0,
               'WARNING' , 1,
               'CRITICAL', 2,
               );

# Get our variables, do our checking:
init();

# Get the numbers:
my ($free_memory_kb,$used_memory_kb,$caches_kb,$hugepages_kb) = get_memory_info();
print "$free_memory_kb Free\n$used_memory_kb Used\n$caches_kb Cache\n" if ($opt_v);
print "$hugepages_kb Hugepages\n" if ($opt_v and $opt_h);

if ($opt_C) { #Do we count caches as free?
    $used_memory_kb -= $caches_kb;
    $free_memory_kb += $caches_kb;
}
   
if ($opt_h) {
    $used_memory_kb -= $hugepages_kb;
}

print "$used_memory_kb Used (after Hugepages)\n" if ($opt_v);

# Round to the nearest KB
$free_memory_kb = sprintf('%d',$free_memory_kb);
$used_memory_kb = sprintf('%d',$used_memory_kb);
$caches_kb = sprintf('%d',$caches_kb);

# Tell Nagios what we came up with
tell_nagios($used_memory_kb,$free_memory_kb,$caches_kb,$hugepages_kb);

sub tell_nagios {
    my ($used,$free,$caches,$hugepages) = @_;
    
    # Calculate Total Memory
    my $total = $free + $used;
    print "$total Total\n" if ($opt_v);

    my $perf_warn;
    my $perf_crit;
    if ( $opt_u ) {
        $perf_warn = int(${total} * $opt_w / 100);
        $perf_crit = int(${total} * $opt_c / 100);
    } else {
        $perf_warn = int(${total} * ( 100 - $opt_w ) / 100);
        $perf_crit = int(${total} * ( 100 - $opt_c ) / 100);
}

my $perfdata = "|TOTAL=${total}KB;;;; USED=${used}KB;${perf_warn};${perf_crit};; FREE=${free}KB;;;; CACHES=${caches}KB;;;;";
$perfdata .= " HUGEPAGES=${hugepages}KB;;;;" if ($opt_h);

if ($opt_f) {
    my $percent = sprintf "%.1f", ($free / $total * 100);
    if ($percent <= $opt_c) {
        finish("CRITICAL - $percent% ($free kB) free!$perfdata",$exit_codes{'CRITICAL'});
    }
    elsif ($percent <= $opt_w) {
        finish("WARNING - $percent% ($free kB) free!$perfdata",$exit_codes{'WARNING'});
    }   
    else {
        finish("OK - $percent% ($free kB) free.$perfdata",$exit_codes{'OK'});
    }
}
elsif ($opt_u) {
    my $percent = sprintf "%.1f", ($used / $total * 100);
    if ($percent >= $opt_c) {
        finish("CRITICAL - $percent% ($used kB) used!$perfdata",$exit_codes{'CRITICAL'});
    }
    elsif ($percent >= $opt_w) {
        finish("WARNING - $percent% ($used kB) used!$perfdata",$exit_codes{'WARNING'});
    }
    else {
        finish("OK - $percent% ($used kB) used.$perfdata",$exit_codes{'OK'});
    }
  }
}

# Show usage
sub usage() {
    print "\ncheck_mem.pl v1.0 - Nagios Plugin\n\n";
    print "usage:\n";
    print " check_mem.pl -<f|u> -w <warnlevel> -c <critlevel>\n\n";
    print "options:\n";
    print " -f Check FREE memory\n";
    print " -u Check USED memory\n";
    print " -C Count OS caches as FREE memory\n";
    print " -h Remove hugepages from the total memory count\n";
    print " -w PERCENT Percent free/used when to warn\n";
    print " -c PERCENT Percent free/used when critical\n";
    print "\nCopyright (C) 2000 Dan Larsson <dl\@tyfon.net>\n";
    print "check_mem.pl comes with absolutely NO WARRANTY either implied or explicit\n";
    print "This program is licensed under the terms of the\n";
    print "MIT License (check source code for details)\n";
    exit $exit_codes{'UNKNOWN'};
}

sub get_memory_info {
    my $used_memory_kb = 0;
    my $free_memory_kb = 0;
    my $total_memory_kb = 0;
    my $caches_kb = 0;
    my $hugepages_nr = 0;
    my $hugepages_size = 0;
    my $hugepages_kb = 0;

    my $uname;
    if ( -e '/usr/bin/uname') {
        $uname = `/usr/bin/uname -a`;
    }
    elsif ( -e '/bin/uname') {
        $uname = `/bin/uname -a`;
    }
    else {
        die "Unable to find uname in /usr/bin or /bin!\n";
    }
    print "uname returns $uname" if ($opt_v);
    if ( $uname =~ /Linux/ ) {
        my @meminfo = `/bin/cat /proc/meminfo`;
        foreach (@meminfo) {
            chomp;
        if (/^Mem(Total|Free):\s+(\d+) kB/) {
            my $counter_name = $1;
            if ($counter_name eq 'Free') {
                $free_memory_kb = $2;
            }  
            elsif ($counter_name eq 'Total') {
                $total_memory_kb = $2;
            }
        }
        elsif (/^(Buffers|Cached|SReclaimable):\s+(\d+) kB/) {
            $caches_kb += $2;
        }
        elsif (/^Shmem:\s+(\d+) kB/) {
            $caches_kb -= $1;
        }
        elsif (/^HugePages_Total:\s+(\d+)/) {
            $hugepages_nr = $1;
        }
        elsif (/^Hugepagesize:\s+(\d+) kB/) {
            $hugepages_size = $1;
        }
    }
    $hugepages_kb = $hugepages_nr * $hugepages_size;
    $used_memory_kb = $total_memory_kb - $free_memory_kb;
}
elsif ( $uname =~ /HP-UX/ ) {
    # HP-UX, thanks to Christoph F�rstaller
    my @meminfo = `/usr/bin/sudo /usr/local/bin/kmeminfo`;
    foreach (@meminfo) {
    chomp;
    if (/^Physical memory\s\s+=\s+(\d+)\s+(\d+.\d)g/) {
        $total_memory_kb = ($2 * 1024 * 1024);
    }
    elsif (/^Free memory\s\s+=\s+(\d+)\s+(\d+.\d)g/) {
        $free_memory_kb = ($2 * 1024 * 1024);
    }
  }
 $used_memory_kb = $total_memory_kb - $free_memory_kb;
}
elsif ( $uname =~ /FreeBSD/ ) {
    # The FreeBSD case. 2013-03-19 www.claudiokuenzler.com
    # free mem = Inactive*Page Size + Cache*Page Size + Free*Page Size
    my $pagesize = `sysctl vm.stats.vm.v_page_size`;
    $pagesize =~ s/[^0-9]//g;
    my $mem_inactive = 0;
    my $mem_cache = 0;
    my $mem_free = 0;
    my $mem_total = 0;
    my $free_memory = 0;
        my @meminfo = `/sbin/sysctl vm.stats.vm`;
        foreach (@meminfo) {
            chomp;
            if (/^vm.stats.vm.v_inactive_count:\s+(\d+)/) {
            $mem_inactive = ($1 * $pagesize);
            }
            elsif (/^vm.stats.vm.v_cache_count:\s+(\d+)/) {
            $mem_cache = ($1 * $pagesize);
            }
            elsif (/^vm.stats.vm.v_free_count:\s+(\d+)/) {
            $mem_free = ($1 * $pagesize);
            }
            elsif (/^vm.stats.vm.v_page_count:\s+(\d+)/) {
            $mem_total = ($1 * $pagesize);
            }
    }
    $free_memory = $mem_inactive + $mem_cache + $mem_free;
    $free_memory_kb = ( $free_memory / 1024);
    $total_memory_kb = ( $mem_total / 1024);
    $used_memory_kb = $total_memory_kb - $free_memory_kb;
    $caches_kb = ($mem_cache / 1024);
}
elsif ( $uname =~ /joyent/ ) {
  # The SmartOS case. 2014-01-10 www.claudiokuenzler.com
  # free mem = pagesfree * pagesize
  my $pagesize = `pagesize`;
  my $phys_pages = `kstat -p unix:0:system_pages:pagestotal | awk '{print \$NF}'`;
  my $free_pages = `kstat -p unix:0:system_pages:pagesfree | awk '{print \$NF}'`;
  my $arc_size = `kstat -p zfs:0:arcstats:size | awk '{print \$NF}'`;
  my $arc_size_kb = $arc_size / 1024;

  print "Pagesize is $pagesize" if ($opt_v);
  print "Total pages is $phys_pages" if ($opt_v);
  print "Free pages is $free_pages" if ($opt_v);
  print "Arc size is $arc_size" if ($opt_v);
  
  $caches_kb += $arc_size_kb;
  
  $total_memory_kb = $phys_pages * $pagesize / 1024;
  $free_memory_kb = $free_pages * $pagesize / 1024;
  $used_memory_kb = $total_memory_kb - $free_memory_kb;
}
elsif ( $uname =~ /SunOS/ ) {
    eval "use Sun::Solaris::Kstat";
    if ($@) { #Kstat not available
        if ($opt_C) {
            print "You can't report on Solaris caches without Sun::Solaris::Kstat available!\n";
            exit $exit_codes{UNKNOWN};
        }
        my @vmstat = `/usr/bin/vmstat 1 2`;
        my $line;
        foreach (@vmstat) {
          chomp;
          $line = $_;
        }
        $free_memo  ry_kb = (split(/ /,$line))[5] / 1024;
        my @prtconf = `/usr/sbin/prtconf`;
        foreach (@prtconf) {
            if (/^Memory size: (\d+) Megabytes/) {
                $total_memory_kb = $1 * 1024;
            }
        }
        $used_memory_kb = $total_memory_kb - $free_memory_kb;

}
else { # We have kstat
    my $kstat = Sun::Solaris::Kstat->new();
    my $phys_pages = ${kstat}->{unix}->{0}->{system_pages}->{physmem};
    my $free_pages = ${kstat}->{unix}->{0}->{system_pages}->{freemem};
    # We probably should account for UFS caching here, but it's unclear
    # to me how to determine UFS's cache size. There's inode_cache,
    # and maybe the physmem variable in the system_pages module??
    # In the real world, it looks to be so small as not to really matter,
    # so we don't grab it. If someone can give me code that does this,
    # I'd be glad to put it in.
    my $arc_size = (exists ${kstat}->{zfs} && ${kstat}->{zfs}->{0}->{arcstats}->{size}) ?
        ${kstat}->{zfs}->{0}->{arcstats}->{size} / 1024
        : 0;
    $caches_kb += $arc_size;
    my $pagesize = `pagesize`;

    $total_memory_kb = $phys_pages * $pagesize / 1024;
    $free_memory_kb = $free_pages * $pagesize / 1024;
    $used_memory_kb = $total_memory_kb - $free_memory_kb;
  }
}
elsif ( $uname =~ /Darwin/ ) {
  $total_memory_kb = (split(/ /,`/usr/sbin/sysctl hw.memsize`))[1]/1024;
  my $pagesize = (split(/ /,`/usr/sbin/sysctl hw.pagesize`))[1];
  $caches_kb = 0;
  my @vm_stat = `/usr/bin/vm_stat`;
  foreach (@vm_stat) {
        chomp;
        if (/^(Pages free):\s+(\d+)\.$/) {
            $free_memory_kb = $2*$pagesize/1024;
        }
        # 'caching' concept works different on MACH
        # this should be a reasonable approximation
        elsif (/^Pages (inactive|purgable):\s+(\d+).$/) {
            $caches_kb += $2*$pagesize/1024;
        }
    }
    $used_memory_kb = $total_memory_kb - $free_memory_kb;
}
elsif ( $uname =~ /AIX/ ) {
    my @meminfo = `/usr/bin/vmstat -vh`;
    foreach (@meminfo) {
        chomp;
        if (/^\s*([0-9.]+)\s+(.*)/) {
            my $counter_name = $2;
            if ($counter_name eq 'memory pages') {
                $total_memory_kb = $1*4;
            }
            if ($counter_name eq 'free pages') {
                $free_memory_kb = $1*4;
            }
            if ($counter_name eq 'file pages') {
                $caches_kb = $1*4;
            }
            if ($counter_name eq 'Number of 4k page frames loaned') {
                $free_memory_kb += $1*4;
            }
        }
    }
    $used_memory_kb = $total_memory_kb - $free_memory_kb;
}
else {
    if ($opt_C) {
        print "You can't report on $uname caches!\n";
        exit $exit_codes{UNKNOWN};
    }
 my $command_line = `vmstat | tail -1 | awk '{print \$4,\$5}'`;
 chomp $command_line;
    my @memlist = split(/ /, $command_line);

    # Define the calculating scalars
    $used_memory_kb = $memlist[0]/1024;
    $free_memory_kb = $memlist[1]/1024;
    $total_memory_kb = $used_memory_kb + $free_memory_kb;
 }
 return ($free_memory_kb,$used_memory_kb,$caches_kb,$hugepages_kb);
}

sub init {
    # Get the options
    if ($#ARGV le 0) {
        &usage;
    }
    else {
      getopts('c:fuChvw:');
    }

    # Shortcircuit the switches
    if (!$opt_w or $opt_w == 0 or !$opt_c or $opt_c == 0) {
        print "*** You must define WARN and CRITICAL levels!\n";
        &usage;
    }
    elsif (!$opt_f and !$opt_u) {
        print "*** You must select to monitor either USED or FREE memory!\n";
        &usage;
    }

    # Check if levels are sane
    if ($opt_w <= $opt_c and $opt_f) {
      print "*** WARN level must not be less than CRITICAL when checking FREE memory!\n";
      &usage;
    }
    elsif ($opt_w >= $opt_c and $opt_u) {
        print "*** WARN level must not be greater than CRITICAL when checking USED memory!\n";
        &usage;
    }
}
sub finish {
    my ($msg,$state) = @_;
    print "$msg\n";
    exit $state;
}

```
3. Renombrar el archivo como <code>check_mem.pl</code>

4. Darle permisos con el comando <code>chmod a+x check_mem.pl</code>

💡 **Puede ejecutar el comando con** <code>perl check_mem.pl</code> **para ver el uso del mismo.**

5. Edite el archivo <code>/usr/local/nagios/etc/nrpe.cfg</code> y agregue lo siguiente en la sección en la que se definen los comandos:

```
command[check_users]=/usr/local/nagios/libexec/check_users -w 5 -c 10
command[check_load]=/usr/local/nagios/libexec/check_load -r -w .15,.10,.05 -c .30,.25,.20
command[check_hda1]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /dev/sda1
command[check_zombie_procs]=/usr/local/nagios/libexec/check_procs -w 5 -c 10 -s Z
command[check_total_procs]=/usr/local/nagios/libexec/check_procs -w 150 -c 200

command[check_swap]=/usr/local/nagios/libexec/check_swap -w 20% -c 10%
command[check_uptime]=/usr/local/nagios/libexec/check_uptime

#Agregue la siguiente linea#
command[check_mem]=perl /usr/local/nagios/libexec/check_mem.pl -f -C -w 20% -c 10%

```
6. Reiniciar el servicio de Nagios.


<a name="52-inspser"></a>
### 5.2 Configuración en el servidor

1. Editar el archivo <code>/usr/local/nagios/etc/objects/linux.cfg</code> y agregar el servicio:

```
define service{
        use                     generic-service
        host_name               cliente_linux
        service_description     Memoria Ram
        check_command           check_nrpe!check_mem
}

```

2. Reiniciar el servicio de NRPE.


<a name="6-insa"></a>
## 6. Monitoreo del servicio de Apache

Del servicio HTTP se puede:

- Monitorear que el proceso exista y esté corriendo o no.

- Monitorear el puerto en el que opera Apache.

- Monitorear que la página de Apache sirve esté disponible.

- Monitorear el tiempo de respuesta del servicio.

- Monitorear los slots del servicio.

  * Cantidad de slots ocupados.

  * Cantidad de slots disponibles.

  * Cantidad de slots inactivos.

<a name="61-confcli"></a>
### 6.1 Configuración del cliente

1. Instalar el servicio de Apache con:

<code>yum install -y httpd</code>

2. Habilitar por defector el servicio de HTTP con:

<code>chkconfig httpd on</code>

3. En caso de tener el Firewall habilitado agregar el servicio de <code>httpd</code>y el puerto <code>80</code>

4. Habilitar el server status agregando al final las siguientes líneas al archivo principal de configuración del apache que se encuentra en <code>/etc/httpd/conf</code> y lleva por nombre <code>httpd.conf</code>

```
<location /server-status>
        setHandler server-status
        Order deny,allow
        Deny from all
        Allow from 192.168.50.0/24      #Red desde la que se podrá acceder al location
</location>

``` 

5. Crear y editar el archivo <codeindex.html</code> con <code>vim /var/www/html/index.html</code> y agregar las líneas

```
<h1>Hola apache</h1></br>
<p>Página de prueba del servicio de apache en el cliente linux 192.168.50.2</p>

``` 

6. Editar el archivo <code>/usr/local/nagios/etc/nrpe.cfg</code>

7. Agregar la siguiente línea para definir el comando:

```
command[check_users]=/usr/local/nagios/libexec/check_users -w 5 -c 10
command[check_load]=/usr/local/nagios/libexec/check_load -r -w .15,.10,.05 -c .30,.25,.20
command[check_hda1]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /dev/sda1
command[check_zombie_procs]=/usr/local/nagios/libexec/check_procs -w 5 -c 10 -s Z
command[check_total_procs]=/usr/local/nagios/libexec/check_procs -w 150 -c 200

command[check_swap]=/usr/local/nagios/libexec/check_swap -w 20% -c 10%
command[check_uptime]=/usr/local/nagios/libexec/check_uptime

command[check_mem]=perl /usr/local/nagios/libexec/check_mem.pl -f -C -w 20% -c 10%

#Agregue la siguiente linea#
command[check_proc_apache]=/usr/local/nagios/libexec/check_procs -c 1: -C httpd

```

8. Reiniciar el servicio de Apache con:

<code>service httpd restart</code>

💡 **Puede verificar que todo funciona correctamente apuntando desde el navegador a la** <code>ip</code> **del cliente y luego agregando en
la url** <code>/server-status</code>

9. Reiniciar el servicio de NRPE con <code>service nrpe restart</code>


<a name="62-confser"></a>
### 6.2 Configuración del servidor

1. Dirigirse al directorio <code>/usr/local/nagios/libexec</code>

2. Crear y editar el archivo <code>check_apachestatus</code> y pegar en el lo siguiente:


```
#!/usr/bin/perl -w
####################### check_apachestatus.pl #######################
# Version : 1.1
# Date : 27 Jul 2007
# Author : De Bodt Lieven (Lieven.DeBodt at gmail.com)
# Licence : GPL - http://www.fsf.org/licenses/gpl.txt
#############################################################
#
# help : ./check_apachestatus.pl -h

use strict;
use Getopt::Long;
use LWP::UserAgent;
use Time::HiRes qw(gettimeofday tv_interval);

# Nagios specific
use lib "/usr/local/nagios/libexec";
use utils qw(%ERRORS $TIMEOUT);
#my %ERRORS=('OK'=>0,'WARNING'=>1,'CRITICAL'=>2,'UNKNOWN'=>3,'DEPENDENT'=>4);

# Globals
    
my $Version='1.0';
my $Name=$0;

my $o_host = undef; # hostname
my $o_help= undef; # want some help ?
my $o_port = undef; # port
my $o_version= undef; # print version
my $o_warn_level= undef; # Number of available slots that will cause a warning
my $o_crit_level= undef; # Number of available slots that will cause an error
my $o_timeout= 15; # Default 15s Timeout

# functions

sub show_versioninfo { print "$Name version : $Version\n"; }

sub print_usage {
    print "Usage: $Name -H <host> [-p <port>] [-t <timeout>] [-w <warn_level> -c <crit_level>] [-V]\n";
}

# Get the alarm signal
$SIG{'ALRM'} = sub {
    print ("ERROR: Alarm signal (Nagios time-out)\n");
    exit $ERRORS{"CRITICAL"};
};

sub help {
    print "Apache Monitor for Nagios version ",$Version,"\n";
    print "GPL licence, (c)2006-2007 De Bodt Lieven\n\n";
    print_usage();
    print <<EOT;
-h, --help
    print this help message
-H, --hostname=HOST
    name or IP address of host to check
-p, --port=PORT
    Http port
-t, --timeout=INTEGER
    timeout in seconds (Default: $o_timeout)
-w, --warn=MIN
    number of available slots that will cause a warning
    -1 for no warning
-c, --critical=MIN
    number of available slots that will cause an error
-V, --version
    prints version number
Note :
    The script will return
        * Without warn and critical options:
            OK if we are able to connect to the apache server's status page,
            CRITICAL if we aren't able to connect to the apache server's status page,,
        * With warn and critical options:
            OK if we are able to connect to the apache server's status page and #available slots > <warn_level>,
            WARNING if we are able to connect to the apache server's status page and #available slots <= <warn_level>,
            CRITICAL if we are able to connect to the apache server's status page and #available slots <= <crit_level>,
            UNKNOWN if we aren't able to connect to the apache server's status page

Perfdata legend:
"_;S;R;W;K;D;C;L;G;I;."
_ : Waiting for Connection
S : Starting up
R : Reading Request
W : Sending Reply
K : Keepalive (read)
D : DNS Lookup
C : Closing connection
L : Logging
G : Gracefully finishing
I : Idle cleanup of worker
. : Open slot with no current process

EOT
}

sub check_options {
    Getopt::Long::Configure ("bundling");
    GetOptions(
        'h' => \$o_help, 'help' => \$o_help,
        'H:s' => \$o_host, 'hostname:s' => \$o_host,
        'p:i' => \$o_port, 'port:i' => \$o_port,
        'V' => \$o_version, 'version' => \$o_version,
        'w:i' => \$o_warn_level, 'warn:i' => \$o_warn_level,
        'c:i' => \$o_crit_level, 'critical:i' => \$o_crit_level,
        't:i' => \$o_timeout, 'timeout:i' => \$o_timeout,

);

    if (defined ($o_help)) { help(); exit $ERRORS{"UNKNOWN"}};
    if (defined($o_version)) { show_versioninfo(); exit $ERRORS{"UNKNOWN"}};
    if (((defined($o_warn_level) && !defined($o_crit_level)) || (!defined($o_warn_level) && defined($o_crit_level))) || ((defined($o_warn_l
evel) && defined($o_crit_level)) && (($o_warn_level != -1) && ($o_warn_level <= $o_crit_level)))) {
        print "Check warn and crit!\n"; print_usage(); exit $ERRORS{"UNKNOWN"}
    }
    # Check compulsory attributes
    if (!defined($o_host)) { print_usage(); exit $ERRORS{"UNKNOWN"}};
}

########## MAIN ##########

check_options();

my $ua = LWP::UserAgent->new( protocols_allowed => ['http'], timeout => $o_timeout);
my $timing0 = [gettimeofday];
my $response = undef;
if (!defined($o_port)) {
    $response = $ua->get('http://' . $o_host . '/server-status');
} else {
    $response = $ua->get('http://' . $o_host . ':' . $o_port . '/server-status');
}
my $timeelapsed = tv_interval ($timing0, [gettimeofday]);

my $webcontent = undef;
if ($response->is_success) {
    $webcontent=$response->content;
    my @webcontentarr = split("\n", $webcontent);
    my $i = 0;
    my $BusyWorkers=undef;
    my $IdleWorkers=undef;
    # Get the amount of idle and busy workers(Apache2)/servers(Apache1)
    while (($i < @webcontentarr) && ((!defined($BusyWorkers)) || (!defined($IdleWorkers)))) {
        if ($webcontentarr[$i] =~ /(\d+)\s+requests\s+currently\s+being\s+processed,\s+(\d+)\s+idle\s+....ers/) {
            ($BusyWorkers, $IdleWorkers) = ($webcontentarr[$i] =~ /(\d+)\s+requests\s+currently\s+being\s+processed,\s+(\d+)\s+idle\s+....er
s/);
        }
        $i++;
     }

    # Get the scoreboard
    my $ScoreBoard = "";
    $i = 0;
    my $PosPreBegin = undef;
    my $PosPreEnd = undef;
    while (($i < @webcontentarr) && ((!defined($PosPreBegin)) || (!defined($PosPreEnd)))) {
        if (!defined($PosPreBegin)) {
            if ( $webcontentarr[$i] =~ m/<pre>/i ) {
                $PosPreBegin = $i;
            }
        }
        if (defined($PosPreBegin)) {
            if ( $webcontentarr[$i] =~ m/<\/pre>/i ) {
                $PosPreEnd = $i;
            }
        }
        $i++;
}
for ($i = $PosPreBegin; $i <= $PosPreEnd; $i++) {
    $ScoreBoard = $ScoreBoard . $webcontentarr[$i];
}
$ScoreBoard =~ s/^.*<[Pp][Rr][Ee]>//;
$ScoreBoard =~ s/<\/[Pp][Rr][Ee].*>//;

my $CountOpenSlots = ($ScoreBoard =~ tr/\.//);
if (defined($o_crit_level) && ($o_crit_level != -1)) {
    if (($CountOpenSlots + $IdleWorkers) <= $o_crit_level) {
        # printf("CRITICAL %f seconds response time. Idle %d, busy %d, open slots %d | %d;%d;%d;%d;%d;%d;%d;%d;%d;%d;%d\n", $timeelapsed, $I
dleWorkers, $BusyWorkers, $CountOpenSlots, ($ScoreBoard =~ tr/\_//), ($ScoreBoard =~ tr/S//),($ScoreBoard =~ tr/R//),($ScoreBoard =~ tr/
W//),($ScoreBoard =~ tr/K//),($ScoreBoard =~ tr/D//),($ScoreBoard =~ tr/C//),($ScoreBoard =~ tr/L//),($ScoreBoard =~ tr/G//),($ScoreBoard
=~ tr/I//), $CountOpenSlots);
    printf("CRITICAL %f seconds response time. Idle %d, busy %d, open slots %d | 'Response_Time'=%f;;;; 'Idle'=%d;;;; 'Busy'=%d;;;; Open_Sl
  ots=%d;;;; \n", $timeelapsed, $IdleWorkers, $BusyWorkers, $CountOpenSlots, $timeelapsed, $IdleWorkers, $BusyWorkers, $CountOpenSlots);
        exit $ERRORS{"CRITICAL"}
      }
    }
    if (defined($o_warn_level) && ($o_warn_level != -1)) {
        if (($CountOpenSlots + $IdleWorkers) <= $o_warn_level) {
          #printf("WARNING %f seconds response time. Idle %d, busy %d, open slots %d | %d;%d;%d;%d;%d;%d;%d;%d;%d;%d;%d\n", $timeelapsed, $Id
leWorkers, $BusyWorkers, $CountOpenSlots, ($ScoreBoard =~ tr/\_//), ($ScoreBoard =~ tr/S//),($ScoreBoard =~ tr/R//),($ScoreBoard =~ tr/
W//),($ScoreBoard =~ tr/K//),($ScoreBoard =~ tr/D//),($ScoreBoard =~ tr/C//),($ScoreBoard =~ tr/L//),($ScoreBoard =~ tr/G//),($ScoreBoard
=~ tr/I//), $CountOpenSlots);
    printf("WARNING %f seconds response time. Idle %d, busy %d, open slots %d | 'Response_Time'=%f;;;; 'Idle'=%d;;;; 'Busy'=%d;;;; Open_Slo
ts=%d;;;; \n", $timeelapsed, $IdleWorkers, $BusyWorkers, $CountOpenSlots, $timeelapsed, $IdleWorkers, $BusyWorkers, $CountOpenSlots);
        exit $ERRORS{"WARNING"}
    }
  }
  #printf("OK %f seconds response time. Idle %d, busy %d, open slots %d | %d;%d;%d;%d;%d;%d;%d;%d;%d;%d;%d\n", $timeelapsed, $IdleWorker
s, $BusyWorkers, $CountOpenSlots, ($ScoreBoard =~ tr/\_//), ($ScoreBoard =~ tr/S//),($ScoreBoard =~ tr/R//),($ScoreBoard =~ tr/W//),($Sco
reBoard =~ tr/K//),($ScoreBoard =~ tr/D//),($ScoreBoard =~ tr/C//),($ScoreBoard =~ tr/L//),($ScoreBoard =~ tr/G//),($ScoreBoard =~ tr/
I//), $CountOpenSlots);
    printf("OK %f seconds response time. Idle %d, busy %d, open slots %d | 'Response_Time'=%f;;;; 'Idle'=%d;;;; 'Busy'=%d;;;; Open_Slots=%
d;;;; \n", $timeelapsed, $IdleWorkers, $BusyWorkers, $CountOpenSlots, $timeelapsed, $IdleWorkers, $BusyWorkers, $CountOpenSlots);
        exit $ERRORS{"OK"}
}
else {
    if (defined($o_warn_level) || defined($o_crit_level)) {
        printf("UNKNOWN %s\n", $response->status_line);
        exit $ERRORS{"UNKNOWN"}
    } else {
    printf("CRITICAL %s\n", $response->status_line);
    exit $ERRORS{"CRITICAL"}
    }
  }

```

3. Renombrar el archivo como <code>check_apachestatus.pl</code>

4. Darle permisos con el comando <code>chmod a+x check_apachestatus.pl</code>

5. Ejecutar el comando <code>yum install cpan -y</code>

6. Ejecutar el comando <code>perl -MCPAN -e 'install Bundle::LWP’</code>

💡 **Verifique el uso del plugin de monitoreo de Apache o si hay algún error ejecutando el mismo con el comando** <code>perl check_apachestatus.pl</code>

7. Agregar la definición del comando a <code>/usr/local/nagios/etc/objects/commands.cfg</code>


```
define command{
    command_name        check_apache
    command_line        $USER1$/check_apachestatus.pl -H $HOSTADDRESS$ -p 80 -t 15 -w 10 -c 5
}

```

8. Agregar la definición de los servicios a <code>/usr/local/nagios/etc/objects/linux.cfg</code>

```
define service{
        use                             generic-service
        host_                           name cliente_linux
        service_description             Apache
        check_command                   check_apache
}

define service{
        use                             generic-service
        host_name                       cliente_linux
        service_description             Puerto 80
        check_command                   check_tcp!80
}

define service{
        use                             generic-service
        host_name                       cliente_linux
        service_description             Control de httpd
        check_command                   check_nrpe!check_proc_apache
}

define service{
        use                             generic-service
        host_name                       cliente_linux
        service_description             Control http
        check_command                   check_http
}

```

💡 **El comando no lleva** <code>check_nrpe!</code> **ya que es un plugin que Nagios ejecuta localmente por el servidor de Nagios, y obtiene el mismo la información del estado del servicio sin conectarse al host monitoreado a través del NRPE.**


9. Ejecutar el comando <code>nagios_check</code> y si todo está correcto reiniciar el servicio de Nagios.

10. Reiniciar el servicio de Nagios.

<a name="7-apar"></a>
## 7. Cambiar la apariencia de Nagios

1. Ejecutar el comando <code>wget https://github.com/ynlamy/vautour-style/releases/latest/download/vautour_style.zip</code>

2. Hacer una copia de la pagina actual con <code>tar -czvf backup-pagina.tar.gz /usr/local/nagios/share/</code>

3. Ejecutar el comando <code>unzip -d /usr/local/nagios/share/ vautour_style.zip</code>

4. Reiniciar el servicio de Nagios.

<a name="8-grafic"></a>
## 8. Gráficos con PNP4Nagios

<a name="81-confserver"></a>
### 8.1 Configuración en el servidor

1. Instalar los paquetes <code>yum install rrdtool perl-Time-HiRes rrdtool-perl php-gd php-xml -y</code>

2. Descargar las fuentes de:


```
https://sourceforge.net/projects/pnp4nagios/files/

```

en la máquina virtual ejecutar:

<code>wget https://downloads.sourceforge.net/project/pnp4nagios/PNP-0.6/pnp4nagios-0.6.26.tar.gz</code>

3. Descomprimir el archivo descargado con <code>tar -xzvf pnp4nagios-0.6.26.tar.gz</code>

4. Acceder al directorio <code>pnp4nagios-0.6.26</code> y ejecutar la configuración con <code>./configure</code>

5. Ejecutar el comando <code>make all</code> para compilar la configuración y luego un <code>make fullinstall</code>

6. Ejecutar el comando <code>chkconfig npcd on</code> para habilitar el servicio de inicio.

7. Reiniciar el servicio de apache con <code>service httpd restart</code>

8. Renombrar el archivo <code>/usr/local/pnp4nagios/share/install.php</code> con <code>mv /usr/local/pnp4nagios/share/install.php /usr/local/pnp4nagios/share/install.php.BKP</code>

9. Editar el archivo <code>/usr/local/nagios/etc/nagios.cfg</code> y agregar al final las siguientes líneas para habilitar el guardado de los datos de performance de los servicios y hosts y decirle a Nagios como manejar tales datos.

```
process_performance_data=1

  service_perfdata_file=/usr/local/pnp4nagios/var/service-perfdata
  service_perfdata_file_template=DATATYPE::SERVICEPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tSERVICEDESC::$SERVICEDESC$\tSERVICEPERF
DATA::$SERVICEPERFDATA$\tSERVICECHECKCOMMAND::$SERVICECHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$\tSERVICESTAT
E::$SERVICESTATE$\tSERVICESTATETYPE::$SERVICESTATETYPE$
  service_perfdata_file_mode=a
  service_perfdata_file_processing_interval=15
  service_perfdata_file_processing_command=process-service-perfdata-file

  host_perfdata_file=/usr/local/pnp4nagios/var/host-perfdata
  host_perfdata_file_template=DATATYPE::HOSTPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tHOSTPERFDATA::$HOSTPERFDATA$\tHOSTCHECKCOMMAN
D::$HOSTCHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$
  host_perfdata_file_mode=a
  host_perfdata_file_processing_interval=15
  host_perfdata_file_processing_command=process-host-perfdata-file

```

10. Editar el archivo <code>/usr/local/nagios/etc/objects/commands.cfg</code> y agregar al final las siguientes líneas para el manejo de los archivos de servicios y host. Lo que se hace es mover los archivos de un directorio a otro y poner una marca de tiempo.

```
define command {
    command_name process-service-perfdata-file
    command_line /bin/mv /usr/local/pnp4nagios/var/service-perfdata /usr/local/pnp4nagios/var/spool/service-perfdata.$TIMET$
 }
 define command {
    command_name process-host-perfdata-file
    command_line /bin/mv /usr/local/pnp4nagios/var/host-perfdata /usr/local/pnp4nagios/var/spool/host-perfdata.$TIMET$
 }

```

11. Editar el archivo <code>/usr/local/nagios/etc/objects/templates.cfg</code> y agregar al final las siguientes líneas para agregar los botones desde los que se puede acceder a PNP4Nagios.

```
define host {
    name host-pnp
    action_url /pnp4nagios/index.php/graph?host=$HOSTNAME$&srv=_HOST_' class='tips' rel='/pnp4nagios/index.php/popup?host=$HOSTNAME$&srv=
_HOST_
    register 0
  }

  define service {
    name srv-pnp
    action_url /pnp4nagios/index.php/graph?host=$HOSTNAME$&srv=$SERVICEDESC$' class='tips' rel='/pnp4nagios/index.php/popup?host=$HOSTNAM
E$&srv=$SERVICEDESC$
    register 0
  }

```

12. En el directorio <code>pnp4nagios-0.6.26</code> copiar el fichero <code>contrib/ssi/status-header.ssi</code> a la carpeta <code>/usr/local/nagios/share/ssi/</code> con el comando <code>cp contrib/ssi/status-header.ssi /usr/local/nagios/share/ssi/</code> para agregar un hover al botón previamente agregado.

13. Reiniciar el servicio de <code>npcd</code>

14. Reiniciar el servicio de Nagios.

15. Editar el archivo <code>/usr/local/nagios/etc/objects/linux.cfg</code> y agregar <code>host-pnp</code> en la definición de los servidores y <code>srv-pnp</code> en todos los servicios.

```
define host{
        use                     linux-server,host-pnp
        host_name               cliente_linux
        alias                   Linux Centos
        address                 192.168.50.2
}

define service{
        use                     generic-service,srv-pnp
        host_name               cliente_linux
        service_description     Hard Disk
        check_command           check_nrpe!check_hda1
}

```

16. Ejecutar el comando <code>sed -i 's:if(sizeof($pages:if(is_array($pages) && sizeof($pages:'/usr/local/pnp4nagios/share/application/models/data.php</code>

17. Cambia todas las líneas donde se define Class Services_JSON por Class __Services_JSON en el archivo <code>/usr/local/pnp4nagios/share/application/lib/json.php</code>

18. Reiniciar el servicio de Nagios.


<a name="9-bibl"></a>
## 9. Bibliografí

- [Curso de monitoreo con Nagios Core](https://www.youtube.com/playlist?list=PLf0g2cV4iCkE9vRbsSoe5f428zMNlasKd)

- [Extra packages for Enterprise Linux](https://docs.fedoraproject.org/en-US/epel/#How_can_I_use_these_extra_packages.3F)

- [Nagios Core - Installing Nagios Core From Source](https://support.nagios.com/kb/article/nagios-core-installing-nagios-core-from-source-96.html#RHEL)

- [Nagios Plugins](https://support.nagios.com/kb/article.php?id=569#CentOS)

- [How to install pnp4nagios to Display Graphics in Nagios?](https://sysadminote.com/how-to-install-pnp4nagios-to-display-graphics-in-nagios/)

- [pnp4nagios #148 issue](https://github.com/lingej/pnp4nagios/issues/148)






