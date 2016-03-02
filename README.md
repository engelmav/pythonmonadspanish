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
