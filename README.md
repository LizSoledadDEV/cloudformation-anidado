# Práctica: CloudFormation con plantillas anidadas

## 1. Objetivo

Refactorizar una plantilla de AWS CloudFormation mediante la separación de la infraestructura en plantillas secundarias. Utilizar Git y GitHub para registrar y sincronizar los cambios realizados durante el desarrollo.

Los cambios se despliegan automáticamente en AWS mediante GitHub Actions: cualquier integrante del grupo modifica una plantilla, hace push y, al llegar el cambio a `main`, la infraestructura se actualiza.

## 2. Estructura del proyecto

```text
cloudformation-anidado/
├── .github/workflows/deploy.yml
├── .cfnlintrc
├── main.yaml
├── README.md
└── templates/
    ├── red.yaml
    ├── seguridad.yaml
    └── servidor.yaml
```

### Descripción de los archivos

- **main.yaml:** plantilla principal que coordina las plantillas secundarias mediante recursos `AWS::CloudFormation::Stack`.
- **templates/red.yaml:** define la VPC, la subred pública, el Internet Gateway y las rutas de red.
- **templates/seguridad.yaml:** define el grupo de seguridad y la regla de entrada HTTP.
- **templates/servidor.yaml:** define una instancia EC2 y sus parámetros de configuración.
- **.github/workflows/deploy.yml:** pipeline que valida y despliega las plantillas en AWS.
- **.cfnlintrc:** configuración de `cfn-lint`.
- **README.md:** documenta la estructura y los procedimientos de validación.

## 3. Funcionamiento de las plantillas anidadas

La plantilla principal organiza los recursos mediante tres stacks secundarios:

1. **Red:** crea la infraestructura de red y expone los identificadores de la VPC y la subred.
2. **Seguridad:** recibe el identificador de la VPC y expone el identificador del grupo de seguridad.
3. **Servidor:** recibe el identificador de la subred y el del grupo de seguridad para configurar la instancia EC2.

Las dependencias se establecen mediante `!Ref` y `!GetAtt`, que permiten pasar parámetros y consultar las salidas de otros stacks.

## 4. Validación local

Para revisar la sintaxis y detectar posibles problemas en las plantillas, se utiliza `cfn-lint`.

Desde la carpeta raíz del proyecto, ejecutar:

```bash
cfn-lint
```

`cfn-lint` toma las plantillas y las reglas desde `.cfnlintrc`.

Si el comando termina sin mostrar errores, significa que no se reportaron problemas por parte de `cfn-lint`. Esto no garantiza que las plantillas puedan desplegarse correctamente en AWS.

También se puede comprobar que no existan errores de formato en los cambios de Git:

```bash
git diff --check
```

## 5. Despliegue automático con GitHub Actions

El workflow `.github/workflows/deploy.yml` se ejecuta en cada push:

| Evento | Validar (`cfn-lint`) | Desplegar en AWS |
|---|---|---|
| Push a cualquier rama | Sí | No |
| Pull request hacia `main` | Sí | No |
| Push o merge a `main` | Sí | Sí |

Al desplegar, el workflow:

1. Crea (si no existe) el bucket `cfn-anidado-<cuenta>-<region>`.
2. Ejecuta `aws cloudformation package`, que sube `templates/*.yaml` a S3 y reemplaza las rutas locales de `TemplateURL` por sus URL.
3. Ejecuta `aws cloudformation deploy` sobre el stack `asistencia`, que crea o actualiza solo los recursos modificados.
4. Muestra las salidas del stack (VPC, subred, grupo de seguridad e instancia).

### Configuración inicial (una sola vez)

En GitHub: **Settings → Secrets and variables → Actions → New repository secret**:

| Secret | Valor |
|---|---|
| `AWS_ACCESS_KEY_ID` | Access key de AWS |
| `AWS_SECRET_ACCESS_KEY` | Secret key de AWS |
| `AWS_SESSION_TOKEN` | Solo si las credenciales son temporales (por ejemplo, AWS Academy) |

Además, cada integrante debe ser colaborador del repositorio (**Settings → Collaborators**).

> Las credenciales de AWS Academy vencen al cerrar la sesión del laboratorio. Hay que volver a copiarlas en los secrets antes de cada despliegue.

### Flujo de trabajo de un integrante

```bash
git checkout main
git pull origin main
git checkout -b cambio-nombre-servidor

# Editar la plantilla, por ejemplo en templates/servidor.yaml:
#   Value: !Sub '${EnvironmentName}-servidor-web'

git add templates/servidor.yaml
git commit -m "feat: cambiar nombre del servidor"
git push origin cambio-nombre-servidor
```

Después, en GitHub se abre un **Pull Request** hacia `main`. Cuando la validación pasa, se hace el merge y el workflow despliega el cambio. El avance se ve en la pestaña **Actions**, y el resultado en la consola de AWS: **CloudFormation → Stacks → asistencia** y **EC2 → Instancias**.

## 6. Consideraciones importantes

- La regla HTTP permite conexiones desde cualquier dirección IPv4. En un entorno real, debe limitarse según los requisitos de seguridad.
- El despliegue crea recursos reales en AWS (incluida una instancia `t3.micro`). Para eliminarlos: `aws cloudformation delete-stack --stack-name asistencia`.
- Cambiar etiquetas como `Name` actualiza la instancia sin reemplazarla. Cambiar `InstanceType` la detiene y la reinicia, y cambiar la AMI o la subred la reemplaza.

## 7. Resultado esperado

El proyecto queda organizado en plantillas reutilizables, con responsabilidades separadas, dependencias explícitas y un historial de versiones administrado mediante Git y GitHub. Cada cambio que un integrante sube a `main` se valida y se aplica automáticamente en AWS.
