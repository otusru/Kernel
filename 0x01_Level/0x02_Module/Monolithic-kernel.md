## 🐧 Linux: Монолитное ядро

### Архитектура Linux

Linux — это монолитное ядро, что означает, что вся операционная система работает в привилегированном режиме (kernel mode). Оно включает в себя:

* **Управление процессами**: планировщик задач, синхронизация, межпроцессное взаимодействие.
* **Управление памятью**: виртуальная память, страничная адресация, кэширование.
* **Файловая система**: абстракция устройств хранения, поддержка различных файловых систем.
* **Управление устройствами**: драйверы устройств, обработка прерываний.

### Пример исходного кода

Рассмотрим файл `main.c` из ядра Linux, который является точкой входа в систему:

```c
// main.c

#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Author");
MODULE_DESCRIPTION("A simple Linux driver");

static int __init hello_init(void) {
    printk(KERN_INFO "Hello, World!\n");
    return 0;
}

static void __exit hello_exit(void) {
    printk(KERN_INFO "Goodbye, World!\n");
}

module_init(hello_init);
module_exit(hello_exit);
```

Этот код представляет собой минимальный модуль ядра, который выводит сообщение при загрузке и выгрузке модуля.

### Сборка и тестирование

Для сборки и тестирования модуля:

1. **Установите необходимые пакеты**:

   ```bash
   sudo apt-get install build-essential linux-headers-$(uname -r)
   ```

2. **Создайте Makefile**:

   ```makefile
   obj-m += hello.o

   all:
       make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

   clean:
       make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
   ```

3. **Соберите модуль**:

   ```bash
   make
   ```

4. **Загрузите модуль в ядро**:

   ```bash
   sudo insmod hello.ko
   ```

5. **Проверьте вывод в журнале ядра**:

   ```bash
   dmesg | tail
   ```

6. **Выгрузите модуль**:

   ```bash
   sudo rmmod hello
   ```

---

## 🦝 Minix 3: Микроядро

### Архитектура Minix 3

Minix 3 — это микроядро, где минимальный набор функций выполняется в привилегированном режиме, а остальные компоненты работают в пользовательском пространстве. Это повышает надежность и безопасность системы.

Основные компоненты:

* **Микроядро**: управление процессами, памятью и коммуникацией между процессами (IPC).
* **Сервисы**: драйверы устройств, файловая система, сеть — реализованы как отдельные процессы в пользовательском пространстве.

### Пример исходного кода

Рассмотрим файл `main.c` из ядра Minix 3:

```c
// main.c

#include "kernel.h"

int main(void) {
    printf("Hello from Minix 3 kernel!\n");
    return 0;
}
```

Этот код представляет собой точку входа в ядро Minix 3, где инициализируются основные компоненты системы.

### Сборка и тестирование

Для сборки и тестирования Minix 3:

1. **Клонируйте репозиторий**:

   ```bash
   git clone https://github.com/o-oconnell/minixfromscratch.git
   cd minixfromscratch
   ```

2. **Соберите систему**:

   ```bash
   make
   ```

3. **Запустите в эмуляторе QEMU**:

   ```bash
   qemu-system-i386 -kernel minix-3.1.0/kernel
   ```

---

## 🔍 Сравнение Linux и Minix 3

| Характеристика           | Linux (монолит)              | Minix 3 (микроядро)                    |
| ------------------------ | ---------------------------- | -------------------------------------- |
| **Архитектура**          | Все в ядре                   | Минимум в ядре, остальное в user space |
| **Надежность**           | Зависит от стабильности ядра | Высокая из-за изоляции компонентов     |
| **Производительность**   | Высокая                      | Немного ниже из-за IPC                 |
| **Сложность разработки** | Высокая                      | Средняя                                |

---

## 📚 Ресурсы для дальнейшего изучения

* **Linux Kernel**: [https://github.com/torvalds/linux](https://github.com/torvalds/linux)
* **Minix 3**: [https://github.com/o-oconnell/minixfromscratch](https://github.com/o-oconnell/minixfromscratch)
* **Руководство по разработке ядра Linux**: [https://www.kernel.org/doc/html/latest/](https://www.kernel.org/doc/html/latest/)
* **Книга "Linux Kernel Development"**: [https://www.amazon.com/Linux-Kernel-Development-Robert-Love/dp/0672329468](https://www.amazon.com/Linux-Kernel-Development-Robert-Love/dp/0672329468)
