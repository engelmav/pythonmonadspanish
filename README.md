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


## Ejemplo 2: operaciones en vectores (alias "Lista")

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


