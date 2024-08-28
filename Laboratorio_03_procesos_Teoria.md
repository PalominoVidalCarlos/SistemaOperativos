## Creación de procesos

- Todos los procesos tienen un identificador de proceso único
  - Las llamadas al sistema `getpid()`, `getppid()` permiten a los procesos obtener su información
- Creación de procesos
  - La llamada al sistema `fork()` crea una copia de un proceso y retorna en ambos procesos, pero con un valor de retorno diferente
  - `exec()` reemplaza un espacio de direcciones con un nuevo programa
- Terminación de procesos, señalización
  - Las llamadas al sistema `signal()`, `kill()` permiten que un proceso sea terminado o que se le envíen señales específicas


## Uso del `getpid` y `getppid`

1. Abre una terminal.
2. Escribe el código en un archivo usando un editor, por ejemplo: `nano pid_example.c`.
3. Guarda el archivo y sal del editor.
4. Compila el programa con `gcc pid_example.c -o pid_example`.
5. Ejecuta el programa compilado con `./pid_example`.

### Código de ejemplo en C

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    printf("El PID de este proceso es: %d\n", getpid());
    printf("El PID del proceso padre es: %d\n", getppid());
    return 0;
}
```

### Instalación del compilador `gcc`

```bash
sudo apt install gcc --fix-missing
sudo apt install gcc
sudo apt install build-essential
```


## Uso del `fork`

### Código de ejemplo en C

```c
#include <unistd.h>

int main() {
    pid_t pid;
    pid = fork();

    // Creamos un nuevo proceso
    if (pid < 0) {
        // Si fork() retorna un valor negativo, la creación del proceso hijo falló
        fprintf(stderr, "Fallo en fork()");
        return 1;
    } else if (pid == 0) {
        // fork() retorna 0 al proceso hijo
        printf("Soy el proceso hijo. Mi PID es %d y el PID de mi proceso padre es %d.\n", getpid(), getppid());
    } else {
        // fork() retorna el PID del proceso hijo al proceso padre
        printf("Soy el proceso padre. Mi PID es %d y el PID de mi proceso hijo es %d.\n", getpid(), pid);
    }

    // Este printf se ejecuta en ambos procesos
    printf("Este mensaje es de %s.\n", (pid == 0) ? "hijo" : "padre");

    return 0;
}
```

### Descripción

- `fork()` crea un nuevo proceso duplicando el proceso actual.
- Si `fork()` retorna un número negativo, significa que la creación del proceso hijo falló.
- Si `fork()` retorna `0`, significa que estamos en el proceso hijo.
- Si `fork()` retorna un número positivo, este es el PID del proceso hijo, y significa que estamos en el proceso padre.
- `getpid()` devuelve el PID del proceso que llama a la función.
- `getppid()` devuelve el PID del proceso padre del proceso que llama a la función.

Puedes escribir este código en un archivo `.c`, compilarlo con `gcc` y ejecutar el binario resultante en tu terminal de Ubuntu para ver cómo funciona la creación de procesos con `fork()`.

## Uso del `EXEC`

El `exec()` en C es una familia de funciones que reemplazan el espacio de direcciones del proceso actual con un nuevo programa. Se utiliza con frecuencia en conjunto con `fork()`, donde el proceso hijo llama a `exec()` para ejecutar un nuevo programa. Aquí tienes un ejemplo de cómo se podría usar:

### Código de ejemplo en C

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    pid_t pid;

    // Crear un proceso hijo
    pid = fork();

    if (pid < 0) {
        // Error al crear el proceso hijo
        perror("Fork failed");
        exit(1);
    } else if (pid == 0) {
        // Estamos en el proceso hijo
        printf("Proceso hijo (PID: %d) a punto de ejecutar 'ls' con execvp...\\n", getpid());

        // Un arreglo con los argumentos para execvp. El primer argumento es el programa.
        // El último argumento debe ser NULL para indicar el fin del arreglo.
        char *args[] = {"/bin/ls", "-l", "/home", NULL};

        // Reemplazar el proceso hijo con el programa 'ls'
        execvp(args[0], args);

        // Si execvp retorna, hubo un error
        perror("execvp");
        exit(1);
    } else {
        // Estamos en el proceso padre
        printf("Proceso padre (PID: %d) esperando a que el hijo termine...\\n", getpid());
        wait(NULL); // Esperar a que termine el proceso hijo
        printf("Proceso hijo terminó la ejecución.\\n");
    }

    return 0;
}
```

- Después de que `fork()` crea un nuevo proceso, el proceso hijo usa `execvp()` para ejecutar `ls -l /home`. `execvp()` toma dos argumentos: el nombre del programa y un arreglo que contiene el programa y sus argumentos, terminando con `NULL`.
- Si `execvp()` se ejecuta correctamente, el programa `ls` reemplaza el proceso hijo, y no se ejecutarán las líneas siguientes a `execvp()`.
- Si hay un error en `execvp()`, imprimirá un mensaje de error y terminará el proceso hijo.
- El proceso padre espera a que el proceso hijo termine con `wait(NULL)` antes de imprimir que el proceso hijo ha terminado.

### Instrucciones

1. Guarda el código en un archivo, por ejemplo `exec_example.c`.
2. Abre una terminal y navega al directorio donde guardaste el archivo.
3. Compila el código con `gcc exec_example.c -o exec_example`.
4. Ejecuta el programa con `./exec_example`. Verás la salida del comando `ls -l /home` en la terminal.

## Uso del `signal`

El manejo de señales en C se realiza con la función `signal()` para establecer un manejador de señales, que es una función que se llamará cuando el proceso reciba una señal específica.

Un ejemplo de cómo capturar la señal SIGINT (generalmente generada por la combinación de teclas Ctrl+C en la terminal) y manejarla con una función personalizada.

### Código de ejemplo en C

```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

// Esta es la función de manejo de señales que se llamará cuando se reciba SIGINT
void handle_sigint(int sig) {
    printf("Capturada señal SIGINT: %d\\n", sig);
    printf("Pero no voy a terminar!\\n");
    // Aquí se podrían liberar recursos o realizar una limpieza si fuera necesario
}

int main() {
    // Establecer la función handle_sigint como manejador de la señal SIGINT
    if (signal(SIGINT, handle_sigint) == SIG_ERR) {
        perror("Error al configurar el manejador de señal");
        return EXIT_FAILURE;
    }

    // Un bucle infinito que mantendrá vivo el proceso hasta que se reciba una señal
    while (1) {
        printf("Esperando señal...\\n");
        sleep(2); // Dormir por dos segundos para evitar una salida demasiado rápida
    }

    return EXIT_SUCCESS;
}
```
Este programa configura un manejador de señales para SIGINT. Cuando ejecutes este programa y presiones Ctrl+C, en lugar de terminar el proceso inmediatamente, imprimirá los mensajes definidos en `handle_sigint()` y continuará ejecutándose.

### Instrucciones

1. Guarda el código en un archivo, por ejemplo `signal_example.c`.
2. Compila el programa con `gcc signal_example.c -o signal_example`.
3. Ejecuta el programa con `./signal_example`.
4. Mientras el programa está corriendo, intenta interrumpirlo con Ctrl+C y observa la salida.

## Uso del `kill`

La función `kill()` en C se usa para enviar señales a otros procesos.

Un ejemplo en el que un proceso padre crea un hijo, y después de un breve periodo, envía una señal al proceso hijo para terminarlo usando `kill()`.


### Código de ejemplo en C

```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
    pid_t pid = fork(); // Creamos un proceso hijo

    if (pid == -1) {
        // Error al crear el proceso hijo
        perror("fork");
        return 1;
    }

    if (pid > 0) {
        // Proceso padre
        printf("Proceso padre: PID = %d, PID del hijo = %d\\n", getpid(), pid);
        sleep(5); // Darle tiempo al hijo para que haga algo

        printf("Proceso padre: Enviando SIGTERM al proceso hijo\\n");
        if (kill(pid, SIGTERM) == -1) {
            perror("kill");
            exit(1);
        }
    } else {
        // Proceso hijo
        while(1) {
            printf("Proceso hijo: PID = %d, trabajando...\\n", getpid());
            sleep(1); // Simula hacer algo
        }
    }

    return 0;
}
```
- `fork()` crea un proceso hijo.
- El proceso padre (`pid > 0`) espera 5 segundos para darle tiempo al hijo de "hacer algo", que en este caso es simplemente imprimir un mensaje cada segundo.
- Después de la espera, el proceso padre utiliza `kill()` para enviar la señal SIGTERM al proceso hijo, pidiéndole que termine.
- El proceso hijo tiene un bucle infinito que sería interrumpido por la señal SIGTERM, la cual por defecto hará que el proceso termine. No necesitas un manejador de señales para SIGTERM si la acción predeterminada (terminar el proceso) es lo que quieres.

### Instrucciones

1. Guarda el código en un archivo llamado `kill_example.c`.
2. Abre una terminal y compila el programa con `gcc kill_example.c -o kill_example`.
3. Ejecuta el programa compilado con `./kill_example`.
4. Observa cómo el proceso padre envía la señal después de 5 segundos y cómo reacciona el proceso hijo.

