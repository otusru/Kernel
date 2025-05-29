# 🧠 МОДУЛЬ 7: Реализация подсистем ядра — планировщик, память, драйверы

---

## 🎯 Цель модуля:

* Реализовать базовый **планировщик задач** (многозадачность);
* Написать **управление памятью**: страничная и физическая;
* Настроить **обработку прерываний (IRQ)** и **драйверы устройств (клавиатура, таймер)**;
* Построить ядро, которое может запускать задачи, обрабатывать ввод и распределять ресурсы.

---

## 🧩 Структура ядра на этом этапе

```text
mykernel/
├── kernel/
│   ├── main.c             ← запуск ядра
│   ├── memory/            ← аллокаторы, карта памяти
│   ├── sched/             ← планировщик задач
│   ├── drivers/           ← устройства: timer, keyboard
│   └── isr/               ← прерывания (ISR/IRQ)
├── lib/                   ← memset, memcpy, string
├── include/
│   ├── kernel.h           ← API ядра
│   ├── memory.h
│   ├── isr.h
│   └── sched.h
```

---

## 🔹 Часть 1: Базовый планировщик (cooperative multitasking)

### Что нужно:

* Простая очередь задач (task queue);
* Переключение контекста вручную;
* `task_create(func)` — добавляет задачу в очередь;
* `schedule()` — переключает на следующую задачу.

```c
typedef struct task {
    uint32_t *stack;
    void (*entry_point)(void);
    struct task *next;
} task_t;

task_t *current_task;
task_t *task_queue;
```

### Пример задач:

```c
void task1() {
    while (1) print("Task 1\n");
}
void task2() {
    while (1) print("Task 2\n");
}
```

---

## 🔹 Часть 2: Управление памятью

### Этапы:

1. **Физическая память**:

   * Битмап или массив фреймов;
   * `alloc_frame()`, `free_frame()`.

2. **Страничная память (paging)**:

   * Таблицы страниц (page table);
   * Переключение адресов через `CR3`.

```c
// page_entry_t, page_directory_t
void paging_init();
void map_page(uint32_t virt, uint32_t phys);
```

3. **Аллокатор ядра (heap)**:

   * Простая реализация malloc/free:

     * bump allocator
     * buddy system

---

## 🔹 Часть 3: Прерывания (IRQ, ISR)

### Что нужно:

1. Реализовать IDT (таблицу дескрипторов прерываний);
2. Обработчики ISR (0–31) и IRQ (32–47);
3. Обработка клавиатуры и таймера.

```c
void isr_install();        // Регистрация всех ISR
void irq_install();        // Регистрация IRQ + переопределение PIC
void irq_set_handler(int n, handler_func);
```

### Пример: таймер

```c
void timer_callback() {
    ticks++;
    schedule(); // переключение задач
}
```

---

## 🔹 Часть 4: Драйверы устройств (ввод/вывод)

### Таймер (IRQ0)

```c
void timer_init(uint32_t frequency) {
    // настройка PIT через порты 0x40, 0x43
}
```

### Клавиатура (IRQ1)

```c
void keyboard_handler() {
    uint8_t scancode = inb(0x60);
    print(scancode_to_ascii(scancode));
}
```

---

## 🔹 Пример запуска ОС с задачами и вводом

```c
void kernel_main() {
    isr_install();         // прерывания
    irq_install();         // IRQ + PIC
    timer_init(50);        // 50 Гц
    keyboard_install();    // обработка ввода
    paging_init();         // виртуальная память
    heap_init();           // аллокатор

    task_create(task1);
    task_create(task2);
    while (1);
}
```

---

## 📘 Что изучить дополнительно

* [Paging (OSDev)](https://wiki.osdev.org/Paging)
* [Task Switching (OSDev)](https://wiki.osdev.org/Task_Switching)
* [Interrupts (OSDev)](https://wiki.osdev.org/Interrupts)
* [Keyboard Driver (OSDev)](https://wiki.osdev.org/PS2_Keyboard)

---

## ✅ Результаты модуля:

* Ты построил **ядро, способное выполнять несколько задач**;
* У тебя есть **виртуальная память** и **аллокатор памяти**;
* Работает **обработка прерываний и ввод с клавиатуры**;
* Ты готов к следующему шагу: **файловая система и запуск пользовательских программ**.

---
