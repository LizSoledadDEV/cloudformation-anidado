# Práctica: CloudFormation con plantillas anidadas

## 1. Objetivo

Refactorizar una plantilla de AWS CloudFormation mediante la separación de la infraestructura en plantillas secundarias. Utilizar Git y GitHub para registrar y sincronizar los cambios realizados durante el desarrollo.

Esta práctica se trabaja de forma local. No es necesario desplegar recursos en AWS.

## 2. Estructura del proyecto

```text
cloudformation-anidado/
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
cfn-lint main.yaml templates/red.yaml templates/seguridad.yaml templates/servidor.yaml
```

Si el comando termina sin mostrar errores, significa que no se reportaron problemas por parte de `cfn-lint`. Esto no garantiza que las plantillas puedan desplegarse correctamente en AWS.

También se puede comprobar que no existan errores de formato en los cambios de Git:

```bash
git diff --check
```

## 5. Control de versiones con Git

Para revisar el estado del repositorio:

```bash
git status
```

Para registrar cambios:

```bash
git add README.md
git commit -m "docs: documentar practica de CloudFormation anidado"
```

Para sincronizar los cambios con GitHub:

```bash
git push origin main
```

Estos comandos permiten mantener un historial de cambios y sincronizar el trabajo local con el repositorio remoto.

## 6. Consideraciones importantes

- Las direcciones `TemplateURL` de `main.yaml` son marcadores de posición. Deben reemplazarse por URL válidas de plantillas almacenadas en una ubicación compatible antes de un despliegue real.
- La regla HTTP permite conexiones desde cualquier dirección IPv4. En un entorno real, debe limitarse según los requisitos de seguridad.
- La instancia EC2 y los demás recursos no se crean por validar las plantillas localmente.
- Esta práctica no contempla el despliegue de infraestructura ni la creación de recursos facturables en AWS.

## 7. Resultado esperado

El proyecto queda organizado en plantillas reutilizables, con responsabilidades separadas, dependencias explícitas y un historial de versiones administrado mediante Git y GitHub. La validación local permite detectar problemas antes de considerar cualquier despliegue.
