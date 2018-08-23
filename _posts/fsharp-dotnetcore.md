---
layout: post
title: F# y .Net Core 2.0
categories: es tutorials
---

# Introducción

Como ya sabéis .Net Core 2.0 ya está aquí, y eso significa que vamos a poder crear aplicaciones de .Net no solo desde Windows, sino también desde macOX o Linux.

Tengo que reconocer que cada dia me siento más cómodo usando macOX como mi principal entorno de desarrollo, y la verdad, no me gusta nada tener que ponerle Parallels u otra herramienta que te crea una máquina virtual corriendo Windows, creo que de verdad afecta al rendimiento de la máquina para poder hacer un trabajo medio en condiciones.

Bueno, al caso que nos lleva. Sigo avanzando poco a poco en el apredizaje de F# y he encontrado en .Net Core la herramienta perfecta para crear mis soluciones y proyectos en los que voy trasteando conforme voy aprendiendo.

# Intalación de .Net Core

Lo primero, por aupuesto, es instalar .Net Core desde el siguiente enlace:

Una vez instalado, ya podemos empezar a trastear con él:

- `dotnet new`. Con esta sentencia podemos ver los tipos de templates que tenemos instalados para poder crear un proyecto con .Net Core: 

![dotnet new]({{ site.url }}/images/dotnetcore1.png)

## Otros comandos de dotnet core

- `dotnet new sln`: Crear una solución en blanco.

- `dotnet new library -lang F# -o FSLibrary`: Crear un proyecto de tipo **library**.

- `dotnet new xunit -lang F# -o FSTests`: Crear un proyecto de xUnit para tus tests.

- `dotnet sln add FSLibrary/FSLibrary.fsproj`: Añadir un proyecto a una solución.

- `dotnet restore`: Restaura los paquetes nugets que pueda utilizar tu solción.

- `dotnet build`: Compila la solución.

- `dotnet test`: Ejecuta los tests que tuvieras implementados.

- `dotnet new -i giraffe-template::*`: En este caso voy a crear un proyecto basado en (Giraffe)[http://www.google.com]. Se trata de un framework muy similar a **Suave** pero para .Net Core. Como funciona este framework lo trataré en un futuro post, porque la verdad, merece mucho la pena.

# .Net Core y Type Providers

Lamentablemente TypeProviders no funcionan correctamente todavía con .NetCore. Voy a explicaros aquí un workarraound para poder habilitarlos facilmente. Todos los pasos a seguir están descritos en el siguiente link: [https://github.com/Microsoft/visualfsharp/issues/3303](https://github.com/Microsoft/visualfsharp/issues/3303).

# The Work-arround

Lo que vamos a hacer, es que sea, msbuild el que compile nuestro proyecto, en vez de dotnetcore. Aunque con esta solución, vamos a poder seguir utilizando los comandos de dotnetcore en terminal sin problema, como: `dotnet build`.

Como pre-requisito, vamos a necesitar tener instalado .Net Framework o Mono dependiendo si nuestro entorno de programación es Windows, OS o Linux.

1. Incluir el siguiente archivo `fsc.props` en el directorio raiz de nuestra solución:

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project>
  <!-- Type providers currently can't run inside the .NET Core 2.0 hosted compiler, see https://github.com/Microsoft/visualfsharp/pull/3658#issuecomment-334773415 -->
  <PropertyGroup>
    <IsWindows Condition="'$(OS)' == 'Windows_NT'">true</IsWindows>
    <IsOSX Condition="'$([System.Runtime.InteropServices.RuntimeInformation]::IsOSPlatform($([System.Runtime.InteropServices.OSPlatform]::OSX)))' == 'true'">true</IsOSX>
    <IsLinux Condition="'$([System.Runtime.InteropServices.RuntimeInformation]::IsOSPlatform($([System.Runtime.InteropServices.OSPlatform]::Linux)))' == 'true'">true</IsLinux>
  </PropertyGroup>  
  <PropertyGroup Condition="'$(IsWindows)' == 'true' AND Exists('C:\Program Files (x86)\Microsoft SDKs\F#\4.1\Framework\v4.0\fsc.exe')">
    <FscToolPath>C:\Program Files (x86)\Microsoft SDKs\F#\4.1\Framework\v4.0</FscToolPath>
    <FscToolExe>fsc.exe</FscToolExe>
  </PropertyGroup>
  <PropertyGroup Condition="'$(IsOSX)' == 'true'  AND Exists('/Library/Frameworks/Mono.framework/Versions/Current/Commands/fsharpc')">
    <FscToolPath>/Library/Frameworks/Mono.framework/Versions/Current/Commands</FscToolPath>
    <FscToolExe>fsharpc</FscToolExe>
  </PropertyGroup>
  <PropertyGroup Condition="'$(IsLinux)' == 'true' AND Exists('/usr/bin/fsharpc')">
    <FscToolPath>/usr/bin</FscToolPath>
    <FscToolExe>fsharpc</FscToolExe>
  </PropertyGroup>
</Project>
```

2. Modificar vuestros archivos `.fsproj` incluyendo el archivo anterior creado y modificar el TargetFramework a `netcoreapp2.0;net461`. Esta modificación solo tendremos que hacerla en aquellos proyectos de nuestra solución que utlicen `TypeProviders`: 

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <!-- Type providers currently can't run inside the .NET Core 2.0 hosted compiler, see https://github.com/Microsoft/visualfsharp/pull/3658#issuecomment-334773415 -->
    <!-- <TargetFrameworks>netcoreapp2.0</TargetFrameworks>  -->
     <TargetFrameworks>netcoreapp2.0;net461</TargetFrameworks>
     <IsPackable>false</IsPackable>
  </PropertyGroup>
  <Import Project="..\fsc.props" />
  <ItemGroup>
    <!-- Rest of the file -->
  </ItemGroup>
</Project>
```

# Conclusión

Si queréis empezar con F#, .Net core es muy buena herramienta para hacerlo, pero por lo que habéis visto con los **Type Providers** aun le queda un poco para que esté fino del todo, pero eso no quita que podamos empezar a jugar con él ¿no?.

Saludos