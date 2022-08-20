# Домашнее задание к занятию "3.2. Работа в терминале, лекция 2"

1. Какого типа команда `cd`? Попробуйте объяснить, почему она именно такого типа; опишите ход своих мыслей, если считаете что она могла бы быть другого типа.

**Ответ:** встроенная команда bash терминала, так как:
- man cd - ничего не выдает
- which cd и whereis cd - тоже ничего не выдает
- в справке (man bash / cp) описано: "the cd builtin command"
    ```bash
    martin@vm2:~$ man cd
    No manual entry for cd
    martin@vm2:~$ which cd
    martin@vm2:~$ whereis cd
    cd:
    martin@vm2:~$ 
     ```
Это логично, так как нужно управление указателем на текущий каталог внутри текущей сесси, а не создание новой со своим окружением.


2. Какая альтернатива без pipe команде `grep <some_string> <some_file> | wc -l`? `man grep` поможет в ответе на этот вопрос. Ознакомьтесь с [документом](http://www.smallo.ruhr.de/award.html) о других подобных некорректных вариантах использования pipe.

**Ответ:** `grep <some_string> <some_file> -c`
- `wc -l, --lines` - print the newline counts
- `grep -c` - Suppress normal output; instead print a count of matching  lines  for  each  input  file

3. Какой процесс с PID `1` является родителем для всех процессов в вашей виртуальной машине Ubuntu 20.04?

**Ответ:**  systemd
```
martin@vm2:~$ pstree -p
systemd(1)─┬─ModemManager(706)─┬─{ModemManager}(709)
           │                   └─{ModemManager}(713)
           ├─agetty(668)
           ├─agetty(670)
           ├─cron(645)
```

4. Как будет выглядеть команда, которая перенаправит вывод stderr `ls` на другую сессию терминала?

**Ответ:**  `ls /nothing 2> /dev/pts/n+1`, где `n` - номер текущей сессии
```
martin@vm2:~$ tty
/dev/pts/0
martin@vm2:~$ ls /nothing 2> /dev/pts/1
martin@vm2:~$ 

martin@vm2:~$ tty
/dev/pts/1
martin@vm2:~$ ls: cannot access '/nothing': No such file or directory
```


5. Получится ли одновременно передать команде файл на stdin и вывести ее stdout в другой файл? Приведите работающий пример.

**Ответ:** да.
```
martin@vm2:~$ cat file1.txt 
one
two
martin@vm2:~$ cat stdout_file1_from_wc.txt
cat: stdout_file1_from_wc.txt: No such file or directory
martin@vm2:~$ wc  < file1.txt > stdout_file1_from_wc.txt
martin@vm2:~$ cat stdout_file1_from_wc.txt
2 2 8
martin@vm2:~$
```

6. Получится ли находясь в графическом режиме, вывести данные из PTY в какой-либо из эмуляторов TTY? Сможете ли вы наблюдать выводимые данные?



7. Выполните команду `bash 5>&1`. К чему она приведет? Что будет, если вы выполните `echo netology > /proc/$$/fd/5`? Почему так происходит?

**Ответ:**
- `bash 5>&1` - создаст файл нового дескриптора и перенаправит его поток в STDOUT
```
martin@vm2:~$ ls -al /proc/$$/fd
total 0
dr-x------ 2 martin martin  0 Aug 18 23:20 .
dr-xr-xr-x 9 martin martin  0 Aug 18 23:20 ..
lrwx------ 1 martin martin 64 Aug 18 23:20 0 -> /dev/pts/0
lrwx------ 1 martin martin 64 Aug 18 23:20 1 -> /dev/pts/0
lrwx------ 1 martin martin 64 Aug 18 23:20 2 -> /dev/pts/0
lrwx------ 1 martin martin 64 Aug 18 23:20 255 -> /dev/pts/0
martin@vm2:~$ bash 5>&1
martin@vm2:~$ ls -al /proc/$$/fd
total 0
dr-x------ 2 martin martin  0 Aug 18 23:21 .
dr-xr-xr-x 9 martin martin  0 Aug 18 23:21 ..
lrwx------ 1 martin martin 64 Aug 18 23:21 0 -> /dev/pts/0
lrwx------ 1 martin martin 64 Aug 18 23:21 1 -> /dev/pts/0
lrwx------ 1 martin martin 64 Aug 18 23:21 2 -> /dev/pts/0
lrwx------ 1 martin martin 64 Aug 18 23:21 255 -> /dev/pts/0
lrwx------ 1 martin martin 64 Aug 18 23:21 5 -> /dev/pts/0
martin@vm2:~$ 
```

- STDOUT команды echo перенаправим в дескриптор 5
```
martin@vm2:~$ echo netology > /proc/$$/fd/5
netology

martin@vm2:~$ echo netology > /proc/$$/fd/6
bash: /proc/963/fd/6: No such file or directory

martin@vm2:~$ echo netology > /proc/$$/fd/4
bash: /proc/963/fd/4: No such file or directory
```

8. Получится ли в качестве входного потока для pipe использовать только stderr команды, не потеряв при этом отображение stdout на pty? Напоминаем: по умолчанию через pipe передается только stdout команды слева от `|` на stdin команды справа.
Это можно сделать, поменяв стандартные потоки местами через промежуточный новый дескриптор, который вы научились создавать в предыдущем вопросе.

**Ответ:** получилось - по подсветвке видно, что в первом случае сработал grep по "No"
```
martin@vm2:~$ ls ~/Nothing 7>&2 2>&1 1>7 | grep "No"
ls: cannot access '/home/martin/Nothing': No such file or directory

martin@vm2:~$ ls ~/Nothing | grep "No"
ls: cannot access '/home/martin/Nothing': No such file or directory
martin@vm2:~$ 
```
![](/Users/almarchenko/DevOps/mylabs/03-sysadmin-02-terminal/no.png)



9. Что выведет команда `cat /proc/$$/environ`? Как еще можно получить аналогичный по содержанию вывод?

**Ответ:** Переменные окружения текущего PID, еще команды: `env` и `printevn`


10. Используя `man`, опишите что доступно по адресам `/proc/<PID>/cmdline`, `/proc/<PID>/exe`.

**Ответ:**
- `/proc/<PID>/cmdline` - это файл только для чтения, который содержит полную информацию о процессе из командной строки
- `/proc/<PID>/exe` - это символическая ссылка на фактически работающую программу

11. Узнайте, какую наиболее старшую версию набора инструкций SSE поддерживает ваш процессор с помощью `/proc/cpuinfo`.

**Ответ:** я не понял, где найти эту версию в файле, так как в`cat /proc/cpuinfo` `sse` встречается только в разделе `flags	:`, и встречаются значения sse: sse, sse1, sse2, sse3, sse4_1 и sse4_2

12. При открытии нового окна терминала и `vagrant ssh` создается новая сессия и выделяется pty. Это можно подтвердить командой `tty`, которая упоминалась в лекции 3.2. Однако:

    ```bash
	vagrant@netology1:~$ ssh localhost 'tty'
	not a tty
    ```
    итайте, почему так происходит, и как изменить поведение.

**Ответ:**  нашел следующее обьяснение на https://unix.stackexchange.com/questions/48527/ssh-inside-ssh-fails-with-stdin-is-not-a-tty

```commandline
By default, when you run a command on the remote machine using ssh, a TTY is not allocated for the remote session. This lets you transfer binary data, etc. without having to deal with TTY quirks. This is the environment provided for the command executed on computerone.

However, when you run ssh without a remote command, it DOES allocate a TTY, because you are likely to be running a shell session. This is expected by the ssh otheruser@computertwo.com command, but because of the previous explanation, there is no TTY available to that command.

If you want a shell on computertwo, use this instead, which will force TTY allocation during remote execution:

ssh -t user@computerone.com 'ssh otheruser@computertwo.com'
This is typically appropriate when you are eventually running a shell or other interactive process at the end of the ssh chain. If you were going to transfer data, it is neither appropriate nor required to add -t, but then every ssh command would contain a data-producing or -consuming command, like:
```

13. Бывает, что есть необходимость переместить запущенный процесс из одной сессии в другую. Попробуйте сделать это, воспользовавшись `reptyr`. Например, так можно перенести в `screen` процесс, который вы запустили по ошибке в обычной SSH-сессии.

**Ответ:** 
```commandline
/proc/sys/kernel/yama/ptrace_scope  - через sudo nano установил 0
/etc/sysctl.d/10-ptrace.conf - через sudo nano установил 0

martin@vm2:~$ tty
/dev/pts/0
martin@vm2:~$ top
top - 23:36:27 up 8 min
CTRL+Z

martin@vm2:~$ tty
/dev/pts/2
martin@vm2:~$ jobs -l
martin@vm2:~$ reptyr 1105
top - 23:37:57 up 10 min
```

3. `sudo echo string > /root/new_file` не даст выполнить перенаправление под обычным пользователем, так как перенаправлением занимается процесс shell'а, который запущен без `sudo` под вашим пользователем. Для решения данной проблемы можно использовать конструкцию `echo string | sudo tee /root/new_file`. Узнайте что делает команда `tee` и почему в отличие от `sudo echo` команда с `sudo tee` будет работать.

**Ответ:** 
- `sudo` не выполняет перенаправление вывода, это произойдет как непривилегированный пользователь
- команда `tee` читает из стандартного ввода и записывает как в стандартный вывод, в один или несколько файлов одновременно
 
```commandline
martin@vm2:~$ sudo echo string > /root/new_file
-bash: /root/new_file: Permission denied

martin@vm2:~$ echo string | sudo tee /root/new_file
string

martin@vm2:~$ sudo ls /root
new_file  snap
```

 ---

## Как сдавать задания

Обязательными к выполнению являются задачи без указания звездочки. Их выполнение необходимо для получения зачета и диплома о профессиональной переподготовке.

Задачи со звездочкой (*) являются дополнительными задачами и/или задачами повышенной сложности. Они не являются обязательными к выполнению, но помогут вам глубже понять тему.

Домашнее задание выполните в файле readme.md в github репозитории. В личном кабинете отправьте на проверку ссылку на .md-файл в вашем репозитории.

Также вы можете выполнить задание в [Google Docs](https://docs.google.com/document/u/0/?tgif=d) и отправить в личном кабинете на проверку ссылку на ваш документ.
Название файла Google Docs должно содержать номер лекции и фамилию студента. Пример названия: "1.1. Введение в DevOps — Сусанна Алиева".

Если необходимо прикрепить дополнительные ссылки, просто добавьте их в свой Google Docs.

Перед тем как выслать ссылку, убедитесь, что ее содержимое не является приватным (открыто на комментирование всем, у кого есть ссылка), иначе преподаватель не сможет проверить работу. Чтобы это проверить, откройте ссылку в браузере в режиме инкогнито.

[Как предоставить доступ к файлам и папкам на Google Диске](https://support.google.com/docs/answer/2494822?hl=ru&co=GENIE.Platform%3DDesktop)

[Как запустить chrome в режиме инкогнито ](https://support.google.com/chrome/answer/95464?co=GENIE.Platform%3DDesktop&hl=ru)

[Как запустить  Safari в режиме инкогнито ](https://support.apple.com/ru-ru/guide/safari/ibrw1069/mac)

Любые вопросы по решению задач задавайте в чате учебной группы.

---
