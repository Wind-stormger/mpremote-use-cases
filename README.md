# mpremote-use-cases

I've tried a lot of different 3rd party micropython tools, some are Windows apps, some are VScode plugins.

But when I tried MicroPython's mpremote tool, I almost decided it was going to be my go-to tool. 😄

I usually use it in conjunction with VScode. Its code highlighting, auto-completion, and auto-indentation are very useful. It is also easy to open other plugins at the same time. The important thing is that VScode is also a multi-platform IDE.

Maybe some minimalists, I think the mpremote tool will also be suitable, just use the system's terminal with any text editor, if you are a terminal command line veteran, you only need a terminal to do everything.

[Document Page](https://docs.micropython.org/en/latest/reference/mpremote.html)

[GitHub Page](https://github.com/micropython/micropython/tree/master/tools/mpremote)

[PyPI Page](https://pypi.org/project/mpremote/)

----------
Most of the use cases are pretty clear in the documentation, but I'm not a command line veteran, so I'll put some use cases I figured out here, maybe it will help some newbies like me.

Existing use cases for this post:

[toc]

> 1. cp
> 1.1 Copy file, in the current path of the terminal
> 1.2 Copy files，in absolute paths
> 1.3 Copy multiple files  at the same time
> 2. exec, run
> 3. mount
> 3.1 Mount the local directory on the remote device
> 3.2 Continue to use the script in the original flash after mounting the local directory
> 3.3 A way to speed up testing

## 1. cp

This is probably the most common command besides repl, used to copy files from local to device, or from device to local. Probably because I am not yet familiar with the terminal command syntax, at first I could not fully grasp the usage of the cp command from the several use cases in the documentation.



### 1.1 Copy the file, in the current path of the terminal

First create a clean temporary folder and write a `main.py` into it.
```python
print ("start")
for i in range(2):
    print(i)
print ("end")
```

Simplest use case, enter this folder path in terminal, copy file from local to device :
```
mpremote connect COM1 cp main.py : 

mpremote connect COM1 cp main.py :main.py
```
The two commands achieve the exact same function.

After the `:` symbol, if you enter a filename, the file will be renamed to this filename when copied to the device.

Copy file from device to local :
```
mpremote connect COM1 cp :main.py .

mpremote connect COM1 cp :main.py main.py
```
The two commands achieve the exact same function.

If you want to rename, you can delete the `.` symbol and enter the filename you want.

### 1.2 Copy files in absolute paths

It is a little more complicated.

On Windows, use the absolute path to the file to copy from local to device:
```
mpremote connect COM1 cp D:\temp\main.py :main.py
```

Copy file from device to local :
```
mpremote connect COM1 cp :main.py D:\temp\main.py
```

> At present, special attention should be paid to the fact that the target filename cannot be omitted in Windows!
> Relevant contents have been transferred to issue #9132 .
> PR #9148 Support for Windows pathname separators.

In Linux, such as Ubuntu, copy a file from an absolute path can omit the target filename:
```
mpremote connect /dev/ttyACM0 cp ~/temp/main.py :

mpremote connect /dev/ttyACM0 cp /home/wind/temp/main.py :

mpremote connect /dev/ttyACM0 cp :main.py ~/temp/

mpremote connect /dev/ttyACM0 cp :main.py /home/wind/temp/
```

### 1.3 Copy multiple files  at the same time

In Linux, such as Ubuntu, copy multiple files from local to device with absolute path :
```
mpremote connect /dev/ttyACM0 cp ~/temp/main.py ~/temp/main2.py :
```

Copy multiple files from device to local absolute path:
```
mpremote connect /dev/ttyACM0 cp :main.py :main2.py ~/temp/
```

## 2. exec, run

These commands are used to control the remote device to run Python code or scripts without copying files.

### 2.1 execute the given Python code

```
mpremote connect COM1 exec "print(1234)"
```

Just like entering a line of Python code  in REPL.

### 2.2 run a script from the local filesystem

```
mpremote connect COM1 run test_1.py
```

Just like entering paste mode in the REPL, copy and paste the code into the specified Python script, then run it.

## 3. mount

I haven't thought about this path yet, take a look at the following use case and try to understand it, you'll be as excited as I am.

### 3.1 Mount the local directory on the remote device

First create a clean temporary directory and write  some Python script into it, as follows:

```python
# numbers.py
num_1 = 21
num_2 = 22
num_3 = 23
num_4 = 24

```

```python
# test_1.py
print("test_1 start")
import numbers
print(numbers.num_1)
print(numbers.num_2)
print(numbers.num_4)
print("test_1 end")

```

```python
# test_2.py
print("test_2 start")
import numbers
temp1 = numbers.num_3 - numbers.num_2
print(temp1)
temp1 = numbers.num_3 - numbers.num_1
print(temp1)
print("test_2 end")

```
Enter the path to this directory in the terminal.

Let's confirm some information, list files on the device:
```
mpremote connect COM1 ls

ls :
         139 boot.py
```
Enter REPL and confirm again:
```
mpremote connect COM1 repl
```

```python
>>> uos.listdir()
['boot.py']
>>>
```

Exit REPL, mount the local directory, enter repl again:

```
mpremote connect COM1 mount . repl
```

Confirm files again:

```python
>>> uos.listdir()
['numbers.py','test_1.py', 'test_2.py']
```
There is no `boot.py` here, but the Python script we created appears in the list.

Import and run two test scripts:
```python
>>> import test_1,test_2
test_1 start
21
22
24
test_1 end
test_2 start
1
2
test_2 end
>>>
```

View a file:
```python
>>> f=open("numbers.py")
>>> print(f.read())
# numbers.py
num_1 = 21
num_2 = 22
num_3 = 23
num_4 = 24

>>>
```

Beyond surprise, you might wonder, if the script files were all uploaded to the device and stored? which I did initially.

Now, we keep the terminal in REPL, modify this file locally:
```python
# numbers.py
num_1 = 1
num_2 = 12
num_3 = 23
num_4 = 35

```

Go back to REPL and check again:
```python
>>> f=open("numbers.py")
>>> print(f.read())
# numbers.py
num_1 = 1
num_2 = 12
num_3 = 23
num_4 = 35

>>>
```
All doubts dissipated, the file was only saved in the local directory, and this directory was mounted on the device.

Soft reset is handled and will re-mount the directory.

@jimmo explained more about this in [discussioncomment-3500600](https://github.com/micropython/micropython/discussions/9096#discussioncomment-3500600) , [discussioncomment-3526060](https://github.com/micropython/micropython/discussions/9096#discussioncomment-3526060)

This is like a mobile hard disk, or a NAS, cloud disk, connected to the MicroPython device. Obviously, this function can greatly save the flash life of the device. Usually, you only need to copy the python script file to flash when you have to run it offline.

It is recommended to execute python scripts using the following combined commands:
```
mpremote connect COM1 mount . exec "import test_1"
```

### 3.2 Continue to use the script in the original flash after mounting the local directory

Use the `uos.listdir("/")` command in the REPL and you will see the files originally stored in flash:
```python
>>> uos.listdir("/")
['remote', 'boot.py', 'main.py']
```

If you want to use flash scripts (such as `main.py`) while keeping the local directory mounted, you can use the following command to add the original flash root directory path to the `sys.path` list:
```python
>>> import sys
>>> sys.path
['', '.frozen', '/lib']
>>> sys.path.append("/")
>>> sys.path
['', '.frozen', '/lib', '/']
>>> import main
```

If there is already `main.py` in the mounted local directory, only the `main.py` of the local directory will be run. You can use `reverse()` to change the list order:
```python
>>> sys.path
['', '.frozen', '/lib', '/']
>>> sys.path.reverse()
>>> sys.path
['/', '/lib', '.frozen', '']
>>> import main
```

Scripts in flash will now run first.

###  3.3 A way to speed up testing

If we need to test and modify a script again and again, and the script has imported many script modules stored locally, in this case, each mount test will take several seconds or even tens of seconds.

I've come up with a way to effectively reduce the runtime by `import` modules in the REPL that are not duplicated.

Take testing SSD1306 OLED display as an example, `main.py` is the main program, `ssd1306.py` is the driver module, and the program code will not be listed in full here.

Use time.tick_ms() to verify the time required to test the `main.py` script in the local directory, which takes about 6 seconds :
```python
>>> import time;t1 = time.ticks_ms();import main;time.ticks_diff(time.ticks_ms(),t1)
6032
```

Use `sys.modules` to view the currently imported script modules, you can see that `ssd1306.py` has been imported:
```python
>>> import sys;sys.modules
{'main': <module 'main' from 'main.py'>, 'ssd1306': <module 'ssd1306' from 'ssd1306.py'>, 'flashbdev': <module 'flashbdev' from 'flashbdev.py'>}
```

We just need to remove `main` and keep the other modules:
```python
>>> del main;sys.modules.pop('main')
<module 'main' from 'main.py'>
>>> import sys;sys.modules
{'ssd1306': <module 'ssd1306' from 'ssd1306.py'>, 'flashbdev': <module 'flashbdev' from 'flashbdev.py'>}
```

Test again:
```python
>>> import time;t1 = time.ticks_ms();import main;time.ticks_diff(time.ticks_ms(),t1)
1612
```

Significantly improved!


( To Be Continued)
