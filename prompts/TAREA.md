# Tarea: Mi prompt avanzado

Herramienta de IA usada: Claude. Cada version la envie en un chat nuevo.

## Tarea elegida

Explicar un error de compilacion de Java. Use el error que me salio en el Laboratorio 06 cuando copie el simulador de temperatura desde el Word: un comentario se partio en dos lineas y quedo un `return` de mas, y javac marco `unreachable statement`.

## Version 1: prompt basico

```text
Explica este error:
SimuladorTemperatura.java:8: error: unreachable statement
            prob[0] = 1.0; return prob; // puntaje
```

- **Tecnica:** ninguna, solo pegue el error (zero-shot).
- **Que paso:** la IA explico que es un error de compilacion y me dio una lista de causas posibles (un `return` antes, un `break` en un `switch` o un bucle infinito), cada una con un ejemplo. Al final me pidio que le pegara las lineas 1 a 10 del archivo.
- **Problema:** como no le di el codigo, no sabia cual de las causas era la mia. Tuve que adivinar yo.

## Version 2

```text
Actua como profesor de Java que ayuda a un estudiante de segundo ciclo. Al compilar mi simulador me sale este error:

SimuladorTemperatura.java:8: error: unreachable statement
            prob[0] = 1.0; return prob; // puntaje

Este es el metodo donde ocurre:

static double[] calcularProbabilidades(double t) {
    double[] prob = new double[puntajes.length];
    if (t <= 0) {
        return prob; // temperatura 0: siempre la opcion con mayor
        prob[0] = 1.0; return prob; // puntaje
    }
    double suma = 0;
    for (int i = 0; i < puntajes.length; i++) {
        prob[i] = Math.exp(puntajes[i] / t);
        suma += prob[i];
    }
    for (int i = 0; i < prob.length; i++) {
        prob[i] = prob[i] / suma;
    }
    return prob;
}

Explicame que significa el error y como lo corrijo.
```

- **Tecnica agregada:** role prompting (profesor de Java para un estudiante de segundo ciclo), y ademas le pase el codigo como contexto.
- **Por que:** en la v1 la IA no sabia que codigo tenia ni a quien le explicaba.
- **Que mejoro:** encontro el problema exacto: el `return prob;` de la linea 7 termina el metodo y la linea 8 nunca se ejecuta. Incluso adivino que el comentario se partio en dos. Pero la respuesta fue larga y desordenada: dio una correccion minima, otra "recomendada", consejos extra sobre `Math.exp` y una seccion "para recordar", todo mezclado.

## Version 3: prompt final

```text
<rol>Actua como profesor de Java que ayuda a un estudiante de segundo ciclo que recien aprende a leer errores del compilador.</rol>
<contexto>Copie desde un documento Word el metodo calcularProbabilidades de un simulador y al compilar con javac me salio este error:
SimuladorTemperatura.java:8: error: unreachable statement
            prob[0] = 1.0; return prob; // puntaje
Este es el metodo:
static double[] calcularProbabilidades(double t) {
    double[] prob = new double[puntajes.length];
    if (t <= 0) {
        return prob; // temperatura 0: siempre la opcion con mayor
        prob[0] = 1.0; return prob; // puntaje
    }
    double suma = 0;
    for (int i = 0; i < puntajes.length; i++) {
        prob[i] = Math.exp(puntajes[i] / t);
        suma += prob[i];
    }
    for (int i = 0; i < prob.length; i++) {
        prob[i] = prob[i] / suma;
    }
    return prob;
}
</contexto>
<tarea>Piensa paso a paso: primero explica que linea se ejecuta antes que la linea 8 y por que Java considera que la linea 8 nunca se ejecuta. Luego da la correccion.</tarea>
<formato>Responde con estas partes y en este orden, como en este ejemplo:
Error: missing return statement
Que significa: el metodo promete devolver un valor, pero hay un camino que termina sin return.
Por que pasa: el return esta dentro de un if y no hay otro return fuera de el.
Como se arregla: agrega un return al final del metodo.
Despues de esas partes, muestra el metodo corregido en un bloque de codigo java.</formato>
<revision>Antes de terminar, revisa que el metodo corregido compile y que con temperatura 0 la primera opcion tenga probabilidad 1.0. Si encuentras un problema en tu respuesta, corrigelo e indica que cambiaste.</revision>
```

- **Tecnicas agregadas:** prompt estructurado con etiquetas, chain of thought ("piensa paso a paso"), few-shot (un ejemplo con el formato de respuesta, usando otro error) y autocritica (la etiqueta `<revision>`).
- **Por que:** en la v2 el contenido estaba bien, pero el orden y el largo de la respuesta los decidio la IA.
- **Que mejoro:** respondio con las partes del ejemplo y en el mismo orden: Error, Que significa, Por que pasa (en 3 pasos: entra al `if`, se ejecuta el `return` de la linea 7, la linea 8 queda despues y nunca se alcanza) y Como se arregla (borrar el `return prob;` de la linea 7). Despues mostro el metodo corregido. En la revision compilo mi version original (salio el mismo error) y la corregida (compilo sin errores), probo que con temperatura 0 queda `[1.0, 0.0, 0.0]` y dijo que no encontro errores en su respuesta. Tambien aviso que `prob[0] = 1.0` solo funciona si el primer puntaje es el mas alto.

## Tecnicas usadas en el prompt final

| Tecnica | Parte del prompt final |
|---|---|
| Role prompting | `<rol>Actua como profesor de Java que ayuda a un estudiante de segundo ciclo que recien aprende a leer errores del compilador.</rol>` |
| Prompt estructurado | Las etiquetas `<rol>`, `<contexto>`, `<tarea>`, `<formato>` y `<revision>` |
| Chain of thought | `Piensa paso a paso: primero explica que linea se ejecuta antes que la linea 8...` |
| Few-shot | El ejemplo del formato con el error `missing return statement` |
| Autocritica | `<revision>Antes de terminar, revisa que el metodo corregido compile...</revision>` |

## Evaluacion del resultado

| Criterio | v1 | v2 | v3 |
|---|---|---|---|
| Encuentra la causa exacta de mi error | No | Si | Si |
| Explica el error con palabras para un estudiante | Si | Si | Si |
| Explica paso a paso por que la linea 8 nunca se ejecuta | No | Si | Si |
| Responde con el formato del ejemplo y en ese orden | No | No | Si |
| Muestra el metodo corregido completo | No | Si | Si |
| Comprueba que la correccion compile | No | No | Si |

## Por que elegi estas tecnicas

Elegi role prompting porque un error de compilacion se puede explicar muy tecnico o muy simple, y yo necesitaba que me lo explicaran como estudiante. Chain of thought sirve porque este error se entiende siguiendo el orden en que Java ejecuta las lineas. El ejemplo few-shot y las etiquetas me sirvieron para que la respuesta siempre tenga las mismas partes, asi puedo usar el mismo prompt con otros errores y comparar. Y la autocritica la agregue porque lo que mas me importa es que la correccion compile. No use descomposicion porque la tarea es corta y se resuelve en un solo pedido.