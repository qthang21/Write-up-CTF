# Are You Pylingual
<img width="523" height="627" alt="image" src="https://github.com/user-attachments/assets/eadd3c09-4036-4a70-8e4b-854479b7f7ce" />

**Mô tả:**

 Challenge này cho 1 file .pyc ta chuyển sang .py bằng pylingual.io sau đó dịch ngược lại chương trình thì sẽ tìm ra được Flag.

**Phân tích**
```py
# Decompiled with PyLingual (https://pylingual.io)
# Internal filename: pylinguese.py
# Bytecode version: 3.12.0rc2 (3531)
# Source timestamp: 2025-09-06 18:41:22 UTC (1757184082)

import pyfiglet
file = open('flag.txt', 'r')
flag = file.read()
font = 'slant'
words = 'MASONCC IS THE BEST CLUB EVER'
flag_track = 0
art = list(pyfiglet.figlet_format(words, font=font))
i = len(art) % 10
for ind in range(len(art)):
    if ind == i and flag_track < len(flag):
        art[ind] = flag[flag_track]
        i += 28
        flag_track += 1
art_str = ''.join(art)
first_val = 5
second_val = 6
first_half = art_str[:len(art_str) // 2]
second_half = art_str[len(art_str) // 2:]
first = [~ord(char) ^ first_val for char in first_half]
second = [~ord(char) ^ second_val for char in second_half]
output = second + first
print(output)
```

`list(pyfiglet.figlet_format(words, font=font))` trả về một chuỗi nhiều dòng (string) chứa các ký tự ' ', '_', '/' hay '\n' để tạo hình chữ `MASONCC IS THE BEST CLUB EVER`.

`i = len(art) % 10` vị trí bắt đầu phụ thuộc vào tổng độ dài ASCII-art

```py
for ind in range(len(art)):
    if ind == i and flag_track < len(flag):
        art[ind] = flag[flag_track]
        i += 28
        flag_track += 1
```
Đoạn code trên thực hiện việc nhúng flag vào ASCII-art theo cách sau:
- bắt đầu từ vị trí i = len(art) % 10.

- cứ mỗi 28 ký tự lại thay 1 ký tự ASCII-art bằng 1 ký tự flag.

- tiếp tục cho đến khi hết flag hoặc hết vị trí hợp lệ.

```py
art_str = ''.join(art)
first_val = 5
second_val = 6
first_half = art_str[:len(art_str) // 2]
second_half = art_str[len(art_str) // 2:]
```

Sau khi có được chuỗi ASCII chứa Flag được nhúng vào rồi thì chia đôi chuỗi ra.

```py
first = [~ord(char) ^ first_val for char in first_half]
second = [~ord(char) ^ second_val for char in second_half]
output = second + first
print(output)
```
Sau khi chia ra rồi thì mã hóa chúng và ghép chúng lại thì ta có được output.

### Cách giải
Ta chia đôi output ra làm 2 phần.
Sau đó giải mã từng phần và ghép lại thì ra được chuỗi ASCII có chứa Flag bên trong.
Ghép các ký tự của đã được nhúng bên trong lại ta ra được Flag.

### Script
```py
#!/usr/bin/env python3

import ast

# Load output array 
with open("output.txt") as f:
    data = ast.literal_eval(f.read().strip().replace("output =", ""))

output = data
n = len(output)

first_val = 5
second_val = 6

# output = second + first
half = n // 2
second_enc = output[:half]
first_enc = output[half:]

def decode(arr, key):
    res = []
    for v in arr:
        c = (~(v ^ key)) & 0xFF
        res.append(chr(c))
    return "".join(res)

first_half = decode(first_enc, first_val)
second_half = decode(second_enc, second_val)

flag = first_half + second_half
print("Recovered flag:")
print(flag)
```
<img width="684" height="409" alt="image" src="https://github.com/user-attachments/assets/7b425916-ac69-4f37-86b4-f05fe0da76f0" />

Flag = pctf{obFusc4ti0n_i5n't_EncRypt1oN}

