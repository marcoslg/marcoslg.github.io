---
layout: page
#
# Content
#
subheadline: "Transformaciones de archivos .config"
title: "VISUAL STUDIO 2012 PARA REALIZAR TRANSFORMACIONES DE LOS ARCHIVOS .CONFIG"
teaser: ""
categories:
  - visual studio
tags:
  - vs
  - vs2012
  - vs 2012
  - .config

#
# Styling
#
#header: no

---

# CONFIGURAR PROYECTO EN VISUAL STUDIO 2012 PARA REALIZAR TRANSFORMACIONES DE LOS ARCHIVOS .CONFIG

Nuestro proyecto tiene como necesidad tener varios archivos de configuración (.config). Debido a esta necesidad y al tener varios entornos (dev, test y prod) necesitamos que dependiendo del entorno estos archivos tengan una configuración especifica por entorno.

Para conseguir que en estos archivos se realice la transformación dependiendo del entorno necesitamos realizar una serie de pasos que indico a continuación.

1. Modificar las propiedades de todos los archivos de configuración excepto el principal
    1. Build Action : None
    2. Copy to output Directory : Do not Copy
2. Descargar de la solución el proyecto.
3. Editar el proyecto (boton derecho -> editar …)
4. Añadir las lineas correspondientes para forzar la transformación.
5. Modificar proyecto de azure.


## Proyecto

Para forzar la transformación debemos añadir una instrucción para indicarle que después de compilar tiene que realizar las transformaciones.

```xml
<Target Name="AfterCompile" Condition="Exists('Sql.$(Configuration).config')">
```
Ahora debemos indicarle dentro de la etiqueta que archivos debe transformar si o si. En este caso seria para el Sql.config

```xml
<TransformXml Source="Sql.config" Destination="$(IntermediateOutputPath)Sql.config" Transform="Sql.$(Configuration).config" />
<ItemGroup>
    <AppConfigWithTargetPath Remove="Sql.config" />
    <AppConfigWithTargetPath Include="$(IntermediateOutputPath)Sql.config">
        <TargetPath>Sql.config</TargetPath>
    </AppConfigWithTargetPath>
</ItemGroup>
```

A continuación mostrare como tiene que quedar.

```xml
<Target Name="AfterCompile" Condition="Exists('Sql.$(Configuration).config')">
    <TransformXml Source="Sql.config" Destination="$(IntermediateOutputPath)Sql.config" Transform="Sql.$(Configuration).config" />
    <ItemGroup>
        <AppConfigWithTargetPath Remove="Sql.config" />
        <AppConfigWithTargetPath Include="$(IntermediateOutputPath)Sql.config">
            <TargetPath>Sql.config</TargetPath>
        </AppConfigWithTargetPath>
    </ItemGroup>
    <TransformXml Source="config\AppSettings.config" Destination="$(IntermediateOutputPath)AppSettings.config" Transform="config\AppSettings.$(Configuration).config" />
    <ItemGroup>
        <AppConfigWithTargetPath Remove="config\AppSettings.config" />
        <AppConfigWithTargetPath Include="$(IntermediateOutputPath)AppSettings.config">
            <TargetPath>config\AppSettings.config</TargetPath>
        </AppConfigWithTargetPath>
    </ItemGroup>
    <TransformXml Source="Unity.config" Destination="$(IntermediateOutputPath)Unity.config" Transform="Unity.$(Configuration).config" />
    <ItemGroup>
        <AppConfigWithTargetPath Remove="Unity.config" />
        <AppConfigWithTargetPath Include="$(IntermediateOutputPath)Unity.config">
            <TargetPath>Unity.config</TargetPath>
        </AppConfigWithTargetPath>
    </ItemGroup>
</Target>
```
## Proyecto de azure

En el momento de publicar en azure tenemos el mismo problema que en el proyecto. Se generan las transformaciones pero no las tiene en cuenta para crear el paquete que después se publicara, por lo tanto tenemos que forzarle para que las incluya en el paquete. ¿Como se hace esto? pues siguiendo las instrucciones aquí detalladas.

1. Descargar de la solución el proyecto de azure.
2. Editar el proyecto (boton derecho -> editar …)
3. Añadir las lineas correspondientes para forzar que empaquete las transformaciones.

Para indicarle que queremos incluir las transformaciones tenemos que buscar en el archivo editado del proyecto la linea "\<Import Project=”$(CloudExtensionsDir)Microsoft.WindowsAzure.targets” />". Justo debajo tenemos que añadir el siguiente código.

```xml
<Target Name="CopyWorkerRoleConfigurations" AfterTargets="CopyWorkerRoleFiles">
    <Copy SourceFiles="$(WebTargetDir)sql.config" DestinationFolder="$(IntermediateOutputPath)<<nombreProyecto>>" OverwriteReadOnlyFiles="true" />
    <Copy SourceFiles="$(WebTargetDir)unity.config" DestinationFolder="$(IntermediateOutputPath)<<nombreProyecto>>" OverwriteReadOnlyFiles="true" />
    <Copy SourceFiles="$(WebTargetDir)config\AppSettings.config" DestinationFolder="$(IntermediateOutputPath)<<nombreProyecto>>\config\" OverwriteReadOnlyFiles="true" />
    <Copy SourceFiles="$(WebTargetDir)ApplicationInsights.config" DestinationFolder="$(IntermediateOutputPath)<<nombreProyecto>>" OverwriteReadOnlyFiles="true" />
</Target>
```

Cada \<copy> es un archivo de configuración que queremos obligar a que coja su transformación y no su original (archivo raiz).

## ¿Como puedo saber si funciona?

Para probar que la publicación en azure realizara las transformaciones debemos de generar el paquete. Una vez se ha generado el paquete, con un compresor de archivos zip (por ejemplo winrar) abrimos el archivo *.cspkg, nos vamos a ********.cssx y vamos a mirar en la carpeta sitesroot/0 si los archivos están transformados.