# Actividad: ciclo de vida de un usuario iam

Crear, consultar, editar, probar y eliminar un usuario IAM desde la AWS CLI, aplicando el principio de menor privilegio.

---

### *Captura o salida de los pasos 3, 6, 8, 12, 13, 15 y 19.*

###### PASO 3 - PARTE 1 (CREAR USUARIO): Mostrar datos del usuario creado sin permisos/políticas asignadas

```powershell
PS C:\Users\Juan Cañas> aws iam get-user --user-name dev-juan
{
    "User": {
        "Path": "/",
        "UserName": "dev-juan",
        "UserId": "AIDAULLIDC6XKS27HOSOR",
        "Arn": "arn:aws:iam::299255404462:user/dev-juan",
        "CreateDate": "2026-11-20T22:21:59+00:00"
    }
}

PS C:\Users\Juan Cañas> aws iam list-attached-user-policies --user-name dev-juan
{
    "AttachedPolicies": []
}
```

###### PASO 6 - PARTE 2 (EDITAR USUARIO): Etiqueta y nuevo nombre de usuario

```powershell
PS C:\Users\Juan Cañas> aws iam get-user --user-name lector-juan
{
    "User": {
        "Path": "/",
        "UserName": "lector-juan",
        "UserId": "AIDAULLIDC6XKS27HOSOR",
        "Arn": "arn:aws:iam::299255404462:user/lector-juan",
        "CreateDate": "2026-11-20T22:21:59+00:00",
        "Tags": [
            {
                "Key": "area",
                "Value": "desarrollo"
            }
        ]
    }
}
```

###### PASO 8 - PARTE 3 (DAR PERMISOS AL USUARIO): Listar las políticas del usuario y confirmar que aparece

```powershell
PS C:\Users\Juan Cañas> aws iam list-attached-user-policies --user-name lector-juan
{
    "AttachedPolicies": [
        {
            "PolicyName": "AmazonEC2ReadOnlyAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess"
        }
    ]
}
```

###### PASO 12 - PARTE 4 (PROBAR MENOR PRIVILEGIO): Ejecutar una acción/política permitida

```powershell
PS C:\Users\Juan Cañas> aws ec2 describe-instances --profile lector
{
    "Reservations": []
}
```

###### PASO 13 - PARTE 4 (PROBAR MENOR PRIVILEGIO): Ejecutar una acción/política no permitida

```powershell
PS C:\Users\Juan Cañas> aws iam list-users --profile lector

aws: [ERROR]: An error occurred (AccessDenied) when calling the ListUsers operation: User: arn:aws:iam::299255404462:user/lector-juan is not authorized to perform: iam:ListUsers on resource: arn:aws:iam::299255404462:user/ because no identity-based policy allows the iam:ListUsers action. Go to https://us-east-1.console.aws.amazon.com/iam/home?region=us-east-1#/authorization-details/4l9johvc8tak6n7rxcal16wgu for complete details, or call the GetRequestAuthorizationDetails API with the following authorization id: 4l9johvc8tak6n7rxcal16wgu
```

###### PASO 15 - PARTE 5(QUITAR PERMISOS AL USUARIO): Repetir el comando del paso 12 con el perfil `lector`

```powershell
PS C:\Users\Juan Cañas> aws ec2 describe-instances --profile lector

aws: [ERROR]: An error occurred (UnauthorizedOperation) when calling the DescribeInstances operation: You are not authorized to perform this operation. User: arn:aws:iam::299255404462:user/lector-juan is not authorized to perform: ec2:DescribeInstances because no identity-based policy allows the ec2:DescribeInstances action
```

###### PASO 19 - PARTE 6 (ELIMINAR USUARIO): Listar los usuarios y confirma que ya no aparece.

**Reto previo: eliminar usuario sin haber borrado sus llaves de acceso:**

```powershell
PS C:\Users\Juan Cañas> aws iam delete-user --user-name lector-juan

aws: [ERROR]: An error occurred (DeleteConflict) when calling the DeleteUser operation: Cannot delete entity, must delete access keys first.
```

**Eliminar lector-juan luego de borrar sus llaves de acceso:**

```powershell
PS C:\Users\Juan Cañas> aws iam delete-user --user-name lector-juan
PS C:\Users\Juan Cañas> aws iam list-users
{
    "Users": [
        {
            "Path": "/",
            "UserName": "admin-juan-1",
            "UserId": "AIDAULLIDC6XJ7XCS3T2F",
            "Arn": "arn:aws:iam::299255404462:user/admin-juan-1",
            "CreateDate": "2026-11-20T00:07:32+00:00"
        }
    ]
}
```

---

### *Preguntas de reflexión*

##### 1. ¿Qué podía hacer el usuario justo después de crearlo, antes de adjuntarle una política? ¿Por qué?

No podía realizar ninguna acción dentro de AWS, porque al crear un usuario IAM no se le asignan permisos automáticamente. AWS aplica el principio de denegación por defecto, por lo que necesita tener una política que le permita realizar determinadas acciones.

##### 2. En el error del paso 13, ¿qué tres datos te dice el mensaje?

El error `AccessDenied` me permite identificar tres datos importantes:

+ **Quién:** el usuario IAM `lector-juan` es quien intenta ejecutar la acción.
+ **Qué acción:** `iam:ListUsers`, es decir, listar los usuarios de la cuenta.
+ **Por qué:** no tiene una política de permisos que autorice esa acción.

Esto demuestra que el usuario puede consultar información de EC2, pero no tiene permisos para administrar o consultar usuarios IAM.

##### 3. ¿Por qué cambió el `Arn` al renombrar el usuario?

Porque el ARN identifica al recurso dentro de AWS y contiene el nombre del usuario en su ruta. Al cambiar el nombre de `dev-juan` a `lector-juan`, también cambió esa parte del ARN. Aunque el usuario conserva su identidad interna (`UserId`), su ARN se actualiza para reflejar el nuevo nombre.

##### 4. ¿Por qué AWS obliga a borrar las llaves antes de borrar el usuario?

Porque las llaves de acceso permiten que el usuario se autentique en AWS desde cualquier computador donde se configuren mediante `aws configure`. 

Por ejemplo, si alguien obtiene el Access Key ID y el Secret Access Key, puede configurar esas credenciales en otro PC y utilizar la identidad del usuario para ejecutar acciones permitidas por las políticas. Por eso AWS exige eliminar primero las llaves, para evitar que queden credenciales activas al usuario que se quiere eliminar.

##### 5. ¿Qué pasaría si le hubieras dado `AdministratorAccess` en vez de solo lectura? ¿En qué caso real estaría justificado?

El usuario tendría permisos administrativos muy amplios, pudiendo realizar muchas acciones sobre los servicios y recursos de AWS, incluyendo crear, modificar y eliminar recursos. Esto sería un riesgo de seguridad si solo necesita realizar tareas específicas, de ahí el principio de menor privilegio.

Estaría justificado en un caso real cuando se necesita una cuenta administrativa para gestionar completamente los recursos de AWS, por ejemplo, un administrador encargado de configurar y mantener la infraestructura. Aun así, debería utilizarse únicamente cuando sea necesario y con las medidas de seguridad correspondientes.

