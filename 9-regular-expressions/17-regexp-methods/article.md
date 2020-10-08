# Métodos de RegExp y String

En este artículo vamos a abordar varios métodos que funcionan con expresiones regulares a fondo.

## str.match(regexp)

El método `str.match(regexp)` encuentra coincidencias para las expresiones regulares (`regexp`) en la cadena (`str`).

Tiene 3 modos:

1. Si la expresión regular (`regexp`) no tiene la bandera `pattern:g`, retorna una lista con los grupos capturados y las propiedades `index` (posición de la coincidencia), `input` (cadena de entrada, igual a `str`):

    ```js run
    let str = "Amo JavaScript";

    let result = str.match(/Java(Script)/);

    alert( result[0] );     // JavaScript (full match)
    alert( result[1] );     // Script (primer grupo capturado)
    alert( result.length ); // 2

    // Información adicional:
    alert( result.index );  // 0 (posición de la coincidencia)
    alert( result.input );  // Amo JavaScript (cadena de entrada)
    ```

2. Si la expresión regular (`regexp`) tiene la bandera `pattern:g`, retorna una lista de todas las coincidencias como cadenas, sin capturar grupos y otros detalles.
    ```js run
    let str = "Yo amo JavaScript";

    let result = str.match(/Java(Script)/g);

    alert( result[0] ); // JavaScript
    alert( result.length ); // 1
    ```

3. Si no hay coincidencias, no importa si tiene la bandera `pattern:g` o no, `null` es retornado.

    Esto es algo mmuy importante. Si no hay coincidencias, no vamos a obtener un array vacío, pero si un `null`. Es fácil cometer un error olvidandolo, ej.:

    ```js run
    let str = "Yo amo JavaScript";

    let result = str.match(/HTML/);

    alert(result); // null
    alert(result.length); // Error: Cannot read property 'length' of null
    ```

    Si queremos que el resultado sea una lista, podemos escribirlo así:

    ```js
    let result = str.match(regexp) || [];
    ```

## str.matchAll(regexp)

[recent browser="new"]

El método `str.matchAll(regexp)` es una variante ("nueva y mejorada") de `str.match`.

Es usado principalmente para buscar por todas las coincidencias con todo todos los grupos.

Hay 3 diferencias con `match`:

1. Retorna un objecto iterable con las coincidencias en lugar de una lista. Podemos convertirlo en una lista usando el método `Array.from`.
2. Cada coincidencia es retornada como una lista con los grupos capturados (el mismo formato de `str.match` sin la bandera `pattern:g`).
3. Si no hay resultados, no retorna `null`, pero si un objecto iterable vacío.

Ejemplo de uso:

```js run
let str = '<h1>Hola, mundo!</h1>';
let regexp = /<(.*?)>/g;

let matchAll = str.matchAll(regexp);

alert(matchAll); // [object RegExp String Iterator], no es una lista, pero si un objeto iterable

matchAll = Array.from(matchAll); // ahora es una lista

let firstMatch = matchAll[0];
alert( firstMatch[0] );  // <h1>
alert( firstMatch[1] );  // h1
alert( firstMatch.index );  // 0
alert( firstMatch.input );  // <h1>Hola, mundo!</h1>
```

Si usamos `for..of` para iterar todas las coincidencias de `matchAll`, no necesitamos `Array.from`.

## str.split(regexp|substr, limit)

Divide la cadena usando la expresión regular (o una sub-cadena) como delimitador.

Podemos usar `split` con cadenas, así:

```js run
alert('12-34-56'.split('-')) // lista de [12, 34, 56]
```

O también dividir una cadena usando una expresión regular de la misma forma:

```js run
alert('12, 34, 56'.split(/,\s*/)) // lista de [12, 34, 56]
```

## str.search(regexp)

El método `str.search(regexp)` retorna la posición de la primera coincidencia o `-1` si no encuentra nada:

```js run
let str = "Una gota de tinta puede hacer pensar a un millón";

alert( str.search( /ink/i ) ); // 10 (posición de la primera coincidencia)
```

**Limitación importante: solo `search` encuentra la primera coincidencia.**

Si necesitamos las posiciones de las demás coincidencias, deberíamos usar otros medios, como encontrar todos con `str.matchAll(regexp)`.

## str.replace(str|regexp, str|func)

Este es un método genérico para buscar y reemplazar, uno de los más útiles. La navaja suiza para buscar y reemplazar.  

Podemos usarlo sin expresiones regular, para buscar y reemplazar una sub-cadena:

```js run
// reemplazar guión por dos puntos
alert('12-34-56'.replace("-", ":")) // 12:34-56
```

Sin embargo hay una trampa:

**Cuando el primer argumento de `replace` es una cadena, solo reemplaza la primera coincidencia.**

Puedes ver eso en el ejemplo anterior: solo el primer `"-"` es reemplazado por `":"`.

Para encontrar todos los guiones, no necesitamos usar un cadena `"-"` sino una expresión regular `pattern:/-/g` con la bandera `pattern:g` obligatoria:

```js run
// reemplazar todos los guioes por dos puntos
alert( '12-34-56'.replace( *!*/-/g*/!*, ":" ) )  // 12:34:56
```

El segundo argumento es la cadena de reemplazo. Podemos usar caracteres especiales:

| Símbolos | Acción en la cadena de reemplazo |
|--------|--------|
|`$&`|inserta toda la coincidencia|
|<code>$&#096;</code>|inserta una parte de la cadena antes de la coincidencia|
|`$'`|inserta una parte de la cadena después de la coincidencia|
|`$n`|si `n` es un 1-2 dígito númerico, inserta el contenido del enésimo grupo capturado, para más detalles ver [](info:regexp-groups)|
|`$<nombre>`|inserta el contenido de los paréntesis con el `nombre` dado, para más detalles ver [](info:regexp-groups)|
|`$$`|inserts character `$` |

Por ejemplo:

```js run
let str = "John Smith";

// intercambiar el nombre con el apellido
alert(str.replace(/(john) (smith)/i, '$2, $1')) // Smith, John
```

**Para situaciones que requieran reemplazos "inteligentes", el segundo argumento puede ser una función.**

Puede ser llamado por cada coincidencia y el valor retornado puede ser insertado como un reemplazo.

La función es llamada con los siguientes argumentos `func(match, p1, p2, ..., pn, offset, input, groups)`:

1. `match` -- la coincidencia,
2. `p1, p2, ..., pn` -- contenido de los grupos capturados (si hay alguno),
3. `offset` -- posición de la coincidencia,
4. `input` -- la cadena de entrada,
5. `groups` -- un objecto con los grupos nombrados.

Si hay paréntesis en la expresión regular, entonces solo son 3 argumentos: `func(str, offset, input)`.

Por ejemplo, hacer mayúsculas todas las coincidencias:

```js run
let str = "html and css";

let result = str.replace(/html|css/gi, str => str.toUpperCase());

alert(result); // HTML and CSS
```

Reemplazar cada coincidencia por su posición en la cadena:

```js run
alert("Ho-Ho-ho".replace(/ho/gi, (match, offset) => offset)); // 0-3-6
```

En el ejemplo anterior hay dos paréntesis, entonces la función de reemplazo es llamada con 5 argumentos: el primero es toda la coincidencia, luego dos paréntesis, y después (no usado en el ejemplo) la posición de la coincidencia y la cadena de entrada:

```js run
let str = "John Smith";

let result = str.replace(/(\w+) (\w+)/, (match, name, surname) => `${surname}, ${name}`);

alert(result); // Smith, John
```

Si hay muchos grupos, es conveniente usar parámetros rest para acceder a ellos:

```js run
let str = "John Smith";

let result = str.replace(/(\w+) (\w+)/, (...match) => `${match[2]}, ${match[1]}`);

alert(result); // Smith, John
```

O, si estamos usando grupos nombrados, entonces el objecto `groups` con ellos es siempre el último, entonces podemos obtener algo como esto:

```js run
let str = "John Smith";

let result = str.replace(/(?<name>\w+) (?<surname>\w+)/, (...match) => {
  let groups = match.pop();

  return `${groups.surname}, ${groups.name}`;
});

alert(result); // Smith, John
```

Usando una función nos da todo el poder del reemplazo, porque obtiene toda la información de la coincidencia, teniendo acceso a variables externas y puede hacer todo.

## regexp.exec(str)

El método `regexp.exec(str)` retorna una coincidencia por expresión regular (`regexp`) en la cadena (`str`).  A diferencia de los metodos anteriores, se llama en una expresión regular en lugar de en una cadena.


Se comporta de manera diferente dependiendo de si la expresión regular tiene la bandera `pattern:g`.

Si no hay la bandera `pattern:g`, entonces `regexp.exec(str)` retorna la primera coindicencia igual que `str.match(regexp)`. Este comportamiento no trae nada nuevo.

Pero si hay la bandera `pattern:g`, entonces:
- Una llamda a `regexp.exec(str)` retorna la primera coincidencia and y guarda la posición inmediatamante después en `regexp.lastIndex`.
- La siguente llamada de la busqueda comienza desde la posición de `regexp.lastIndex`, retorna la siguiente coincidencia y guarda la posición inmediatamante después en `regexp.lastIndex`.
- ...y así consecutivamente.
- Si no hay coincidencias, `regexp.exec` retorna `null` y resetea `regexp.lastIndex` a `0`.

Entonces, repetidas llamadas todas las coincidencias una tras otra, usando la propiedad `regexp.lastIndex` para realizar el rastreo de la posición actual de la busqueda.

En el pasado, antes el método `str.matchAll` fue agregado a JavaScript, llamadas de `regexp.exec` se utilizaron en el ciclo para obtener todas las coincidencias con sus grupos:

```js run
let str = 'Más sobre JavaScript en https://javascript.info';
let regexp = /javascript/ig;

let result;

while (result = regexp.exec(str)) {
  alert( `Found ${result[0]} at position ${result.index}` );
  // Se encontró JavaScript en la posición 11, luego
  // Se encontró javascript en la posición 33
}
```

Esto también, aunque para navegadores modernos `str.matchAll` es lo más conveniente.

**Podemos usar `regexp.exec` para buscar desde una posición dada configurando manualmente el `lastIndex`.**

Por ejemplo:

```js run
let str = 'Hola, mundo!';

let regexp = /\w+/g; // sin la bandera "g", la propiedad `lastIndex` es ignorada
regexp.lastIndex = 5; // buscar desde la 5ta posición (desde la coma)

alert( regexp.exec(str) ); // mundo
```

Si la expresión regular tiene la bandera `pattern:y`, entonces la busqueda se realizará exactamente en la posición del `regexp.lastIndex`, no más adelante.

Vamos a reemplazar la bandera `pattern:g` con `pattern:y` en el ejemplo anterior. No habrá coincidencias, ya que no hay palabra en la posiicón `5`:

```js run
let str = 'Hello, world!';

let regexp = /\w+/y;
regexp.lastIndex = 5; // buscar exactamente en la posición 5

alert( regexp.exec(str) ); // null
```

Es conveniente para situaciones cuando necesitamos "leer" algo de la cadena por una expresión regular en una posición exacta. That's convenient for situations when we need to "read" something from the string by a regexp at the exact position, no en otro lugar.

## regexp.test(str)

El método `regexp.test(str)` busca por una coincidencia y retorna `true/false` si existe.

Por ejemplo:

```js run
let str = "Yo amo JavaScript";

// estas dos pruebas hacen lo mismo
alert( *!*/love/i*/!*.test(str) ); // true
alert( str.search(*!*/love/i*/!*) != -1 ); // true
```

Un ejemplo con respuesta negativa:

```js run
let str = "Bla-bla-bla";

alert( *!*/love/i*/!*.test(str) ); // false
alert( str.search(*!*/love/i*/!*) != -1 ); // false
```

Si la expresión regular tiene la bandera `pattern:g`, el método `regexp.test` busca la propiedad `regexp.lastIndex` y la actualiza, igual que `regexp.exec`.

Entonces podemos usarlo para buscar desde un posición dada:

```js run
let regexp = /love/gi;

let str = "Yo amo JavaScript";

// comienza la busqueda desde la posición 10:
regexp.lastIndex = 10;
alert( regexp.test(str) ); // false (sin coincidencia)
```

````warn header="La misma expresión regular probada (de manera global) repetidamente en diferentes lugares puede fallar"
Si nosotros aplicamos la misma expresión regular (de manera global) a diferentes entradas, puede causar resultados incorrectos, porque `regexp.test` anticipa las llamadas usando la propiedad `regexp.lastIndex`, por lo que la busqueda en otra cadena puede comenzar desde una posición distinta a cero.

Por ejemplo, podemos llamarlo aquí `regexp.test` dos veces en el mismo texto y en la segunda vez falla:

```js run
let regexp = /javascript/g;  // (expresión regular creada: regexp.lastIndex=0)

alert( regexp.test("javascript") ); // true (regexp.lastIndex=10 ahora)
alert( regexp.test("javascript") ); // false
```

Eso es exactamente porque `regexp.lastIndex` no es cero en la segunda prueba.

Para solucionarlo, podemos establecer `regexp.lastIndex = 0` antes de cada busqueda. O en lugar de llamar a los metodos en la expresión regular usar los metodos de cadena `str.match/search/...`, ellos no usan el `lastIndex`.
````
