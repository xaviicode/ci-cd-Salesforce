
```markdown
# Proyecto Salesforce DX: mi-proyecto-cicd

Bienvenido a la documentación dinámica y completa para el proyecto Salesforce DX **"mi-proyecto-cicd"**. Aquí encontrarás todo lo necesario para crear, conectar, versionar y automatizar tu proyecto Salesforce, incluyendo buenas prácticas, solución de errores y comandos útiles para CI/CD.

---

## Índice

- [1. Creación del Proyecto](#1-creación-del-proyecto)
- [2. Configuración de la Conexión con Salesforce](#2-configuración-de-la-conexión-con-salesforce)
- [3. Recuperación y Versionado de Metadatos](#3-recuperación-y-versionado-de-metadatos)
- [4. Pipeline CI/CD con GitHub Actions](#4-pipeline-cicd-con-github-actions)
- [5. Manejo de Claves y Seguridad](#5-manejo-de-claves-y-seguridad)
- [6. Solución de Problemas Comunes](#6-solución-de-problemas-comunes)
- [7. Consulta y Verificación de Conexión](#7-consulta-y-verificación-de-conexión)
- [8. Buenas Prácticas](#8-buenas-prácticas)
- [9. Recursos Útiles](#9-recursos-útiles)

---

## 1. Creación del Proyecto

- Inicializa el proyecto Salesforce DX:
  ```
  sf project generate -n mi-proyecto-cicd
  ```
- Abre el proyecto en Visual Studio Code para una mejor gestión y edición de archivos[1].

---

## 2. Configuración de la Conexión con Salesforce

- Autoriza tu org de Salesforce:
  ```
  sf org login web -a devorg
  ```
- Genera claves JWT para autenticación segura:
  ```
  openssl genrsa -out server.key 2048
  openssl req -new -x509 -key server.key -out server.crt -days 365
  ```
- Crea una Connected App en Salesforce, sube el certificado y copia el Consumer Key[2].
- Configura los permisos en la Connected App:
  - Edit Policies > Permitted Users: “Admin approved users are pre-authorized”
  - Manage Profiles > Asigna “System Administrator”[2].

---

## 3. Recuperación y Versionado de Metadatos

- Crea el archivo `manifest/package.xml` con los tipos de metadatos relevantes.
- Recupera los metadatos desde tu org:
  ```
  sf project retrieve start --manifest manifest/package.xml --target-org devorg
  ```
- Inicializa Git y haz tu primer commit:
  ```
  git init
  git add .
  git commit -m "Primer commit: Proyecto Salesforce DX conectado"
  ```

---

## 4. Pipeline CI/CD con GitHub Actions

- Estructura recomendada:
  ```
  / (raíz)
    |-- sfdx-project.json
    |-- force-app/
    |-- manifest/
    |-- .github/
        |-- workflows/
            |-- deploy.yml
  ```
- Ejemplo de workflow `.github/workflows/deploy.yml`:
  ```
  name: Deploy to Salesforce Dev Org

  on:
    push:
      - main
    workflow_dispatch:

  jobs:
    deploy:
      runs-on: ubuntu-latest
      steps:
        - name: Checkout
          uses: actions/checkout@v3

        - name: Setup Salesforce CLI
          uses: salesforce-actions/setup-sfdx@v1

        - name: Write server.key file
          run: |
            mkdir -p assets
            echo "${{ secrets.SF_JWT_KEY }}" > assets/server.key

        - name: Authenticate to Salesforce (JWT)
          run: |
            sfdx auth:jwt:grant \
              --clientid ${{ secrets.SF_CLIENT_ID }} \
              --jwtkeyfile assets/server.key \
              --username ${{ secrets.SF_USERNAME }} \
              --instanceurl ${{ secrets.SF_INSTANCE_URL }} \
              --setdefaultusername

        - name: Remove server.key
          run: rm -f assets/server.key

        - name: Deploy source to org (Check Only)
          run: sfdx force:source:deploy -p force-app -u ${{ secrets.SF_USERNAME }} --checkonly

        - name: Run Apex tests
          run: sfdx force:apex:test:run -u ${{ secrets.SF_USERNAME }} --resultformat human --wait 10
  ```
- Configura los secrets en GitHub:  
  `SF_CLIENT_ID`, `SF_USERNAME`, `SF_JWT_KEY`, `SF_INSTANCE_URL`[3].

---

## 5. Manejo de Claves y Seguridad

- Guarda `server.key` y `server.crt` en una carpeta local como `secrets/` o `assets/` y agrégala a `.gitignore`:
  ```
  secrets/
  assets/
  *.key
  *.crt
  ```
- Nunca subas archivos sensibles al repositorio. Usa el contenido de `server.key` como secret en GitHub Actions[2].

---

## 6. Solución de Problemas Comunes

| Error | Causa | Solución |
|-------|-------|----------|
| `user is not admin approved to access this app` | Usuario sin acceso a la Connected App | Asigna el perfil/permiso en la Connected App[2] |
| `This command is required to run from within an SFDX project` | Comando fuera de la raíz del proyecto | Ejecuta desde la raíz donde está `sfdx-project.json`[1] |
| `No source backed components present in the package` | Carpetas vacías, archivos ignorados o manifest incompleto | Revisa `.forceignore`, estructura y `package.xml`[3] |

---

## 7. Consulta y Verificación de Conexión

- Verifica orgs conectadas:
  ```
  sf org list
  ```
- Prueba autenticación JWT:
  ```
  sfdx auth:jwt:grant --client-id  --jwt-key-file server.key --username  --instance-url https://login.salesforce.com --set-default
  ```
- Consulta componentes desplegados en la org desde la interfaz de Salesforce.

---

## 8. Buenas Prácticas

- Usa siempre secrets para claves y credenciales.
- Versiona todos los cambios y documenta el pipeline en el README.
- Elimina archivos sensibles después de usarlos en el pipeline.
- Mantén actualizado el archivo `.gitignore` para proteger información privada.

---

## 9. Recursos Útiles

- [Guía oficial de configuración de Salesforce DX][1]
- [JWT Bearer Flow para la CLI][2]
- [Automatización con GitHub Actions y Salesforce][3]

---

**Con este README tienes toda la documentación para crear, conectar, versionar y automatizar tu proyecto Salesforce DX "mi-proyecto-cicd" de forma robusta y segura.**

[1]: https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_workspace_setup.htm
[2]: https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_auth_jwt_flow.htm
[3]: https://trailhead.salesforce.com/content/learn/projects/automate-app-development-with-continuous-integration-and-delivery/set-up-github-actions
```







# Salesforce DX Project: Next Steps

Now that you’ve created a Salesforce DX project, what’s next? Here are some documentation resources to get you started.

## How Do You Plan to Deploy Your Changes?

Do you want to deploy a set of changes, or create a self-contained application? Choose a [development model](https://developer.salesforce.com/tools/vscode/en/user-guide/development-models).

## Configure Your Salesforce DX Project

The `sfdx-project.json` file contains useful configuration information for your project. See [Salesforce DX Project Configuration](https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_ws_config.htm) in the _Salesforce DX Developer Guide_ for details about this file.

## Read All About It

- [Salesforce Extensions Documentation](https://developer.salesforce.com/tools/vscode/)
- [Salesforce CLI Setup Guide](https://developer.salesforce.com/docs/atlas.en-us.sfdx_setup.meta/sfdx_setup/sfdx_setup_intro.htm)
- [Salesforce DX Developer Guide](https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_intro.htm)
- [Salesforce CLI Command Reference](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference.htm)
