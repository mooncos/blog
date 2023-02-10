---
title: "Menos es más: Las ventajas de los lenguajes de programación tersos y funcionales"
date: 2023-02-09T10:26:41Z
draft: true
summary: "Los lenguajes de programación tersos, que normalmente incluyen características de programación funcional, pueden simplificar la comprensión y el desarrollo de código en tus proyectos. En este artículo se explora la refactorización de código de un problema para reducir el número de líneas de código en un 80%"
cover:
    image: "algo.jpg"
    alt: "Disipador de calor de una CPU con luz morada y azul espectral"
    relative: false
showtoc: true
tags:
    - programación funcional
    - c
    - rust
    - haskell
    - lenguajes de programación
categories:
    - programación
math: true
---
Según mi [WakaTime](https://wakatime.com/), paso casi la mitad del tiempo que uso mi ordenador con un editor de código abierto (probablemente VS Code), un 30% diseñando circuitos electrónicos y PCBs en [Altium Designer](https://www.altium.com/es/altium-designer) (relacionado con mi trabajo) y el restante usando Internet en general o usando [GoodNotes](https://www.goodnotes.com/).

Si te dedicas a algo relacionado con desarrollo software o hardware o incluso modelos de IA, seguro que sabes de lo que hablo...

El problema de base es que la mayor parte del tiempo no estamos escribiendo código nuevo, sino que estamos haciendo una de estas tres cosas:
1. Leyendo código ya escrito de otras personas o código tuyo anterior.
2. Pensando como traducir en código el algoritmo/proceso/modelo que necesitas implementar.
3. Solucionando bugs que pudiseses tener o probando código que recién has escrito.

Me atrevería a decir que estas tres cosas son las que andamos haciendo el **80% del tiempo** y el 20% del tiempo lo pasas realmente editando código como tal.

Con el tiempo me he dado cuenta ---y al parecer también [cientos](https://softwareengineering.stackexchange.com/questions/299796) [de](https://www.reddit.com/r/learnprogramming/comments/yb2pb4/is_it_normal_to_spend_so_many_hours_trying_to/) [miles](https://www.reddit.com/r/Python/comments/zkzqrq/how_much_time_do_you_spend_debugging/) [de](https://www.quora.com/How-can-I-code-more-efficiently-and-spend-less-time-debugging) [desarrolladores](https://news.ycombinator.com/item?id=11729806) [más](https://www.quora.com/It-is-true-that-developers-spend-most-of-their-time-reading-code-than-writing-code)--- de que esto es bastante ineficiente y que el buen código debería ser posible entenderlo en una, o como mucho, un par de pasadas.

Afortunadamente a lo largo del tiempo diversas personas con este problema, y que además tienen la capacidad de idear lenguajes de programación, concibieron la [programación funcional](https://en.wikipedia.org/wiki/Functional_programming).

## La programación funcional y su *concisa expresividad*

La programación funcional ofrece un paradigma muy distinto a la programación secuencial o imperativa. Este último es el clásico, y se basa en el hecho de que el estado de un programa (o los datos que maneja) se van modificando según la ejecución de unas instrucciones o sentencias, ejecutando una variación esencial de los datos por instrucción. Nada es más ilustrativo que ver este concepto con un ejemplo. Para ello vamos a definir un problema simple a resolver:

> Dados dos números enteros $a$ y $b$, calcular la suma de los enteros pares comprendidos entre $a$ y $b$, ambos incluidos, si y sólo si $b \ge a$; de lo contrario el resultado es $0$.

Consideraremos que el $0$ es par.

Veamos primero unos ejemplos de lo que queremos conseguir. El *input* se muestra a la izquierda (en el orden $a$, $b$) y el resultado que se espera a la derecha:
```txt
5, 12 → 36
0, 4 → 6
5, 3 → 0
```
Para visualizar mejor el problema, expandiremos estos ejemplos con la lista de números enteros que se encuentran entre $a$ y $b$ con ambos incluidos y posteriormente la lista de sólo los enteros pares:
```txt
5, 12 → [5, 6, 7, 8, 9, 10, 11, 12] → [6, 8, 10, 12] → 36
0, 4 → [0, 1, 2, 3, 4] → [0, 2, 4] → 6
5, 3 → No cumple condición b >= a → 0
```

Como se puede ver, es un problema sencillo de resolver pero que nos permitirá explorar las diferencias entre programación funcional y secuencial.

Para comenzar vamos a realizar un algoritmo en pseudocódigo para resolver este problema con el paradigma clásico de programación imperativa:

```txt
recoger enteros a y b en variables a y b
si b >= a
    definir suma = 0
    por cada entero i de a hasta b incluidos
        si i divisible entre 2
            añadir a suma i
    devolver suma
por el contrario
    devolver 0
```
Okay, de momento todo bien.


Ahora exploraremos la utilidad de la programación funcional para resolver este problema de manera más sencilla. La programación funcional se basa en el concepto de [composición de funciones](https://es.wikipedia.org/wiki/Funci%C3%B3n_compuesta) del álgebra funcional. Este concepto está relacionado con formar nuevas funciones basadas en el resultado de una función anterior, o lo que es lo mismo, expandir una función con las operaciones algebraicas de otra. Formalmente, dadas dos funciones:
$${\displaystyle {\begin{array}{rccl}f:&X&\longrightarrow &Y \newline &x&\longmapsto &y=f(x)\end{array}}}$$
y
$${\displaystyle {\begin{array}{rccl}g:&Y&\longrightarrow &Z \newline &y&\longmapsto &z=g(y)\end{array}}}$$
donde la imagen de $f$ está contenida en el dominio de $g$, entonces se define la *composición de $f$ sobre $g$* tal que:
$${\displaystyle {\begin{array}{rccl}(g\circ f):&X&\longrightarrow &Z \newline &x&\longmapsto &z=g(f(x))\end{array}}}$$
**En palabras simples, se aplica $f$ a $x$ y sobre este resultado se aplica $g$.**

Si pensamos en un algoritmo de tipo funcional para aplicarlo a nuestro problema, podría parecerse al siguiente pseudocódigo:
```txt
recoger enteros a y b en variables a y b
suma(seleccionar_elementos_pares(rango_inclusivo(a, b)))
```
Considerando que la función `rango_inclusivo(inicio, fin)` devuelve una lista de un elemento único `[0]` si no se cumple que `fin >= inicio`.

Como se puede apreciar, el algoritmo es mucho más declarativo y conciso ---es decir *terso*--- y las operaciones de dicho algoritmo son mucho más fáciles de visualizar de un vistazo.

Pero es más, los lenguajes con soporte de programación funcional normalmente incluyen optimizaciones en el código ensamblador generado al compilarse o ejecutarse que reducen el tiempo de ejecución porque utilizan algoritmos y mecanismos de procesamiento más optimizados. Si bien es cierto que aunque no incluyeran estos beneficios ocasionales en tiempo de cómputo, el beneficio que se obtiene por la claridad y concisión es más que bienvenido.

De hecho, los lenguajes de tipo funcional están siendo cada vez más utilizados en aplicaciones que requieren procesado de datos muy elevado como puede ser en Inteligencia Artificial (un ejemplo de ello es la composición de modelos de Deep Learning a partir de capas en [TensorFlow](https://keras.io/guides/making_new_layers_and_models_via_subclassing/)), en servidores que procesan peticiones o computan datos de forma masiva (por ejemplo el uso de [Erlang](https://www.erlang.org/) o su primo [Elixir](https://elixir-lang.org/) en [servidores y aplicaciones](https://medium.com/@obvyyyyes/the-emergence-of-scalable-functional-server-languages-65ca9068e94a)) o por ejemplo en [sistemas de comunicaciones en tiempo real](https://elixir-lang.org/blog/2020/10/08/real-time-communication-at-scale-with-elixir-at-discord/).

Continuemos con nuestro problema para intentar implementarlo en la realidad usando lenguajes de programación que tengan soporte para programación funcional.

## ¿Y cómo narices programo *funcionalmente*?

Para resolver el problema vamos a hacer uso de tres lenguajes utilizados en la actualidad con diversas características:
- [**C**](https://en.cppreference.com/w/c/language). El ancestro de los ancestros de los ancestros de todos los lenguajes de programación actuales. Probablemente el compilador o intérprete de todos los lenguajes de programación existentes están programados en C. No tiene soporte para programación funcional.
- [**Haskell**](https://www.haskell.org/). Un lenguaje puramente funcional que exprime y aprovecha al máximo las maravillas de dicho paradigma de programación.
- [**Rust**](https://www.rust-lang.org/). Un lenguaje con tipado estático que tiene soporte tanto para programación imperativa (la clásica) como para programación funcional. Mucha gente (me incluyo) lo considera el futuro sustituto a C por su seguridad de memoria y sus capacidades de programación flexibles.

En todos ellos definiremos una función con el identificador `calculate` y que tome dos argumentos `bottom` y `top`, que resuelva nuestro problema.

Para establecer una métrica de concisión o *tersidad* utilizaremos las SLOC (Significant Lines of Code) que tenga la función que implementemos en cada lenguaje. Las SLOC son las líneas de código que tienen uso útil, es decir, las que aportan al funcionamiento del código. Se computan del total de líneas de código quitando aquellas que sirvan para los delimitadores (los corchetes) o que estén en blanco.

### Implementación en C

Veamos como implementaríamos este algoritmo en *C*, de manera secuencial o imperativa, basándonos en nuestro pseudocódigo anterior:
```c
int calculate(int bottom, int top) 
{
    if (top >= bottom) 
    {
        int sum = 0;

        for (int number = bottom; number <= top; number++) 
        {
            if (number % 2 == 0) 
            {
                sum += number;
            }
        }

        return sum;
    } 
    else 
    {
        return 0;
    }
}
```
Este código tiene **16 SLOC**. Se puede observar lo muy *verboso* que es, pero sin embargo es muy expresivo, por lo que se puede ver paso a paso cómo se modifica la suma y el resultado que se muestra. De hecho, a la programación secuencial normalmente se le llama expresiva haciendo alusión a este nivel de detalle. Pero bien es cierto que el detalle a veces es excesivo y esta implementación es un ejemplo de ello. Para quien tenga experiencia en programación, comprenderá que implementar este algoritmo es sencillo. 

### Implementación en Haskell

Ahora exporaremos soluciones implementadas en lenguajes que tienen soporte para programación funcional. Normalmente el desarrollo algoritmos funcionales se realiza a partir de destilar un algoritmo previamente secuencial y expresarlo en forma de composición de resultados.

Uno de los lenguajes funcionales por excelencia que además es **puramente** funcional es *Haskell*. Primero veremos la implementación como tal para luego comentar y analizar la sintaxis de *Haskell*:

```haskell
calculate :: Int -> Int -> Int
calculate bottom top = sum $ filter even [bottom..top]
```
Este resultado tiene la increíble cantidad de **2 SLOC**, una reducción de casi el 86% con respecto a la implementación en *C*. Observemos qué se realiza en cada línea:

**Línea 1**
```haskell
calculate :: Int -> Int -> Int
```
Necesitamos declarar una función que tome dos argumentos y devuelva un entero ---la suma---. Haskell tiene una forma específica de tratar funciones al ser un lenguaje puramente funcional. En un inicio podríamos intentar declarar una función con varios argumentos como:
```haskell
calculate :: (Int x Int) -> Int
```
En este caso la función `calculate` tomaría dos argumentos de tipo entero (en nuestro caso serían `bottom` y `top`) y devuelve un entero. Sin embargo, Haskell es un lenguaje que se basa en programación funcional y por lo tanto en composición de funciones, si queremos componer la función `calculate` con otra que por ejemplo multiplique un entero por 2 (llamémosla `double`), necesitaríamos recordar la cantidad de argumentos que cada función toma y eso dificultaría la composición de funciones.

En este caso tendríamos que recordar que `calculate` toma dos argumentos y si queremos multiplicar antes con `double` el valor de los argumentos, tendríamos que componer de una forma tal que así:
```haskell
double_for_calculate :: (Int x Int) -> (Int x Int)
```
para que la salida de `double` sea compatible con `calculate`. **¡Entonces tendríamos que definir la función `double` para cada tipo de salida!**

Haskell soluciona esto expresando la declaración de funciones de varios argumentos en forma [*curry*](https://www.cmi.ac.in/~madhavan/courses/pl2009/lecturenotes/lecture-notes/node69.html). Esta forma trata funciones de varios parámetros como varias funciones (una composición de funciones) de un parámetro, que se van utilizando en el orden en el que se consumen según la definición de la función. El nombre se debe al matemático [Haskell Curry](https://es.wikipedia.org/wiki/Haskell_Curry), quien ideó este concepto y del que también se inspira el nombre del lenguaje.

De esta forma la declaración de la función queda como en la línea 1:
```haskell
calculate :: Int -> Int -> Int
```
En donde se utilizará primero un argumento, luego otro y finalmente devolverá un entero.

**Línea 2**
```haskell
calculate bottom top = sum $ filter even [bottom..top]
```
Esta es la definición de la función. Primero se especifica el nombre de la función que se define y luego el conjunto de argumentos que serán procesados. *Haskell* permite también varias definiciones de una misma función dependiendo de los argumentos que se usen o especifiquen, pero [esto es algo más complejo fuera del objetivo de este post](https://www.cmi.ac.in/~madhavan/courses/pl2009/lecturenotes/lecture-notes/node70.html).

En *Haskell* la apliación de funciones sobre variables o valores se hace con la notación espacio:
```haskell
sort "julie"
"eijlu"
```

Recordemos la composición de funciones y la notación algebraica:
$$(g\circ f)(x) = g(f(x)) $$
Se observa cómo las funciones se aplican por notación de derecha a izquierda, es decir, primero se aplica $f$ y luego $g$. Esto se denomina asociatividad derecha, es decir: la función especificada a la derecha tiene precedencia asociativa antes que la de la izquierda. *Haskell* por defecto tiene asociatividad izquierda, pero incluye un operador `$` que convierte la asociatividad izquierda en derecha y le da la menor prioridad posible (se ejecuta al final de todas).

Mejor verlo con un ejemplo utilizando dos funciones. Usaremos la función `sort` de antes y la función `++` que es una función infijo (los argumentos están a ambos lados de la función, como el infijo de suma de matemáticas `2 + 1` = 3) que concatena las strings que se encuentren a ambos lados.
```haskell
sort "julie" ++ "moronuki"
"eijlumoronuki"

sort $ "julie" ++ "moronuki"
"eiijklmnooruu"
```
Al utilizar el operador `$` que tiene prioridad mínima y asociatividad derecha, se ejecuta al final del todo, cuando todo lo que está a la derecha ya se ha ejecutado y el argumento es el resultado de todo lo que se encuentra a su derecha. De lo contrario, primero se ejecuta lo de la izquierda y se ejecuta con prioridad máxima. Un resultado similar al último caso se puede obtener usando paréntesis:
```haskell
sort ("julie" ++ "moronuki")
"eiijklmnooruu"
```
que también sirven para denotar preferencia en la asociatividad en *Haskell* al igual que en el álgebra.

Entonces volviendo a nuestra definición de la función:
```haskell
calculate bottom top = sum $ filter even [bottom..top]
```
Y recordamos nuestro pseudocódigo:
```txt
suma(seleccionar_elementos_pares(rango_inclusivo(bottom, top)))
```
Lo primero que se hace es generar un array que comience en `bottom` y acabe en `top`. Esto en *Haskell* se consigue con la notación `[bottom..top]`. Ya tenemos el componente `rango_inclusivo(bottom, top)` que necesitamos en nuestro algoritmo. Renombremos este array en nuestra mente a algo como *r*. Tenemos ahora algo así:
```txt
suma(seleccionar_elementos_pares(r))
```
Para seleccionar elementos que cumplan cierta condición dentro de un iterable (algo que se puede iterar, recorrer; como un array o una lista) se suele aplicar el concepto de *filtrar*. Esta operación consiste en modificar una lista que se proporciona como argumento, dejando sólo los elementos que cumplan cierta condición. En *Haskell* existe una función que se denomina [`filter`](http://zvon.org/other/haskell/Outputprelude/filter_f.html), que toma dos argumentos: el primero es una función que debe devolver un booleano y el segundo un iterable al que aplicar el filtrado. Convenientemente *Haskell* tiene una función que encapsula la comprobación de `elemento % 2 == 0` de *C* para comprobar si un entero es par. Esta función es [`even`](http://zvon.org/other/haskell/Outputprelude/even_f.html) y toma un sólo parámetro, el entero a comprobar. De esta forma podemos filtrar el rango *r* obtenido antes de la forma:
```haskell
filter even r
```
Al array resultante de este filtrado lo llamaremos en nuestra mente *r_pares*.
Por último tenemos algo así:
```txt
suma(r_pares)
```
Lo último que queda es sumar todos los elementos del array filtrado. En *Haskell* hay una función que realiza esto y es [`sum`](http://zvon.org/other/haskell/Outputprelude/sum_f.html), que toma un iterable (array) como argumento y devueve la suma. En nuestro caso , la suma se debe realizar sólo sobre los números pares (los ya filtrados) y por lo tanto esta operación deberá hacerse al final de todo y una vez todo el filtrado se haya realizado. Por ello es necesario utilizar el operador `$` para la suma, pues `sum` debe esperar a que se calcule todo el nuevo array para sumar los elementos. Esto se consigue haciendo:
```haskell
sum $ r_pares
```
Componiendo una a una las partes del código y aplicaciones de funciones que hemos ido haciendo, se obtiene nuestra segunda línea de código *Haskell*:
```haskell
calculate bottom top = sum $ filter even [bottom..top]
```
¡Y realmente ya estaría! Ahora nuestra función se podrá usar en nuestro programa *Haskell* siempre que queramos de la forma `calculate 5 12`, por ejemplo.

### Implementación en Rust

*Rust* es un lenguaje de programación con características tanto de programación imperativa como de programación funcional. Vamos a implementar en *Rust* el algoritmo de nuestro pseudocódigo funcional para resolver el problema y después comentaremos dicha implementación:
```rust
fn calculate(bottom: i32, top: i32) -> i32 {
    (bottom..=top).filter(|e| e % 2 == 0).sum()
}
```
Esta solución tiene **2 SLOC**, igual que la solución implementada en *Haskell*. Lo primero que observamos es que la sintaxis es muy similar a *C* en el estilo de declaración y definición de la función con sus argumentos. Vamos línea a línea:

*Línea 1*
```rust
fn calculate(bottom: i32, top: i32) -> i32
```
En esta línea declaramos la función con los parámteros que toma, el tipo de cada parámetro y el tipo del resultado que devuelve la función. Se puede ver cómo es muy similar a *C*. En *Rust* las funciones se declaran con la palabra reservada `fn` seguido del nombre de la función, los parámetros que toma separados por comas y con el tipo especificado después de dos puntos y el tipo que devuelve después de una flecha `->`. *Rust* también tiene funcionalidades de [*traits*](https://doc.rust-lang.org/book/ch10-02-traits.html) y [*lifetimes*](https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/lifetimes.html) que se pueden especificar en las declaraciones de funciones, pero que están fuera del objetivo de este post. Nos mantendremos en los principios básicos del lenguaje.

El tipo entero con signo de *Rust* por defecto es `i32`, un entero con signo de 32 bits. Usamos este tipo para todos los argumentos de la función y para el tipo de resultado que devuelve. Entre corchetes, se expresa entonces la definición de la función (lo que hace).

*Línea 2*
```rust
(bottom..=top).filter(|e| e % 2 == 0).sum()
```

La gente que esté familiarizada con *JavaScript* le resultará familiar este tipo de componer y expresar funciones. En *Rust* la programación funcional se implementa *encadenando* funciones, es decir, aplicando una nueva función directamente al resultado de otra en forma de postfijo. Esto quiere decir que algo del estilo `h(g(f(x)))` se expresa de la forma `x.f().g().h()`.

Recordando nuestro pseudocódigo:
```txt
suma(seleccionar_elementos_pares(rango_inclusivo(bottom, top)))
```
Lo reformateamos para expresarlo en la manera encadenada de componer funciones:
```txt
rango_inclusivo(bottom, top).seleccionar_elementos_pares().suma()
```
Lo primero que necesitamos hacer es generar el array de enteros entre bottom y top que incluya a ambos. Esto se puede hacer en *Rust* con la expresión `(bottom..=top)`. Por defecto, los arrays que se crean en *Rust* con la notación `..` similar a *Haskell* no incluyen el elemento final (igual que los rangos en Python, por ejemplo), es decir, se genera un array `[bottom, bottom+1, ..., top-1]`. Para incluir el último elemento, se añade un `=` después del último punto. Se esta forma conseguimos el rango adecuado. Si `top < bottom`, entonces esta expresión devuelve un array vacío `[]`. Al igual que el ejemplo de *Haskell*, recordaremos en nuestra cabeza este resultado como *r*.
Tenemos entonces el pseudocódigo restante:
```txt
r.seleccionar_elementos_pares().suma()
```

Para seleccionar los elementos que son pares de un iterable se emplea el concepto de filtrar (al igual que *Haskell*). *Rust* incluye también una función particular para este fin [`filter(function)`](https://doc.rust-lang.org/std/iter/struct.Filter.html) que toma como argumento una función para filtrar que devuelve un booleano que es la que se aplica a cada elemento del iterable (muy similar a *Haskell*). *Rust* sin embargo, no tiene una función conveniente como el `even` de *Haskell* por lo que debemos pasar como argumento una función nuestra que implemente la comprobación `elemento % 2 == 0` de *C*.

Al igual que *JavaScript*, *Rust* tiene maneras de definir una función anónima, es decir una función expresada concisamente para su uso como argumentos a este tipo de funciones que solicitan una función. En *Rust* estas funciones anónimas se denominan [*closures*](https://doc.rust-lang.org/book/ch13-01-closures.html) y tienen la sintaxis `|argumentos| cuerpo`. Podemos ver que son muy similares a las funciones [*arrow*](https://developer.mozilla.org/en-US/docs/Web/*JavaScript*/Reference/Functions/Arrow_functions) de *JavaScript* `(argumentos) => cuerpo `. En este caso definimos una *closure* que tome un elemento como argumento y que devuelva `true` si es múltiplo de 2 o `false` en caso contrario: `|e| e % 2 == 0`. No es necesario poner `return` en los cuerpos de funciones de *Rust* si el último elemento es una expresión con un resultado (igual que en *JavaScript*). Al resultado de esta aplicación del filtrado y por lo tanto al array filtrado resultante lo llamaremos *r_pares*, como en la implementación anterior. El pseudocódigo queda así:
```txt
r_pares.suma()
```
Ya sólo nos queda sumar los elementos de este array. *Rust* muy convenientenemente incluye una función en la librería por defecto `sum()` que aplicada sobre un tipo iterable, devuelve la suma de los elementos que lo componen (en nuestro caso, suma los enteros de un array). así que simplemente encadenando esta función es suficiente.

Por último, en *Rust* se puede omitir el `return` si se quiere devolver una expresión quitando el `;` al final de la última:
`return 2 + 1;` es igual a `2 + 1`.

Finalmente, obtenemos la segunda línea de nuestra implementación en *Rust* componiendo cada una de las partes:
```rust
(bottom..=top).filter(|e| e % 2 == 0).sum()
```

¡Y ya lo tenemos! ¿A qué es muy sencillo? En nuestro programa *Rust* podremos llamar a nuestra función de forma muy natural y similar a *C*: `calculate(5, 12);`.

## Análisis de la programación funcional

Es interesante explorar los beneficios en tiempo de ejecución u otras optimizaciones que nos proporciona este paradigma de programación. Para ello vamos a compilar un programa en los tres lenguajes donde se define la función y se realizan dos ejecuciones, una con parámetros `5, 12` y otra con parámetros `5, 3` (esta última debe dar `0`) y observar el número de instrucciones en ensamblador que se generan en cada caso. Los resultados se muestran en el siguiente gráfico:

{{< chart 100 auto >}}
{
    type: 'bar',
    data: {
  "labels": [
    "C (x86-64 clang 15.0.0 -O3)",
    "Haskell (x86-64 ghc 9.2.2 -O3)",
    "Rust (x86-64 rustc 1.67.0 -C opt-level=3)"
  ],
  "datasets": [
    {
      "label": "",
      "backgroundColor": "#575757",
      "fill": true,
      "data": [
        "139",
        "212",
        "50"
      ]
    }
  ]
},
    options: {
  "title": {
    "display": true,
    "text": "Instrucciones en ensamblador generadas por lenguaje de programación",
    "position": "top",
    "fontFamily": "sans-serif",
    "fontSize": 15,
    "fullWidth": false,
    "fontStyle": "bold"
  },
  "legend": {
    "display": false
  },
  "scales": {
    "yAxes": [
      {
        "ticks": {
          "beginAtZero": true
        }
      }
    ]
  }
}
}
{{< /chart >}}

También los dejo en formato de tabla, se han añadido enlaces a un compilador online para comprobar interactivamente el resultado y jugar con los programas:
| Lenguaje | Compilador                         | No. instrucciones | Enlace                          |
|----------|------------------------------------|-------------------|---------------------------------|
| C        | `x86-64 clang 15.0.0 -O3`            | 139               | [source.c](https://godbolt.org/z/GPs6hvjYh) |
| Haskell  | `x86-64 ghc 9.2.2 -O3`               | 212               | [source.hs](https://godbolt.org/z/r85dxe9YG) |
| Rust     | `x86-64 rustc 1.67.0 -C opt-level=3` | 50                | [source.rs](https://godbolt.org/z/GWeqzbaTx) |

Se puede observar cómo *C* tiene un elevado número de instrucciones incluso con el nivel de optimización del compilador configurado al máximo `-O3`. Sin embargo, es *Haskell* el lenguaje que produce un programa con mayor número de instrucciones. Esto se debe a que en cada programa, *Haskell* linkea la librería estandar (llamada *Prelude*) independientemente del tamaño del archivo, esto provoca que para código muy pequeño sea algo ineficiente. Sin embargo, si observamos sólo las instrucciones relacionadas con la ejecución de la función, observamos que tiene 84 instrucciones, alrededor de la mitad que el programa en *C*.

No obstante, el claro vencedor en esta batalla es *Rust* que genera un ejecutable con un increíble **total de 50 instrucciones**, verdaderamente asombroso. El compilador de rust, *rustc*, es alardeado como muy muy eficiente y es un esfuerzo increíble de la comunidad opensource que desarrolla el lenguaje, está basado en [LLVM](https://llvm.org/), al igual que *clang*, pero está muy optimizado para sólo incluir en el programa las librerías enlazadas y funciones estrictamente necesarias en cada compilación.

*Rust* además tiene una [documentación](https://doc.rust-lang.org/beta/) increíble y un [tutorial para aprender el lenguaje en forma de libro](https://doc.rust-lang.org/stable/book/) que es muy llevadero. Estoy explorando y aprendiendo este lenguaje y la verdad que estoy muy sorprendido de las ventajas que tiene a pesar de su curva de aprendizaje difícil.

Si te has quedado con las ganas de explorar más la programación funcional y ver ejemplos de como implementarla en *Rust*, puedes leer más en [este maravilloso post de Sylvain Kerkour](https://kerkour.com/rust-functional-programming).

## Una cosa más...

La programación funcional está en mil sitios, también en lenguajes más populares como Python. Estate atentx para un próximo blog sobre cómo mejorar tu código aplicando conceptos de programación funcional en programas escritos en Python :)