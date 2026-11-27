# Actividad: tu primer servidor en EC2

Lanzamiento de una instancia EC2 en `us-east-2` con Apache como servidor web, reglas de security group para HTTP y SSH, y un rol de IAM (`ec2-lector-s3`) para leer un archivo de S3 sin guardar credenciales en el servidor.

---

## Contenido de `inicio.sh`

```bash
#!/bin/bash
dnf install -y httpd
systemctl enable --now httpd
echo "<h1>Hola desde mi instancia EC2, soy Juan Canas.</h1>" > /var/www/html/index.html
```

---

## Salidas de los pasos solicitados

### Paso 12: Reglas del security group (`web-juan`)

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2> aws ec2 describe-security-groups --group-ids sg-07bae58f0aa56c668 --query "SecurityGroups[].IpPermissions[].[FromPort,IpRanges[0].CidrIp]" --output table
-----------------------------
|   DescribeSecurityGroups  |
+----+----------------------+
|  80|  0.0.0.0/0           |
|  22|  191.156.252.27/32   |
+----+----------------------+
```

### Paso 16: IP pública inicial de la instancia (`i-01f1f6a84f2045f8e`)

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws ec2 describe-instances --instance-ids i-01f1f6a84f2045f8e --query "Reservations[0].Instances[0].PublicIpAddress" --output text
3.145.203.253
```

### Paso 17: Prueba del servidor web (navegador y `curl`)

Se accedió en el navegador a `http://3.145.203.253` y se validó la respuesta HTTP desde PowerShell:

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> curl.exe -i --connect-timeout 10 http://3.145.203.253
HTTP/1.1 200 OK
Server: Apache/2.4.71 (Amazon Linux)

<h1>Hola desde mi instancia EC2, soy Juan Canas.</h1>
```

### Paso 20: Identidad con la que trabaja la instancia

```bash
[ec2-user@ip-172-31-26-229 ~]$ aws sts get-caller-identity
{
    "UserId": "AROAULLIDC6XHC7RP7X4X:i-01f1f6a84f2045f8e",
    "Account": "299255404462",
    "Arn": "arn:aws:sts::299255404462:assumed-role/ec2-lector-s3/i-01f1f6a84f2045f8e"
}
```

### Paso 21: Lectura del archivo en S3 desde la instancia (permitido)

```bash
[ec2-user@ip-172-31-26-229 ~]$ aws s3 cp s3://reportes-juan-1125/reportes.txt -
Este es un reporte de prueba
```

### Paso 22: Intento de listar el bucket desde la instancia (denegado)

```bash
[ec2-user@ip-172-31-26-229 ~]$ aws s3 ls s3://reportes-juan-1125/

aws: [ERROR]: An error occurred (AccessDenied) when calling the ListObjectsV2 operation: User: arn:aws:sts::299255404462:assumed-role/ec2-lector-s3/i-01f1f6a84f2045f8e is not authorized to perform: s3:ListBucket on resource: "arn:aws:s3:::reportes-juan-1125" because no identity-based policy allows the s3:ListBucket action
```

### Paso 23: Intento de listar usuarios de IAM desde la instancia (denegado)

```bash
[ec2-user@ip-172-31-26-229 ~]$ aws iam list-users

aws: [ERROR]: An error occurred (AccessDenied) when calling the ListUsers operation: User: arn:aws:sts::299255404462:assumed-role/ec2-lector-s3/i-01f1f6a84f2045f8e is not authorized to perform: iam:ListUsers on resource: arn:aws:iam::299255404462:user/ because no identity-based policy allows the iam:ListUsers action. Go to https://us-east-1.console.aws.amazon.com/iam/home?region=us-east-1#/authorization-details/6fdrmc0xw0zfq0x06h9z8kw4o for complete details, or call the GetRequestAuthorizationDetails API with the following authorization id: 6fdrmc0xw0zfq0x06h9z8kw4o
```

### Paso 27: Cambio de IP pública al apagar y volver a encender

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws ec2 stop-instances --instance-ids i-01f1f6a84f2045f8e
{
    "StoppingInstances": [
        {
            "InstanceId": "i-01f1f6a84f2045f8e",
            "CurrentState": {
                "Code": 64,
                "Name": "stopping"
            },
            "PreviousState": {
                "Code": 16,
                "Name": "running"
            }
        }
    ]
}

PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws ec2 describe-instances --instance-ids i-01f1f6a84f2045f8e --query "Reservations[0].Instances[0].State.Name" --output text
stopped

PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws ec2 start-instances --instance-ids i-01f1f6a84f2045f8e
{
    "StartingInstances": [
        {
            "InstanceId": "i-01f1f6a84f2045f8e",
            "CurrentState": {
                "Code": 0,
                "Name": "pending"
            },
            "PreviousState": {
                "Code": 80,
                "Name": "stopped"
            }
        }
    ]
}

PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws ec2 describe-instances --instance-ids i-01f1f6a84f2045f8e --query "Reservations[0].Instances[0].State.Name" --output text
running

PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws ec2 describe-instances --instance-ids i-01f1f6a84f2045f8e --query "Reservations[0].Instances[0].PublicIpAddress" --output text
18.188.171.58
```

Al comparar con el paso 16, la IP pública cambió de `3.145.203.253` a `18.188.171.58`.

### Paso 31: Confirmación de que no quedaron instancias activas ni detenidas

Luego de terminar la instancia (`terminated`) y eliminar el security group, el par de llaves, el perfil de instancia, el rol de IAM y el bucket de S3:

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws ec2 describe-instances --filters "Name=instance-state-name,Values=running,stopped" --query "Reservations[].Instances[].[InstanceId,State.Name]" --output table
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2>
```

La consulta no devolvió resultados, confirmando que no quedó ninguna instancia en estado `running` ni `stopped`. También se verificó la eliminación del security group (`sg-07bae58f0aa56c668`), el par de llaves (`clave-juan`), el perfil y rol `ec2-lector-s3`, y el bucket `reportes-juan-1125`.

---

## Problemas encontrados durante la práctica en Windows

### 1. BOM en el archivo `inicio.sh`

Cuando creé `inicio.sh` desde PowerShell con `-Encoding utf8`, no sabía que Windows PowerShell le agrega una marca oculta al inicio del archivo (el BOM - Byte Order Mark). Por eso, cuando la instancia arrancó, no entendió bien la primera línea (`#!/bin/bash`) y no ejecutó el script para instalar la página web. Al guardarlo con `Set-Content -Encoding ascii .\inicio.sh` ya quedó limpio y funcionó sin problema.

### 2. Permisos de la llave `.pem` en Windows

Al principio intenté proteger la llave con `chmod 400` desde Git Bash (porque PowerShell no reconoce ese comando), pero en Windows no fue suficiente y al intentar entrar por SSH me seguía rechazando la llave por tener permisos muy abiertos. Me tocó arreglarlo desde PowerShell con `icacls` para quitarle los permisos heredados (`icacls .\clave-juan.pem /inheritance:r`) y dejar que solo mi usuario Windows (Juan Cañas) pudiera leerla (`icacls .\clave-juan.pem /grant:r "$($env:USERNAME):(R)"`), sin que quedara expuesta a otros usuarios del computador.

---

## Preguntas de reflexión

### 1. En el paso 20, la instancia responde con un rol y no con un usuario. ¿De dónde sacó esas credenciales si nunca corriste `aws configure`?

Las sacó del perfil de instancia `ec2-lector-s3` que le puse al crearla. Como la instancia tiene ese rol conectado, AWS se encarga por debajo de darle credenciales temporales mientras está encendida, así que la AWS CLI las toma automáticamente sin tener que usar `aws configure` ni guardar claves fijas dentro del servidor.

### 2. ¿Por qué el puerto 80 puede estar abierto a todo internet y el 22 no?

Porque el puerto 80 es por donde entra la gente a ver la página web, así que tiene que estar abierto para todo el mundo (`0.0.0.0/0`). En cambio, el puerto 22 es el de SSH, que da acceso directo a la terminal de la máquina; si lo dejo abierto a todo internet cualquiera podría intentar meterse a la fuerza, por eso solo se abre para mi IP.

### 3. La instancia pudo leer `reporte.txt` pero no listar el bucket. ¿Dónde se decidió eso?

Eso quedó definido en la política de permisos (`leer-reportes`) que le dimos al rol `ec2-lector-s3` en la práctica anterior. Ahí solo le dimos permiso para leer archivos puntuales (`s3:GetObject`) si uno ya sabe el nombre, pero no para ver la lista completa del bucket (`s3:ListBucket`) ni para consultar usuarios de IAM. Por eso dejó leer el archivo y bloqueó lo demás.

### 4. ¿Por qué cambió la IP pública al apagar y encender? ¿Qué problema traería eso en un sitio web real?

Porque la IP pública que da EC2 por defecto es temporal y solo dura mientras la instancia está prendida. Cuando se apaga (`stopped`), AWS la suelta y al volverla a encender le asigna otra diferente (pasó de `3.145.203.253` a `18.188.171.58`). En una página real sería un problema porque si la IP cambia, el dominio dejaría de apuntar al servidor y la página se caería hasta volver a configurar la dirección; para eso se usaría una IP fija (Elastic IP).

### 5. EC2 es infraestructura como servicio (IaaS). En esta práctica, ¿qué fue responsabilidad tuya y qué fue responsabilidad de AWS?

AWS se encargó de toda la parte física y base: los servidores reales, el internet del centro de datos, la luz y que la máquina virtual funcionara. A mí me tocó la parte de configuración y uso: elegir la imagen del sistema, abrir los puertos del security group, cuidar la llave `.pem`, instalar el servidor web con `inicio.sh`, asignarle el rol y apagar y borrar todo al final.

### 6. Para tener este mismo servidor en una oficina, sin la nube, ¿qué habrías tenido que comprar y configurar?

Habría tocado comprar un computador o servidor físico, una batería (UPS) por si se va la luz y un lugar donde tenerlo encendido todo el día. Además habría tenido que pagar un internet empresarial con IP fija, configurar el router para abrir los puertos 80 y 22, instalar Linux y el servidor web desde cero y encargarme si algún componente físico se daña.
