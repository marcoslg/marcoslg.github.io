---
layout: page
#
# Content
#
subheadline: "rendimiento"
title: "EF6 - precompilar vistas"
teaser: "Consideraciones de rendimiento"
categories:
  - entity framework
tags:
  - ef
  - precompilar
  - vistas
  - rendimiento
  - performance

#
# Styling
#
#header: no

---

# COMO PRECOMPILAR LAS VISTAS EN ENTITY FRAMEWORK

Dentro de los [procesos internos](https://docs.microsoft.com/es-es/previous-versions/dotnet/netframework-4.0/cc853327(v=vs.100)?redirectedfrom=MSDN)
 del propio Entity Framework (a partir de ahora EF) hay uno que hay que destacar que es el de la compilación/generación de las vistas en tiempo de ejecución. Estas vistas son una serie de metadatos que genera el EF para poder generar las sentencias SQL del modelo de datos. Este proceso es el más costos de todos, por este motivo la primera vez que se ejecuta una sentencia el EF tarda más que en la segunda vez. ¿Como podemos solucionar esto? pues depende de la versión del EF.

En las versiones EF inferiores a la 6 se precompilaban en el momento de compilar el proyecto. Para más información http://msdn.microsoft.com/es-es/library/vstudio/bb896240%28v=vs.100%29.aspx

En la versión inicial de EF, la 6.0.0, se han reportado bugs en el proceso de generación de las vistas haciendolo aún más lento, por esta razon se recomienda actualizar a la última versión (6.1.1).
Para la versiones superiores o igual a la 6, la manera de generar las vistas ha cambiado por lo tanto ya no nos sirve la precompilación que se hacia en la versión 5. Ahora no es necesario generar los metadatos del edmx, ejecutar el edmgen, etc. Tenemos la posibilidad de modificar la cache de la generación de las vistas (siendo esta la nueva manera).

![]("/assets/files/2014-02-09-ef6-precompilar-vistas-efinteractiveviews.png")


Para facilitarnos un poco la tarea nos tenemos que descargar el paquete nuget [“Interactive Pregenerated Views for Entity Framework 6”](https://efinteractiveviews.codeplex.com)

## Como usar Interactive Pregenerated Views for Entity Framework 6

Hay dos maneras de configurar la pregeneración de las vistas.

1. XML Local
2. Conexión a una base de datos.

### XML Local

```c#
InteractiveViews.SetViewCacheFactory(context, 
new FileViewCacheFactory(String.Format("{0}{1}{2}", AppDomain.CurrentDomain.BaseDirectory, context.Database.Connection.Database,".xml")));

```

### Conexión a una base de datos
```c#
InteractiveViews.SetViewCacheFactory(context,
 new SqlServerViewCacheFactory(context.Database.Connection.ConnectionString));
```


### ¿Puede generar algún error?

Sí, podria ser que el EF conserve este cambio, sobretodo si se crean los contextos con un intervalo de tiempo bajo. Para solucionar este problema antes hay que comprobar si ya tiene alguna asignación a la cache de las vistas.

```c#
public static bool Attached(DbContext context)
{
    var oCtx = ((IObjectContextAdapter)context).ObjectContext;
    var viewCache = (StorageMappingItemCollection)oCtx.MetadataWorkspace.GetItemCollection(DataSpace.CSSpace);
    return (viewCache.MappingViewCacheFactory is SqlServerViewCacheFactory || viewCache.MappingViewCacheFactory is FileViewCacheFactory);
}
```

### ¿Dónde se pone?

En un principio da igual el momento en el que se llame, pero como es obvio, tenemos que tener creado el contexto así que la recomendación es ponerlo en el constructor del contexto

```c#
public EjemploModelContext()
: base(...)
{
    InteractiveViewsHelper.AttachLocal(this); <-- Aquí ya se comprueba si ya tiene la cache asignada.
}

```

------------
Procesos internos del EF: http://msdn.microsoft.com/es-es/library/vstudio/cc853327%28v=vs.100%29.aspx
Interactive Pregenerated Views for Entity Framework 6: https://efinteractiveviews.codeplex.com/