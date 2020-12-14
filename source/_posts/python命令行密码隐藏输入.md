---
title: Python命令行密码隐藏输入
tags:
  - Python
id: '10'
categories:
  - - Python
date: 2018-01-29 22:22:27
---

有时候，我们需要在命令行交互中输入用户名跟密码，而且不希望输入的密码被别看到，找了下好像没看到有密码隐藏输入的库，这里整理了下网上找到的一些资料，自己试了可以使用的：

```python
import sys
import termios
import tty


def getch():
    fd = sys.stdin.fileno()
    old_settings = termios.tcgetattr(fd)
    try:
        tty.setraw(sys.stdin.fileno())
        ch = sys.stdin.read(1)
    finally:
        termios.tcsetattr(fd, termios.TCSADRAIN, old_settings)
    return ch


def getpass(maskchar="*"):
    password = ""
    while True:
        ch = getch()
        if ch == "\r" or ch == "\n":
            return password
        elif ch == "\b" or ord(ch) == 127:
            if len(password) > 0:
                sys.stdout.write("\b \b")
                sys.stdout.flush()
                password = password[:-1]
        else:
            if maskchar:
                sys.stdout.write(maskchar)
                sys.stdout.flush()
            password += ch


x = input("please input your gitlab username:\n")
print('please input your password:')
y = getpass()

print('\nYour username is:' + x + ' and password is:' + y)</pre>
```

试了下效果还可以，就先用上了