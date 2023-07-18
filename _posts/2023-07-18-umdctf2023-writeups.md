---
layout: post
title: "UMDCTF 2023 - write-ups"
date: 2023-07-18 17:00:00 +0000
categories: security
tags: ctf
---

Editor note: Apologies for the long time between of this post and the contest.
I have finished writing this long ago, but I forgot to edit and publish it.
The original complete date of the draft is 2023-05-02.

UMDCTF 2023 over the weekend leaves me a great impression.
Our team, SavedByTheShell, managed to finish in 10th place in scoreboard
and 3rd place in academic rank.

Since I appreciate the challenges, I committed writing writeup for
all 9 challenges that I solved or helped solving, in chronological order.

## \[rev\] Welcome to Python!

> Jimmy has a Python program which compiled into something he's never seen. Can you fix it?

We were given an executable `chal`. Putting into Cutter (or any RE framework, I suppose),
we found several strings refer to Python. Suspecting this to be a PyInstaller
packed program, a popular packer to create single executable from a Python script,
we use `pyinsxtract` to extract it, and wallah, we got `chal.pyc`. `.pyc` stands for
Python bytecode, and this requires an additional step to convert back to Python,
as these two languages/formats are somewhat interchangeable. I first tried `uncompyle6`,
but sadly, the tool only support as high as Python 3.9 while the bytecode was translated
from a newer version. A quick online search points me to an alternative tool,
`Decompyle++`, and smoothly we receive the Python code after minimal cleanup.

```py
# Source Generated with Decompyle++
# File: chal.pyc (Python 3.10)

from math import sqrt, sin, cos
from ctypes import c_uint32, c_float
from sys import exit as exit_
source = [
    ...
]
seed = 64

def wandom(x):
    return x * x * cos(x) * sin(x) / 1000


def evil_bit_hack(y):
    return int(c_uint32.from_buffer(c_float(y)).value)

def main():
    print('==========================================')
    print('Professional flag checker service (v 97.2)')
    print('==========================================')
    flag = input('Show me the flag: ')
    lf = len(flag)
    ls = len(source)
    l = lf if lf < ls else ls
    for i in range(seed, seed + l):
        w = wandom(i)
        c = ~(~ord(flag[i - seed]) ^ evil_bit_hack(wandom(wandom(w))) & evil_bit_hack(w)) + 1
        if source[i - seed] != c:
            print("Uh oh! We don't think your flag is correct... :(")
            exit_(1)
    if lf == ls:
        print('Your flag is correct!')
        return None
    None('Some of your flag is correct...')

main()
```

Now we cannot directly run the above script, because some part of the code seems
irregular, perhaps mistranslated. Luckily, here is the only important logic
we need to focus on.

```py
w = wandom(i)
c = ~(~ord(flag[i - seed]) ^ evil_bit_hack(wandom(wandom(w))) & evil_bit_hack(w)) + 1
```

Reading the definition of function `wandom`, the return of the function is consistent
and we can predict the input. So, we don't need to care much.
For the `evil_bit_hack`, it is easily recognized as a casting that force reading
a raw float value as an int32 value, which can be roughly interpreted as
`*reinterpret_cast<int32_t*>(&value)` in C++.

In fact, the `evil_bit_hack` and `wandom` calls are computed using predetermined
input, so we don't have to reverse them. `XOR` is reversible, so we can
spice up a short Python script to solve it:

```py
from math import sqrt, sin, cos
from ctypes import c_uint32, c_float

def wandom(x):
    return x * x * cos(x) * sin(x) / 1000

def evil_bit_hack(y):
    return int(c_uint32.from_buffer(c_float(y)).value)

def unc(c, i):
    w = wandom(i)
    down = (~(c - 1)) ^ (evil_bit_hack(wandom(wandom(w))) & evil_bit_hack(w))
    flag = ~down
    return chr(flag)

source = [
    ...
]
seed = 64

# Attempted fixed `seed`, didn't work
print("".join([unc(source[i], seed + i) for i in range(len(source))]))
```

## \[rev\] Cleffa's Surprise

> Cleffa worked some magic on this binary but I can't figure out what the spell is doing!

By putting into a RE framework, we know we were given a macOS ARM 64-bit binary.

![`main` excerpt]({{ "/assets/images/umdctf2023_cleffa-1.png" | relative_url }})

At a quick glance, we see the following calls, in series:

-   A memory allocation call (`mmap`)
-   A memory copy call (`memcpy`)
-   A memory permission set call (`mprotect`)
-   And an indirect function call. (`(*pcVar2)()`)
Following the source address of memory copy call, we find out code likely to be later
indirectly executed. It has multiple `MOVK` instructions.

![indirect function]({{ "/assets/images/umdctf2023_cleffa-2.png" | relative_url }})

A little background info: in some architecture, with code size
limitation, supporting moving(`MOV`) number into register can be difficult and require
tricks up their sleeve. In ARM cases, with 4 bytes for an instruction, it sets
higher-order bits using `MOVK`. This instruction has a 16-bit field for immediate
and will optionally shift the value 16n bits.
From a different perspective, we realize the program is packing multiple characters
in to what looks like a string.

At this point, either writing a script or putting them into CyberChef will give
us the flag.

## \[rev\] Introduction to C

> "Welcome to CMSC216. Weeeeee have a lecture worksheet that the TAs will now hand out. You must write your name, student ID, and discussion session CORRECTLY at the top of the worksheet. I have 369 students, so any time I need to spend finding out who to grade will cause YOU to lose credit." (Larry Herman)

This is an interesting challenge. We were instead given a PNG, a TXT, and a
description suggesting a relationship between them.
Suspecting that the array of tuple of 3 values in TXT to be colors, I examined
PNG using color picker tools, and they indeed match. With the array size of 128,
this is likely a mapping of ASCII-RGB. I then filter the data (In VS Code
hotkey, it is `Ctrl+D`), put them in a Python script to transpose them,
and use Pillow to convert into a complete image:

```py
from PIL import Image

color_dict = [
    (0, 0, 0),
    (210, 126, 15),
    (164, 252, 30),
    (118, 122, 45),
    (72, 248, 60),
    (26, 118, 75),
    (236, 244, 90),
    (190, 114, 105),
    (144, 240, 120),
    (98, 110, 135),
    (52, 236, 150),
    (6, 106, 165),
    (216, 232, 180),
    (170, 102, 195),
    (124, 228, 210),
    (78, 98, 225),
    (32, 224, 240),
    (242, 94, 255),
    (196, 220, 14),
    (150, 90, 29),
    (104, 216, 44),
    (58, 86, 59),
    (12, 212, 74),
    (222, 82, 89),
    (176, 208, 104),
    (130, 78, 119),
    (84, 204, 134),
    (38, 74, 149),
    (248, 200, 164),
    (202, 70, 179),
    (156, 196, 194),
    (110, 66, 209),
    (64, 192, 224),
    (18, 62, 239),
    (228, 188, 254),
    (182, 58, 13),
    (136, 184, 28),
    (90, 54, 43),
    (44, 180, 58),
    (254, 50, 73),
    (208, 176, 88),
    (162, 46, 103),
    (116, 172, 118),
    (70, 42, 133),
    (24, 168, 148),
    (234, 38, 163),
    (188, 164, 178),
    (142, 34, 193),
    (96, 160, 208),
    (50, 30, 223),
    (4, 156, 238),
    (214, 26, 253),
    (168, 152, 12),
    (122, 22, 27),
    (76, 148, 42),
    (30, 18, 57),
    (240, 144, 72),
    (194, 14, 87),
    (148, 140, 102),
    (102, 10, 117),
    (56, 136, 132),
    (10, 6, 147),
    (220, 132, 162),
    (174, 2, 177),
    (128, 128, 192),
    (82, 254, 207),
    (36, 124, 222),
    (246, 250, 237),
    (200, 120, 252),
    (154, 246, 11),
    (108, 116, 26),
    (62, 242, 41),
    (16, 112, 56),
    (226, 238, 71),
    (180, 108, 86),
    (134, 234, 101),
    (88, 104, 116),
    (42, 230, 131),
    (252, 100, 146),
    (206, 226, 161),
    (160, 96, 176),
    (114, 222, 191),
    (68, 92, 206),
    (22, 218, 221),
    (232, 88, 236),
    (186, 214, 251),
    (140, 84, 10),
    (94, 210, 25),
    (48, 80, 40),
    (2, 206, 55),
    (212, 76, 70),
    (166, 202, 85),
    (120, 72, 100),
    (74, 198, 115),
    (28, 68, 130),
    (238, 194, 145),
    (192, 64, 160),
    (146, 190, 175),
    (100, 60, 190),
    (54, 186, 205),
    (8, 56, 220),
    (218, 182, 235),
    (172, 52, 250),
    (126, 178, 9),
    (80, 48, 24),
    (34, 174, 39),
    (244, 44, 54),
    (198, 170, 69),
    (152, 40, 84),
    (106, 166, 99),
    (60, 36, 114),
    (14, 162, 129),
    (224, 32, 144),
    (178, 158, 159),
    (132, 28, 174),
    (86, 154, 189),
    (40, 24, 204),
    (250, 150, 219),
    (204, 20, 234),
    (158, 146, 249),
    (112, 16, 8),
    (66, 142, 23),
    (20, 12, 38),
    (230, 138, 53),
    (184, 8, 68),
    (138, 134, 83),
    (92, 4, 98),
    (46, 130, 113),
]


im = Image.open("intro_to_c.png")
decoded = "".join(
    [chr(color_dict.index(d)) for d in list(im.getdata()) if d != (255, 255, 255)]
)
print(decoded)
```

But it is not done yet, we now got a C or C++ source code:

```c
#include <stdio.h>

#define LEN(array) sizeof(array) / sizeof(*array)

#define SALT_1 97
#define SALT_2 4563246763
const long numbers[] = {4563246815, 4563246807, 4563246800, 4563246797, 4563246816, 4563246802, 4563246789, \
4563246780, 4563246783, 4563246850, 4563246843, 4563246771, 4563246765, 4563246825, 4563246781, 4563246784, \
4563246796, 4563246784, 4563246843, 4563246765, 4563246825, 4563246786, 4563246844, 4563246803, 4563246800, \
4563246825, 4563246775, 4563246852, 4563246843, 4563246778, 4563246825, 4563246781, 4563246849, 4563246782, \
4563246843, 4563246778, 4563246769, 4563246825, 4563246796, 4563246782, 4563246769, 4563246781, 4563246821, \
4563246823, 4563246827, 4563246827, 4563246827, 4563246791};

int main(void)
{
    size_t i;
    char undecyphered_char;

    for (i = 0; i < LEN(numbers); i++)
    {
        undecyphered_char = (char)((numbers[i] - SALT_2) ^ 97);

        printf("%c", undecyphered_char);
    }

    printf("\n");

    return 0;
}
```

This is in fact a simple XOR challenge. Reversing the numbers gives us the flag.

## \[mobile\] Who's That Pokemon?

> Who's That Pokemon? Enter the Pokemon's name to find the flag!

Scanning the APK using JADX, we see a resource named `encrypted`.
Tracing resource usage, we can find references that the data was AES-256 encrypted, 
its first 16 bytes is the IV. The last step to completely decrypt the content
is to find the encryption key. Reading the source code surrounding the current line, we
find that it checks the key with one of a string resource, right padded with spaces.
Cross-referencing the string resource key, we find the encryption key and
successfully capture the flag.

## \[HW\] beep-boop

We were given an audio file with a Matlab script that is likely used to generate
it. Believing it is a signal challenge, I opened Sonic Visualizer, selected
Spectrogram, and Wallah.

![Beepboop view]({{ "/assets/images/umdctf2023_beep-boop-1.png" | relative_url }})

With a little bit of filtering, we got this

![Beepboop filtered view]({{ "/assets/images/umdctf2023_beep-boop-2.png" | relative_url }})

Selecting _File > Export Annotation Layer..._, we got the top two values,
matching the frequency functions mentioned in the Matlab code.


```matlab
function freq = get_frequency_1(char)
    freq = char * 13;
end

function freq = get_frequency_2(char)
    freq = (char - 50) * 11;
end
```


![Beepboop encoded data]({{ "/assets/images/umdctf2023_beep-boop-3.png" | relative_url }})

From here, we write a Python script that reverse the operation and get the
original output. Due to signal on power-of-2 bitrate,
a little bit of rounding was used, and then we got the flag.

```py
import csv

r = lambda x, y: (x - y - 550)/2
c = lambda x, y: chr(int(r(x, y)))

data = []
with open('test.csv', newline='\r\n') as csvfile:
    raw = csv.reader(csvfile, delimiter='\t')
    for row in raw:
        if len(row) == 5:
            data.append((round(float(row[1])), round(float(row[3]))))

print(data)
print("".join([c(x, y) for y, x in data]))
```

## \[forensics\] Telekinetic Warfare

We were given a gif image. Opening it shows different QR Code display in each frame.
Attempt to decode the first frame hints it to be a PDF file. Parsing the rest
reveal a PDF that contains the flag.

```py
from PIL import Image
from pyzbar.pyzbar import decode
from base64 import b64decode

newfile = b""

with Image.open('bruh.gif') as im:
    with open('bruh.pdf', "wb") as pd:
        for i in range(im.n_frames):
            im.seek(i)
            qr = decode(im.copy())
            data = b64decode(qr[0].data)
            pd.write(data)
```

## \[mobile\] JNIdorino

> I hate using native libraries so much! I think I forgot a function call somewhere. Can you find it?

Despite being mobile, this one delivers an Android app that uses native function,
also known as JNI in Java world. Based on the description, we extract
function names from both Java code and native binary (using RE framework).
For the APK, I use JADX, while for binary, I used Ghidra. We extract function
names from both binary into separate text files for ease of access.
Using UNIX `sort` and `uniq`, we can quickly filter and find the missing/unique
function. That sole function contains the flag we are looking for.

## \[mobile\] Flamecamp

> Flamecamp is a new social media app exclusive to Fire type Pokemon. Can you get access to it?

This is quite a fun one, particularly because of my experience with Firebase
before it was bought by Google. The android app connects to a
Firebase Cloud Storage and download a file under the path `flamecamp_welcome.jpg`
in the online database.
The rest of the app looks generic, so we thought we have to find ways to login
in to the database.
Originally, my teammate `pow-a` first tried to access the bucket through
Google Cloud API, but seems like it is not allowed.

I then remembered from experience that a Firebase app may allow Registration
and Anonymous Login. Before I tried the former, I found a string resource
mentions that registration is impossible, so I immediately try the later way.
Spending little time reading Firebase SDK and scripting short JavaScript Node.js
code, I was able to quickly fetch and download the file using Anonymous Login.
The flag is (poorly) handwritten in the image.

```js
import { initializeApp } from "firebase/app";
import { getBytes, getStorage, ref } from "firebase/storage";
import { getAuth, signInAnonymously } from "firebase/auth";
import { writeFileSync } from "fs";

const firebaseConfig = {
    apiKey: "AIzaSyANqtzq5t7AJ1FDJjK9if77WsV_ebvktiA",
    authDomain: "flamecamp.firebaseapp.com",
    // The value of `databaseURL` depends on the location of the database
    databaseURL: "https://flamecamp.firebaseio.com",
    projectId: "flamecamp",
    storageBucket: "flamecamp.appspot.com",
    messagingSenderId: "1090871602799",
    appId: "1:1090871602799:android:c1d198201cd7635db5279a",
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);

const auth = getAuth();
await signInAnonymously(auth);

// Initialize Cloud Storage and get a reference to the service
const storage = getStorage(app);
const pathReference = ref(storage, 'flamecamp_welcome.jpg');

const data = await getBytes(pathReference);
writeFileSync("flamecamp_welcome.jpg", Buffer.from(data));
```

Additional `package.json` to run the Node.js script.

```json
{
  "scripts": {
    "start": "node solve.js"
  },
  "type": "module",
  "dependencies": {
    "firebase": "^9.21.0"
  }
}
```