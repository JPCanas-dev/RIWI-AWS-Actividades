# RIWI - Actividades AWS Cloud

Repositorio de actividades prácticas sobre servicios fundamentales de AWS utilizando AWS CLI.

## Índice

- [Semana 1](#semana-1)
  - [A1: Ciclo de vida de un usuario IAM](#a1-ciclo-de-vida-de-un-usuario-iam)
- [Semana 2](#semana-2)
  - [A1: Ciclo de vida de un bucket en S3](#a1-ciclo-de-vida-de-un-bucket-en-s3)
  - [A2: Un rol para tu instancia EC2](#a2-un-rol-para-tu-instancia-ec2)
  - [A3: Tu primer servidor en EC2](#a3-tu-primer-servidor-en-ec2)

---

## Descripción de actividades

### Semana 1

#### A1: Ciclo de vida de un usuario IAM
- **Carpeta:** [`Semana_1/A1-ciclo-de-vida-usuario-iam`](Semana_1/A1-ciclo-de-vida-usuario-iam/enunciado-actividad.md)
- **Descripción:** Creación, consulta, etiquetado y renombrado de un usuario IAM desde AWS CLI. Incluye la asignación y revocación de la política `AmazonEC2ReadOnlyAccess`, pruebas de acciones permitidas y denegadas bajo el principio de menor privilegio con un perfil temporal, y la eliminación ordenada de llaves, permisos y usuario.

### Semana 2

#### A1: Ciclo de vida de un bucket en S3
- **Carpeta:** [`Semana_2/A1-ciclo-de-vida-bucket-s3`](Semana_2/A1-ciclo-de-vida-bucket-s3/practica-s3/activity.md)
- **Descripción:** Creación de un bucket S3, carga, consulta, descarga y modificación de objetos mediante AWS CLI. Se trabaja la generación de enlaces temporales para compartir archivos sin hacer público el bucket, pruebas de menor privilegio con un usuario limitado a `s3:GetObject` y la limpieza completa de los recursos.

#### A2: Un rol para tu instancia EC2
- **Carpeta:** [`Semana_2/A2-rol-para-instancia-ec2`](Semana_2/A2-rol-para-instancia-ec2/practica-roles/activity-roles.md)
- **Descripción:** Preparación de un bucket S3 con un archivo de prueba y creación del rol `ec2-lector-s3` con política de confianza exclusiva para EC2 y permisos de solo lectura (`s3:GetObject`) sobre el bucket. El rol se integra en un perfil de instancia y se validan reglas de conflicto al intentar eliminarlo antes de tiempo.

#### A3: Tu primer servidor en EC2
- **Carpeta:** [`Semana_2/A3-primer-servidor-ec2`](Semana_2/A3-primer-servidor-ec2/practica-ec2/activity-ec2.md)
- **Descripción:** Lanzamiento de una instancia EC2 (`t3.micro` con Amazon Linux 2023) configurada con un script de inicio para levantar un servidor web, par de llaves `.pem` y security group (puerto 80 abierto y puerto 22 restringido a la IP propia). Se verifica desde la instancia el consumo de S3 usando el rol de IAM sin credenciales locales, el cambio de IP pública entre estados (`stopped`/`running`) y la eliminación final de todos los recursos.
