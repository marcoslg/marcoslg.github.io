---
layout: page
#
# Content
#
subheadline: ""
title: "EF6 - cache nivel 2"
teaser: "Consideraciones de rendimiento"
categories:
  - entity framework
tags:
  - ef
  - cache
  - rendimiento
  - performance

#
# Styling
#
#header: no

---

## ¿Que es Cache de nivel 2 en entity framework?
Como todos sabemos entity framework, a partir de ahora ef, construye y ejecute instrucciones SQL a una base de datos transformando el resultado a un objeto que luego sera controlado dentro de un contexto. Para poder cachear estos objetos debido a que estan controlados y asociados a un determinado contexto si los cacheamos directamente luego cuando los recuperemos de la cache donde el contexto es otro nuevo no podremos añadirlo al nuevo contexto de una forma sencilla.

Para solucionar este problema, en lugar de cachear los objetos se cachean los resultados antes de ser transformados en objeto esto es lo que se llama una cache de nivel 2.



![]("/assets/files/2014-09-09-ef6-cache-nivel-2-1.png")


## ¿Que implementaciones existentes hay?

Tenemos varias soluciones ya implementadas, como por ejemplo Ncache y EntityFramework.Cache, que se basan en los mismo. Nosotros trabajamos con EntityFramework.Cache, disponible desde los paquetes Nuget. Es una implementación sencilla y básica.


## EntityFramework.Cache
### Instalación

Abrimos el gestor de nugets y buscamos EntityFramework.Cache e instalamos el paquete.

![]("/assets/files/2014-09-09-ef6-cache-nivel-2-nuget-efcacahe.png")

1. XML Local
2. Conexión a una base de datos.

### Implementación

Para utilizar el paquete debemos crear una clase de configuración para el ef. La clase para configurar el ef debe de heredar de DbConfiguration, hay que tener en cuenta que solo se puede tener una configuración, no pueden existir configuraciones diferentes por modelo.

```c#
public class CustomDbConfiguration : DbConfiguration
{

    public CustomDbConfiguration()
    : base()
    {

        var transactionHandler = new CacheTransactionHandler(new CacheLevel2());
        AddInterceptor(transactionHandler);

        var cachingPolicy = new CachingPolicySelected();
        Loaded += (sender, args) => args.ReplaceService<DbProviderServices>(
            (s, _) => new CachingProviderServices(s, transactionHandler, cachingPolicy));

    }

}

```

Como podemos ver en el ejemplo, ya tenemos implementada la cache de nivel 2 del entity, con este código interceptamos las operaciones sql y los resultados de las sentencias sql para poder cachearlas.

**CacheLevel2** es quien se encargará de gestionar la cache con las operaciones básicas y  **CachingPolicySelected** indica la política a utilizar, en nuestro caso utilizamos una propia.

```
public class CachingPolicySelected : CachingPolicy
{

    protected override bool CanBeCached(ReadOnlyCollection<EntitySetBase> affectedEntitySets, string sql, IEnumerable<KeyValuePair<string, object>> parameters)
    {

        //cacheamos resultados ConfigurationDAL
        Dictionary<string, List<string>> cacheList = new Dictionary<string, List<string>>()
            {
                {“BCApplicationsModelStoreContainer”, new List<string>(){ “Configurations”, “Companies”}},
                {“ContractsStoreContainer”, new List<string>(){ “Configurations”}}
            };

        if (affectedEntitySets.Count == 1)
        {
            var containerStoreName = affectedEntitySets.FirstOrDefault().EntityContainer.Name;
            var setName = affectedEntitySets.FirstOrDefault().Name;
            return cacheList.ContainsKey(containerStoreName) && cacheList[containerStoreName].Contains(setName);
        }
        return false;

    }

}
```

------------

#### Bibliografía
Cache level 2: http://blog.3d-logic.com/2014/03/20/second-level-cache-for-ef-6-1/