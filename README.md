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


