# Лабораторная работа №1
## Исследование компилятора GCC и анализ ассемблерного кода

---

# 1. Создание программы на C++

## Исходный файл программы

Создаем файл `main.cpp` и добавляем в него код программы для вычисления факториала числа.

```cpp
#include <iostream>

long long factorial(int n)
{
    if (n <= 1)
        return 1;

    return n * factorial(n - 1);
}

int main()
{
    int value;

    std::cout << "number: ";
    std::cin >> value;

    std::cout << "factorial = "
              << factorial(value)
              << std::endl;
}
```

### Скриншот исходного кода:
![Код программы](images/lab1.png)

---

## Компиляция и запуск программы

Для компиляции программы используется команда:

```bash
g++ main.cpp -o factorial
```

Запуск программы:

```bash
./factorial
```

### Результат компиляции и запуска:
![Результат](images/lab2.png)

---

# 2. Генерация ассемблерного кода

## Компиляция без оптимизации (-O0)

Создаем ассемблерный файл без оптимизации:

```bash
g++ -O0 -S main.cpp -o main_O0.s
```

### Ассемблерный код без оптимизации:
```
	.file	"main.cpp"                     # Указывает исходный файл программы
	.text                                   # Начало секции машинного кода

	.globl	_Z9factoriali                 # Делает функцию factorial глобальной
	.def	_Z9factoriali;	.scl	2;	.type	32;	.endef   # Описание символа функции

	.seh_proc	_Z9factoriali                # Начало описания функции для Windows SEH
_Z9factoriali:
.LFB2623:                                 # Метка начала функции

	pushq	%rbp                           # Сохраняем базовый указатель стека
	.seh_pushreg	%rbp                   # Информация для обработчика исключений

	pushq	%rbx                           # Сохраняем регистр RBX
	.seh_pushreg	%rbx                   # Информация для SEH

	subq	$40, %rsp                      # Выделяем 40 байт памяти в стеке
	.seh_stackalloc	40                  # Описание выделения памяти

	leaq	32(%rsp), %rbp                # Настраиваем новый стековый кадр
	.seh_setframe	%rbp, 32             # Устанавливаем frame pointer

	.seh_endprologue                  # Конец пролога функции

	movl	%ecx, 32(%rbp)                # Сохраняем аргумент n в стек

	cmpl	$1, 32(%rbp)                  # Сравниваем n с 1
	jg	.L2                             # Если n > 1 — переход к рекурсии

	movl	$1, %eax                       # Возвращаем 1
	jmp	.L3                             # Переход к завершению функции

.L2:
	movl	32(%rbp), %eax                # Загружаем n в eax
	movslq	%eax, %rbx                   # Расширяем eax до 64 бит и сохраняем в rbx

	movl	32(%rbp), %eax                # Снова загружаем n
	subl	$1, %eax                      # Вычисляем n - 1

	movl	%eax, %ecx                    # Передаем n-1 как аргумент
	call	_Z9factoriali                 # Рекурсивный вызов factorial(n - 1)

	imulq	%rbx, %rax                   # Умножаем результат на n

.L3:
	addq	$40, %rsp                     # Освобождаем память стека

	popq	%rbx                           # Восстанавливаем регистр RBX
	popq	%rbp                           # Восстанавливаем базовый указатель

	ret                                   # Возврат из функции

	.seh_endproc                         # Конец описания функции

	.section .rdata,"dr"                 # Секция константных данных

.LC0:
	.ascii "number: \0"                  # Строка "number: "

.LC1:
	.ascii "factorial = \0"              # Строка "factorial = "

	.text                                 # Возврат в секцию кода

	.globl	main                           # Глобальная функция main

	.def	main;	.scl	2;	.type	32;	.endef   # Описание функции main

	.seh_proc	main                        # Начало функции main

main:
.LFB2624:

	pushq	%rbp                           # Сохраняем rbp
	.seh_pushreg	%rbp

	pushq	%rbx                           # Сохраняем rbx
	.seh_pushreg	%rbx

	subq	$56, %rsp                      # Выделяем память под локальные переменные
	.seh_stackalloc	56

	leaq	48(%rsp), %rbp                # Настраиваем frame pointer
	.seh_setframe	%rbp, 48

	.seh_endprologue                  # Конец пролога

	call	__main                         # Инициализация среды C++

	leaq	.LC0(%rip), %rdx              # Загружаем адрес строки "number: "

	movq	.refptr._ZSt4cout(%rip), %rax # Получаем cout
	movq	%rax, %rcx                    # Передаем cout

	call	_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc
                                            # cout << "number: "

	leaq	-4(%rbp), %rdx                # Адрес переменной number

	movq	.refptr._ZSt3cin(%rip), %rax  # Получаем cin
	movq	%rax, %rcx                    # Передаем cin

	call	_ZNSirsERi                    # cin >> number

	leaq	.LC1(%rip), %rdx              # Загружаем строку "factorial = "

	movq	.refptr._ZSt4cout(%rip), %rax # Получаем cout
	movq	%rax, %rcx

	call	_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc
                                            # cout << "factorial = "

	movq	%rax, %rbx                    # Сохраняем поток вывода

	movl	-4(%rbp), %eax                # Загружаем number
	movl	%eax, %ecx                    # Передаем аргумент в factorial

	call	_Z9factoriali                 # Вызов factorial(number)

	movq	%rax, %rdx                    # Результат factorial
	movq	%rbx, %rcx                    # cout

	call	_ZNSolsEx                     # Вывод результата

	movq	%rax, %rcx                    # cout

	movq	.refptr._ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_(%rip), %rax
                                            # Получаем std::endl

	movq	%rax, %rdx                    # Передаем endl

	call	_ZNSolsEPFRSoS_E              # cout << endl

	movl	$0, %eax                      # Возвращаем 0

	addq	$56, %rsp                     # Освобождаем стек

	popq	%rbx                           # Восстанавливаем rbx
	popq	%rbp                           # Восстанавливаем rbp

	ret                                   # Завершение main

	.seh_endproc                         # Конец main

	.def	__main;	.scl	2;	.type	32;	.endef   # Объявление __main
```

В данной версии присутствует большое количество дополнительных инструкций, связанных с организацией стека и вызовами функций.

---

## Компиляция с оптимизацией (-O2)

Создаем оптимизированную версию ассемблерного кода:

```bash
g++ -O2 -S main.cpp -o main_O2.s
```

### Ассемблерный код с оптимизацией:
```
	.file	"main.cpp"                          # Исходный файл программы
	.text                                        # Начало секции кода

	.section	.text$_ZNKSt5ctypeIcE8do_widenEc,"x"
                                                   # Секция для функции do_widen

	.linkonce discard                             # Удалять дубликаты при линковке

	.align 2                                      # Выравнивание по 2 байта
	.p2align 4                                    # Выравнивание по степени двойки

	.globl	_ZNKSt5ctypeIcE8do_widenEc         # Глобальная функция do_widen

	.def	_ZNKSt5ctypeIcE8do_widenEc;	.scl	2;	.type	32;	.endef
                                                   # Описание символа функции

	.seh_proc	_ZNKSt5ctypeIcE8do_widenEc       # Начало процедуры SEH

_ZNKSt5ctypeIcE8do_widenEc:
.LFB2410:

	.seh_endprologue                         # Конец пролога функции

	movl	%edx, %eax                            # Копируем символ в eax
	ret                                          # Возврат из функции

	.seh_endproc                              # Конец процедуры

	.text                                        # Возврат к секции кода

	.p2align 4                                 # Выравнивание

	.globl	_Z9factoriali                       # Глобальная функция factorial

	.def	_Z9factoriali;	.scl	2;	.type	32;	.endef
                                                   # Описание factorial

	.seh_proc	_Z9factoriali                    # Начало factorial

_Z9factoriali:
.LFB2666:

	.seh_endprologue                         # Конец пролога

	movl	%ecx, %eax                            # Копируем аргумент n в eax

	cmpl	$1, %ecx                              # Сравниваем n с 1
	jle	.L13                                    # Если n <= 1 — переход

	movl	%eax, %ecx                            # ecx = n
	movq	%rax, %rdx                            # rdx = n

	subq	$1, %rax                              # n - 1

	cmpl	$1, %eax                              # Проверка n-1
	jle	.L3                                     # Если <=1, переход

	andl	$1, %ecx                              # Проверка четности числа
	je	.L6                                     # Если четное — переход

	imulq	%rax, %rdx                           # result *= n-1

	subq	$1, %rax                              # Уменьшаем счетчик

	cmpl	$1, %eax                              # Проверка
	jle	.L3                                     # Переход если <=1

	.p2align 5
	.p2align 4
	.p2align 3

.L6:
	imulq	%rax, %rdx                           # Умножение result *= значение

	leaq	-1(%rax), %rcx                       # rcx = rax - 1

	subq	$2, %rax                              # rax -= 2

	imulq	%rcx, %rdx                           # Дополнительное умножение

	cmpl	$1, %eax                              # Проверяем условие цикла
	jg	.L6                                     # Продолжаем цикл

.L3:
	movq	%rdx, %rax                            # Возвращаем результат
	ret                                          # Выход из factorial

	.p2align 4,,10
	.p2align 3

.L13:
	movl	$1, %edx                              # factorial(0/1)=1
	movq	%rdx, %rax                            # Возврат 1
	ret                                          # Выход

	.seh_endproc                              # Конец factorial

	.section .rdata,"dr"                      # Секция констант

.LC0:
	.ascii "number: \0"                       # Строка "number: "

.LC1:
	.ascii "factorial = \0"                   # Строка "factorial = "

	.section	.text.startup,"x"              # Секция startup-кода

	.p2align 4                                 # Выравнивание

	.globl	main                                # Глобальная функция main

	.def	main;	.scl	2;	.type	32;	.endef
                                                   # Описание main

	.seh_proc	main                             # Начало main

main:
.LFB2667:

	pushq	%rbx                                  # Сохраняем rbx
	.seh_pushreg	%rbx

	subq	$64, %rsp                             # Выделяем память стека

	.seh_stackalloc	64                       # Описание выделения

	.seh_endprologue                         # Конец пролога

	call	__main                                # Инициализация среды C++

	movq	.refptr._ZSt4cout(%rip), %rbx        # Загружаем cout

	movl	$8, %r8d                              # Длина строки

	leaq	.LC0(%rip), %rdx                     # Адрес строки "number: "

	movq	%rbx, %rcx                            # cout -> rcx

	call	_ZSt16__ostream_insertIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_PKS3_x
                                                   # Вывод строки

	movq	.refptr._ZSt3cin(%rip), %rcx         # Загружаем cin

	leaq	60(%rsp), %rdx                       # Адрес переменной

	call	_ZNSirsERi                           # cin >> number

	leaq	.LC1(%rip), %rdx                     # Адрес строки "factorial = "

	movl	$12, %r8d                             # Длина строки

	movq	%rbx, %rcx                            # cout

	call	_ZSt16__ostream_insertIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_PKS3_x
                                                   # Вывод строки

	movl	60(%rsp), %eax                       # Загружаем number

	movq	%rax, %rdx                            # result = number

	cmpl	$1, %eax                              # Проверяем number
	jle	.L29                                    # Если <=1

	movl	%eax, %ecx                            # ecx = number

	subq	$1, %rax                              # number-1

	cmpl	$1, %eax                              # Проверка
	jle	.L20                                    # Переход

	andl	$1, %ecx                              # Проверка четности

	je	.L21                                    # Переход

	imulq	%rax, %rdx                           # result *= значение

	subq	$1, %rax                              # Уменьшение

	cmpl	$1, %eax                              # Проверка
	jle	.L20                                    # Переход

	.p2align 5
	.p2align 4
	.p2align 3

.L21:
	imulq	%rax, %rdx                           # Умножение

	leaq	-1(%rax), %rcx                       # rcx = rax-1

	subq	$2, %rax                              # rax -= 2

	imulq	%rcx, %rdx                           # Дополнительное умножение

	cmpl	$1, %eax                              # Проверка цикла
	jg	.L21                                    # Повтор

.L20:
	movq	%rbx, %rcx                            # cout

	call	_ZNSo9_M_insertIxEERSoT_             # Вывод результата

	movq	%rax, %r8                             # Сохраняем поток

	movq	(%rax), %rax                          # Получаем vtable

	movq	-24(%rax), %rax                       # Смещение

	movq	240(%r8,%rax), %rcx                  # Получаем ctype

	testq	%rcx, %rcx                            # Проверка на null

	je	.L32                                    # Если null — ошибка

	cmpb	$0, 56(%rcx)                          # Проверка инициализации

	je	.L18                                    # Если не инициализировано

	movsbl	67(%rcx), %edx                       # Получаем символ '\n'

.L19:
	movq	%r8, %rcx                             # cout

	call	_ZNSo3putEc                          # cout.put()

	movq	%rax, %rcx

	call	_ZNSo5flushEv                        # flush()

	xorl	%eax, %eax                            # return 0

	addq	$64, %rsp                             # Освобождение стека

	popq	%rbx                                  # Восстановление rbx

	ret                                          # Выход из main

.L18:
	movq	%r8, 40(%rsp)                        # Сохраняем значения

	movq	%rcx, 32(%rsp)

	call	_ZNKSt5ctypeIcE13_M_widen_initEv     # Инициализация widen

	movq	32(%rsp), %rcx

	movq	40(%rsp), %r8

	leaq	_ZNKSt5ctypeIcE8do_widenEc(%rip), %r9
                                                   # Адрес do_widen

	movl	$10, %edx                             # Символ '\n'

	movq	(%rcx), %rax                          # vtable

	movq	48(%rax), %rax                        # Метод widen

	cmpq	%r9, %rax                             # Сравнение функций

	je	.L19                                    # Если совпадает

	movq	%r8, 32(%rsp)

	movl	$10, %edx

	call	*%rax                                 # Вызов widen

	movq	32(%rsp), %r8

	movsbl	%al, %edx                             # Символ результата

	jmp	.L19                                    # Переход к выводу

.L29:
	movl	$1, %edx                              # factorial=1

	jmp	.L20                                    # Переход к выводу

.L32:
	call	_ZSt16__throw_bad_castv              # Генерация bad_cast

	nop                                          # Пустая инструкция

	.seh_endproc                              # Конец main
```

После оптимизации структура программы изменилась. 
Компилятор заменил часть рекурсивных вызовов более эффективными инструкциями и циклами, что повысило производительность программы.
---

# 3. Создание Makefile

Создаем файл `Makefile` для автоматической сборки проекта.

```Makefile
all:
	g++ main.cpp -o factorial

optimized:
	g++ -O2 main.cpp -o factorial_opt

clean:
	del factorial.exe factorial_opt.exe
```
## Выполнение сборки
Запускаем команду:

```bash
make
```

### Результат выполнения make:
![Результат](images/lab3.png)

---

# 4. Улучшение программы

## Добавление многопоточности

Для повышения производительности в программу была добавлена поддержка потоков.

Пример использования потоков:

```cpp
#include <iostream>
#include <thread>

long long factorial(int n)
{
    if (n <= 1)
        return 1;

    return n * factorial(n - 1);
}

void task()
{
    std::cout << "Thread is open" << std::endl;
}

int main()
{
    int number;

    std::cout << "Number: ";
    std::cin >> number;

    std::thread t(task);

    t.join();

    std::cout << "Factorial = "
              << factorial(number)
              << std::endl;

    return 0;
}
```

![Результат](images/lab4.png)
---

## Обновленный Makefile

После добавления потоков файл Makefile был изменен.

### Новый Makefile:
```
all:
	g++ main.cpp -o factorial -pthread

optimized:
	g++ -O2 main.cpp -o factorial_opt -pthread

clean:
	del factorial.exe factorial_opt.exe
```
---

# Вывод

В ходе выполнения лабораторной работы была изучена работа компилятора GCC. Была создана программа на языке C++, выполнена генерация ассемблерного кода с различными уровнями оптимизации, а также настроена автоматическая сборка проекта с помощью Makefile. Дополнительно была рассмотрена работа многопоточности в C++.
