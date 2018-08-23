---
layout: post
title: F# .NetCore & Type Providers
categories: es tutorials
---

# Introducción

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