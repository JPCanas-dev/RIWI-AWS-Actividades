# Registro completo de comandos y salidas (`evidencia-raw`)

## Parte 1: Reconocer el terreno

### Paso 1: Confirma quién eres y en qué región trabajas

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2> aws sts get-caller-identity
{
    "UserId": "AIDAULLIDC6XJ7XCS3T2F",
    "Account": "299255404462",
    "Arn": "arn:aws:iam::299255404462:user/admin-juan-1"
}

PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2> aws configure get region
us-east-2
```

### Paso 2: Confirma que el perfil de instancia `ec2-lector-s3` existe y contiene el rol

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2> aws iam get-instance-profile --instance-profile-name ec2-lector-s3
{
    "InstanceProfile": {
        "Path": "/",
        "InstanceProfileName": "ec2-lector-s3",
        "InstanceProfileId": "AIPAULLIDC6XJ6C3HMF32",
        "Arn": "arn:aws:iam::299255404462:instance-profile/ec2-lector-s3",
        "Roles": [
            {
                "Path": "/",
                "RoleName": "ec2-lector-s3",
                "RoleId": "AROAULLIDC6XHC7RP7X4X",
                "Arn": "arn:aws:iam::299255404462:role/ec2-lector-s3"
            }
        ]
    }
}
```

### Paso 3: Busca el ID de la AMI de Amazon Linux 2023 en tu región y anótalo

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2> aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64 --query "Parameters[0].Value" --output text
ami-01e4a7224fcbfe81f
```

### Paso 4: Busca el ID de tu VPC por defecto y anótalo

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2> aws ec2 describe-vpcs --filters Name=isDefault,Values=true --query "Vpcs[].VpcId" --output text
vpc-02d4ab8b4d59be894
```

---

## Parte 2: Par de llaves

### Paso 5: Crea un par de llaves llamado `clave-<tu-nombre>` y guarda la llave privada en un archivo `.pem`

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2> aws ec2 create-key-pair --key-name clave-juan --query "KeyMaterial" --output text | Set-Content -Encoding ascii .\clave-juan.pem
```

### Paso 6: Protege el archivo con permisos de solo lectura para tu usuario

Como el sistema operativo es Windows, se utilizaron permisos NTFS con `icacls` en lugar de `chmod`.

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2> icacls .\clave-juan.pem /inheritance:r
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2> icacls .\clave-juan.pem /grant:r "$($env:USERNAME):(R)"
```

### Paso 7: Lista el archivo y confirma que aparece

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2> Get-ChildItem .\clave-juan.pem | Select-Object Name

Name
----
clave-juan.pem
```

---

## Parte 3: Security group

### Paso 8: Crea un security group llamado `web-<tu-nombre>` y anota su `sg-...`

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2> aws ec2 create-security-group --group-name web-juan --description "HTTP abierto y SSH solo desde mi IP" --query "GroupId" --output text
sg-07bae58f0aa56c668
```

### Paso 9: Consulta tu IP pública

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2> curl.exe -s https://checkip.amazonaws.com
191.156.252.27
```

### Paso 10: Permite el puerto 80 desde cualquier lugar

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2> aws ec2 authorize-security-group-ingress --group-id sg-07bae58f0aa56c668 --protocol tcp --port 80 --cidr 0.0.0.0/0
{
    "Return": true
}
```

### Paso 11: Permite el puerto 22 solo desde tu IP

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2> aws ec2 authorize-security-group-ingress --group-id sg-07bae58f0aa56c668 --protocol tcp --port 22 --cidr 191.156.252.27/32
{
    "Return": true
}
```

### Paso 12: Muestra las reglas del security group

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2> aws ec2 describe-security-groups --group-ids sg-07bae58f0aa56c668 --query "SecurityGroups[].IpPermissions[].[FromPort,IpRanges[0].CidrIp]" --output table
-----------------------------
|   DescribeSecurityGroups  |
+----+----------------------+
|  80|  0.0.0.0/0           |
|  22|  191.156.252.27/32   |
+----+----------------------+
```

---

## Parte 4: Lanzar la instancia

### Paso 13: Crea el archivo `inicio.sh` con un script que instale el servidor web y publique una página con tu nombre

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> @'
#!/bin/bash
dnf install -y httpd
systemctl enable --now httpd
echo "<h1>Hola desde mi instancia EC2, soy Juan Canas.</h1>" > /var/www/html/index.html
'@ | Set-Content -Encoding ascii .\inicio.sh
```

> **Nota de Windows:** En Windows PowerShell 5.1, `Set-Content -Encoding utf8` guarda el archivo como UTF-8 con BOM. Ese marcador al comienzo del archivo impedía que `cloud-init` reconociera correctamente la línea `#!/bin/bash` y ejecutara el User Data. Para este script, que contiene caracteres ASCII, se debe usar `-Encoding ascii` para evitar el BOM.

### Paso 14: Lanza una instancia `t3.micro` con tu AMI, tu par de llaves, tu security group, el perfil `ec2-lector-s3`, el script `inicio.sh` y el nombre `web-<tu-nombre>`. Anota su `i-...`

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws ec2 run-instances `
    --image-id ami-01e4a7224fcbfe81f `
    --count 1 `
    --instance-type t3.micro `
    --key-name clave-juan `
    --security-group-ids sg-07bae58f0aa56c668 `
    --subnet-id subnet-0ded3fccebc7eb353 `
    --iam-instance-profile Name=ec2-lector-s3 `
    --user-data file://inicio.sh `
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=web-juan}]' `
    --query "Instances[0].InstanceId" `
    --output text
i-01f1f6a84f2045f8e
```

### Paso 15: Espera a que la instancia esté encendida

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws ec2 wait instance-running --instance-ids i-01f1f6a84f2045f8e
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws ec2 describe-instances --instance-ids i-01f1f6a84f2045f8e --query "Reservations[0].Instances[0].State.Name" --output text
running

PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws ec2 wait instance-status-ok --instance-ids i-01f1f6a84f2045f8e
```

### Paso 16: Consulta su IP pública y anótala

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws ec2 describe-instances --instance-ids i-01f1f6a84f2045f8e --query "Reservations[0].Instances[0].PublicIpAddress" --output text
3.145.203.253
```

---

## Parte 5: Probar el servidor web

### Paso 17: Espera dos minutos y abre `http://<ip>` en el navegador

Abrir en el navegador:

`http://3.145.203.253`

Comprobación desde PowerShell:

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> curl.exe -i --connect-timeout 10 http://3.145.203.253
HTTP/1.1 200 OK
Server: Apache/2.4.71 (Amazon Linux)

<h1>Hola desde mi instancia EC2, soy Juan Canas.</h1>
```

---

## Parte 6: Entrar y probar el rol

### Paso 19: Conéctate a la instancia por SSH

```powershell
D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> ssh -i clave-juan.pem ec2-user@3.145.203.253
```

### Paso 20: Dentro de la instancia, pregunta con qué identidad trabaja

```bash
[ec2-user@ip-172-31-26-229 ~]$ aws sts get-caller-identity
{
    "UserId": "AROAULLIDC6XHC7RP7X4X:i-01f1f6a84f2045f8e",
    "Account": "299255404462",
    "Arn": "arn:aws:sts::299255404462:assumed-role/ec2-lector-s3/i-01f1f6a84f2045f8e"
}
```

### Paso 21: Dentro de la instancia, muestra el contenido de `reporte.txt` desde tu bucket

```bash
[ec2-user@ip-172-31-26-229 ~]$ aws s3 cp s3://reportes-juan-1125/reportes.txt -
Este es un reporte de prueba
```

### Paso 22: Dentro de la instancia, intenta listar el contenido del bucket

```bash
[ec2-user@ip-172-31-26-229 ~]$ aws s3 ls s3://reportes-juan-1125/

aws: [ERROR]: An error occurred (AccessDenied) when calling the ListObjectsV2 operation: User: arn:aws:sts::299255404462:assumed-role/ec2-lector-s3/i-01f1f6a84f2045f8e is not authorized to perform: s3:ListBucket on resource: "arn:aws:s3:::reportes-juan-1125" because no identity-based policy allows the s3:ListBucket action
```

### Paso 23: Dentro de la instancia, intenta listar los usuarios de IAM

```bash
[ec2-user@ip-172-31-26-229 ~]$ aws iam list-users

aws: [ERROR]: An error occurred (AccessDenied) when calling the ListUsers operation: User: arn:aws:sts::299255404462:assumed-role/ec2-lector-s3/i-01f1f6a84f2045f8e is not authorized to perform: iam:ListUsers on resource: arn:aws:iam::299255404462:user/ because no identity-based policy allows the iam:ListUsers action. Go to https://us-east-1.console.aws.amazon.com/iam/home?region=us-east-1#/authorization-details/6fdrmc0xw0zfq0x06h9z8kw4o for complete details, or call the GetRequestAuthorizationDetails API with the following authorization id: 6fdrmc0xw0zfq0x06h9z8kw4o
```

### Paso 24: Sal de la instancia con `exit`

```bash
[ec2-user@ip-172-31-26-229 ~]$ exit
logout
Connection to 3.145.203.253 closed.
```

---

## Parte 7: Estados de la instancia

### Paso 25: Apaga la instancia y consulta su estado hasta que sea `stopped`

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
```

### Paso 26: Enciéndela de nuevo, espera a que esté `running` y consulta su IP pública

```powershell
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

---

## Parte 8: Eliminar

### Reto: intenta borrar el security group antes de que la instancia quede `terminated` y lee el error

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws ec2 delete-security-group --group-id sg-07bae58f0aa56c668
aws: [ERROR]: An error occurred (DependencyViolation) when calling the DeleteSecurityGroup operation: resource sg-07bae58f0aa56c668 has a dependent object
```

### Paso 28: Termina la instancia y espera a que quede `terminated`

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws ec2 terminate-instances --instance-ids i-01f1f6a84f2045f8e
{
    "TerminatingInstances": [
        {
            "InstanceId": "i-01f1f6a84f2045f8e",
            "CurrentState": {
                "Code": 32,
                "Name": "shutting-down"
            },
            "PreviousState": {
                "Code": 16,
                "Name": "running"
            }
        }
    ]
}

PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws ec2 describe-instances --instance-ids i-01f1f6a84f2045f8e --query "Reservations[0].Instances[0].State.Name" --output text
terminated
```

### Paso 29: Borra el security group

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws ec2 delete-security-group --group-id sg-07bae58f0aa56c668
{
    "Return": true,
    "GroupId": "sg-07bae58f0aa56c668"
}

PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws ec2 describe-security-groups --group-ids sg-07bae58f0aa56c668

aws: [ERROR]: An error occurred (InvalidGroup.NotFound) when calling the DescribeSecurityGroups operation: The security group 'sg-07bae58f0aa56c668' does not exist
```

### Paso 30: Borra el par de llaves en AWS y el archivo `.pem` de tu computador

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws ec2 describe-security-groups --group-ids sg-07bae58f0aa56c668

aws: [ERROR]: An error occurred (InvalidGroup.NotFound) when calling the DescribeSecurityGroups operation: The security group 'sg-07bae58f0aa56c668' does not exist
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2> aws ec2 delete-key-pair --key-name clave-juan
{
    "Return": true,
    "KeyPairId": "key-0f612bee12aac229d"
}

PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws ec2 describe-instances --filters "Name=instance-state-name,Values=running,stopped" --query "Reservations[].Instances[].[InstanceId,State.Name]" --output table
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2>
```

### Paso 31: Lista las instancias y confirma que ninguna está en `running` ni `stopped`

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws ec2 describe-instances --filters "Name=instance-state-name,Values=running,stopped" --query "Reservations[].Instances[].[InstanceId,State.Name]" --output table
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2>
```

### Limpieza de la actividad de roles: perfil de instancia, política, rol y bucket

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws iam remove-role-from-instance-profile --instance-profile-name ec2-lector-s3 --role-name ec2-lector-s3
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws iam delete-instance-profile --instance-profile-name ec2-lector-s3
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws iam delete-role-policy --role-name ec2-lector-s3 --policy-name leer-reportes
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws iam delete-role --role-name ec2-lector-s3
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws s3 ls s3://reportes-juan-1125/
2026-11-25 11:44:25         28 reportes.txt
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws s3 rm s3://reportes-juan-1125/ --recursive
delete: s3://reportes-juan-1125/reportes.txt
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\05-EC2\practica-ec2> aws s3 rb s3://reportes-juan-1125
remove_bucket: reportes-juan-1125
```
