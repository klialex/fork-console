## THIS REPOSITORY IS NO LONGER MAINTAINED

[This fork](https://github.com/bgomberg/anchor) should be considered the primary fork at this point and has a few changes / fixes versus what's in this repository and is being actively maintained by the original author of this repository.

# Anchor

Anchor is a collection of various embedded firmware libraries developed by Skip
Transport, Inc. These libraries were all developed to be easily re-usable and
not dependant on a specific MCU or RTOS.

# Libraries

This repository is organized as a collection of libraries which are largely
independent of each other. You are free to pick and chose the libraries which
are most useful in your application, without needing to include them all. Each
library is organized within a subdirectory, with a README providing information
specific to the library.

In general, including a library is as simple as adding all the `.c` files
within `<lib>/src` to your build system as C sources to be compiled, and adding
`<lib>/include` to your build system's include path.

## Logging

The logging library makes it easy to write out consistent and useful debug
logs. This is often done over a UART connection, but the library is agnostic to
the protocol being used.

```
  0:00:00.626 WARN  system.c:199: Last reset due to software reset
  0:00:00.632 INFO  system.c:243: Serial number is 003b001b524b501020353548
  0:00:00.639 INFO  system.c:257: Detected DVT hardware
  0:00:00.648 INFO  piezo.c:45: is_present=1
  0:00:16.732 ERROR SONAR:link_layer.c:161: Invalid packet: Response sequence number does not match request
```

## FSM

The FSM library is a very lightweight set of macros and functions for defining
event-based finite state machines with enter/exit handlers for each state. See
the README within the `fsm` subdirectory for more information and an example.

## Console

The console library can be used to provide a debug interface to your device.
The library is designed to be highly configurable to allow the user to decide
between more features and code-space / runtime efficiencies.

```
> help
Available commands:
  help     - List all commands, or give details about a specific command
  gpio_set - Sets a GPIO pin
  gpio_get - Gets a GPIO pin value
> help gpio_get
Gets a GPIO pin value
Usage: gpio_get port pin
  port - The port <A,B,C>
  pin  - The pin <0-15>
> gpio_set A 1 1
> gpio_get A 1
value=1
> gpio_set A 1 0
> gpio_get A 1
value=0
```

# Contributing

If you find a bug, or have an idea for how we can improve this library, please
let us know by creating an issue. Opening up a pull request is even better!

# License

These libraries are made available under the MIT license. See the LICENSE file
for more information.

Проектируя сложное устройство часто требуется како-нибудь отладочный терминал, чтобы можно было легко заглянуть в недра контроллера, выполнять команды, посмотреть логику. Вмешаться в ход событий, что-то запустить. Да банально логи посмотреть и ошибки считать.

Обычно для этого применяют UART и терминальный доступ в текстовом режиме. А для простоты и универсальности весь интерфейс реализуют прям в самом контроллере. Благо ресурсов это много не требует. Задача столь частая, что существуют хорошие готовые библиотеки терминальных систем для контроллеров. Одну из них я давно применяю и хочу познакомить читателей с ней.

Написана на Си и с большим количеством макросов. Состоит из трех файлов. Умеет автодополнение, контроль параметров, самодокументирущуся справку (!!!), историю.

Общение может выглядеть так:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
> help
Available commands:
  help     - List all commands, or give details about a specific command
  gpio_set - Sets a GPIO pin
  gpio_get - Gets a GPIO pin value
> help gpio_get
Gets a GPIO pin value
Usage: gpio_get port pin
  port - The port <a ,B,C>
  pin  - The pin &lt;0-15>
> gpio_set A 1 1
> gpio_get A 1
value=1
> gpio_set A 1 0
> gpio_get A 1
value=0</a>

Ну обо всем по порядку. Установка консоли в проект довольно проста, но имеет ряд нюансов. Надо подключить в проект файлы:

console.c
console.h
console_config.h
console_private_helpers.h
Библиотека требует наличие типа bool, для этого можно либо подключить stdbool.h или пробежаться по файлам библиотеки и заменить bool на _Bool. Мне пришлось так сделать, т.к. на GD32, в его версии SPL, булевый тип криво прибит гвоздями в хидерах библиотеки и возникает гемор ненужный с переопределением типов.

Также я создал отдельный файл для конфиугурации, собственно, консоли, где прописал команды и всякие функции

myconsole.h
myconsole.с
Идем в console_config.h и настраиваем параметры:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
#pragma once
 
// See the README for documentation on these options.
 
// Максимальное число команд. Потом правите под свои нужды. 
#ifndef CONSOLE_MAX_COMMANDS
#define CONSOLE_MAX_COMMANDS 16
#endif
 
// Максимальная длина командной строки, ака самая длинная команда которую можно ввести (со всеми параметрами)
#ifndef CONSOLE_MAX_LINE_LENGTH
#define CONSOLE_MAX_LINE_LENGTH 32
#endif
 
// Вид строки приглашения. Тут на свой вкус у меня это PB II>
#ifndef CONSOLE_PROMPT
#define CONSOLE_PROMPT "PB II>"
#endif
 
// Настроки расположения буфера. Это для компоновщика GCC я не вникал и оставил по дефолту. 
#ifndef CONSOLE_BUFFER_ATTRIBUTES
#define CONSOLE_BUFFER_ATTRIBUTES
#endif
 
// Нужна ли встроенная справка. 
#ifndef CONSOLE_HELP_COMMAND
#define CONSOLE_HELP_COMMAND 1
#endif
 
// Давать ли полное управление. Например, можно ли стирать символы, т.е. править команду. Я включаю. 
#ifndef CONSOLE_FULL_CONTROL
#define CONSOLE_FULL_CONTROL 1
#endif
 
// Включать ли автодополнение. Штука удобная, хотя работает немного кривовато. Часто не дополняет (визуально), но при этом работает 
#ifndef CONSOLE_TAB_COMPLETE
#define CONSOLE_TAB_COMPLETE 0
#endif
 
#if CONSOLE_TAB_COMPLETE && !CONSOLE_FULL_CONTROL
#error "CONSOLE_TAB_COMPLETE requires CONSOLE_FULL_CONTROL to be enabled"
#endif
 
// История ввода. Я выключаю, чтобы не забивать память. 
#ifndef CONSOLE_HISTORY
#define CONSOLE_HISTORY 0
#endif
 
#if CONSOLE_HISTORY && !CONSOLE_FULL_CONTROL
#error "CONSOLE_HISTORY requires CONSOLE_FULL_CONTROL to be enabled"
#endif
С этим файлом разобрались. Теперь надо настроить ввод-вывод. В файле myconsole.с я прописываю обработчики и задачи.

1
2
3
4
void console_write_function(const char* str)
{
    xprintf(str);
}
Это функция вывода. Я использую xprintf от ElmChan, очень маленькая и быстрая библиотека, в отличии от той, что в stdio.h даже по сравнению с newlib-nano и не трахает мозги буфферизацией. Приколы с буфферизацией стандартной библиотеки вывода я наверное отдельной статьей распишу. Заодно и про xprintf расскажу. В данном случае не важно, что за функция вывода используется. Главное она должна получать ASCIIZ строку (т.е. строку с 0х00 в конце) и выводить ее в UART.

Поскольку у меня FreeRTOS, то я запускаю консоль в отдельной задаче:

1
xTaskCreate(ConsoleTask,"ConsoleTask", 	configMINIMAL_STACK_SIZE, NULL, tskIDLE_PRIORITY + 1, NULL);
А сама задача выглядит так:

1
2
3
4
5
6
7
8
9
10
11
void ConsoleTask (void *pvParameters)
{
uint8_t Buffer[64];
uint32_t len;
 
    while(1)
    {
        len = USART1_Receive(0x0D,Buffer,sizeof(Buffer),portMAX_DELAY); // принимаем строку оканчивающуюся на 0x0D и portMAX_DELAY - т.е. ждем до талого. 
        console_process(Buffer, len);
    }
}
И вот тут надо развернуть ряд нюансов. Для работы консоли ей надо просто скормить буфер который содержит введенную команду с 0x0A (LF он же \n) байтом на конце. И длину строки.

А, например, PuTTy в конце строки отдает не LF, а CR (0x0D).

Поэтому я написал специализированный обработчик данных из UART который принимает строку ограничивающуюся на 0x0D, а потом подменяет его на 0x0A.

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
uint32_t USART1_Receive(uint8_t chr, uint8_t *Buffer, uint32_t Length, TickType_t TimesUp)
{
uint8_t Receive;
uint32_t i=0;
 
    while(1)
    {
        if(pdTRUE!=xQueueReceive(USART1_In,&Receive,TimesUp)) return 0;
 
        if(Receive == chr)
        {
            Buffer[i++] = 0x0A; //LF  OLD = chr
            return i;
        }
 
        if(i<Length)
        {
            Buffer[i++]=Receive;
        }
        else
        {
            return i;
        }
    }
}
chr — символ который надо ждать и по которому мы отбиваем ввод. Это позволяет ждать сколько угодно. Сам ввод сделан на очередях FreeRTOS, т.е. в прерывании USART данные приходящие пихаются в очередь. А ожидающая задача на другом конце просыпается и по одному байту их зажевывает. УАРТ штука медленная, поэтому всякие DMA с кольцевыми буферами тут, ИМХО, излишняя заморочка. Удобней очередь, т.к. она сама разблокируется, сама передаст. Ну так вот, в выше указанной функции мы ждем конец строки, что для PuTTy 0x0D, либо до заполнения буфера, а когда встречаем 0x0D подменяем его на 0x0A и уже это отдаем консоли.

Сама инициализация консоли при этом выглядит следующим образом:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
void MyConsole_Setup(void)
{
 
// Прописываем функцию вывода. Обратите, кстати, на обращение к полю структуры без имени. Сразу через точку :) 
const console_init_t init_console = {
        .write_function = console_write_function,
    };
 
//Инициализируем
    console_init(&init_console);
 
//Регистрируем команды. В данном случае это команды hello, led, get
    console_command_register(hello);
    console_command_register(led);
    console_command_register(get);
 
// Запускаем задачу консоли. Размер стэка должен вместить её буфер!!!  В системах на конечном автомате можно же просто 
// периодически запускать консольную службу, да  хоть по таймеру
 
    xTaskCreate(ConsoleTask,"ConsoleTask", 	configMINIMAL_STACK_SIZE, NULL, tskIDLE_PRIORITY + 1, NULL);
}
Теперь о создании самой задачи и ее хэндле. Задача описывается регистрирующим макросом и самой функцией:

Макрос:

1
2
3
4
5
6
7
8
9
10
11
12
 
CONSOLE_COMMAND_DEF(hello, "Say hello!");
 
CONSOLE_COMMAND_DEF(led, "Sets a LED No",
    CONSOLE_INT_ARG_DEF(led, "The LED 0...3"),
    CONSOLE_INT_ARG_DEF(value, "The value 0-1")
);
 
CONSOLE_COMMAND_DEF(get, "Gets a GPIO pin value",
    CONSOLE_STR_ARG_DEF(port, "The port <A,B,C>"),
    CONSOLE_INT_ARG_DEF(pin, "The pin <0-15>")
);
Обратите внимание на то, что тут сразу прописывается имя команды, а также подсказка которая будет выводиться для системе в целом (help) . Как в общей подсказке ,так и индивидуальная подсказка по каждой команде (например help get )!!!

Существует два типа параметров. Строковые и численные. Строковый это

CONSOLE_STR_ARG_DEF позволяет засунуть в себя строку, которую ограничит нулем. Не забывайте про размер буфера (CONSOLE_MAX_LINE_LENGTH) который прописан в настройках.
CONSOLE_INT_ARG_DEF целочисленное значение в виде long числа.
Сколько надо параметров столько и прописываете в эти макросы.

Потом надо сделать обработчики:

1
2
3
4
static void hello_command_handler(const hello_args_t* args)
{
    xprintf("Hi!\n");
}
Для команды без параметров все просто. Она выполняется когда попадает в консоль. Обратите внимание на то, что имя тут параметры тут содержат название команды.

static void ####_command_handler(const ####_args_t* args)

Если у команды есть цифровые параметры, то их можно достать через поля структуры которая намазана на буффер ввода:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
static void led_command_handler(const led_args_t* args)
{
 
// Берем значение из строки ввода команды, которую нам передают сюда. 
uint8_t led = args->led;
uint8_t value = args->value;
 
// Поскольку консоль никак не проверят валидность значений это нам обязательно надо делать 
// На входе. Светодиодов у нас 3, поэтому проверяем, чтобы не больше 3. А значение может быть либо 
// 0 либо 1, поэтому не больше 1. 
    if( led>3 || value>1 )
    {
        xprintf("What is this shit?\n");
        return;
    }
 
    switch(led)
    {
        case 0: IO_SetLine(io_LED0,value); break;
        case 1: IO_SetLine(io_LED1,value); break;
        case 2: IO_SetLine(io_LED2,value); break;
        case 3: IO_SetLine(io_LED3,value); break;
        default: break;
    }
}
Для строковых параметров все похожим образом. Вообще передаваемая командная строка args просто содержит массив, где параметры лежат рядочком друг за другом. И его разбирают структурой.

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
static void get_command_handler(const get_args_t* args)
{
uint8_t i=0;
 
// Сначала пересчитаем сколько у нас символов в имени порта. Т.к. имя порта передатся строковой переменной
// оканчивающейся на 0х00
    while(args->port[i] !='\0')
    {
        i++;
    }
 
// Если всего один символ, значит это скорей то что нужно. Разбираем его через SWITCH
    if(i==1)
    {
       switch (args->port[0])
       {
            case 'A':
            case 'a':
            {
                if (args->pin < 16 ) // Второй параметр проверяем на максимальное значение. 
                {                           // Так как пинов у порта всего 16. И выводим значение в консоль. 
                    xprintf("GPIO A.%d = %d \n",args->pin,(GPIOA->IDR & 1<<(args->pin)) ? 1:0  );
                }
                else xprintf("Error: Wrong pin!\n");
                break;
            }
            case 'B':
            case 'b':
            {
                if (args->pin < 16 )
                {
                    xprintf("GPIO B.%d = %d \n",args->pin,(GPIOB->IDR & 1<<(args->pin)) ? 1:0  );
                }
                else xprintf("Error: Wrong pin!\n");
                break;
            }
            case 'C':
            case 'c':
            {
                if (12 < args->pin && args->pin < 16 )  // У STM32F103C8T6 порт С не полный
                {
                    xprintf("GPIO C.%d = %d \n",args->pin,(GPIOB->IDR & 1<<(args->pin)) ? 1:0  );
                }
                else xprintf("Error: Wrong pin!\n");
                break;
            }
            default: xprintf("Bullshit Port!\n");
       }
    }
    else
    {
        xprintf("To long name!\n");
    }
}
Программа готова. Осталось только сделать небольшую настройку PuTTy, чтобы она корректно отображала ввод и переносы
