---
title: "Raíz amarga es la del ensamblador; pero muy dulce su fruto"
date: 2023-04-04
draft: false
summary: "Entender cómo funcionan los compiladores nos puede ayudar a producir código más eficiente y compacto"
cover:
    image: "huellas.jpg"
    alt: "Huellas sobre la arena de la playa"
    relative: false
showtoc: true
tags:
    - ensamblador
    - compilador
    - assembly
    - c++
    - compiler optimizations
categories:
    - programación
math: true
---
Leer *desensamblado* ---que es como denomino al producto de una compilación justo antes de ser traducido a código máquina, es decir, en lenguaje *Assembly*--- es más parecido a seguir pistas que a leer un libro. Para leer un libro hay que conocer el idioma. En cambio, seguir pistas, aunque mejora con la destreza y la experiencia, requiere sobre todo **atención e imaginación**.

Como el buen [refranero español](https://elpais.com/elpais/2016/05/24/ciencia/1464081951_497823.html) ---fuente de inspiración para el título de este post--- dice, el esfuerzo de pensar y concentrarse en algo para ver qué esta pasando, tiene sus frutos. En este caso, aunque sea bien complicado y enrevesado el proceso de compilación y desensamblado de programas para ordenador, es útil conocer cómo funciona para extraer su máximo potencial.

La mayoría de las veces leemos desensamblado sólo para respondernos una simple pregunta: *¿está haciendo el compilador lo que queremos que haga?* Con tres sencillos ejercicios, voy a mostrarte que muchas veces tú también puedes responderte esta pregunta aunque no tengas conocimientos previos de ensamblador. En estos ejemplos uso C++ como lenguaje de programación, pero lo que intento mostrar es más o menos universal, así que no importa si programas en C o Swift, C# o Rust. Si compilas a cualquier tipo de código máquina, puedes beneficiarte de entender tu compilador.

Además, en los ejemplos de este post uso el [lenguaje ensamblador de sistemas x86-64](https://cs.brown.edu/courses/csci1260/spring-2021/lectures/x86-64-assembly-language-reference.html) (que utilizan los procesadores Intel y AMD), que es más universal y utilizado que el de, por ejemplo, ARM. 

Por último, a lo largo del post hay enlaces a notas al pie de página, te permitirán profundizar en algunos conceptos, si bien es cierto que no es necesario para comprender la esencia de lo que trata el post.

## Cálculos en tiempo de compilación

Cualquier compilador **que se haga respetar** se esfuerza por hacer que tu código máquina no sólo sea correcto, sino también rápido. Esto implica hacer el menor trabajo posible en tiempo de ejecución. A veces puede incluso realizar todo tipo de cálculos en tiempo de compilación, de modo que tu código máquina sólo contendrá la respuesta precalculada.

Por ejemplo, este código fuente define el número de bits en un byte y devuelve el tamaño de un `int` en bits.

```cpp
static int BITS_EN_UN_BYTE = 8;

int main() {
    return sizeof(int)*BITS_EN_UN_BYTE;
}
```

El compilador sabe el tamaño de un `int`. Supongamos que para la plataforma objetivo ---a la que programamos--- el tamaño es de 4 bytes. También fijamos explícitamente el número de bits de un byte. Dado que todo lo que queremos es una simple multiplicación, y ambos valores son conocidos durante la compilación, un compilador puede simplemente calcular el número resultante por sí mismo en lugar de generar el código que calcula el mismo número una y otra vez cada vez que se ejecuta.

Cuidao, que es importante recordar que un compilador puede o no ofrecer esta optimización. No es algo que esté garantizado en el [estándar de ANSI C](https://www.iso-9899.info/wiki/The_Standard).

Ahora observa dos posibles desensamblados para este código fuente y decide qué variante realiza el cálculo en tiempo de compilación y cuál no.

**Opción A**:
```nasm
BITS_EN_UN_BYTE:
  .long 8
main:
  mov eax, DWORD PTR BITS_EN_UN_BYTE[rip]
  cdqe
  sal eax, 2
  ret
```
**Opción B**:
```nasm
main:
  mov eax, 32
  ret
```

<details>
  <summary><strong>Solución y explicación</strong></summary>

En efecto, la **Opción B** lo hace.

En una plataforma de 32 bits el tamaño de `int` es de 4 bytes, que son 32 bits, que es exactamente el número que aparece en el código ensamblador de la **Opción B**. Puede que no sepas que una función entera normalmente devuelve su salida en `eax`[^eax], pero no importa. El caso es que el código de la **Opción B** ya tiene una respuesta incluida, así que incluso da igual.
</details>

[^eax]: `eax` es un registro. Hay bastantes registros, pero los más importantes para nosotros son los de propósito general, más concretamente `eax`, `ebx`, `ecx` y `edx`. Sus nombres son respectivamente: **a**cumulador, **b**ase, **c**ontador y **d**atos. No son necesariamente intercambiables. Puedes pensar en ellos como variables ultrarrápidas de tamaño conocido. Por ejemplo, `rax` tiene 64 bits. Los 32 bits inferiores de éste son accesibles con el nombre `eax`. Los 16 bits inferiores de la misma como `ax`, que a su vez consta de dos bytes `ah` y `al`. Todas estas son partes del mismo registro. Los registros no residen en la RAM, por lo que no se puede leer y escribir ningún registro por su dirección. Los corchetes suelen indicar manipulaciones de la dirección. `mov rax, dword ptr [BITS_IN_BYTE]` significa poner lo que esté en la dirección de `BITS_IN_BYTE` en el registro `rax` como una double word.

## Anidado de funciones

Llamar a una función implica cierta sobrecarga al preparar los datos de entrada en un orden determinado; luego iniciar la ejecución desde otra parte de la memoria; luego preparar los datos de salida; y luego retornar.

No es que sea demasiado lento, pero si sólo quieres llamar a una función una vez, no tienes por qué llamar a la función. Lo lógico es copiar o "anidar" el cuerpo de la función en el lugar desde el que se llama y saltarse todas las formalidades. A menudo, los compiladores se encargan de hacerlo sin que tengas que preocuparte.

Si el compilador usa esta optimización, este código:

```cpp
inline int cuadrado(int x)  {
    return x * x;
}


int main(int argc, char** argv)  {
    return cuadrado(argc);
}
```

Virtualmente se convierte en éste:

```cpp
// esto no es realmente código fuente, simplemente estoy ilustrando la idea
int main(int argc, char** argv)  {
    return argc * argc;
}
```

Ojo porque el estándar no asegura que todas las funciones marcadas como `inline` serán anidadas. Es más una sugerencia que una directiva.

Ahora observa estas dos variantes de desensamblado y elige aquella en la que la función se anida.

**Opción A**:
```nasm
main:
  imul edi, edi
  mov eax, edi
  ret
```
**Opción B**:
```nasm
cuadrado(int):
  imul edi, edi
  mov eax, edi
  ret
main:
  sub rsp, 8
  call cuadrado(int)
  add rsp, 8
  ret
```

<details>
  <summary><strong>Solución y explicación</strong></summary>

Elemental querido Watson, es la **Opción A**.

No tienes por qué saberlo pero la instrucción que se usa para llamar a una función se denomina `call`[^call]. Vemos que en el código de la **Opción A** ni siguiera hay mención de la función `cuadrado`, lo que nos hace intuir que la función se ha integrado directamente como una multiplicación. 
</details>

[^call]: `call` almacena un registro especial que apunta a la siguiente instrucción tras la llamada a la función en la pila y después fija su valor a la dirección donde se encuentra la función. Entonces, el procesador salta a ejecutar la función. Dentro de la función se usa `ret` para obtener una dirección almacenada en la pila de vuelta al registro, provocando al procesador regresar al punto donde previamente se había llamado a la función. La pila es una porción de memoria organizada de forma que el primero en entrar es el último en salir (LIFO), así que si, por ejemplo, haces `push` de `rax` y `rbx` ahí y luego haces `pop` de `rax` y `rbx`, estos dos registros se intercambiarán.

## Desenrollado de bucles

Al igual que ocurre con las llamadas a funciones, entrar en bucles conlleva cierta sobrecarga. Hay que incrementar el contador, compararlo con algún valor y volver al principio del bucle.

Los compiladores saben que en algunas situaciones es más eficaz desenrollar el bucle. Esto quiere decir que algún trozo de código se repetirá varias veces seguidas en lugar de andar liándose con la comparación del contador y saltando aquí y allí.

Veamos qué significa esto partiendo del código fuente que tenemos aquí:

```cpp
int main(int argc, char**) {
    int resultado = 1;
    for(int i = 0; i < 3; ++i)
        resultado *= argc;
    return resultado;
}
```

El compilador tiene todos los motivos para desenrollar este bucle tan simple, ¡pero ojo! Que también puede decidir no hacerlo.

¿Qué código desensamblado tiene el bucle desenrollado?

**Opción A**:
```nasm
main:
  mov eax, 1
  mov ecx, 3
.LBB0_1:
  imul eax, edi
  dec ecx
  jne .LBB0_1
  ret
```
**Opción B**:
```nasm
main:
  mov eax, edi
  imul eax, edi
  imul eax, edi
  ret
```

<details>
  <summary><strong>Solución y explicación</strong></summary>

Sí sí sí, es la **Opción B**.

Puede que no sepas que `j<*>` es la familia de instrucciones de salto[^jump]. Sin embargo, el código de la **Opción B** tiene un claro patrón de repetición, mientras que el de la **Opción A** tiene ahí un número tres, que es la condición de salida del bucle, y eso debería ser suficiente para llegar a nuestra conclusión.
</details>

[^jump]: Hay un salto incondicional `jmp`, y un montón de saltos condicionales como: `jz` ---salta cuando es cero; `jg` ---salta cuando es mayor; o, como en nuestro código, `jne` ---salta cuando no es igual. Estas instrucciones funcionan en base a las flags booleanas previamente establecidas por el procesador. Las flags son los bits que residen en un registro especial que se activan por instrucciones aritméticas como `add` o `sub`, o por una instrucción especial para comparar cosas `cmp`; dependiendo de su resultado.

## Conclusión

Se podría decir que estos ejemplos los he simplificado a propósito y no reflejan situaciones de la vida real. Aunque esto es cierto en parte, todos proceden de mi propia experiencia y sirven para demostrar conceptos de forma más eficaz.

Al utilizar el dispatch estático en lugar de dinámico, pude hacer que mi pipeline de procesamiento digital de señal fuera hasta cinco veces más rápido. Arreglar un anidado roto me ayudó a recuperar el 50% de la pérdida de rendimiento de una función que calcula distancias entre bordes de imágenes. Al cambiar el tipo de contador para permitir el desenrollado de bucles, el rendimiento de una función que hacía transformaciones de matrices aumentó aproximadamente un 10%. Aunque este último no luzca una mejora significativa, lo conseguí simplemente sustituyendo un `short int` por un `size_t` en un sitio del código. Creo que es una inversión de tiempo que merece la pena por un aumento en rendimiento del 10%.

Sorprendentemente, las versiones antiguas de MSVC (el maldito y horroroso compilador de Microsoft Visual Studio 💩) no pueden desenrollar bucles que usan tipos de contador no nativos. ¿Quién lo hubiera pensado? Incluso si eres consciente de esta idiosincrasia particular, no hay manera de conocer **todas** las peculiaridades de cada compilador que hay por ahí. Por lo tanto, examinar el desensamblado de vez en cuando puede ser útil.

La buena noticia es que no necesitas pasar años aprendiendo cada dialecto de ensamblador para hacer esto. Leer el desensamblado es a menudo más simple de lo que parece. Puedes intentarlo hoy, ahora. [Atrévete](https://godbolt.org/).