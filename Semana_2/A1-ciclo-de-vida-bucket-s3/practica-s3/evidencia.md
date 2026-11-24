# Actividad: ciclo de vida de un bucket en S3

Crear un bucket, subir, consultar, modificar y compartir objetos desde la AWS CLI, y protegerlo con una política de menor privilegio.

---

## Captura o salida de los pasos 3, 7, 13, 16, 21, 22, 23, 24 y 29

### Parte 1 (Crear) - Paso 3: Consultar en qué región quedó el bucket

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\03-AWS-bucket-s3\practica-s3> aws s3api get-bucket-location --bucket practica-juan-1125
{
    "LocationConstraint": "us-east-2"
}
```

---

### Parte 2 (Subir) - Paso 7: Listar el contenido del bucket con tamaños legibles y el total

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\03-AWS-bucket-s3> aws s3 ls s3://practica-juan-1125 --recursive --human-readable --summarize
2026-11-23 19:40:35   18 Bytes notas.txt
2026-11-23 19:45:33   20 Bytes secreto.txt
2026-11-23 19:45:33   19 Bytes tareas.txt

Total Objects: 3
   Total Size: 57 Bytes
```

---

### Parte 4 (Editar) - Paso 13: Mostrar el contenido de `notas.txt` desde el bucket y verificar el cambio

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\03-AWS-bucket-s3\practica-s3> aws s3 cp s3://practica-juan-1125/notas.txt -  
contenido de notas versión 2 para paso 12
```

---

### Parte 5 (Compartir sin hacerlo público) - Paso 16: Comprobación del enlace temporal (habilitado y expirado)

Generación previa del enlace prefirmado de 60 segundos (Paso 14):

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\03-AWS-bucket-s3\practica-s3> aws s3 presign s3://practica-juan-1125/notas.txt --expires-in 60
https://practica-juan-1125.s3.us-east-2.amazonaws.com/notas.txt?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAULLIDC6XD67QPCEZ%2F20261124%2Fus-east-2%2Fs3%2Faws4_request&X-Amz-Date=20261124T012106Z&X-Amz-Expires=60&X-Amz-SignedHeaders=host&X-Amz-Signature=5f8c8490f7f80a2c182d2e141bccbc775ff055cfd442000afb094b3965bf44d9
```

**Paso 15 - Enlace funcionando antes de que expire el minuto:**

![Paso 15 - Enlace habilitado](./P15-enlace-habilitado.png)

**Paso 16 - Enlace expirado tras cumplirse el tiempo límite:**

![Paso 16 - Enlace expirado](./P16-enlace-expirado.png)

---

### Parte 6 (Menor privilegio) - Paso 21: Con el perfil `lector-s3`, mostrar el contenido de `notas.txt` (permitido)

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\03-AWS-bucket-s3\practica-s3> aws s3 cp s3://practica-juan-1125/notas.txt - --profile lector-s3
contenido de notas versión 2 para paso 12
```

---

### Parte 6 (Menor privilegio) - Paso 22: Con el perfil `lector-s3`, intentar listar el contenido del bucket (denegado)

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\03-AWS-bucket-s3\practica-s3> aws s3 ls s3://practica-juan-1125/ --profile lector-s3

aws: [ERROR]: An error occurred (AccessDenied) when calling the ListObjectsV2 operation: User: arn:aws:iam::299255404462:user/lector-s3-juan is not authorized to perform: s3:ListBucket on resource: "arn:aws:s3:::practica-juan-1125" because no identity-based policy allows the s3:ListBucket action
```

---

### Parte 6 (Menor privilegio) - Paso 23: Con el perfil `lector-s3`, intentar borrar `secreto.txt` (denegado)

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\03-AWS-bucket-s3\practica-s3> aws s3 rm s3://practica-juan-1125/secreto.txt --profile lector-s3
delete failed: s3://practica-juan-1125/secreto.txt An error occurred (AccessDenied) when calling the DeleteObject operation: User: arn:aws:iam::299255404462:user/lector-s3-juan is not authorized to perform: s3:DeleteObject on resource: "arn:aws:s3:::practica-juan-1125/secreto.txt" because no identity-based policy allows the s3:DeleteObject action
```

---

### Parte 7 (Eliminar) - Paso 24: Intento de borrar el bucket sin vaciarlo

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\03-AWS-bucket-s3\practica-s3> aws s3 rb s3://practica-juan-1125 
remove_bucket failed: s3://practica-juan-1125 An error occurred (BucketNotEmpty) when calling the DeleteBucket operation: The bucket you tried to delete is not empty
```

---

### Parte 7 (Eliminar) - Paso 29: Confirmar que no quedaron buckets ni usuarios de la actividad

```powershell
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\03-AWS-bucket-s3\practica-s3> aws s3 ls
PS D:\Juan-Canas-D\Escritorio\AWS-Cloud\RIWI-AWS\03-AWS-bucket-s3\practica-s3> aws iam list-users --query "Users[].UserName" --output table
------------------
|    ListUsers   |
+----------------+
|  admin-juan-1  |
+----------------+
```

---

## Preguntas de reflexión

### 1. ¿Por qué el nombre de tu bucket tuvo que ser distinto al de tus compañeros si cada uno tiene su propia cuenta?

Porque en Amazon S3 los nombres de los buckets son únicos a nivel global en todo AWS, no solo dentro de mi propia cuenta.

Esto se debe a que cada bucket se convierte en una dirección web (URL) para acceder a él desde internet (por ejemplo: `practica-juan-1125.s3.us-east-2.amazonaws.com`). Como no pueden existir dos direcciones idénticas en internet, si otra persona en cualquier parte del mundo ya registró ese nombre, AWS no me permite usarlo y me toca elegir uno diferente hasta que el original sea eliminado.

### 2. El usuario del paso 21 pudo leer un archivo, pero en el paso 22 no pudo ver la lista. ¿Qué acción le faltaba en la política?

Le faltaba la acción **`s3:ListBucket`**.

En la política que creamos solo le dimos el permiso `s3:GetObject` sobre los objetos (`practica-juan-1125/*`). En S3 una cosa es el permiso sobre los archivos individuales y otra diferente es el permiso sobre el bucket como contenedor:

- Con `s3:GetObject` el usuario pudo leer `notas.txt` porque ya conocía el nombre exacto del archivo.
- Pero para poder ver la lista de qué archivos hay dentro del bucket se necesita permiso sobre el contenedor con `s3:ListBucket`. Como no lo tenía asignado, AWS bloqueó el comando con `AccessDenied`.

### 3. En la política, ¿qué significa el `/*` al final del `Resource`? ¿Qué pasaría si se quitara?

El `/*` al final (`arn:aws:s3:::practica-juan-1125/*`) funciona como un comodín que representa a **todos los objetos y archivos que están guardados dentro del bucket**.

Si se quitara y dejáramos solo `arn:aws:s3:::practica-juan-1125`, la política estaría apuntando únicamente al bucket (al contenedor) y no a su contenido. Como la acción permitida era `s3:GetObject` (que solo aplica para archivos, no para el bucket), el usuario no tendría permiso sobre ningún archivo interno y el comando del paso 21 habría fallado con error de acceso denegado.

### 4. ¿En qué se diferencia compartir un archivo con un enlace temporal de hacer público el bucket?

La diferencia principal está en la seguridad y el tiempo de acceso:

- Con el **enlace temporal (URL prefirmada)**, el bucket sigue siendo totalmente privado. El enlace se crea firmado con mis credenciales de AWS y tiene un tiempo límite de vida (en este caso 60 segundos). La persona que lo reciba solo puede ver ese archivo puntual y, en cuanto pasa el minuto, el enlace vence solo y ya nadie más puede entrar.
- Por el contrario, **hacer público el bucket** significa quitarle la seguridad para que cualquier persona en internet pueda ver o descargar los archivos cuando quiera y por tiempo indefinido, lo que representa un riesgo grande de seguridad y posibles sobrecostos en la cuenta.

### 5. Según el modelo de responsabilidad compartida, si alguien deja un bucket público con datos de clientes, ¿de quién es la responsabilidad?

La responsabilidad es **100% del cliente (quien administra la cuenta)**.

Según el modelo de responsabilidad compartida:
- **AWS se encarga de la seguridad "de la nube":** que la infraestructura física, los servidores, los centros de datos y el software base de S3 funcionen de forma segura. De hecho, AWS activa por defecto el bloqueo de acceso público (*Block Public Access*).
- **El cliente es responsable de la seguridad "en la nube":** esto incluye cómo configuramos los permisos, las políticas de IAM y qué archivos decidimos dejar privados o públicos.

Si un bucket queda expuesto con información confidencial, no es una falla de AWS, sino una mala configuración o descuido por parte de quien administró el bucket.
