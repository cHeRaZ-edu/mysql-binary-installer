# INSTALAR MYSQL BINARIO PASOS CORRECTOS.

Este proyecto fue creado para poder configurar una versión de MySQL a partir de los binarios, en lugar de usar los comandos por defecto. Puede elegir cualquier compilado para ejecutarlo. No incluye un cliente de interfaz como Workbench o PhpMyAdmin.

Binarios: https://downloads.mysql.com/archives/community/

Instalar las librerias para el funcionamiento correcto

MySQL tiene una dependencia de la biblioteca. Inicialización del directorio de datos y posterior servidor Los pasos de inicio fallan si esta biblioteca no está instalada localmente. Si es necesario, instálelo con el paquete adecuado director. Por ejemplo, en sistemas basados en Yum: libaio

## Descargar binarios
```sh
#Este es un ejemplo con la version mysql 8.0.36
/home/$USER/Downloads/mysql-8.0.36-linux-glibc2.28-x86_64.tar.xz
```

## Actualizar linux
```sh
sudo apt update
```

## Instalar libs en debian

```sh
apt-cache search libaio # search for info
```
```sh
sudo apt-get install libaio1 # install library
```
```sh
sudo apt-get install libncurses5 # EN CASO DE INSTALAR VERSIONES ANTIGUAS COMO  MYSQL 5.7.44
```


## Crear el grupo mysql y usuario mysql
```sh
sudo groupadd mysql
```
```sh
sudo useradd -r -g mysql -s /bin/false mysql
```

## Agregar el binario en local y crear enlace
```sh
cd /usr/local
```
```sh
sudo tar xvf /home/$USER/Downloads/mysql-8.0.36-linux-glibc2.28-x86_64.tar.xz
```
```sh
sudo ln -s /usr/local/mysql-8.0.36-linux-glibc2.28-x86_64 mysql
```
## Crear carpeta de directorios de datos
```sh
cd /usr/local/mysql
```
```sh
sudo mkdir mysql-files
```
```sh
sudo chown mysql:mysql mysql-files
```
```sh
sudo chmod 750 mysql-files
```
## Crear archivo my.cnf opciones para directorio de datos
```sh
cd /etc
```
```sh
sudo touch my.cnf
```
```sh
sudo chown root:root my.cnf
```
```sh
sudo chmod 644 my.cnf
```

### Agregar configuracion en my.cnf

```sh
sudo nano my.cnf
```

```sh
[mysqld]
datadir=/usr/local/mysql/data
socket=/tmp/mysql.sock
port=3306
log-error=/usr/local/mysql/data/localhost.localdomain.err
user=mysql
secure_file_priv=/usr/local/mysql/mysql-files
local_infile=OFF
```

## Preparar datos de directorio
```sh
cd /usr/local/mysql
```
```sh
sudo mkdir data
```
```sh
sudo chmod 750 data
```
```sh
sudo chown mysql:mysql data
```

## Inicializar la configuracion de datos
```sh
cd /usr/local/mysql
```
```sh
sudo bin/mysqld --defaults-file=/etc/my.cnf --initialize
```

### En esta configuracion mysql genera una contraseña alaeatora que se guarda en un log
```sh
cd /usr/local/mysql
```
```sh
sudo cat data/localhost.localdomain.err
```

#### EJEMPLO DE LA SALIDA DEL LOG
```
2024-03-30T20:23:07.225209Z 0 [System] [MY-013169] [Server] /usr/local/mysql-8.0.36-linux-glibc2.28-x86_64/bin/mysqld (mysqld 8.0.36) initializing of server in progress as process 2712
2024-03-30T20:23:07.233881Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2024-03-30T20:23:08.435886Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2024-03-30T20:23:11.121428Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: YOUR_PASSWORD
```
Recuerde la contraseña

## Crear archivo de servicio mysql

### Crear el servicio de systemd
```sh
cd /usr/lib/systemd/system
```
```sh
sudo touch mysqld.service
```
```sh
sudo chmod 644 mysqld.service
```
```sh
sudo nano mysqld.service
```
```sh
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target

[Service]
User=mysql
Group=mysql

# Have mysqld write its state to the systemd notify socket
Type=notify

# Disable service start and stop timeout logic of systemd for mysqld service.
TimeoutSec=0

# Start main service
ExecStart=/usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf $MYSQLD_OPTS 

# Use this to switch malloc implementation
EnvironmentFile=-/etc/sysconfig/mysql

# Sets open_files_limit
LimitNOFILE = 10000

Restart=on-failure

RestartPreventExitStatus=1

# Set environment variable MYSQLD_PARENT_PID. This is required for restart.
Environment=MYSQLD_PARENT_PID=1

PrivateTmp=false

```
#### Comandos con systemd
```sh
sudo systemctl enable mysqld.service # Habilite el servicio
```
```sh
sudo systemctl start mysqld # Inicie el servicio de mysql
```
```sh
sudo systemctl status mysqld # Verifique el estatus del servicio
```
Salida
```
● mysqld.service - MySQL Server
     Loaded: loaded (/lib/systemd/system/mysqld.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2024-03-30 14:28:28 CST; 12min ago
       Docs: man:mysqld(8)
             http://dev.mysql.com/doc/refman/en/using-systemd.html
   Main PID: 2929 (mysqld)
     Status: "Server is operational"
      Tasks: 38 (limit: 9369)
     Memory: 368.9M
        CPU: 10.341s
     CGroup: /system.slice/mysqld.service
             └─2929 /usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf

Mar 30 14:28:26 autie-VirtualBox systemd[1]: Starting MySQL Server...
Mar 30 14:28:28 autie-VirtualBox systemd[1]: Started MySQL Server.
```

### Alternativa Sysvinit

```sh
sudo nano /etc/init.d/mysql
```

```sh
#!/bin/bash
### BEGIN INIT INFO
# Provides:          mi_servicio
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Inicia el servicio mi_servicio
# Description:       Este script inicia y detiene el servicio mi_servicio
### END INIT INFO

# Nombre del programa/servicio
NAME="mysql"
# Ruta del programa a ejecutar
PATH_TO_EXECUTABLE="/usr/local/mysql/bin/mysqld"
PATH_TO_MY_CNF="/etc/my.cnf"
PIDFILE="/var/run/$NAME.pid"
MYSQLD_OPTS="" #path options not defined

start() {
    echo "Iniciando $NAME..."
    if [ -f $PIDFILE ]; then
        echo "$NAME ya está ejecutándose."
    else
        $PATH_TO_EXECUTABLE --defaults-file=$PATH_TO_MY_CNF $MYSQLD_OPTS &
        echo $! > $PIDFILE
        if [ $? -eq 0 ]; then
            echo "$NAME iniciado con éxito."
        else
            echo "Error al iniciar $NAME."
            exit 1
        fi
    fi
}

stop() {
    echo "Deteniendo $NAME..."
    if [ -f $PIDFILE ]; then
        kill $(cat $PIDFILE)
        if [ $? -eq 0 ]; then
            rm -f $PIDFILE
            echo "$NAME detenido."
        else
            echo "Error al detener $NAME."
            exit 1
        fi
    else
        echo "$NAME no está ejecutándose."
    fi
}

restart() {
    echo "Reiniciando $NAME..."
    stop
    start
}

status() {
    if [ -f $PIDFILE ]; then
        echo "$NAME está ejecutándose con PID $(cat $PIDFILE)."
    else
        echo "$NAME no está ejecutándose."
    fi
}


case "$1" in
    start)
        start
        ;;
    stop)
        stop
        ;;
    restart)
        restart
        ;;
    status)
        status
        ;;
    *)
        echo "Uso: /etc/init.d/$NAME {start|stop|restart|status}"
        exit 1
        ;;
esac

exit 0
```

#### Permisos
```sh
sudo chmod 755 /etc/init.d/mysql
```
#### Agregar al inicio de arranque
```sh
sudo update-rc.d mysql defaults
```

#### Comandos
```sh
sudo service mysql start # Inicie el servicio de mysql
```
```sh
sudo service mysql stop # Detener el servicio de mysql
```
```sh
sudo service mysql status # Verifique el estatus del servicio
```
```sh
sudo service mysql restart # Reniciar
```

## Resetar contraseña
Use su contraseña guardada
En mysql cambie la contraseña
```sh
cd /usr/local/mysql
```
```sh
bin/mysql -u root -p
```
Cambie su contraseña
```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';
```
```sql
exit;
```

## Agregar variable de entorno de mysql
```sh
cd ~
```
```sh
sudo nano .bashrc
```
Al final del archivo agregue esto
```sh
PATH=$PATH:/usr/local/mysql/bin
```


## Exportar base de datos
```sh
/usr/local/mysql/bin/mysqldump -u user_example -h host_example -p --no-tablespaces --skip-triggers --routines --events --compact database_example > dump_file_database_00001.sql
```


## PREPARAR CONEXION REMOTA

### PARA MYSQL 8
```sql
CREATE USER 'your_user'@'%' IDENTIFIED BY 'root';
```
```sql
GRANT ALL PRIVILEGES ON *.* TO 'your_user'@'%';
```
```sql
FLUSH PRIVILEGES;
```
### Para MYSQL 5
```sql
CREATE USER 'your_user'@'localhost' IDENTIFIED BY 'root';
```
```sql
GRANT ALL PRIVILEGES ON *.* TO 'your_user'@'localhost';
```
```sql
grant all privileges on *.* to 'your_user'@'%' identified by 'root';
```
```sql
FLUSH PRIVILEGES;
```

## Reniciar mysql
### Systemd
```sh
sudo systemctl restart mysqld
```
### Sysvinit
```sh
sudo service mysql restart
```
