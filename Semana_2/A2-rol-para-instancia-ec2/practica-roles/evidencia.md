# Actividad: un rol para tu instancia EC2

Crear un rol de IAM aplicando el principio de menor privilegio y dejarlo listo dentro de un perfil de instancia (*instance profile*) para usarlo en la práctica de EC2.

---

## Contenido de las políticas (`confianza.json` y `permisos.json`)

### `confianza.json` (Política de confianza)

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "Service": "ec2.amazonaws.com" },
    "Action": "sts:AssumeRole"
  }]
}
```

### `permisos.json` (Política de permisos)

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject"],
    "Resource": "arn:aws:s3:::reportes-juan-1125/*"
  }]
}
```

---

## Salida de los pasos 7, 9, 13, 16, 19 y 20

### Parte 3 (Crear el rol) - Paso 7: Intentar crear el rol sin indicar la política de confianza

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\04-ROLES-IAM\practica-roles> aws iam create-role --role-name ec2-lector-s3

aws: [ERROR]: An error occurred (ParamValidation): the following arguments are required: --assume-role-policy-document
```

---

### Parte 3 (Crear el rol) - Paso 9: Mostrar los datos del rol y anotar su `Arn`

**ARN del rol:** `arn:aws:iam::299255404462:role/ec2-lector-s3`

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\04-ROLES-IAM\practica-roles> aws iam get-role --role-name ec2-lector-s3
{
    "Role": {
        "Path": "/",
        "RoleName": "ec2-lector-s3",
        "RoleId": "AROAULLIDC6XHC7RP7X4X",
        "Arn": "arn:aws:iam::299255404462:role/ec2-lector-s3",
        "CreateDate": "2026-11-25T19:12:44+00:00",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "ec2.amazonaws.com"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        },
        "MaxSessionDuration": 3600,
        "MaxSigninSessionDuration": 3600,
        "RoleLastUsed": {}
    }
}
```

---

### Parte 4 (Dar permisos) - Paso 13: Mostrar el contenido de la política `leer-reportes` desde AWS

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\04-ROLES-IAM\practica-roles> aws iam get-role-policy --role-name ec2-lector-s3 --policy-name leer-reportes
{
    "RoleName": "ec2-lector-s3",
    "PolicyName": "leer-reportes",
    "PolicyDocument": {
    
    
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "s3:GetObject"
                ],
                "Resource": "arn:aws:s3:::reportes-juan-1125/*"
            }
        ]
    }
}
```

---

### Parte 5 (Editar) - Paso 16: Mostrar los datos del rol y sus etiquetas para verificar los cambios

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\04-ROLES-IAM\practica-roles> aws iam get-role --role-name ec2-lector-s3
{
    "Role": {
        "Path": "/",
        "RoleName": "ec2-lector-s3",
        "RoleId": "AROAULLIDC6XHC7RP7X4X",
        "Arn": "arn:aws:iam::299255404462:role/ec2-lector-s3",
        "CreateDate": "2026-11-25T19:12:44+00:00",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "ec2.amazonaws.com"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        },
        "Description": "Rol para que EC2 lea reportes",
        "MaxSessionDuration": 3600,
        "MaxSigninSessionDuration": 3600,
        "Tags": [
            {
                "Key": "semana",
                "Value": "1"
            }
        ],
        "RoleLastUsed": {}
    }
}
```

---

### Parte 6 (Perfil de instancia) - Paso 19: Mostrar el perfil y confirmar que contiene el rol

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\04-ROLES-IAM\practica-roles> aws iam get-instance-profile --instance-profile-name ec2-lector-s3
{
    "InstanceProfile": {
        "Path": "/",
        "InstanceProfileName": "ec2-lector-s3",
        "InstanceProfileId": "AIPAULLIDC6XJ6C3HMF32",
        "Arn": "arn:aws:iam::299255404462:instance-profile/ec2-lector-s3",
        "CreateDate": "2026-11-25T20:12:06+00:00",
        "Roles": [
            {
                "Path": "/",
                "RoleName": "ec2-lector-s3",
                "RoleId": "AROAULLIDC6XHC7RP7X4X",
                "Arn": "arn:aws:iam::299255404462:role/ec2-lector-s3",
                "CreateDate": "2026-11-25T19:12:44+00:00",
                "AssumeRolePolicyDocument": {
                    "Version": "2012-10-17",
                    "Statement": [
                        {
                            "Effect": "Allow",
                            "Principal": {
                                "Service": "ec2.amazonaws.com"
                            },
                            "Action": "sts:AssumeRole"
                        }
                    ]
                }
            }
        ],
        "Tags": []
    }
}
```

---

### Parte 7 (Comprobar las reglas de IAM) - Paso 20: Intentar borrar el rol con `aws iam delete-role` y leer el error

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\04-ROLES-IAM\practica-roles> aws iam delete-role --role-name ec2-lector-s3

aws: [ERROR]: An error occurred (DeleteConflict) when calling the DeleteRole operation: Cannot delete entity, must remove roles from instance profile first.
```

---

## Preguntas de reflexión

### 1. ¿Qué diferencia hay entre la política de confianza y la política de permisos? ¿Qué pasaría si el rol tuviera solo una de las dos?

Aunque ambas van escritas en JSON, cumplen funciones muy distintas dentro del rol:

- **Política de confianza (`confianza.json`): establece quién puede asumir el rol mediante la acción `sts:AssumeRole`. En esta práctica, el servicio `ec2.amazonaws.com` es el único autorizado para asumirlo, es decir, solo una instancia EC2 puede usarlo.

- **Política de permisos (**`**permisos.json**`**):** establece qué acciones puede realizar quien asume el rol y sobre qué recursos. En este caso, permite únicamente la acción `s3:GetObject` sobre los objetos del bucket `reportes-juan-1125`.

Si el rol tuviera solo una de las dos:

- **Solo política de confianza (sin permisos):** el rol se crea y EC2 puede asumirlo, pero como en IAM todo viene denegado por defecto, no podría hacer absolutamente nada en S3 ni en ningún otro servicio.

- **Solo política de permisos (sin confianza):** no es posible crear el rol sin una política de confianza, ya que AWS exige el parámetro `--assume-role-policy-document`, como se comprobó en el paso 7. Esto se debe a que todo rol necesita definir quién está autorizado para asumirlo.

### 2. ¿Por qué es más seguro que la instancia use un rol en vez de guardar una llave de acceso con `aws configure`?

Porque si uso `aws configure` dentro de la instancia, quedan guardadas credenciales permanentes (`Access Key ID` y `Secret Access Key`) en un archivo dentro del servidor (`~/.aws/credentials`). Si alguien llega a entrar a la instancia, lee el disco o se filtra una copia, esas llaves se pueden usar desde cualquier otro computador hasta que alguien las revoque o las borre manualmente.

En cambio, cuando la instancia usa un rol (a través del perfil de instancia), no hay ninguna llave fija guardada en archivos. El servicio STS (*Security Token Service*) entrega credenciales temporales que la AWS CLI toma de forma automática y que se van renovando y venciendo solas cada cierto tiempo, reduciendo mucho el riesgo si algo se llega a comprometer.

### 3. Con la política `leer-reportes`, ¿la instancia podrá listar el contenido del bucket? ¿Y borrar `reporte.txt`? Explica por qué.

- **Listar el contenido del bucket:** **No.** Para listar qué archivos hay dentro de un bucket hace falta la acción `s3:ListBucket` apuntando al recurso del bucket (`arn:aws:s3:::reportes-juan-1125`). La política `leer-reportes` solo tiene `s3:GetObject` sobre `arn:aws:s3:::reportes-juan-1125/*`, así que la instancia solo puede leer/descargar un objeto si ya conoce su nombre exacto, pero si intenta listar el bucket va a recibir un `AccessDenied`.

- **Borrar `reporte.txt`:** **Tampoco.** Para eliminar objetos se necesita el permiso `s3:DeleteObject`, el cual no está en la política. Como IAM aplica denegación por defecto para cualquier acción que no esté permitida explícitamente, también fallará con `AccessDenied`.

### 4. En el error del paso 20, ¿qué le falta por hacer antes de poder borrar el rol?

El error `DeleteConflict` indica que no es posible eliminar el rol porque todavía está asociado al perfil de instancia. AWS muestra el mensaje `Cannot delete entity, must remove roles from instance profile first`.

Antes de poder ejecutar `aws iam delete-role` debo realizar dos pasos previos en el siguiente orden:
1. **Sacar el rol del perfil de instancia** `ec2-lector-s3` utilizando `aws iam remove-role-from-instance-profile`.
2. **Eliminar la política en línea** `leer-reportes`, utilizando `aws iam delete-role-policy`.

### 5. Si mañana la instancia también necesita subir archivos al bucket, ¿qué cambiarías y en cuál de los dos archivos?

Solo tendría que modificar **`permisos.json`**. El archivo `confianza.json` queda igual, porque quien asume el rol sigue siendo el servicio EC2 (`ec2.amazonaws.com`); lo único que cambia es lo que el rol puede hacer una vez asumido.

En `permisos.json` agregaría la acción **`s3:PutObject`** dentro de `"Action"`, manteniendo el mismo recurso `arn:aws:s3:::reportes-juan-1125/*` (porque subir archivos también aplica sobre objetos dentro del bucket):

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject", "s3:PutObject"],
    "Resource": "arn:aws:s3:::reportes-juan-1125/*"
  }]
}
```

Después de guardar el cambio, volvería a aplicar la política al rol con `aws iam put-role-policy` para que el permiso quede actualizado en AWS.
