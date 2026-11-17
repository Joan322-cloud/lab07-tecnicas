# Bitacora de tecnicas avanzadas

Laboratorio 07: Tecnicas Avanzadas de Prompting. Herramienta de IA usada: Claude

## Ejercicio 2: Zero-shot, one-shot y few-shot

| Tipo | Aciertos (de 5) | Formato de la respuesta | Todas con el mismo formato (Si/No) |
|------|-----------------|-------------------------|------------------------------------|
| Zero-shot | 5 | Tabla con numero, comentario y clasificacion, con explicaciones entre parentesis y un resumen al final | No |
| One-shot | 5 | Lista "comentario → etiqueta", pero el neutral vino con una explicacion | No |
| Few-shot | 5 | Solo la etiqueta, una por linea, sin explicaciones | Si |

- En zero-shot acerto todo, pero respondio como quiso: hizo una tabla, puso explicaciones en algunos y uso "Neutro" en vez de "Neutral".
- En few-shot respondio solo con las etiquetas y todas iguales. No repitio el texto de cada comentario, creo que porque en mi prompt quedaron dos comentarios en la misma linea.

## Ejercicio 3: Chain of Thought

| Pedido | Respuesta de la IA | Muestra los pasos (Si/No) | Correcta (Si/No) |
|--------|--------------------|---------------------------|------------------|
| Directo | 318.6 | No | Si |
| Paso a paso | S/ 318,60 (descuento: 90, con IGV: 106,20, por 3 unidades: 318,60, y lo comprobo por otro camino) | Si | Si |

Las dos respuestas fueron correctas y coinciden con lo que calcule con la calculadora: 120 × 0,75 = 90, 90 × 1,18 = 106,20 y 106,20 × 3 = 318,60.
Ver el razonamiento sirve porque puedo revisar cada calculo y, si hubiera un error, sabria exactamente en que paso esta. Con la respuesta directa solo tengo que creerle al numero.

## Ejercicio 4: Role prompting

| Version | Vocabulario (sencillo/tecnico) | Usa ejemplos o codigo | A quien le sirve mas |
|---------|--------------------------------|-----------------------|----------------------|
| A. Sin rol | Intermedio | Si, la caja con etiqueta y codigo en Python | A alguien que ya sabe un poco |
| B. Rol docente | Sencillo | Si, la caja con etiqueta y ejemplos cortos en Python | A alguien que nunca programo |
| C. Rol senior | Tecnico | Si, codigo en Java con tipos de datos | A un programador que ya trabaja |

- La version B explico todo como una caja con etiqueta y aclaro que el signo = no es una igualdad de matematicas.
- La version C hablo de tipado estatico, tipos primitivos y referencias, el heap y el alcance (scope), con codigo Java y consejos de nombres como camelCase.

## Ejercicio 5: Descomposicion

Pedido de una sola vez:

```text
Crea un sistema de inventario para una tienda.
```

- Hizo una aplicacion web en una pagina, no en Java, con productos, movimientos y alertas. Fue algo grande y general que yo no podia revisar por partes.

Pedido por pasos (en el mismo chat):

- **Paso 1:** me dio los 5 requisitos: gestion de productos, control de stock con entradas y salidas, alertas de stock minimo, busqueda y reportes, y guardar los datos en una base de datos.
- **Paso 2:** diseno las clases Producto, Categoria, Proveedor, MovimientoInventario, Usuario y AlertaStock, cada una con sus atributos y su tipo de dato.
- **Paso 3:** escribio la clase Producto con sus 11 atributos, un constructor y los get y set. Aviso que necesita las clases Categoria y Proveedor para compilar.
- **Paso 4:** propuso 3 mejoras: validar los datos en el constructor y los setters, mover la logica del stock a la clase en vez de usar setStockActual, y agregar equals, hashCode y toString.

Comparacion: por pasos el resultado fue en Java y cada parte salio de la anterior (los requisitos, luego las clases y luego el codigo). De una sola vez la IA decidio todo sola e hizo una pagina web.

## Ejercicio 6: Prompt estructurado y autocritica

- **Prompt basico** ("Dame casos de prueba para un login."): me dio una lista muy larga separada por categorias (funcionales, negativos, validacion, seguridad, recuperacion y login social). No era lo que necesitaba y el bloqueo solo aparecia como "bloqueo temporal o CAPTCHA".
- **Prompt estructurado:** me dio exactamente 6 casos en una tabla con las 4 columnas, incluido el bloqueo al tercer intento (CP-03), la contrasena correcta con la cuenta ya bloqueada (CP-04) y el reinicio del contador (CP-05).
- **Autocritica:** la IA reconocio que los campos vacios y el correo sin @ estaban juntos en un solo caso (CP-06) y que faltaba la contrasena con espacios. Separo el CP-06 y agrego del CP-07 al CP-14.

| Que revisar | Cumple (Si / No) |
|-------------|------------------|
| ¿Tiene las 4 columnas pedidas? | Si |
| ¿Incluye el bloqueo despues de 3 intentos? | Si |
| ¿Incluye casos con campos vacios? | Si |
| ¿Indica que casos agrego en la autocritica? | Si |
| ¿Hay algun caso repetido o que no tenga sentido? | No |

Nota: los casos CP-11 y CP-12 (espacios al inicio o al final) dependen de si el sistema recorta los espacios; la misma IA dijo que hay que confirmarlo.

```text
<rol>Actua como analista de pruebas de software.</rol>
<contexto>Login web con correo y contrasena. La cuenta se bloquea despues de 3 intentos fallidos.</contexto>
<tarea>Piensa paso a paso que puede fallar y escribe 6 casos de prueba.</tarea>
<formato>Tabla con las columnas: ID, escenario, datos de entrada, resultado esperado.</formato>

Revisa tu tabla: faltan casos limite como campos vacios, correo sin @ o contrasena con espacios? Agrega los que falten e indica cuales agregaste.
```