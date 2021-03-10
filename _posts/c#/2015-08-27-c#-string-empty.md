---
layout: page
#
# Content
#
subheadline: "strings"
title: "string vacio"
teaser: "Consideraciones de rendimiento"
short:"<p>¿Por qué debo usar string.Empty y no \"\"?</p>
categories:
  - c#
tags:
  - c#
  - string
  - strings  
  - rendimiento
  - performance

#
# Styling
#
#header: no

---

`Hace ya bastante que no publico un post, lo reconozco, me da pereza tener que enrollarme. Por esta razon voy hacer un post corto pero muy util, sobretodo para aquellos que se estan iniciando en c#.`
## ¿Por qué debo usar string.Empty y no “”?

Cuando estamos programando, en ocasiones tenemos la necesidad de utilizar una cadena vacia, es decir, “”. Pero en la mayoria de ocasiones ya sea por pereza, rapidez o simplemente desconocimiento no utilizamos el **string.Empty** cuando es la mejor opción.

Ahora te estaras preguntando pero si es lo mismo ¿no? que más da usar string.Empty que “”, pues sí es importante.

Si usas “” lo que estas haciendo es ocupar memoria de forma innecesaria. Por ejemplo, si tenemos un bucle que se ejecuta 10 veces con “” en su contenido lo que estaremos haciendo es generar 10 objetos “” en memoria, en cambio si utilizas string.Empty no se generaran esos 10 objetos de “” optimizando memoria.

En otro post express explicare como construir correctamente una cadena de strings.

A picar como monos!
