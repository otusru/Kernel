# 🧩 МОДУЛЬ 5: Модули ядра Linux (LKM) — написание драйверов и расширений ядра

---

## 🎯 Цель модуля:

* Понять, что такое модуль ядра и зачем он нужен;
* Написать, собрать и загрузить простой модуль (драйвер);
* Познакомиться с жизненным циклом модуля (init, exit);
* Освоить интерфейсы регистрации устройств, работы с `/proc` и `/dev`.

---

## 🧠 Что такое LKM?

**Loadable Kernel Module (LKM)** — это часть кода, которую можно **динамически загружать** и выгружать в ядро Linux без его перезапуска.

### Примеры:

* Драйвер сетевой карты
* Виртуальное устройство
* Расширение системных вызовов

---

## 🧪 Пример 1: Простейший модуль ядра

### 📁 Файл: `hello.c`

```c
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Ты");
MODULE_DESCRIPTION("Простой модуль ядра");

static int __init hello_init(void) {
    printk(KERN_INFO "Привет из ядра!\n");
    return 0;
}

static void __exit hello_exit(void) {
    printk(KERN_INFO "Пока, ядро!\n");
}

module_init(hello_init);
module_exit(hello_exit);
```

---

### 🛠 Makefile для сборки

```makefile
obj-m += hello.o

all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

---

### 🔧 Сборка и запуск

```bash
make                      # сборка
sudo insmod hello.ko      # загрузка
dmesg | tail              # просмотр лога ядра
sudo rmmod hello          # выгрузка
```

---

## 🧪 Пример 2: Модуль с параметрами

```c
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>

static int value = 0;
module_param(value, int, 0); // Передача параметра при загрузке
MODULE_PARM_DESC(value, "Целочисленный параметр");

static int __init mod_init(void) {
    printk(KERN_INFO "Значение value = %d\n", value);
    return 0;
}

static void __exit mod_exit(void) {
    printk(KERN_INFO "Выход из модуля\n");
}

module_init(mod_init);
module_exit(mod_exit);
MODULE_LICENSE("GPL");
```

Загрузить:

```bash
sudo insmod modparam.ko value=42
```

---

## 🧪 Пример 3: Простое виртуальное устройство в `/proc`

```c
#include <linux/proc_fs.h>
#include <linux/uaccess.h>

#define ENTRY_NAME "myproc"
static struct proc_dir_entry *entry;

ssize_t read_proc(struct file *file, char __user *buf, size_t count, loff_t *ppos) {
    char *msg = "Hello from /proc!\n";
    return simple_read_from_buffer(buf, count, ppos, msg, strlen(msg));
}

static struct file_operations fops = {
    .owner = THIS_MODULE,
    .read = read_proc,
};

static int __init proc_init(void) {
    entry = proc_create(ENTRY_NAME, 0, NULL, &fops);
    return 0;
}

static void __exit proc_exit(void) {
    proc_remove(entry);
}

module_init(proc_init);
module_exit(proc_exit);
MODULE_LICENSE("GPL");
```

После загрузки модуля:

```bash
cat /proc/myproc
```

---

## 🧠 Структура модуля

| Компонент       | Назначение                                    |
| --------------- | --------------------------------------------- |
| `module_init()` | Функция, вызываемая при загрузке              |
| `module_exit()` | Функция, вызываемая при выгрузке              |
| `printk()`      | Логгирование (аналог `printf()` в user space) |
| `MODULE_…`      | Метаданные для ядра                           |
| `Makefile`      | Сборка через `kbuild`                         |

---

## 🧠 Где модуль появляется в системе?

* Загруженные модули: `lsmod`
* Журнал ядра: `dmesg`
* Файл устройств: `/proc/devices`, `/proc/modules`
* Виртуальные устройства: `/dev/`, `/sys/class/`

---

## 📦 Практика

1. Напиши свой модуль, который:

   * Создает файл в `/proc/`;
   * Возвращает текущее значение счетчика;
   * Обновляет значение при чтении;
2. Поэкспериментируй с `module_param()` — передавай значения при загрузке;
3. Посмотри, как работают реальные модули:

   ```bash
   modinfo e1000e
   ```

---

## ✅ Результаты по окончанию модуля:

* Ты знаешь, как создавать модули ядра;
* Умеешь загружать и выгружать LKM;
* Понимаешь интерфейс между user и kernel space через `/proc`;
* Готов перейти к следующему шагу — **созданию собственного ядра ОС с модульной архитектурой** (Модуль 6).
