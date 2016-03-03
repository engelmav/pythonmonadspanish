ADVERTENCIA: Todas las frases que contienen un numero en parentesis requiren que verifiquemos sus traducciones idiomaticas al español. Esta traduccion no es completa.

# Monads(1), Parte 1: Un patrón de diseño(2)

El uso de los monads puede considerarse como un patrón de diseño que nos permite definir de nuevo cómo funciona la composición de funciones.  Antes de que definamos qué es un monad, desarrollemos nuestra intuición con algunos ejemplos motivadores.

## Ejemplo 1: Manejo(3) de Excepciones (o sea, "Maybe")

Para lanzarnos, usemos un subconjunto funcional del Python, parsa describir una función que computa la cociente de ``100`` y algun divisor. En el caso especial de que el divisor sea cero, devolvemos(4) ``None``:

```python
def divide100(divisor):
    if divisor == 0:
        return None
    else:
        return 100 / divisor
```

Escribamos una función para computar la raiz cuadrada(5) de un número. Si el número de entrada(6) es negativo, deolvemos ``None``:

```python
def sqrt(x):
    if x < 0:
        return None
    else:
        return x ** 0.5
```

Podemos componer estas funciones? Qué pasa si quisieramos computar algo como ``sqrt(divide100(sqrt(x)))``? Eso funcionaría, con tal que ``x`` sea positivo. Si acomodamos explícitamente los casos en que cualquier de esas funciones devuelva ``None``, el código se alarga(7) mucho:

```python
a = sqrt(x)
if a is not None:
    b = divide100(a)
    if b is not None:
        c = sqrt(b)
```

Te puedes imaginar lo tedio(8) que sería si tuvieramos que verificar, a mano, los posibles errores para todas nuestras llamadas de función(8). Quizás una solución sería volver a escribir ``divide100`` y ``sqrt`` tal que hagan el manejo de errores por su cuenta. Por ejemplo, quizás los modifiquemos así como lo siguiente:

```python
def composable_divide100(divisor):
    if divisor is None or divisor == 0:
        return None
    else:
        return 100 / divisor

def composable_sqrt(x):
    if x is None or x < 0:
        return None
    else:
        return x**0.5
```

Ahora podemos evaluar expresiones tales como ``composable_sqrt(composable_divide100(composable_sqrt(x)))``. Cuando x ``x <= 0``, la expresión entera evalua a None, como se esperaría.

En vez de modificar todas nuestras funciones para ver si existe ``None``, escribamos un función que envuelve a otras funciones (digamosle ``bind``) para hacer el manejo de errores para nosotros. Esta función acepta un parámetro (o un número o ``None``) y una función (tal como ``divide100`` o ``sqrt``) y aplica la función a ese parámetro. Si el parámetro es ``None``, nos saltamos la aplicación de la función y sólo devolvemos ``None``:

```python
def bind(x, f):
    if x is None:
        return None
    else:
        return f(x)
```

Ahora podemos componer estas funciones, como: ``bind(bind(bind(x, sqrt), divide100), sqrt)``

¡Genial! Ahora tenemos una manera de componer funciones numericales que puedan fallar. Acabas de imple,emtar el monad de ``Maybe`` de Haskell, lo cual es una forma simple del manejo de excepciones. Intentemos algunos ejemplos más complejos.


## Ejemplo 2: operaciones en vectores (alias "Lista" / "List")

Sabemos que. matemáticamente, números positivos tienen dos raices. Modifiquemos ``sqrt``para que devuelva una lista de valores:

```python
def sqrt(x):
    if x < 0:
        return []
    elif x == 0:
        return [0]
    else:
  return [x**0.5, -x**0.5]
```

Así que hay tres casos que tenemos que considerar. Si ``x`` es positivo, devolvemos sus raices cuadradas. Si ``x`` es ``0``, devolvemos ``[0]``. Si ``x`` es negativo, devolvemos una lista vacía.

¡Excelente! Nuestra función ``sqrt``  ahora tiene más sentido matemáticamente, por lo menos para los números reales. Pero tenemos el mismo problema como en Ejemplo 1 -- ya no se puede componer. No podemos simplemente computar ``sqrt(sqrt(x))``, porque la llamada interior de ``sqrt`` devuelve una lista, y la llada exterior espera un número. Como antes, necesitamos definir una función ``vins`` para ayudarnos con la composición.

```python
def bind(x, f): 
  return [j for i in x for j in f(i)]
```

Aquí, ``bind`` toma una lista de números ``x`` y una función ``f``. La comprehensión de listas de dos iteraciones se ve críptico -- lo puedes pensar así: Aplicamos ``f`` a cada valor en ``x``, lo cual nos da una lista de listas. Después aplanamos el resultado en una sola lista y la devolvemos. Ahora podemos computar las raices cuadradas de una lista de números, y después computar todas las raices cuadradas de los resultados:

```python
>>> bind(bind([5, 0, 3], sqrt), sqrt) 
[1.4953487812212205, -1.4953487812212205, 0, 1.3160740129524924, -1.3160740129524924]
```

Funciona! Pero nuestra meta original fue encontrar las raices cuadradas de *un* número. Podríamos siempre escribir ``bind)[x], sqrt) donde ``x`` es un número, pero quizás sería mejor utilizar una función para abstraer la representación de nuestro parámetro. Digamosle a esta función ``unit``:

```python
def unit(x):
  return [x]
```

Sip, esto simplemente pone un valor dentro de una lista. Puede que no se vea justificado ahora, pero escribiremos una versión más útil en el próximo ejemplo. Ahora no tenemos que meter nuestro parámetro dentro de una lista -- en vez de hacer eso, lo pasamos a ``unit``, lo cual lo convierte a su representación necesaria:

```python
>>> bind(bind(unit(4), sqrt), sqrt) 
[1.4142135623730951, -1.4142135623730951]
```

Excelente, ahora podemos inteligentemente componer las funciones que tal vez devuelvan varios valores! Eso es básicamente la monad ``List`` de Haskell.

## Ejemplo 3: depurar salida(9) (Alias "Escritor" / "Writer")

Suponga que tenemos unas funciones ``u`` y ``v``, y ambas funciones toman un número y devuelven un número. No importa realmente cuáles funciones son, así que definamoslas como tal:

```python
def u(x): 
  return x + 4 

def v(x): 
  return x * 2
```

No hay problema allí.  Aún podemos componerlas, así como ``u(v(x))``. ¿Qué pasaría si quisieramos imprimir información adicional junta con el valor normal que las funciones devuelven? Por ejemplo, suponga que quisieramos que cada funcion también devuelva una cadena de carácteres indicando que la función se ha llamado. Quizás modifiquemos ``u`` y ``v`` como tal:

```python
def verbose_u(x): 
  return (x + 4, '[verbose_u was called on ' + str(x) + ']') 

def verbose_v(x): 
  return (x * 2, '[verbose_v was called on ' + str(x) + ']')
```

Ahora tenemos el mismo problema de antes, hemos perdido la habilidad de componer las funciones: ``verbose_u(verbose_v))``. Pero ahora sabemmos que la solución es ``bind``:

```python
def bind(x, f): 
  result, output = f(x[0]) 
  return (result, x[1] + output)
```
