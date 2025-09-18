= Este archivo fue escrito utilizando markdown de gitlab =\
# Machete de GDB

`Las lineas con este formato son comandos`

## Antes de llamar a GDB

Paso 0: se recomienda FUERTEMENTE utilizar gdb-dashboard. "Instalarlo" no es más que correr el comando `wget -P ~ https://git.io/.gdbinit` desde la terminal. No requiere sudo ni ningún permiso especial, así que se puede instalar en las compus de los labos sin problemas (lo único que hace es descargar un archivo llamado ".gdbinit" y ponerlo en el home).

Una vez que se compiló el programa **USANDO LA FLAG -g en nasm** (para incluir debug symbols):

## Usando GDB

Supongamos que tengo los archivos asm.asm, c.c y el programa compilado (binario), llamemoslo ejec

(Mi archivo asm tiene la función (símbolo global) holaMundo:, Y creé dentro de ella la etiqueta .debug: para debuggear parte del código (util para ir rápido en el gdb.))

* Para ejecutar el programa, en la carpeta donde esta el binario llamo
    * `gdb ejec` 
    * `gdb` solo, y luego `file ejec`
    * `gdb --args ejec temperature -i c facil.bmp` para llamar un binario con argumentos, por ejemplo acá llamo al binario ejec con varios argumentos.

* Podemos crear breakpoints en cualquier momento (antes de llamar run, o durante la ejecución del programa) usando el comando `b` (break). Opciones:

* Agregar breakpoints (guía de todo lo que se puede hacer con breakpoints al final).
    * `b asm.asm:20`       // Agrega breakpoint en linea 20 de asm.asm (a veces anda medio raro)
    * `$b c.c:30`          // Agrega breakpoint en linea 30 de c.c
    * `b holaMundo.debug`  // Agrega breakpoint donde está .debug (el que me funciona mejor para asm)
* `run` o `r` para (re)iniciar la ejecución del programa.
* Nos llevará al primer breakpoint, donde podemos hacer
    * `info registers`     // O cualquiera de sus variantes (reg, r,... ver help info)
    * `continue` o `c`
    * `n` (next, avanza de a una linea del fuente)
    * `ni` (next instruction, avanza una linea del assemble) o `si` (step instruction)
    * `p $RXX` (o print, imprimir valor de un registro, variable, expresión)
        * Podemos castear el tipo del registro, y entonces nos lo imprime con el formato correcto. Ejemplos:
            * `p (char*) $RXX` imprime "Susan" por ejemplo, si $RXX es el comienzo de una string
            * `p *(struct_t*) $RXX` imprime todos los campos de un struct de manera ordenada, si $RXX es un puntero a dicho struct (vemos que desreferencié el puntero). Sumamente util para debuggear movimientos por estructuras.
    * Si tenemos variables declaradas por nombre (por ejemplo al debuggear código c) se pueden imprimir. Por ejemplo si tengo una variable var `p var` imprime su valor. gdb ya sabe su tipo dado que c es fuertemente tipado.
    * También se puede imprimir registros XMM, pero hacer tira todas las posibles interpretaciones (float, int, etc.) de los datos (ejemplo de salida al fondo). Si sabemos que los datos en xmm0 son 4 floats, por ejemplo, nos conviene llamar `print $xmm0.v4_float` (los nombres de los formatos aparecen al hacer print $xmm). Listo los nombres de formatos:
        * v4_float (4 floats)
        * v2_double (2 doubles)
        * v16_int8 (16 enteros de 8 bits)
        * v8_int16 (8 enteros de 16 bits)
        * v4_int32 (4 enteros de 32 bits)
        * v2_int64 (2 enteros de 64 bits)
        * uint128 (1 entero sin signo de 128 bits)
        (basicamente lo que está pasando es que trata a $xmm como un struct con las posibles lecturas como elementos, me parece. Por eso hacemos .v4_float, estamos accediendo a uno de los elementos de ese struct. En general la sintaxis de punteros de c funciona en asm para cualquier cosa que sea una dirección de memoria! Incluyendo registros.)

    * `x/1tb dir` (examinar 1B de memoria y expresar como entero en binario. El 1 representa cantidad de unidad, b indica byte y t indica binario. Me resulta útil para leer máscaras que declaré en .rodata o .data antes de bajarlas a registros, por ejemplo. referencia: ftp://ftp. gnu.org/old-gnu/Manuals/gdb/html_chapter/gdb_9.html#SEC56). Algunas formas útiles:
        * `x/4wf dir` lee 4 floats a partir de la dir   (w = words(DE 4 BYTES, nuestra doubleword), f = interpreta como floats)
        * `x/2gf dir` lee 2 doubles a partir de la dir (g = giant word (de 8 bytes, neustra quadword))
        * `x/1dw dir` lee 1 int de 32 bits CON SIGNO (d = signed decimal)
        * `x/1uw dir` lee 1 int de 32 bits SIN SIGNO (u = unsigned decimal)
        * donde dir es alguna dirección de memoria (puede ser una etiqueta o algún puntero que tengo en un registro...)

* En cualquier momento, podemos salir con quit, o reiniciar la ejecución del código volviendo a correr `r` (run). También se puede hacer `make` desde dentro de gdb y volver a correr el ejecutable con `r`, y se cargará el binario nuevo. Útil para mantener los breakpoints preestablecidos luego de una corrección chica. 


### Manejo de breakpoints
Fuente: https://www.cse.unsw.edu.au/~learn/debugging/modules/gdb_conditional_breakpoints/

* `b` (o `break`) por si solo solo crea un breakpoint en la línea actual.
* `b asm.asm:27` crea un breakpoint en la línea 27 del archivo asm.asm (podría ser en un archivo de c también).
* `b foo` crea un breakpoint al comienzo de la función foo
* `b foo.etiqueta_relativa` si usamos etiquetas con un . adelante en asm, son lo que llamo "etiquetas relativas" a la función actual (última etiqueta sin . adelante). Entonces al poner el breakpoint en gdb debemos especificar el nombre de la función y luego el nombre de la etiqueta relativa.
* `b <lugar> if <expresion>` donde lugar se especifica de cualquiera de las maneras mencionadas arriba, y la expresión puede ser, por ejemplo $rdi > 5 o cualquier expresión booleana, en principio (se puede hacer un `p <expresion>` antes de setear el breakpoint para revisar que ande bien).
* `info breakpoints` para ver los breakpoints seteados y sus números correspondientes. Usamos los números de los breakpoints para borrarlos o ponerles condiciones:
* `delete 5` borra el breakpoint con número 5
* `delete` borra todos los breakpoints
* `condition 7 $rdi > 5` agrega una condición al breakpoint número 7, que ahora frenará sólo si el valor de $rdi es mayor a 5
* `condition 7` borra las condiciones del breakpoint 7, es decir frena siempre que el código pase por ahí.

### Ejemplo de salida de `print $xmm`

```
$1 = {
  v8_bfloat16 = {[0] = -nan(0x7f), [1] = -nan(0x7f), [2] = -nan(0x7f), [3] = -nan(0x7f), [4] = -nan(0x7f), [5] = -nan(0x7f), [6] = -nan(0x7f), [7] = -nan(0x7f)},
  v8_half = {[0] = -nan(0x3ff), [1] = -nan(0x3ff), [2] = -nan(0x3ff), [3] = -nan(0x3ff), [4] = -nan(0x3ff), [5] = -nan(0x3ff), [6] = -nan(0x3ff), [7] = -nan(0x3ff)},
  v4_float = {[0] = -nan(0x7fffff), [1] = -nan(0x7fffff), [2] = -nan(0x7fffff), [3] = -nan(0x7fffff)},
  v2_double = {[0] = -nan(0xfffffffffffff), [1] = -nan(0xfffffffffffff)},
  v16_int8 = {[0] = -1 <repeats 16 times>},
  v8_int16 = {[0] = -1, [1] = -1, [2] = -1, [3] = -1, [4] = -1, [5] = -1, [6] = -1, [7] = -1},
  v4_int32 = {[0] = -1, [1] = -1, [2] = -1, [3] = -1},
  v2_int64 = {[0] = -1, [1] = -1},
  uint128 = 340282366920938463463374607431768211455
}
```
