# ssl-flex-bison-projects
Una forma más sencilla de compilar tus proyectos de Flex y Bison.

![image](https://user-images.githubusercontent.com/39303639/111651339-e5a4ad00-87e4-11eb-99af-a479b7ed5068.png)

## ¿Qué debo tener instalado?

<details>
<summary>
¿Qué debo tener instalado en Windows?        
</summary>
    
1. [MinGW y GCC](../../wiki/Instalacion-de-MinGW) con las variables de entorno incluidas.

2. Los siguientes programas de GnuWin32 (la opción que dice "Complete package, except sources") instalado en una carpeta SIN ESPACIOS (puede ser, al igual que para MinGW, `C:\GnuWin32`). Las variables de entorno deben apuntar hacia la subcarpeta `bin` (ej: `C:\GnuWin32\bin`):
    - Flex: http://gnuwin32.sourceforge.net/packages/flex.htm
    - Bison: http://gnuwin32.sourceforge.net/packages/bison.htm
    - Make: http://gnuwin32.sourceforge.net/packages/make.htm

3. Crear una nueva variable de entorno llamada `LIBRARY_PATH` y asignarle como valor la ruta hacia la subcarpeta `lib` (ej: `C:\GnuWin32\lib`) [para que GCC encuentre las bibliotecas de Flex y Bison](https://gcc.gnu.org/onlinedocs/gcc/Environment-Variables.html).

![image](https://user-images.githubusercontent.com/39303639/113922291-828fb000-97bd-11eb-8142-9822bd635523.png)

4. Una consola Bash, que puede ser la que viene con [Git](https://git-scm.com/downloads)

</details>

<details>
<summary>
¿Qué debo tener instalado en Ubuntu?
</summary>

Make y gcc vienen instalados en forma nativa, se puede averiguar con el flag `--version`:
```
$ gcc --version
$ make --version
```
En el caso de Flex y Bison, se pueden obtener con el gestor de paquetes `apt-get`:

```
$ sudo apt-get install libfl-dev
$ sudo apt-get install libbison-dev
```

</details>

## ¿Dónde pongo mis archivos?

En el repo se encuentran dos archivos makefile (uno para [solo flex](flex/makefile) y otro para [flex y bison](bison/makefile)), los cuales se rigen bajo la siguiente estructura para funcionar:

```bash
.
│
└─── src/  
|     └─── <nombre_archivo_final>.l   # Archivo flex
|     └─── <nombre_archivo_final>.y   # Archivo bison (si corresponde)
|     └─── ...                        # Código C (archivos *.c y *.h)
└─── makefile                         # Archivo makefile para compilar el proyecto
```

Al compilar, se generará lo siguiente:
```bash
.
│
└─── obj/  
|     └─── <nombre_archivo_final>.yy.c    # Código C generado por flex
|     └─── <nombre_archivo_final>.tab.c   # Código C generado por bison (si corresponde)
|     └─── <nombre_archivo_final>.tab.h   # Encabezado C generado por bison (si corresponde)
|     └─── <nombre_archivo_final>.output  # Gramática generada por bison (si corresponde)
|     └─── ...                            # Código máquina generado por gcc (archivos *.o)
└─── <nombre_archivo_final>.exe/.out      # Archivo ejecutable generado por gcc
```

## ¿Cómo configuro VSCode?

### Consola

Para que los makefiles funcionen, es fundamental usarlos siempre en una consola **Bash**. Para configurarla por default en VSCode se deberá hacer lo siguiente:

![bash vsc](https://i.imgur.com/w3rKAl4.gif)

### Plugins

Para resaltar la sintaxis de Flex y Bison, recomiendo usar el plugin [Yash](https://marketplace.visualstudio.com/items?itemName=daohong-emilio.yash):

![image](https://user-images.githubusercontent.com/39303639/113889033-3d598700-9799-11eb-982b-a0283d9a10a0.png)

## ¿Cómo se usa?

Para ejecutar los makefiles, nos moveremos a la ruta donde se encuentra nuestro archivo `makefile` y usaremos el comando `make` seguido de alguna de las reglas existentes de ser necesario. Si querés ver cómo funciona el makefile, podés hacerlo en la [wiki de este repo](../../wiki).

### Compilar el proyecto
```
$ make
```
### Eliminar los archivos generados al compilar
```
$ make clean
```
Esto es útil por si uno quiere compilar todo el proyecto desde cero. Si tu intención es eliminar los objetos generados del repo, te aconsejo usar un archivo [gitignore](flex/.gitignore).

### Debugear el proyecto

```
$ make debug
```
#### Aclaración importante

En el caso de Bison, para debugear el proyecto hay que tener dentro del `main()`, antes de `yyparse()`, las siguientes tres líneas:
```c
    #if YYDEBUG
        yydebug = 1;
    #endif
```
Para más información, podés ver la [wiki de este repo](../../wiki#)

#### Consejo

El ejecutable generado va a imprimir por la salida estándar las reglas por las que va derivando:
```bash
Starting parse
Entering state 0
Reducing stack by rule 1 (line 29):
-> $$ = nterm input ()
Stack now 0
Entering state 1
Reading a token: --(end of buffer or a NUL)
```

Probablemente, a medida que el proyecto vaya escalando, leer eso desde la consola te resulte poco práctico, así que una forma rápida en Bash de redirigir `stdout` a un archivo de log sin modificar código es la siguiente:
```
$ ./main.out &> main.log
```
Esto me genera un archivo `main.log` que puede ser abierto por cualquier editor de texto. En VSCode, incluso se puede abrir durante la ejecución y seguirlo en tiempo real como si fuera el `stdout`.

## Troubleshooting

### ¿Cómo incluyo una library?

Al principio del makefile hay un comentario que lo explica muy simple:
```makefile
# Agregar bibliotecas necesarias acá (por ejemplo, -lm para incluir <math.h>)
LIBS+=-lfl
```
Entonces, si queremos incluir una biblioteca como math, simplemente concatenamos ahí el flag correspondiente.

### Make: Nothing to be done for 'all'.

```makefile
make: Nothing to be done for 'all'.
```

¡No es un error! Significa que los archivos en `src/` no fueron modificados desde la última vez que se compiló el proyecto. Asegurate de haber guardado los últimos cambios antes de compilar ;)

### Makefile: missing separator. Stop.

```makefile
makefile:35: *** missing separator.  Stop.
```

Esto se debe a que el make es sensible a los tabs. Probablemente habrás copiado el texto del makefile, y lo habrás pegado en un bloc de notas o en el VSCode, y el editor incluyó espacios en vez de tabs con `\t`.

En VSCode esto se puede resolver [de esta forma](https://stackoverflow.com/a/38083525/14089741).

### No puedo ejecutar 'make' más de una vez / 'make clean' no elimina los archivos correctamente

Probablemente, si estás en Windows y al ejecutar `make` se crea una subcarpeta `-p`, además de aparecer errores del estilo:
```powershell
Ya existe el subdirectorio o el archivo -p.
Error mientras se procesaba: -p.
Ya existe el subdirectorio o el archivo obj.
Error mientras se procesaba: obj.
make: *** [Ejemplo.exe] Error 1
```
y al ejecutar `make clean` también puede que te aparezca algo como:
```powershell
process_begin: CreateProcess(NULL, uname, ...) failed.
rm -f obj/*
process_begin: CreateProcess(NULL, rm -f obj/*, ...) failed.
make (e=2): El sistema no puede encontrar el archivo especificado.
make: *** [clean] Error 2
```
Es porque estás usando la consola cmd o Powershell. Asegurate de cambiarla a Bash como se explica [más arriba](#consola).
