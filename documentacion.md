# Informe técnico final

**Universidad de San Carlos de Guatemala**
**Curso:** Sistemas Operativos 2
**Autor:** Christopher Iván Monterroso Alegría
**Carné:** 201902363

---

## Introducción

Este informe documenta el desarrollo e implementación de un sistema limitador de memoria en Linux. El proyecto tiene como objetivo controlar la cantidad máxima de memoria que los procesos pueden solicitar dinámicamente. Esto se logró mediante la modificación del kernel de Linux para incluir nuevas syscalls que permiten crear, leer, actualizar y eliminar límites de memoria para procesos específicos.

El sistema implementa cuatro nuevas syscalls:

* **SYS\_CHRIS\_ADD\_MEMORY\_LIMIT (557):** Agrega un límite de memoria a un proceso.
* **SYS\_CHRIS\_GET\_MEMORY\_LIMITS (558):** Obtiene la lista de procesos limitados.
* **SYS\_CHRIS\_UPDATE\_MEMORY\_LIMIT (559):** Actualiza el límite de memoria de un proceso.
* **SYS\_CHRIS\_REMOVE\_MEMORY\_LIMIT (560):** Elimina el límite de memoria de un proceso.

---

## Configuración del entorno

Se utilizó una máquina virtual con Linux Mint como entorno de desarrollo. Las herramientas necesarias para la modificación y recompilación del kernel fueron instaladas siguiendo los pasos:

1. Descargar el código fuente del kernel:
   ```
   wget https://www.kernel.org/pub/linux/kernel/v6.x/linux-6.8.tar.xz
   tar -xf linux-6.8.tar.xz
   cd linux-6.8
   ```
2. Instalar dependencias:
   ```
   sudo apt-get install build-essential libncurses-dev bison flex libssl-dev libelf-dev
   ```
3. Configurar y compilar el kernel:
   ```
   cp -v /boot/config-$(uname -r) .config
   make oldconfig
   make -j$(nproc --ignore=1)
   make modules_install
   make install
   update-grub2
   ```
4. Reiniciar el sistema para usar el nuevo kernel.

---

## Diseño e implementación

### Estructura de Datos

Se utilizó una lista enlazada simple en el espacio del kernel para almacenar los procesos y sus límites de memoria. Cada entrada está definida mediante la siguiente estructura:

```
struct memory_limit_entry {
    pid_t pid;
    size_t limit;
    struct list_head list;
};
```

### Syscalls

**1. SYS\_CHRIS\_ADD\_MEMORY\_LIMIT (557):** Agrega un proceso a la lista de procesos limitados.

```
long SYSCALL_DEFINEx(SYS_CHRIS_ADD_MEMORY_LIMIT, pid_t, process_pid, size_t, memory_limit);
```

* **Errores manejados:**
  * Proceso inexistente.
  * Límite no válido.
  * Permisos insuficientes.

**2. SYS\_CHRIS\_GET\_MEMORY\_LIMITS (558):** Obtiene la lista de procesos limitados.

```
long SYSCALL_DEFINEx(SYS_CHRIS_GET_MEMORY_LIMITS, struct memory_limitation*, size_t, max_entries);
```

* **Errores manejados:**
  * Buffer insuficiente.
  * Permisos insuficientes.

**3. SYS\_CHRIS\_UPDATE\_MEMORY\_LIMIT (559):** Actualiza el límite de memoria de un proceso existente.

```
long SYSCALL_DEFINEx(SYS_CHRIS_UPDATE_MEMORY_LIMIT, pid_t, process_pid, size_t, memory_limit);
```

* **Errores manejados:**
  * Proceso no encontrado.
  * Nuevo límite inválido.

**4. SYS\_CHRIS\_REMOVE\_MEMORY\_LIMIT (560):** Elimina un proceso de la lista de procesos limitados.

```
long SYSCALL_DEFINEx(SYS_CHRIS_REMOVE_MEMORY_LIMIT, pid_t, process_pid);
```

* **Errores manejados:**
  * Proceso no encontrado.
  * Permisos insuficientes.

## Resultados y análisis

Se realizaron pruebas con scripts en C para validar el funcionamiento de las syscalls.


| Proceso | Memoria solicitada | Límite asignado | Resultado             |
| ------- | ------------------ | ---------------- | --------------------- |
| 1234    | 50 MB              | 40 MB            | Bloqueado por límite |
| 5678    | 30 MB              | 50 MB            | Aprobado              |

---

## Problemas encontrados


| Problema                                   | Causa                                                     | Solución                                                                       |
| ------------------------------------------ | --------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Kernel panic al agregar procesos repetidos | Falta de validación antes de insertar en la lista.       | Se agregó verificación para evitar duplicados en la lista                     |
| Error al obtener la lista de procesos      | Buffer insuficiente para la cantidad de datos retornados. | Se validó el tamaño del buffer y se informa sobre el tamaño quie se necesita |

---

## Cronograma


| Día        | Actividad                                                              |
| ----------- | ---------------------------------------------------------------------- |
| 27/12/2024  | Investigación sobre asignación de memoria.                           |
| 28/12/2024  | Diseño de la estructura de lista enlazada e implementación del CRUD. |
| 01/01/2025  | Implementación de manejo de errores.                                  |
| 02/01//2025 | Pruebas, validaciones y elaboración de la documentación.             |

### Explicación de Actividades

**Día 1:** Se investigaron algoritmos como malloc y mmap, además de configurar el entorno para compilar el kernel modificado.

**Día 2:** Se diseñó la estructura de lista enlazada e implementaron las funciones CRUD para gestionar procesos limitados.

**Día 3:** Se añadieron validaciones para evitar errores.

**Día 4:** Se realizaron pruebas, se documentaron errores encontrados y se terminó la documentación.

---

## 



## Reflexión personal

A pesar de que para este curso tuve muy poco tiempo y no pude dedicarle el tiempo que me hubiese gustado, todo lo que aprendí me sirvió bastante, empezando porque familiarizarme mas con el kernel de linux ya que en su momento tuve que usar otras versiones de kernel para poder correr los drivers de mi computadora laptop, lo cual en ese momento no entendía bien lo que estaba modificando o como funcionaba, pero ahora lo entiendo de una mejor manera, también la guía para solucionar los problemas que tenía en mi laptop con hyper-visor me sirvió demasiado ya que también fue un problema que no había logrado solucionar cuando se me presentó.
