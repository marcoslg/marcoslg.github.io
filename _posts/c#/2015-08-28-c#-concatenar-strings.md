---
layout: page
#
# Content
#
subheadline: "c#"
title: "Concatenar strings"
teaser: "Consideraciones de rendimiento"
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

`Esta semana estoy que lo peto. Otro post express!! para iniciados.`

## ¿Como unirias dos strings?
string str1 = "hola";
string str2 = "mundo";
string str12 = str1 + str2;

Si esta es tu respuesta date una cleca y sigue leyendo.


## Como unir o generar un string

Esto nos puede servir tanto para c# como para java. Unir strings utilizando el operador “+” es pecado capital en C# y java.

Se debe utilizar el objecto StringBuilder, que como el mismo nombre dice, construye strings. Es muy sencillo de utilizar y se suele usar cuando necesitamos generar un string utilizando bucles.

El ejemplo anterior pero bien hecho.

Opción 1
```c#
StringBuilder  strb = new StringBuilder();
strb.Append("Hola");
strb.Append(" mundo!");

string str12 =strb.ToString();
```
Opción 2
```c#
StringBuilder  strb = new StringBuilder("Hola");
strb.Append(" mundo!");
string str12 =strb.ToString();
```
Opción 3
```c#
StringBuilder  strb = new StringBuilder();
strb.AppendFormat("{0} {1}", "Hola", "mundo!");
string str12 =strb.ToString();
```
 

Utilicar el StringBuilder para unir dos cadenas puede ser un poco engorroso, asi que utilizaremos mejor “string.Format”, esta función no solo nos servira para unir cadenas sino tambien para formatearlas.

Un ejemplo.
```c#
string.Format("{0} {1}", "Hola", " mundo");
```
