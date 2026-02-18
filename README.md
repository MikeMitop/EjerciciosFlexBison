# EjerciciosFlexBison
# Taller: Primeros Pasos en Flex y Bison

**Autor:** Miguel Ángel Celis López  
**Docente:** Joaquín Fernando Sánchez Cifuentes  
**Asignatura:** Lenguajes de Programación  
**Universidad:** Universidad Sergio Arboleda  
**Ciudad y fecha:** Bogotá D.C., 2026

---

## Objetivo general

Desarrollar, ejecutar y analizar los ejemplos 1 al 6 del capítulo 1 del libro **_Flex & Bison_**, con el fin de comprender el funcionamiento básico de analizadores léxicos y sintácticos.

## Objetivos específicos

1. Implementar y probar los ejemplos 1, 2, 3, 4, 5 y 6 del capítulo 1.
2. Resolver los ejercicios propuestos al final del capítulo.
3. Explicar el propósito y la lógica de cada fragmento de código.
4. Documentar la compilación y ejecución correcta de cada ejercicio.

---

## ¿Cómo compilar y ejecutar?

> Requisito (Linux):
>
> ```bash
> sudo dnf install flex bison gcc make
> ```

En Debian/Ubuntu se puede usar:

```bash
sudo apt update
sudo apt install flex bison gcc make
```

---

## Ejercicio 1 (Capítulo 1) – Contador tipo `wc`

### Código fuente (`fb1.1/fb1.1.l`)

```lex
/* just like Unix wc */
%{
#include <stdio.h>
#include <string.h>
int chars = 0;
int words = 0;
int lines = 0;
%}

%%

[a-zA-Z]+  { words++; chars += strlen(yytext); }
\n         { chars++; lines++; }
.          { chars++; }

%%

int main(int argc, char **argv) {
  if (argc > 1) {
    if (!(yyin = fopen(argv[1], "r"))) {
      perror(argv[1]);
      return 1;
    }
  }

  yylex();
  printf("%8d%8d%8d\n", lines, words, chars);
  return 0;
}

int yywrap(void) {
  return 1;
}
```

### Explicación del código

Este escáner cuenta líneas, palabras y caracteres de una entrada.

- `[a-zA-Z]+` identifica secuencias de letras como palabras.
- `\n` incrementa líneas y caracteres.
- `.` captura cualquier otro carácter.
- En `main`, si se pasa un archivo por argumento, lo lee desde `yyin`; de lo contrario, lee desde entrada estándar.

### Compilación y ejecución

```bash
cd fb1.1
flex fb1.1.l
gcc lex.yy.c -o fb1.1 -lfl
./fb1.1 texto.txt
```

También puede ejecutarse con entrada manual:

```bash
./fb1.1
```

(Escribir texto y finalizar con `Ctrl + D`).

---

## Ejercicio 2 (Capítulo 1) – Traductor léxico simple

### Código fuente (`fb1.2/fb1.2.l`)

```lex
%{
#include <stdio.h>
%}

/* English -> American */
%%
"colour"        { printf("color"); }
"flavour"       { printf("flavor"); }
"clever"        { printf("smart"); }
"smart"         { printf("elegant"); }
"conservative"  { printf("liberal"); }
.                { printf("%s", yytext); }
%%

int main(void)
{
    yylex();
    return 0;
}
```

### Explicación del código

Este programa reemplaza ciertas palabras por equivalentes definidos en reglas.

- Si coincide con una palabra específica, imprime su traducción.
- La regla `.` mantiene cualquier otro carácter sin modificar.

### Salida esperada

- `smart` → `elegant`
- `colour` → `color`

### Compilación y ejecución

```bash
cd fb1.2
flex fb1.2.l
gcc lex.yy.c -o fb1.2 -lfl
echo "smart colour" | ./fb1.2
```

---

## Ejercicio 3 (Capítulo 1) – Reconocimiento de tokens

### Código fuente (`fb1.3/fbl1.3.l`)

```lex
/* recognize tokens for the calculator and print them out */
%%
"+"     { printf("PLUS\n"); }
"-"     { printf("MINUS\n"); }
"*"     { printf("TIMES\n"); }
"/"     { printf("DIVIDE\n"); }
"|"     { printf("ABS\n"); }
[1-9]+  { printf("NUMBER %s\n", yytext); }
\n      { printf("NEWLINE\n"); }
[ \t]   { }
.       { printf("Mystery character %s\n", yytext); }
%%
```

### Explicación del código

El escáner tokeniza operadores y números para una calculadora simple:

- Traduce símbolos (`+`, `-`, `*`, `/`, `|`) a nombres de token.
- Detecta enteros positivos (`[1-9]+`).
- Ignora espacios y tabulaciones.
- Reporta caracteres inesperados con un mensaje de error léxico.

### Compilación y ejecución

```bash
cd fb1.3
flex fbl1.3.l
gcc lex.yy.c -o fb1.3 -lfl
echo "2+3*5" | ./fb1.3
```

---

## Ejercicio 4 (Capítulo 1) – Escáner que retorna tokens numéricos

### Código fuente (`fbl1.4/fbl1.4.l`)

```lex
/* recognize tokens for the calculator and print them out */
%{
#include <stdio.h>
#include <stdlib.h>
enum yytokentype {
  NUMBER = 258,
  ADD = 259,
  SUB = 260,
  MUL = 261,
  DIV = 262,
  ABS = 263,
  EOL = 264
};
int yylval;
%}

%%
"+"     { return ADD; }
"-"     { return SUB; }
"*"     { return MUL; }
"/"     { return DIV; }
"|"     { return ABS; }
[0-9]+  { yylval = atoi(yytext); return NUMBER; }
\n      { return EOL; }
[ \t]   { /* ignore whitespace */ }
.       { printf("Mystery character %c\n", *yytext); }
%%

int main(int argc, char **argv) {
  int tok;
  while ((tok = yylex())) {
    printf("%d", tok);
    if (tok == NUMBER) printf(" = %d\n", yylval);
    else printf("\n");
  }
  return 0;
}

int yywrap(void) {
  return 1;
}
```

### Explicación del código

- Convierte números a enteros con `atoi` y los guarda en `yylval`.
- Devuelve constantes (`ADD`, `SUB`, etc.) para ser consumidas por un parser.
- `main` demuestra qué token se leyó y el valor numérico cuando corresponde.

### Compilación y ejecución

```bash
cd fbl1.4
flex fbl1.4.l
gcc lex.yy.c -o fbl1.4 -lfl
echo "12+7" | ./fbl1.4
```

---

## Ejercicio 5 (Capítulo 1) – Calculadora con Bison + Flex

### Archivos

- Parser: `fbl1.5/fb1-5.y`
- Scanner: `fbl1.5/fb1-5.l`

### Explicación del código

Este ejercicio divide responsabilidades:

- **Bison (`.y`)** define la gramática y calcula expresiones con precedencia implícita por niveles:
  - `exp`: suma y resta.
  - `factor`: multiplicación y división.
  - `term`: números y valor absoluto.
- **Flex (`.l`)** reconoce los tokens y alimenta al parser con `NUMBER`, `ADD`, `SUB`, etc.

Cuando se completa una línea (`EOL`), se imprime el resultado con `printf("= %d\n", $2);`.

### Compilación y ejecución

```bash
cd fbl1.5
bison -d fb1-5.y
flex fb1-5.l
gcc fb1-5.tab.c lex.yy.c -o fb1-5 -lfl
./fb1-5
```

Ejemplo de entrada:

```text
2+3
(2+3)*5
```

---

## Ejercicio 6 (Capítulo 1) – Parser con escáner manual en C

### Archivos

- Parser: `fbl1.6/fb1-5.y`
- Scanner manual en C: `fbl1.6/fb1-6.c`

### Explicación del código

Este ejemplo reemplaza Flex por un escáner escrito a mano:

- Lee carácter por carácter con `getc`.
- Construye números de múltiples dígitos manualmente.
- Reconoce operadores, paréntesis y fin de línea.
- Soporta comentarios estilo `//...` hasta salto de línea.

### Ejemplos de salida

```text
2+3
= 5
(2+3)*5
= 25
5/5
= 1
```

### Compilación y ejecución

```bash
cd fbl1.6
bison -d fb1-5.y
gcc fb1-5.tab.c fb1-6.c -o fb1-6
./fb1-6
```

---

## Exercises | ANSWER

### 1) ¿La calculadora acepta una línea que solo contiene un comentario?


- En el escáner manual del ejemplo 6, `// comentario` se consume hasta `\n`, pero no siempre produce un token útil por sí solo para la regla `calclist exp EOL`.


### 2) ¿Cómo volverla calculadora hexadecimal y decimal?

En el scanner, agregar una regla antes de `[0-9]+`:

```lex
0x[0-9a-fA-F]+ { yylval = (int)strtol(yytext, NULL, 16); return NUMBER; }
[0-9]+         { yylval = atoi(yytext); return NUMBER; }
```

Luego, en la impresión del resultado:

```c
printf("= %d (0x%X)\n", valor, valor);
```

Así se muestra el resultado en decimal y hexadecimal.

### 3) (Extra) ¿Qué pasa si `|` también se usa como OR binario?

Hay ambigüedad porque `|` ya es operador unario de valor absoluto (`ABS term`).

- Expresiones como `a | b` pueden interpretarse de varias maneras.
- Bison puede reportar conflictos *shift/reduce* si no se redefinen precedencias y gramática.


### 4) ¿El scanner manual reconoce exactamente los mismos tokens que Flex?

No necesariamente.

Diferencias comunes:

- Patrones de números pueden no coincidir exactamente.
- Manejo de EOF y `ungetc` puede cambiar casos borde.
- Soporte de comentarios y caracteres desconocidos puede ser distinto.

Por lo tanto, no son idénticos salvo que se repliquen cuidadosamente todas las reglas.

### 5) ¿En qué lenguajes Flex no sería buena opción?

Flex no es ideal cuando la tokenización depende de contexto muy profundo o estructura anidada compleja:

- Lenguajes con sintaxis sensible a indentación estricta y múltiples estados semánticos.
- Sistemas con Unicode avanzado y reglas de segmentación complejas.


### 6) Reescribir `wc` en C y comparar
```C
#include <stdio.h>
#include <ctype.h>
#include <stdlib.h> // Necesario para exit o malloc si se usa
#include "fb1-5.tab.h" // Reusamos los tokens de Bison

void yyerror(char *s);
extern int yylval; // yylval se define en el parser de bison, aqui es extern
int seeneof = 0;

int yylex(void) {
  if(seeneof) return 0; /* saw EOF last time */
  
  while(1) {
    int c = getchar(); // Usamos getchar estándar

    if (c == EOF) {
        seeneof = 1;
        return 0;
    }

    if(isdigit(c)) {
      int i = c - '0';
      while(isdigit(c = getchar()))
        i = (10*i) + c-'0';
      yylval = i;
      if(c == EOF) seeneof = 1;
      else ungetc(c, stdin);
      return NUMBER;
    }

    switch(c) {
    case '+': return ADD;
    case '-': return SUB;
    case '*': return MUL;
    case '|': return ABS;
    case '\n': return EOL;
    case ' ':
    case '\t': break;    /* ignore these */
    case '/': 
        // Lógica simple para detectar comentarios o división (simplificado)
        return DIV;
    default: printf("Mystery character %c\n", c); break;
    }
  }
}

---
```

 **Rendimiento:** la versión en C manual es ligeramente más rápida en archivos grandes.
- **Dificultad de depuración:** la versión en Flex fue más fácil de depurar porque las reglas son declarativas; en C manual aparecieron más errores de borde (EOF, saltos de línea y conteos).


## Conclusiones

1. Flex y Bison permiten construir analizadores funcionales de forma incremental: primero lexer, luego parser.
2. Separar análisis léxico y sintáctico mejora la organización del código y facilita extender la gramática.
3. Aunque un scanner manual da mayor control, Flex acelera el desarrollo y reduce errores para la mayoría de casos.
