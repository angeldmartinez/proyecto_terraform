# Que es terraform

![](https://www.developerro.com/public/uploads/2019/07/hashicorp-terraform.png)

## ¿En qué consiste la infraestructura como código?

La infraestructura como código (IaC) trata la infraestructura de configuración como un software de programación. De hecho, esto empieza a desdibujar los límite entre la escritura de aplicaciones y la creación de entornos en los que se ejecutan. Las aplicaciones pueden contener scripts para crear y organizar sus propias máquinas virtuales (VM). Se trata de una parte fundamental de la computación en la nube y esencial para DevOps.

La infraestructura como código permite gestionar las máquinas virtuales de manera programática, eliminando la necesidad de configuración manual y actualización de las piezas individuales de hardware. Gracias a ello, la infraestructura se vuelve altamente «elástica», es decir, repetible y escalable. Un operador puede implementar y gestionar una máquina o 1000 usando el mismo conjunto de códigos. Velocidad, ahorros de costes y reducción del riesgo son las recompensas naturales de la infraestructura como código.

El concepto de infraestructura como código es el marco que ha dado origen a DevOps. Una línea cada vez más delgada entre el código que ejecuta las aplicaciones y el código que configura la infraestructura indica que los desarrolladores y los profesionales de operaciones comparten cada vez las responsabilidades de trabajo.

## ¿Qué es Terraform?

Terraform es una herramienta de orquestación de código abierto desarrollado por Hashicorp que nos permite definir nuestra infraestructura como código , esto quiere decir que es posible escribir en un fichero de texto la definición de nuestra infraestructura usando un lenguaje de programación declarativo y simple.

Terraform tiene soporte para una gran cantidad de proveedores de infraestructura local o en la nube, Amazon Web Services (AWS) , Digital Ocean , Microsoft Azure , VMware vSphere.

## Ventajas de utilizar terraform

Terraform es una herramienta de orquestación de código abierto desarrollado por Hashicorp que nos permite definir nuestra infraestructura como código, esto quiere decir que es posible escribir en un fichero de texto la definición de nuestra infraestructura usando un lenguaje de programación declarativo y simple.

Terraform no se limita a un proveedor en específico.

Proporciona una sintaxis simple y unificada que permite administrar casi cualquier recurso en lugar de requerir que se utilicen herramientas independientes para cada plataforma y servicio.

Las configuraciones que se realizan en Terraform puede ser compartida y reutilizable.

El modelo de su centro de datos puede ser versionado, de esta forma es más sencillo observar el progreso de nuestro servicio y controlar los cambios.

Infraestructura como código: La infraestructura se describe utilizando una sintaxis de alto nivel. Esto permite que un blueprint sea versionado y tratado como lo haría con cualquier otro código. Estos archivos que describen la infraestructura pueden ser compartidos y reutilizados.

Planes de Ejecución: Terraform tiene un paso de “planificación” donde genera un plan de ejecución. El plan de ejecución muestra lo que hará Terraform cuando se ejecute. Esto permite evitar sorpresas.

Gráfico de recursos: Terraform crea un gráfico de todos los recursos y paraleliza la creación y modificación de cualquier recurso. Con esto los operadores obtienen información sobre las dependencias en la infraestructura.

Automatización de cambios: Los conjuntos de cambios complejos se pueden aplicar a su infraestructura con una mínima interacción humana. Con el plan de ejecución y el gráfico de recursos mencionados anteriormente, se sabe exactamente qué cambiará Terraform y en qué orden, evitando muchos posibles errores humanos.

## ¿Cómo usar Terraform?

Instalación de Terraform:

### Instalacion en Windows

1.  Descargar el paquete Descargar el paquete según la plataforma. El paquete debe descargarse desde https://www.terraform.io/downloads.html.

2.  Creamos un directorio para los binarios de Terraform

    [root@wollfex Descargas]# mkdir /opt/terraform

3.  Movemos el archivo descargado anteriormente al interior del directorio:

        [root@wollfex Descargas]# mv terraform_0.11.13_linux_amd64.zip /opt/terraform/

4.  Nos situamos en el directorio terraformy descomprimimos los binarios

        [root@wollfex Descargas]# cd /opt/terraform/
        [root@wollfex terraform]# unzip terraform_0.11.13_linux_amd64.zip
        Archive:  terraform_0.11.13_linux_amd64.zip
        inflating: terraform

5.  Exportamos las variables de entorno para añadir el directorio de Terraform al PATH del sistema (variable \$PATH):

        [root@wollfex terraform]# export PATH="$PATH:/opt/terraform"

    Para hacer persistente el cambio y que el binario de terraform sea reconocido después de esta sesión la terminar debemos agregarlo en ~/.profile o ~/.bashrc:

        [root@wollfex ~]# echo PATH="$PATH:/opt/terraform" >>  ~/.bashrc
        [root@wollfex ~]# source .bashrc

6.  Por último comprobamos que se ha instalado bien ejecutando el comando siguiente:

        [root@wollfex terraform]# terraform --version
        Terraform v0.11.13

## Ejemplos de uso de terraform

1. Configuracion de una maquina

```terraform
    data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-bionic-18.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  owners = ["099720109477"] # Canonical
}

resource "aws_instance" "web" {
  ami           = "${data.aws_ami.ubuntu.id}"
  instance_type = "t3.micro"


  vpc_security_group_ids = [
    "${aws_security_group.allow_ssh_anywhere.id}",
    "${aws_security_group.allow_http_anywhere.id}"
  ]
  user_data = "${file("run.txt")}"
  key_name  = "${aws_key_pair.keypair.key_name}"

  tags = {
    Name = "instance"
  }
}
```
### Ejemplo de implantacion de las keys para el terraform

```terraform
provider "aws" {
  access_key = "XXXXXXXXXXXXXXX"
  secret_key = "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
  region     = "us-east-2"
}
```

### Valores de salida cuando creamos la maquina 

```terraform
output "instance_public_ip" {
  value = "${aws_eip.web_eip.public_ip}"
}

output "security_gruop_id" {
  value = "${aws_security_group.allow_ssh_anywhere.id}"
}

output "security_gruop_name" {
  value = "${aws_security_group.allow_ssh_anywhere.name}"
}

output "security_gruop_desc" {
  value = "${aws_security_group.allow_ssh_anywhere.description}"
}
```

### Tambine podemos lanzar script de inicio aunque no los utilizaremos estos se guardan como .txt

```txt

#!/bin/bash
#Actualizacion de Repositorios

 apt-get update

#Instalamos apache 2

apt-get install apache2 -y

#Instalamos los paquetes para el apache

apt-get install php libapache2-mod-php php-mysql -y

#instalamos adminer

sudo mkdir /var/www/html/adminer

cd /var/www/html/adminer

wget https://github.com/vrana/adminer/releases/download/v4.7.3/adminer-4.7.3-mysql.php

mv adminer-4.7.3-mysql.php index.php

#Instalamos las debconf-utils

apt-get install debconf-utils -y

# Instalamos mySQL Server

DB_ROOT_PASSWD=root

debconf-set-selections <<<"mysql-server mysql-server/root_password password $DB_ROOT_PASSWD"
debconf-set-selections <<<"mysql-server mysql-server/root_password_again password $DB_ROOT_PASSWD"

apt-get install mysql-server -y

cd /var/www/html

git clone https://github.com/josejuansanchez/iaw-practica-lamp

chown www-data:www-data * -R

#Configuramos la aplicacion web

DB_NAME=lamp_db
DB_USER=lamp_user
DB_PASSWD=lamp_user

mysql -u root -p$DB_ROOT_PASSWD <<< "DROP DATABASE IF EXISTS $DB_NAME;"
mysql -u root -p$DB_ROOT_PASSWD <<< "CREATE DATABASE $DB_NAME;"
mysql -u root -p$DB_ROOT_PASSWD <<< "GRANT ALL PRIVILEGES ON $DB_NAME.* TO $DB_USER@'%' IDENTIFIED BY '$DB_PASSWD';"
mysql -u root -p$DB_ROOT_PASSWD <<< "FLUSH PRIVILEGES;"

#importamos la base de datos

mysql -u root -p$DB_ROOT_PASSWD <  /var/www/html/iaw-practica-lamp/db/database.sql


```

en mi caso he utilizado uno de calase y funciona correctamente
