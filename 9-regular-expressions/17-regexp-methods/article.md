# Métodos de RegExp y String

En este artículo vamos a abordar varios métodos que funcionan con expresiones regulares a fondo.

## str.match(regexp)

El método `str.match(regexp)` encuentra coincidencias para las expresiones regulares (`regexp`) en la cadena (`str`).

Tiene 3 modos:

1. Si la expresión regular (`regexp`) no tiene la bandera `pattern:g`, devuelve una lista con los grupos capturados y las propiedades `index` (posición de la coincidencia), `input` (cadena de entrada, igual a `str`):

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

2. Si la expresión regular (`regexp`) tiene la bandera `pattern:g`, devuelve una lista de todas las coincidencias como cadenas, sin capturar grupos y otros detalles.
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

El método `str.search(regexp)` devuelve la posición de la primera coincidencia o `-1` si no encuentra nada:

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

If there's no `pattern:g`, then `regexp.exec(str)` returns the first match exactly as  `str.match(regexp)`. This behavior doesn't bring anything new.

But if there's flag `pattern:g`, then:
- A call to `regexp.exec(str)` returns the first match and saves the position immediately after it in the property `regexp.lastIndex`.
- The next such call starts the search from position `regexp.lastIndex`, returns the next match and saves the position after it in `regexp.lastIndex`.
- ...And so on.
- If there are no matches, `regexp.exec` returns `null` and resets `regexp.lastIndex` to `0`.

So, repeated calls return all matches one after another, using property `regexp.lastIndex` to keep track of the current search position.

In the past, before the method `str.matchAll` was added to JavaScript, calls of `regexp.exec` were used in the loop to get all matches with groups:

```js run
let str = 'More about JavaScript at https://javascript.info';
let regexp = /javascript/ig;

let result;

while (result = regexp.exec(str)) {
  alert( `Found ${result[0]} at position ${result.index}` );
  // Se encontró JavaScript en la posición 11, luego
  // Se encontró javascript en la posición 33
}
```

This works now as well, although for newer browsers `str.matchAll` is usually more convenient.

**Podemos usar `regexp.exec` para buscar desde una posición dada configurando manualmente el `lastIndex`.**

Por ejemplo:

```js run
let str = 'Hola, mundo!';

let regexp = /\w+/g; // sin la bandera "g", la propiedad `lastIndex` es ignorada
regexp.lastIndex = 5; // buscar desde la 5ta posición (desde la coma)

alert( regexp.exec(str) ); // mundo
```

If the regexp has flag `pattern:y`, then the search will be performed exactly at the  position `regexp.lastIndex`, not any further.

Let's replace flag `pattern:g` with `pattern:y` in the example above. There will be no matches, as there's no word at position `5`:

```js run
let str = 'Hello, world!';

let regexp = /\w+/y;
regexp.lastIndex = 5; // buscar exactamente en la posición 5

alert( regexp.exec(str) ); // null
```

Es conveniente para situaciones cuando necesitamos "leer" algo de la cadena por una expresión regular en una posición exacta. That's convenient for situations when we need to "read" something from the string by a regexp at the exact position, no en otro lugar.

## regexp.test(str)

El método `regexp.test(str)` busca por una coincidencia y retorna `true/false` si existe.

For instance:

```js run
let str = "Yo amo JavaScript";

// these two tests do the same
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

````warn header="Same global regexp tested repeatedly on different sources may fail"
If we apply the same global regexp to different inputs, it may lead to wrong result, because `regexp.test` call advances `regexp.lastIndex` property, so the search in another string may start from non-zero position.

For instance, here we call `regexp.test` twice on the same text, and the second time fails:

```js run
let regexp = /javascript/g;  // (regexp just created: regexp.lastIndex=0)

alert( regexp.test("javascript") ); // true (regexp.lastIndex=10 now)
alert( regexp.test("javascript") ); // false
```

That's exactly because `regexp.lastIndex` is non-zero in the second test.

To work around that, we can set `regexp.lastIndex = 0` before each search. Or instead of calling methods on regexp, use string methods `str.match/search/...`, they don't use `lastIndex`.
````
