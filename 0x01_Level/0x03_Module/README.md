# 🧠 МОДУЛЬ 3: Язык C для системного программирования (разработка ядра)

---

## 🎯 Цель модуля:

* Освоить ключевые особенности C, применимые в системном программировании и разработке ядра;
* Понять, как управлять памятью, работать с указателями и структурами;
* Получить готовые примеры кода, близкие к тем, что используются в ядре Linux и микроядрах.

---

## 🔧 Почему C?

Язык C — это низкоуровневый язык с доступом к памяти, прямой работой с адресами и битами, без автоматического управления памятью, что делает его идеальным для создания операционных систем.

---

## 📘 Базовые темы C, которые нужно знать

---

### ✅ 1. Работа с указателями

Указатели — сердце системного программирования.

```c
int x = 42;
int *ptr = &x;
printf("%d\n", *ptr);  // Выведет: 42
```

**Применение в ядре:**

* Обращение к физической памяти
* Передача структур между функциями без копирования

---

### ✅ 2. Работа с памятью: `malloc`, `free`, `memcpy`

Хотя в ядре Linux используется свой аллокатор (`kmalloc`, `kfree`), принцип такой же.

```c
int *arr = malloc(sizeof(int) * 10);
arr[0] = 123;
free(arr);
```

**В ядре:**

```c
int *arr = kmalloc(sizeof(int) * 10, GFP_KERNEL);
kfree(arr);
```

---

### ✅ 3. Структуры и объединения

```c
struct Process {
    int pid;
    char name[16];
};

struct Process p = { .pid = 1, .name = "init" };
```

**В ядре:**

* Все объекты — структуры: `task_struct`, `file`, `inode` и др.

---

### ✅ 4. Макросы и препроцессор

Макросы используются повсеместно:

```c
#define PAGE_SIZE 4096
#define MAX(a,b) ((a) > (b) ? (a) : (b))
```

**Условия компиляции:**

```c
#ifdef DEBUG
    printk("Debug info\n");
#endif
```

---

### ✅ 5. Статические и inline-функции

```c
static inline int max(int a, int b) {
    return a > b ? a : b;
}
```

**Важно:** `static` ограничивает область видимости функции внутри файла ядра.

---

### ✅ 6. Перечисления и битовые флаги

```c
enum ProcessState { READY, RUNNING, BLOCKED };

#define FLAG_READ   0x1
#define FLAG_WRITE  0x2
#define FLAG_EXEC   0x4
```

Флаги удобно объединять:

```c
int perms = FLAG_READ | FLAG_WRITE;
if (perms & FLAG_WRITE) { ... }
```

---

### ✅ 7. Ввод/вывод (низкоуровневый)

```c
#include <unistd.h>
write(1, "Hello\n", 6);
```

В ядре нет `printf`, есть `printk`:

```c
printk(KERN_INFO "Kernel log\n");
```

---

## 🛠️ Практическое упражнение

Создай файл `kernel_style.c` и напиши пример ядроподобной структуры:

```c
#include <stdio.h>
#include <string.h>

#define TASK_NAME_LEN 16

struct task_struct {
    int pid;
    char name[TASK_NAME_LEN];
    int state;
};

void show_task(const struct task_struct *task) {
    printf("PID: %d, Name: %s, State: %d\n", task->pid, task->name, task->state);
}

int main() {
    struct task_struct init_task = { .pid = 0, .name = "init", .state = 1 };
    show_task(&init_task);
    return 0;
}
```

---

## 📘 Дополнительные ресурсы

* 📚 **The C Programming Language (K\&R)** — классика
* 🧠 [Learn C the Hard Way](https://learncodethehardway.org/c/)
* 🧰 [Linux Kernel C Coding Style](https://www.kernel.org/doc/html/latest/process/coding-style.html)
* 🧪 [Godbolt Compiler Explorer](https://godbolt.org/) — смотри, как C превращается в ASM

---

## ✅ Результаты модуля:

* Ты уверенно работаешь с указателями, структурами, макросами;
* Понимаешь, как это применимо в ядре Linux и Minix;
* Готов перейти к следующему этапу — **архитектуре ядра Linux** и его компонентам (модуль 4).
