# 🛠️ МОДУЛЬ 6: Создание собственного ядра ОС — архитектура и стартовая точка

---

## 🎯 Цель модуля:

* Разработать минимальное ядро с нуля;
* Понять, как загружается ядро и как взаимодействовать с "железом";
* Освоить работу с GRUB, multiboot и написание стартового кода;
* Построить архитектуру модульного ядра.

---

## 🚀 Что такое "свое ядро"?

Собственное ядро — это самостоятельная программа, которая:

* Загружается BIOS/UEFI → через GRUB → в память;
* Инициализирует экран, память, устройства;
* Запускает планировщик, обработчики прерываний и файловую систему;
* Может быть **монолитным**, **модульным**, **микроядерным** (по твоему выбору).

---

## 📦 Структура твоей ОС

```text
mykernel/
├── boot/             ← Загрузочный код GRUB
│   └── grub.cfg
├── kernel/           ← Исходники ядра
│   ├── main.c
│   ├── screen.c
│   ├── memory.c
│   └── modules/      ← Подключаемые модули
├── lib/              ← Библиотеки: строковые, память, отладка
├── include/          ← Заголовочные файлы
├── Makefile          ← Сборка ядра
└── myos.iso          ← ISO-образ для QEMU
```

---

## 📁 Базовый проект ядра

### 1. 🎬 Начальная точка: `main.c`

```c
#include "screen.h"

void kernel_main(void) {
    clear_screen();
    print("Добро пожаловать в MyOS Kernel!\n");
}
```

---

### 2. 💻 Вывод текста: `screen.c`

```c
#include <stdint.h>
#include "screen.h"

#define VIDEO_MEMORY (uint16_t*)0xb8000

void print(const char *msg) {
    uint16_t *video = VIDEO_MEMORY;
    while (*msg) {
        *video++ = (*msg++ | 0x0700); // символ + цвет
    }
}

void clear_screen() {
    for (int i = 0; i < 80 * 25; i++) {
        VIDEO_MEMORY[i] = 0x0720; // пробел
    }
}
```

---

### 3. 🧠 Загрузчик GRUB (Multiboot)

#### `grub.cfg`

```cfg
set timeout=0
set default=0

menuentry "My OS" {
    multiboot /boot/kernel.bin
    boot
}
```

---

### 4. 🧱 Makefile + компиляция

```makefile
ISO=myos.iso

all: kernel.bin
	mkdir -p isodir/boot/grub
	cp kernel.bin isodir/boot/
	cp grub.cfg isodir/boot/grub/
	grub-mkrescue -o $(ISO) isodir

kernel.bin: kernel/main.c kernel/screen.c
	i686-elf-gcc -ffreestanding -m32 -c kernel/main.c -o main.o
	i686-elf-gcc -ffreestanding -m32 -c kernel/screen.c -o screen.o
	i686-elf-ld -T linker.ld -o kernel.bin -nostdlib main.o screen.o

clean:
	rm -rf *.o *.bin isodir $(ISO)
```

---

### 5. 🔗 Linker Script: `linker.ld`

```ld
ENTRY(kernel_main)

SECTIONS {
    . = 0x100000;
    .text : {
        *(.text)
    }
    .data : {
        *(.data)
    }
    .bss : {
        *(.bss COMMON)
    }
}
```

---

## 🧪 Запуск в QEMU

```bash
make
qemu-system-i386 -cdrom myos.iso
```

---

## 🧩 Расширение: Модули твоего ядра

Ты можешь построить свою модульную архитектуру (как в Linux):

```c
typedef struct KernelModule {
    const char *name;
    void (*init)();
    void (*shutdown)();
} KernelModule;
```

Пример подключения модуля:

```c
void register_module(KernelModule *mod) {
    print("Загрузка модуля: ");
    print(mod->name);
    print("\n");
    mod->init();
}
```

---

## 📘 Ресурсы

* [osdev.org](https://wiki.osdev.org) — энциклопедия по созданию ОС
* [Writing a Simple Kernel](https://github.com/cfenollosa/os-tutorial)
* [GRUB Multiboot Spec](https://www.gnu.org/software/grub/manual/multiboot/)
* [Book: Operating Systems: From 0 to 1](https://github.com/tuhdo/os01)

---

## ✅ Результаты по окончании модуля:

* Ты собрал своё ядро с выводом на экран;
* Знаешь, как работает boot → kernel\_main();
* Готов расширять ядро: ввод, память, файловая система, многозадачность;
* Умеешь строить модульную архитектуру с собственными интерфейсами.
