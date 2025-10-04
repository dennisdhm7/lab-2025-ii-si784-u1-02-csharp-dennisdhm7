[![Open in Codespaces](https://classroom.github.com/assets/launch-codespace-2972f46106e565e64193e422d61a12cf1da4916b45550586e14ef0a7c637dd04.svg)](https://classroom.github.com/open-in-codespaces?assignment_repo_id=20892037)
# SESION DE LABORATORIO N° 02: PRUEBAS ESTATICAS DE SEGURIDAD DE APLICACIONES CON SNYK

## OBJETIVOS
  * Comprender el funcionamiento de las pruebas estaticas de seguridad de còdigo de las aplicaciones que desarrollamos utilizando Snyk.

## REQUERIMIENTOS
  * Conocimientos: 
    - Conocimientos básicos de Bash (powershell).
    - Conocimientos básicos de Contenedores (Docker).
  * Hardware:
    - Virtualization activada en el BIOS..
    - CPU SLAT-capable feature.
    - Al menos 4GB de RAM.
  * Software:
    - Windows 10 64bit: Pro, Enterprise o Education (1607 Anniversary Update, Build 14393 o Superior)
    - Docker Desktop 
    - Powershell versión 7.x
    - Net 8 o superior
    - Visual Studio Code

## CONSIDERACIONES INICIALES
  * Clonar el repositorio mediante git para tener los recursos necesarios
  * Tener una cuenta de Github valida. 

## DESARROLLO
### Parte I: Configuración de la herramienta de Pruebas Estaticas de Seguridad de la Aplicación
1. Ingrear a la pagina de Snyk (https://snyk.io/), iniciar sesión o registrarse con su cuenta de Github.
2. En la pagina de Snyk, ingresar a la opción Account Settings.
   
   ![image](https://github.com/UPT-FAING-EPIS/lab_calidad_02/assets/10199939/2b08058f-87d7-44ca-91b7-7f032374fd36)
3. En la pagina de Snyk, generar un nuevo token con cualquier nombre, luego de generar el token, guardar el resultado en algún archivo o aplicación de notas puesto que se utilizará más adelante.

   ![image](https://github.com/UPT-FAING-EPIS/lab_calidad_02/assets/10199939/0634dbf8-6721-4dfe-b258-2012594f90e4)
   
4. Iniciar la aplicación Powershell o Windows Terminal en modo administrador 
5. En el terminal, ejecutar los siguientes comandos para instalar snyk.
```Bash
scoop bucket add snyk https://github.com/snyk/scoop-snyk
scoop install snyk   
```
6. En el terminal, ejecutar los siguientes comandos para instalar snyk-to-html
```Bash
scoop install nvm
nvm install lts
nvm use lts
npm install snyk-to-html -g
```
7. Cerrar el terminal para que se actualicen las variables de entorno.

### Parte II: Creación de la aplicación
1. Iniciar la aplicación Powershell o Windows Terminal en modo administrador. 
2. En el terminal, Ejecutar el siguiente comando para crear una nueva solución
```
dotnet new sln -o Bank
dotnet tool install -g dll2mmd
dotnet tool install -g docfx
dotnet tool install -g dotnet-reportgenerator-globaltool
```
3. En el terminal, Acceder a la solución creada y ejecutar el siguiente comando para crear una nueva libreria de clases y adicionarla a la solución actual.
```
cd Bank
dotnet new webapi -o Bank.WebApi
dotnet sln add ./Bank.WebApi/Bank.WebApi.csproj
```
4. En el terminal, Ejecutar el siguiente comando para crear un nuevo proyecto de pruebas y adicionarla a la solución actual
```
dotnet new mstest -o Bank.WebApi.Tests
dotnet sln add ./Bank.WebApi.Tests/Bank.WebApi.Tests.csproj
dotnet add ./Bank.WebApi.Tests/Bank.WebApi.Tests.csproj reference ./Bank.WebApi/Bank.WebApi.csproj
```
5. Iniciar Visual Studio Code (VS Code) abriendo el folder de la solución como proyecto. En el proyecto Bank.Domain, si existe un archivo Class1.cs proceder a eliminarlo. Asimismo en el proyecto Bank.Domain.Tests si existiese un archivo UnitTest1.cs, también proceder a eliminarlo.

6. En el Visual Studio Code, en el proyecto Bank.WebApi proceder la carpeta `Models` y dentro de esta el archivo BankAccount.cs e introducir el siguiente código:
```C#
namespace Bank.WebApi.Models
{
    public class BankAccount
    {
        private readonly string m_customerName;
        private double m_balance;
        private BankAccount() { }
        public BankAccount(string customerName, double balance)
        {
            m_customerName = customerName;
            m_balance = balance;
        }
        public string CustomerName { get { return m_customerName; } }
        public double Balance { get { return m_balance; }  }
        public void Debit(double amount)
        {
            if (amount > m_balance)
                throw new ArgumentOutOfRangeException("amount");
            if (amount < 0)
                throw new ArgumentOutOfRangeException("amount");
            m_balance -= amount;
        }
        public void Credit(double amount)
        {
            if (amount < 0)
                throw new ArgumentOutOfRangeException("amount");
            m_balance += amount;
        }
    }
}
```
7. En el Visual Studio Code, en el proyecto Bank.WepApi.Tests añadir un nuevo archivo BanckAccountTests.cs e introducir el siguiente código:
```C#
using Bank.WebApi.Models;
using NUnit.Framework;

namespace Bank.WebApi.Tests
{
    public class BankAccountTests
    {
        [Test]
        public void Debit_WithValidAmount_UpdatesBalance()
        {
            // Arrange
            double beginningBalance = 11.99;
            double debitAmount = 4.55;
            double expected = 7.44;
            BankAccount account = new BankAccount("Mr. Bryan Walton", beginningBalance);
            // Act
            account.Debit(debitAmount);
            // Assert
            double actual = account.Balance;
            Assert.AreEqual(expected, actual, 0.001, "Account not debited correctly");
        }
    }
}
```
8. En el Visual Studio Code, en la raiz del proyecto crear un archivo Dockerfile con el siguiente contenido:
```Yaml
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
WORKDIR "/src/."
RUN dotnet restore 
RUN dotnet build -o /app/build

FROM build AS publish
RUN dotnet publish -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Bank.WebApi.dll"]
```

9. En el terminal, ejecutar el siguiente comando para ejecutar las pruebas unitarias y el reporte de cobertura
```Bash
dotnet test --collect:"XPlat Code Coverage"
ReportGenerator "-reports:./*/*/*/coverage.cobertura.xml" "-targetdir:Cobertura" -reporttypes:MarkdownSummaryGithub
``` 

10. En el terminal, ejecutar el siguiente comando para optener el diagrama de clases.
```Bash
dll2mmd -f Bank.WebApi/bin/Debug/net8.0/Bank.WebApi.dll -o disenio.md
``` 
11. En el terminal, ejecutar el siguiente comando para generar el diagrama de clases respectivo
```Bash
docfx init -y
```

12. En el Visual Studio Code, eliminar los archivos de la carpeta o directorio docs, seguidamente modificar los archivos con el siguiente contenido:
> docfx.json
```Json
{
  "$schema": "https://raw.githubusercontent.com/dotnet/docfx/main/schemas/docfx.schema.json",
  "metadata": [
    {
      "src": [
        {
          "src": ".",
          "files": [
            "**/*.csproj"
          ]
        }
      ],
      "dest": "docs"
    }
  ],
  "build": {
    "content": [
      {
        "files": [
          "**/*.{md,yml}"
        ],
        "exclude": [
          "_site/**"
        ]
      }
    ],
    "resource": [
      {
        "files": [
          "images/**"
        ]
      }
    ],
    "output": "_site",
    "template": [
      "default",
      "modern"
    ],
    "globalMetadata": {
      "_appName": "Bank.App",
      "_appTitle": "Bank App",
      "_enableSearch": true,
      "pdf": true
    }
  }
}
```
> toc.yml
```Yaml
- name: Docs
  href: docs/
```
> index.md
```Markdown
---
_layout: landing
---

# This is the **HOMEPAGE**.

## [Diagrama de Clases](disenio.md)

## [Pruebas](Cobertura/SummaryGithub.md)
```

13. En el terminal, ejecutar el siguiente comando para generar la documentacion
```Bash
docfx metadata docfx.json
docfx build
```

14. En el terminal, ejecutar el comando para construir la imagen del contenedor:
```Bash
docker build -t api-banco .
```
15. En el terminal, verificar que la imagen se genero correcmente :
```Bash
docker images 
```
> Resultado
```
REPOSITORY                                       TAG       IMAGE ID       CREATED         SIZE  
api-banco                                        latest    949c67f75e5e   2 minutes ago   247MB
```
16. En el terminal, ejecutar el siguiente comando para iniciar sesión en snyk :
```Bash
snyk auth <TOKEN>
```
> Donde:
> - TOKEN: es el token que previamente se genero en la pagina de Snyk.io

17. En el terminal, ejecutar el siguiente coamndo para realizar el analisis de codigo:
```Bash
snyk code test --json | snyk-to-html -o code-test-results.html
```
18. En el paso anterior se genero un archivo `code-test-results.html` el cual contiene el resultado del analisis que deberia ser similar a lo siguiente:

![image](https://github.com/UPT-FAING-EPIS/lab_calidad_02/assets/10199939/1d52853d-713b-4f5b-97e5-3d6d634a8172)

19. En el terminal, ejecutar el siguiente coamndo para realizar el analisis de codigo:
```
snyk container test api-banco --json | snyk-to-html -o container-test-result.html
```
20. En el paso anterior se genero un archivo `container-test-results.html` el cual contiene el resultado del analisis que deberia ser similar a lo siguiente:

![image](https://github.com/UPT-FAING-EPIS/lab_calidad_02/assets/10199939/b7515f19-0d6d-4401-b3ca-386a436ae6bf)

21. Abrir un nuevo navegador de internet o pestaña con la url de su repositorio de Github ```https://github.com/UPT-FAING-EPIS/nombre_de_su_repositorio```, abrir la pestaña con el nombre *Settings*, en la opción *Secrets and Actions*, selecionar Actions y hacer click en el botón *New Respository Token*, en la ventana colocar en Nombre (Name): SNYK_TOKEN y en Secreto (Secret): el valor del token de Snyk Cloud, guardado previamente

![image](https://github.com/user-attachments/assets/2a9d703c-3531-42c8-8f63-d386e86be11f)

22. En el VS Code, proceder a crear la carpeta .github/workflow y dentro de esta crear el archivo snyk.yml con el siguiente contenido.
```Yaml
name: Snyk Analysis
env:
  DOTNET_VERSION: '8.x'                     # la versión de .NET
on: push
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - uses: snyk/actions/setup@master
      - name: Configurando la versión de NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}  
      - name: Snyk monitor
        run: snyk code test --sarif-file-output=snyk.sarif
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}          
      - name: Upload result to GitHub Code Scanning
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: snyk.sarif
```

---
## Actividades Encargadas
1. Adicionar al archivo de snyk.yml los pasos necesarios para generar el reporte en formato HTML y subirlo como un artefacto en el resultado del job.
```Yaml
name: Snyk Analysis
env:
  DOTNET_VERSION: '9.x'   # versión actual de tu proyecto
on: push

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - uses: snyk/actions/setup@master

      - name: Configurando la versión de NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}

      - name: Snyk monitor
        run: snyk code test --json | snyk-to-html -o snyk-report.html
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

      - name: Upload result to GitHub Code Scanning
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: snyk.sarif

      - name: Upload HTML report
        uses: actions/upload-artifact@v3
        with:
          name: snyk-html-report
          path: snyk-report.html

```
2. Completar la documentación de todas las clases y generar una automatizaciòn .github/workflows/publish_docs.yml (Github Workflow) utilizando DocFx (init, metadata y build) y publicar el site de documentaciòn generado en un Github Page.
```Yaml
name: Publish Documentation
on:
  push:
    branches: [ "main" ]

jobs:
  build-docs:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '9.x'

      - name: Install DocFX
        run: dotnet tool install -g docfx

      - name: Build documentation (init, metadata, build)
        run: |
          docfx init -y
          docfx metadata docfx.json
          docfx build

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./_site

```
3. Generar una automatización de nombre .github/workflows/package_nuget.yml (Github Workflow) que ejecute:
   * Pruebas unitarias y reporte de pruebas automatizadas
   * Realice el analisis con SonarCloud.
   * Contruya un archivo .nuget a partir del proyecto Bank.Domain y lo publique como un Paquete de Github

```Yaml
name: Package NuGet
on: push

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '9.x'

      - name: Run Unit Tests and Coverage
        run: dotnet test --collect:"XPlat Code Coverage"

      - name: SonarCloud Analysis
        uses: SonarSource/sonarcloud-github-action@master
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

      - name: Build NuGet Package
        run: dotnet pack ./Bank.WebApi/Bank.WebApi.csproj -c Release -o ./nuget

      - name: Upload NuGet Package as artifact
        uses: actions/upload-artifact@v3
        with:
          name: nuget-package
          path: ./nuget/*.nupkg

```
4. Generar una automatización de nombre .github/workflows/release_version.yml (Github Workflow) que contruya la version (release) del paquete y publique en Github Releases e incluya pero ahi no esta el test unitarios
```Yaml
name: Release Version
on:
  push:
    tags:
      - 'v*.*.*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '9.x'

      - name: Build Project
        run: dotnet build -c Release

      - name: Create NuGet Package
        run: dotnet pack ./Bank.WebApi/Bank.WebApi.csproj -c Release -o ./nuget

      - name: Upload Release Package
        uses: softprops/action-gh-release@v1
        with:
          files: ./nuget/*.nupkg
```


