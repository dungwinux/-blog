---
layout: post
title: "1337UP LIVE CTF - Funny writeup"
date: 2024-11-17 17:00:00 +0000
categories: security
tags: ctf
excerpt: The challenges are challenging, and you should try it
---

Another great CTF, this time from Intigriti. Playing in team `SavedByTheShell`, I solved 3/4 challenges in rev category. Unfortunately, I decided to only do Funny writeup. Buckle up and let's dive into the details.

### Description

464

`created by word_oh`

What is this funny file? 🤪

### Approach

We are given a fascinating binary: `funny.pyc`. Out of all executable formats, they decided to use `pyc`? What's the meaning of this?

> To speed up loading modules, Python caches the compiled version of each module in the `__pycache__` directory under the name module.version.pyc, where the version encodes the format of the compiled file; it generally contains the Python version number.
>
> ...
>
> A program doesn’t run any faster when it is read from a .pyc file than when it is read from a .py file; the only thing that’s faster about .pyc files is the speed with which they are loaded.
> 
> ~ [The Python Tutorial - 6.1.3. “Compiled” Python files](https://docs.python.org/3/tutorial/modules.html#compiled-python-files)

What can you do with a pyc then? Even without Python source code, you can still execute it in a similar fashion: `python funny.pyc`. If you fail to do so (_RuntimeError: Bad magic number in .pyc file_), you probably need python 3.13. Turns out, after every **minor** update in version, they renumber all the opcodes, so there is no way from one specific version of Python to execute pyc compiled using previous or later Python version.

How do I know it is 3.13? There is an open-source tool called [Decompyle++](https://github.com/zrax/pycdc) that can disassemble and decompile different version of Python uniformingly. Using its disassembler `pycdas`, we can meaningfully speculate the file.

```yaml
funny.pyc (Python 3.13)
[Code]
    File Name: chal.py
    Object Name: <module>
    Qualified Name: <module>
    Arg Count: 0
    Pos Only Arg Count: 0
    KW Only Arg Count: 0
    Stack Size: 7
    Flags: 0x00000000
...
```

Unfortunately, we do not have the same success when using `pycdc`, the decompiler.

```py
# Source Generated with Decompyle++
# File: funny.pyc (Python 3.13)

Unsupported opcode: COPY (218)
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
# WARNING: Decompyle incomplete
```

Here comes the most "respectful" thing about open-source. 80% of the comment on the repo are: _please support xxx feature_ without a corresponding pull request. Cool! I found a [blog post by idafchev](https://idafchev.github.io/blog/Decompile_python/) about hacking the tool to do "fake" support on the opcode, which is basically adding cases in `BuildFromCode` function in ASTree.cpp file. And it works just enough for us to see what's going on:

```py
# Source Generated with Decompyle++
# File: funny.pyc (Python 3.13)

from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
a = 0
b = 0
c = 0
d = 0
e = 0
f = 0
g = 0
h = 0
i = 0
j = 0
k = 0
l = 0
m = 0
n = 0
o = 0
p = 0
q = 0
r = 0
s = 0
t = 0
u = 0
v = 0
w = 0
x = 0
y = 0
z = 0
key = None('Key > ').encode()
iv = None('IV  > ').encode()
if None(key) == 32 and None(iv) == 16:
    decrypted = None(AES.new(key, AES.MODE_CBC, iv).decrypt(...), AES.block_size)
    if b'INTIGRITI' in decrypted:
        None('gg')
    None(decrypted.decode())
del decrypted
del key
del iv
i_wonder_what_this_decrypts_to = ... # Some super long cipher text
(AES.new(..., AES.MODE_CBC, ...).decrypt(i_wonder_what_this_decrypts_to), AES.block_size)
# 1627 lines of just variables?
return None
if ImportError:
    None('run `pip install pycryptodome` to be able to play this challenge')
    None()
```

The decompiler could not put the correct function name, but no worries, as we can fix this by hand. If we look back at the disassembler, we got a list of names used in the program.

`Crypto.Cipher`, `AES`, `Crypto.Util.Padding`, `pad`, `unpad`, `ImportError`, `print`, `exit`, `a`, `b`, `c`, `d`, `e`, `f`, `g`, `h`, `i`, `j`, `k`, `l`, `m`, `n`, `o`, `p`, `q`, `r`, `s`, `t`, `u`, `v`, `w`, `x`, `y`, `z`, `input`, `encode`, `key`, `iv`, `len`, `new`, `MODE_CBC`, `decrypt`, `block_size`, `decrypted`, `decode`, `i_wonder_what_this_decrypts_to`

So I started matching them through guessing:

|Code                                                                   |Guess              |
|:----------------------------------------------------------------------|:-----------------:|
| `None('Key > ').encode()` <br> `None('IV  > ').encode()`              | `input`           |
| `None(key) == 32` <br> `None(iv) == 16`                               | `len`             |
| `None(AES.new(key, AES.MODE_CBC, iv).decrypt(...), AES.block_size)`   | `unpad`           |
| `None('gg')` <br> `None(decrypted.decode())` <br> `None('run ...')`   | `print`           |
| `None()`                                                              | `exit`            |

There is another `AES.new` at the bottom with improper enclosure, so I assumed it does the same thing as above and wrapped it in an `unpad` call.
We then get a better view at the source code:

```py
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
a = 0
b = 0
c = 0
d = 0
e = 0
f = 0
g = 0
h = 0
i = 0
j = 0
k = 0
l = 0
m = 0
n = 0
o = 0
p = 0
q = 0
r = 0
s = 0
t = 0
u = 0
v = 0
w = 0
x = 0
y = 0
z = 0
key = input('Key > ').encode()
iv = input('IV  > ').encode()
if len(key) == 32 and len(iv) == 16:
    decrypted = unpad(AES.new(key, AES.MODE_CBC, iv).decrypt(...), AES.block_size)
    if b'INTIGRITI' in decrypted:
        print('gg')
    print(decrypted.decode())
del decrypted
del key
del iv
i_wonder_what_this_decrypts_to = ... # Some super long cipher text
unpad(AES.new(..., AES.MODE_CBC, ...).decrypt(i_wonder_what_this_decrypts_to), AES.block_size)
```

### A potential shortcut?

A summary of the program is as follow:
1.  User inputs in key and IV ([Wikipedia: Block cipher mode of operation](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Initialization_vector_(IV)))
2.  Check if key length is 32, IV length is 16.
    -   If true, use them to decrypt pre-determined string. Print _gg_ if the decrypted text contains _INTIGRITI_. Then print the decrypted text.
3.  Using a different key and IV, decrypt `i_wonder_what_this_decrypts_to`.

The check in step 2 can easily be bypassed if we patch the string it decrypts to match with something we input. However, step 3 is more tempting and does not require step 2 to be success. So I installed Python 3.13, ran `python -i funny.pyc` (i for interactive), and paste the last line to get decrypted text of `i_wonder_what_this_decrypts_to`.

Here comes the flag. And we have successfully solved the challenge. Thank you for reading the writeup. Like and...

But wait.

```
If you've uncovered this message, you're probably on the right track.

The goal for this challenge is to figure out the correct key and IV to decrypt the flag. You must have noticed all the names being pushed and popped and wondered what they're for...
I promise it's not too scary. The key to decrypt this flag is written in ASCII art using these names, so you must reconstruct the source for everything under this
message decryption down to the exact spacing. The IV is the characters at positions
[(476, 6), (468, 5), (282, 6), (506, 6), (420, 3), (492, 0), (192, 6), (56, 6), (144, 3), (324, 0), (360, 1), (352, 6), (30, 1), (260, 0), (298, 1), (480, 3)]
in that exact order joined together where (0, 0) is the *TOP LEFT* of where the ASCII art begins.

For example, consider the following *FAKE* ASCII art where the IV positions are [(0, 0), (5, 1), (2, 6), (10, 6)]

axxxx   x       xxxxxxxxx
x    b  x           x
x    x  x           x
xxxxx   x           x
x    x  x           x
x    x  x           x
xxcxx   xxdxxx      x

In this example, the key would be "BLT" and the IV would be "abcd" since the character at (0, 0) is 'a', (5, 1) is 'b', (2, 6) is 'c', and (10, 6) is 'd'.

To clarify further, all semicolons are placed "normally" meaning that they're placed directly after a statement with no spacing (ex: a;b;   c;  d;), the final IV will only contain characters a-z, and I highly recommend
using a text editor that uses an equal width for all characters to avoid misalignment.

Good luck!
```

A message from the author! The real flag is in fact in the check earlier and can only be found with the correct key and IV. To do so, we need to guess the ASCII art positioning after the decryption.

### Recovering ASCII art from the line table

If we take a look at the decompiler output from earlier, we can see a long list of variables with no exact uses. We also got the hint that ASCII art is written in the form of `a;b;   c;  d;`. So how do all of these work? One misconception must be cleared first: `;` (semicolon) is meaningful in Python. Functionally, it is very similar to `\n` (new line), but more restricted. You can write `a = 3; b = 4` and it would run as you would expect in other languages. The annoying thing is that you cannot end a line with `;`. So what the bytecode actually tells us is the list of variables, in the order of left-to-right, low-to-high. And to get the coordinates, we need to look at line table.

_Line table_ might be a horrible name, because it is very difficult to search its documentation. Most links point to [PEP 626 – Precise line numbers for debugging and other tools](https://peps.python.org/pep-0626/), which is not helpful for what we are doing. As I was struggling to figure out, I found this StackOverflow [answer](https://stackoverflow.com/a/59431935). In short, for Python 3.13, we can parse pyc using the following code:

```py
import dis
import marshal

with open('funny.pyc', 'rb') as f:
    f.seek(16)
    dis.dis(marshal.load(f))
```

But `dis.dis` would just print out in format similar to what we saw from `pycdas` earlier. Not helpful. However, if we look at its [documentation](https://docs.python.org/3/library/dis.html#dis.dis):

> `dis.dis(x=None, *, file=None, depth=None, show_caches=False, adaptive=False)`
> 
> Disassemble the x object. x can denote either a module, a class, a method, a function, a generator, an asynchronous generator, a coroutine, a code object, a string of source code or a byte sequence of raw bytecode...

So in our case, what is the type of x? I thought it is _a byte sequence of raw bytecode_ so we are limited by the library, but I was wrong. It is in fact _a code object_ as a result of `marshal.load`, which leads us to another awesome [function](https://docs.python.org/3/library/dis.html#dis.get_instructions):

> `dis.get_instructions(x, *, first_line=None, show_caches=False, adaptive=False)`
> 
> Return an iterator over the instructions in the supplied function, method, source code string or code object.
> 
> The iterator generates a series of Instruction named tuples giving the details of each operation in the supplied code.
> 
> ...

This means we can iterate and get every bytecode instruction!

```py
import dis
import marshal

with open('funny.pyc', 'rb') as f:
    f.seek(16)
    a = (marshal.load(f))
print([i for i in dis.get_instructions(a)])

# [Instruction(opname='RESUME', opcode=149, arg=0, argval=0, argrepr='', offset=0, start_offset=0, starts_line=True, line_number=0, label=None, positions=Positions(lineno=0, end_lineno=1, col_offset=0, end_col_offset=0), cache_info=None), ...]
```

Pretty cool, right? It is even better when you realized that among them, there are Position informations!

```py
for i in dis.get_instructions(a):
    pos = i.positions
    # Print name and relevant location in the source code
    print(i.opname, (pos.lineno, pos.col_offset))

"""
RESUME (0, 0)
NOP (1, 0)
LOAD_CONST (2, 4)
LOAD_CONST (2, 4)
...
"""
```

Woah! Did Python leak information or exfil-ed the source code? No. Remember that sometimes Python asserts and informs you the line and show it to you using arrow. That is exactly what the position is used for.

```py
File "test.py", line 29, in <module>
    key = input('Key > ').encode()
          ~~~~~^^^^^^^^^^
KeyboardInterrupt
```

Even without the source code, Python can still show you the line to look at.

```py
  File "chal.py", line 9, in <module>
KeyboardInterrupt
```

With that, we can now reconstruct the actual ASCII art. Since it is after the decryption part and before the `ImportError`, I limited the offset to be within 600 and 2778. I observed that each of the symbol (`a`, `b`, ...) correlates with a `LOAD_NAME` opcode, and each of the `LOAD_NAME` instruction contains positions information. To simplify the matter, I created a virtual text file `vt` that allows writing to any specific line and column.

```py
for i in dis.get_instructions(a):
    # We only want to look at the list of variables after AES decryption
    if 2778 > i.offset > 600 and i.opname == "LOAD_NAME":
        pos = i.positions
        # Write out the symbol at the exact location.
        # This assumes the symbol has length of 1
        vt.write((pos.lineno, pos.col_offset), i.argrepr)
```

Printing out the reconstructed text, we can find the key we were looking for!

```
b;r;y;b;e;f;        e;j;n;k;i;      d;o;s;b;o;g;u;  
x;          x;    x;          b;    h;              
p;          t;    j;        i;l;    e;              
k;s;d;p;s;b;      j;    m;    g;    e;k;y;y;r;d;    
r;                d;g;        v;                i;  
p;                e;          m;    z;          m;  
o;                  f;y;n;g;r;        b;k;a;l;d;    ...
```

Now, to get the IV, the message from earlier was again helpful to give us a python array to use. We just need to remap into our text file and wallah, we got it!

```py
# From the earlier message
iv_mapper = [(476, 6), (468, 5), (282, 6), (506, 6), (420, 3), (492, 0), (192, 6), (56, 6), (144, 3), (324, 0), (360, 1), (352, 6), (30, 1), (260, 0), (298, 1), (480, 3)]
actual_mapper = [(y + 24, x) for (x, y) in iv_mapper]
actual = ''.join([vt.get(c) for c in actual_mapper])
print(actual)
```

The program should accept both the key and IV we found, and here is the flag we are looking for.

```
gg
INTIGRITI{y0u_7ruly_4r3_7h3_pyc_p4r51n6_m4573r}
```

Interestingly, when I compared with the official writeup after the CTF, the author parses from `code.co_positions` manually rather than using `dis.get_instructions(code)`. I guess the Python documentation is not informational on Python best practices.

If you want to see the code, scroll down to the [bottom](#final-words), because the next part is...

### Bonus: Whole source code reconstruction

Please hear me out. If I can produce a pyc with the same byte code and line table, then I might have the same source code (minus stuff that are not explicitly or implicitly recorded inside pyc, such as comments). I did this before recovering from the line table, because I thought I need to recover the whole source code file!

Using the source code from earlier (under file name test.py), I ran `python -m compileall test.py` to force it to produce `pyc` file.

At first, I tried to match the bytecode. I used `pycdas` to produce bytecode listing of both files (test.cpython-313.pyc and funny.pyc), and then compared them side-by-side (`vimdiff` or `vscode`, your choice). Let's look at the differences!

| funny.pyc                                 | test.cpython-313.pyc                    |
|-------------------------------------------|-----------------------------------------|
| `LOAD_CONST                      0: 0`    | `LOAD_CONST                      0: 0`  |
| `COPY                            1   `    |                                         |
| `STORE_NAME                      8: a`    | `STORE_NAME                      5: a`  |
| `COPY                            1   `    | `LOAD_CONST                      0: 0`  |
| `STORE_NAME                      9: b`    | `STORE_NAME                      6: b`  |
| `COPY                            1   `    | `LOAD_CONST                      0: 0`  |
| `STORE_NAME                      10: c`   | `STORE_NAME                      7: c`  |
| `COPY                            1   `    | `LOAD_CONST                      0: 0`  |

First, we see `COPY` opcode again. Per documenation, `COPY(i)` would copy the i-th of the stack and push it on top of the stack. How can we produce this opcode? Thankfully, from the Decompyle++ repo, we got a [minimal reproducible example](https://github.com/zrax/pycdc/issues/452#issuecomment-1999660941) for this:

```py
def COPY():
    a = 10
    b = 10
    c = 10
    return a == b == c
```

The expression after return is what produces the `COPY` opcode. For our case, we simply chain the assignment into one line. Here, I also remove space to align with the line table.

```py
a=b=c=d=e=f=g=h=i=j=k=l=m=n=o=p=q=r=s=t=u=v=w=x=y=z=0
```

Next, we see another extra section of bytecode from funny.pyc:

```
2776    PUSH_EXC_INFO                   
2778    LOAD_NAME                       5: ImportError
2780    CHECK_EXC_MATCH                 
2782    POP_JUMP_IF_FALSE               19 (to 2822)
2786    POP_TOP                         
2788    LOAD_NAME                       6: print
2790    PUSH_NULL                       
2792    LOAD_CONST                      3: 'run `pip install pycryptodome` to be able to play this challenge'
2794    CALL                            1
2802    POP_TOP                         
2804    LOAD_NAME                       7: exit
2806    PUSH_NULL                       
2808    CALL                            0
2816    POP_TOP                         
2818    POP_EXCEPT                      
2820    JUMP_BACKWARD_NO_INTERRUPT      1396 (to 32)
```

There are several interesting opcodes here: `PUSH_EXC_INFO`, `CHECK_EXC_MATCH`, `POP_EXCEPT`, `JUMP_BACKWARD_NO_INTERRUPT`. All of them suggest that there might be exception catching.
From earlier link, we can find another helpful reproducible example:

```py
def PUSH_EXC_INFO():
    try:
        raise Exception("This is an exception")
    except Exception as e:
        raise e
```

With a little bit of imagination, I successfully reconstructed them as the following code:

```py
try:
    ...
except ImportError:
    print('run `pip install pycryptodome` to be able to play this challenge')
    exit()
...
```

There are also several parts where decompiler guesses the indent incorrectly, which can be seen in different jump destination when compare.
For example, in the check, both print are next to each other, not on different scope:

```diff
     if b'INTIGRITI' in decrypted:
         print('gg')
+        print(decrypted.decode())
-    print(decrypted.decode())
```

Then the `del decrypted` is within the check rather than outside

```diff
 if len(key) == 32 and len(iv) == 16:
     decrypted = ...
     ...
+    del decrypted
-del decrypted
```

Finally, `del key` and `del iv` can be combined into one line to match the line table.

```diff
+del key, iv
-del key
-del iv
```

From this point, only the line table differs. Except for the ASCII art, The rest of the code are guessed through trial-and-error. I used [biodiff](https://github.com/8051Enthusiast/biodiff) to compare pyc side-by-side. After 2 hours, I was able to reconstruct the whole Python source code with no differences in both bytecode and line table! The only difference is the timestamp in the header, which we can ignore. (It is _Thursday, October 31, 2024 3:27:40 PM_, but you can just do time travel in your system)

You can find my final reconstruction here: [dungwinux/1337up-live:funny/chal.py](https://github.com/dungwinux/1337up-live/blob/master/funny/chal.py)

If you have read through this far, congratulation, because you have gone down the rabbit hole of pyc. I hope you learned something new through the challenge.

### Final words

You can find all of my code here: [dungwinux/1337up-live:funny/](https://github.com/dungwinux/1337up-live/tree/master/funny). I want to give my thank to the author for the wonderful and creative challenge, because I really enjoyed it, and I hope you did as well.
