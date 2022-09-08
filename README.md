# devops-netology
### ./terraform/.gitignore
Будут проигнорированы служебные файлы (файлы состояния tfstate, файлы со значениями переменных tfvars и пр.)

## 02-git-04-tools
1. Найдите полный хеш и комментарий коммита, хеш которого начинается на aefea.
```bash
git rev-parse aefea
aefead2207ef7e2aa5dc81a34aedf0cad4c32545
```
2. Какому тегу соответствует коммит 85024d3?
```bash
git show 85024d3
...
tag: v0.12.23
...
```
3. Сколько родителей у коммита b8d720? Напишите их хеши.
```bash
git rev-list --parents -n 1 b8d720
b8d720f8340221f2146e4e4870bf2ee0bc48f2d5 56cd7859e05c36c06b56d013b55a252d0bb7e158 9ea88f22fc6269854151c571162c5bcf958bee2b
```
4. Перечислите хеши и комментарии всех коммитов которые были сделаны между тегами v0.12.23 и v0.12.24.
```bash
git log v0.12.23..v0.12.24 --pretty=oneline
33ff1c03bb960b332be3af2e333462dde88b279e (tag: v0.12.24) v0.12.24
b14b74c4939dcab573326f4e3ee2a62e23e12f89 [Website] vmc provider links
3f235065b9347a758efadc92295b540ee0a5e26e Update CHANGELOG.md
6ae64e247b332925b872447e9ce869657281c2bf registry: Fix panic when server is unreachable
5c619ca1baf2e21a155fcdb4c264cc9e24a2a353 website: Remove links to the getting started guide's old location
06275647e2b53d97d4f0a19a0fec11f6d69820b5 Update CHANGELOG.md
d5f9411f5108260320064349b757f55c09bc4b80 command: Fix bug when using terraform login on Windows
4b6d06cc5dcb78af637bbb19c198faff37a066ed Update CHANGELOG.md
dd01a35078f040ca984cdd349f18d0b67e486c35 Update CHANGELOG.md
225466bc3e5f35baa5d07197bbc079345b77525e Cleanup after v0.12.23 release
```
5. Найдите коммит в котором была создана функция func providerSource, ее определение в коде выглядит так func providerSource(...) (вместо троеточего перечислены аргументы).
```bash
git log -S 'func providerSource' --oneline
5af1e6234a main: Honor explicit provider_installation CLI config when present
8c928e8358 main: Consult local directories as potential mirrors of providers
git show 8c928e8358
...
+func providerSource(services *disco.Disco) getproviders.Source {
...

```
6. Найдите все коммиты в которых была изменена функция globalPluginDirs.
```bash
git log -L :globalPluginDirs:plugins.go --pretty=format:"%h %s" --no-patch
78b1220558 Remove config.go and update things using its aliases
52dbf94834 keep .terraform.d/plugins for discovery
41ab0aef7a Add missing OS_ARCH dir to global plugin paths
66ebff90cd move some more plugin search path logic to command
8364383c35 Push plugin discovery down into command package
```
7. Кто автор функции synchronizedWriters?
```bash
git log -S "func synchronizedWriters" --oneline
bdfea50cc8 remove unused
5ac311e2a9 main: synchronize writes to VT100-faker on Windows
git show 5ac311e2a9
...
Author: Martin Atkins
...
+func synchronizedWriters(targets ...io.Writer) []io.Writer {
...
```
## 03-sysadmin-01-terminal
1-7. У меня дома сервер с Proxmox на борту, так что ДЗ выполняю в виртуалке на нём.

8. Ознакомиться с разделами `man bash`, почитать о настройках самого bash:
* какой переменной можно задать длину журнала `history`, и на какой строчке manual это описывается?

  ```bash
  HISTSIZE (строка 862 в мануле)
  ```
* что делает директива `ignoreboth` в bash?

  ```bash
  Не сохранять команды начинающиеся с пробела и одинаковые команды, выполненные подряд (ignorespace + ignoredup)
  ```
  
9. В каких сценариях использования применимы скобки `{}` и на какой строчке `man bash` это описано?

  ```bash
  Это Brace Expansion (строка 1091 в мануле). Может использоваться для получения списка аргументов.
  echo {a..z} # алфавит
  echo {1..9} # последовательность
  echo {1..20..2} # инкремент в последовательности
  echo {a,b,c,d} # просто список
  ```

10. С учётом ответа на предыдущий вопрос, как создать однократным вызовом `touch` 100000 файлов? Получится ли аналогичным образом создать 300000? Если нет, то почему?

  ```bash
  touch {1..100000}.tmp
  ```
  
11. В man bash поищите по `/\[\[`. Что делает конструкция `[[ -d /tmp ]]`
  ```bash
  Двойные фигурные скобки - это расширение CONDITIONAL EXPRESSIONS. Данная конструкция проверяет существование папки /tmp.
  ```

12. Основываясь на знаниях о просмотре текущих (например, PATH) и установке новых переменных; командах, которые мы рассматривали, добейтесь в выводе type -a bash в виртуальной машине наличия первым пунктом в списке:
  ```bash
  bash is /tmp/new_path_directory/bash
  bash is /usr/local/bin/bash
  bash is /bin/bash
  ```
(прочие строки могут отличаться содержимым и порядком)

В качестве ответа приведите команды, которые позволили вам добиться указанного вывода или соответствующие скриншоты.

  ```bash
  mkdir -p /tmp/new_path_directory && touch /tmp/new_path_directory/bash
  PATH="/tmp/new_path_directory:$PATH"
  type -a bash
  ```

13. Чем отличается планирование команд с помощью `batch` и `at`?

  ```bash
  Обе являются частью пакета at.
  at - запуск команд в определённое время
  batch - выполнение команд когда загрузка системы ниже указанного значения (по дефолту 1.5)
  ```

14. Завершите работу виртуальной машины чтобы не расходовать ресурсы компьютера и/или батарею ноутбука.

## 03-sysadmin-02-terminal

1. Какого типа команда `cd`? Попробуйте объяснить, почему она именно такого типа; опишите ход своих мыслей, если считаете что она могла бы быть другого типа.
  
2. Какая альтернатива без pipe команде `grep <some_string> <some_file> | wc -l`? `man grep` поможет в ответе на этот вопрос. Ознакомьтесь с [документом](http://www.smallo.ruhr.de/award.html) о других подобных некорректных вариантах использования pipe.
  ```
  grep <some_string> <some_file> -c
  ```
3. Какой процесс с PID `1` является родителем для всех процессов в вашей виртуальной машине Ubuntu 20.04?
  ```
  systemd
  ```
4. Как будет выглядеть команда, которая перенаправит вывод stderr `ls` на другую сессию терминала?
  ```
  ls > /proc/<SESSION_PID>/fd/2
  ```
5. Получится ли одновременно передать команде файл на stdin и вывести ее stdout в другой файл? Приведите работающий пример.
  ```
  cat package_list.txt
  apt
  apt-utils
  apt-transport-https
  
  apt show $(<package_list.txt) > out.txt
  ```
6. Получится ли находясь в графическом режиме, вывести данные из PTY в какой-либо из эмуляторов TTY? Сможете ли вы наблюдать выводимые данные?
7. Выполните команду `bash 5>&1`. К чему она приведет? Что будет, если вы выполните `echo netology > /proc/$$/fd/5`? Почему так происходит?
8. Получится ли в качестве входного потока для pipe использовать только stderr команды, не потеряв при этом отображение stdout на pty? Напоминаем: по умолчанию через pipe передается только stdout команды слева от `|` на stdin команды справа.
Это можно сделать, поменяв стандартные потоки местами через промежуточный новый дескриптор, который вы научились создавать в предыдущем вопросе.
9. Что выведет команда `cat /proc/$$/environ`? Как еще можно получить аналогичный по содержанию вывод?
10. Используя `man`, опишите что доступно по адресам `/proc/<PID>/cmdline`, `/proc/<PID>/exe`.
11. Узнайте, какую наиболее старшую версию набора инструкций SSE поддерживает ваш процессор с помощью `/proc/cpuinfo`.
12. При открытии нового окна терминала и `vagrant ssh` создается новая сессия и выделяется pty. Это можно подтвердить командой `tty`, которая упоминалась в лекции 3.2. Однако:

    ```bash
	vagrant@netology1:~$ ssh localhost 'tty'
	not a tty
    ```

	Почитайте, почему так происходит, и как изменить поведение.
13. Бывает, что есть необходимость переместить запущенный процесс из одной сессии в другую. Попробуйте сделать это, воспользовавшись `reptyr`. Например, так можно перенести в `screen` процесс, который вы запустили по ошибке в обычной SSH-сессии.
14. `sudo echo string > /root/new_file` не даст выполнить перенаправление под обычным пользователем, так как перенаправлением занимается процесс shell'а, который запущен без `sudo` под вашим пользователем. Для решения данной проблемы можно использовать конструкцию `echo string | sudo tee /root/new_file`. Узнайте что делает команда `tee` и почему в отличие от `sudo echo` команда с `sudo tee` будет работать.
