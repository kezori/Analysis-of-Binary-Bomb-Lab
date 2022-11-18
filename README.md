# Analysis of A Binary Bomb Lab / Giải bomb nhị phân

## Thang Long University - Computer Architecture - CS212 - Semester I/2022

```
Start working on 11/17/2022 - During the period of the final exam of the first term in 2022
Please feel free to fork or star if helpful! (^^ゞ
Created by: A41316 - Nguyễn Hữu Khoa
```

_The article is referenced from:_

- [https://github.com/sc2225/Bomb-Lab](https://github.com/sc2225/Bomb-Lab)

## Overview / Tổng quan về giải bomb

Read in this [post](https:;github.com/kr4zym3nvn/Analysis-of-Binary-Bomb-Lab/blob/master/Analysis%20of%20CME%20bomb%20lab%20program.md)

## Getting Strings and Objdump

Run the following commands to create text files which we will look at later:

_Chạy các dòng lệnh sau để tạo file text mà chúng ta sẽ cần sau này_

```
strings bomb > strings.txt
objdump -d bomb > assembly.txt
```

You should now have two files: strings.txt and assembly.txt. Now let’s get started with Phase 1!

_Bây giờ chúng ta sẽ có 2 tệp strings.txt và assembly.txt. Bây giờ chúng ta sẽ bắt đầu với Phase 1!_

## Phase 1:

```assembly
0000000000400ef0 <phase_1>:
=> 0x0000000000400ef0 <+0>:     sub    $0x8,%rsp ; building stack frame with 8 more bytes
   0x0000000000400ef4 <+4>:     mov    $0x401ae8,%esi ; what is this being moved to esi? (0x401ae8)
   0x0000000000400ef9 <+9>:     call   0x4012de <strings_not_equal> ; call strings_not_equal function
   0x0000000000400efe <+14>:    test   %eax,%eax ; test eax register
   0x0000000000400f00 <+16>:    je     0x400f07 <phase_1+23> ; jump if equal
   0x0000000000400f02 <+18>:    call   0x4016b3 <explode_bomb> ; call explode_bomb function
   0x0000000000400f07 <+23>:    add    $0x8,%rsp
   0x0000000000400f0b <+27>:    ret
```

Chúng ta có thể thấy rằng hàm `<strings_not_equal>` đang được gọi với hàm ý so sánh hai chuỗi.
Việc đầu tiên là cần xác định được bối cảnh thủ tục hay chính là các biến được truyền vào hàm là gì.
Dễ dàng nhận thấy biến sử dụng trong so sánh là giá trị của địa chỉ lưu trong thanh ghi `%eax`. Ngay trước khi gọi hàm, ở trên có xuất hiện thanh ghi `%esi` cũng liên quan. Ta sử dụng địa chỉ đó trong bộ nhớ và xem nó chứa gì dưới dạng chuỗi.

```objdump
400ef4:	be e8 1a 40 00       	mov    $0x401ae8,%esi
```

Lets examine what is being moved from address 0x401ae8. We know it has to be a string of some sort so we use '/s'.

```assembly
(gdb) x/s 0x401ae8
0x401ae8:       "Science isn't about why, it's about why not?"
```

Như vậy chuỗi này được chuyển vào thanh ghi %esi và tiếp đó được truyền vào hàm <strings_not_equal> để so sánh. Cùng phân tích hàm này.

```assembly
00000000004012de <strings_not_equal>:
   0x00000000004012de <+0>:     push   %r12
   0x00000000004012e0 <+2>:     push   %rbp
   0x00000000004012e1 <+3>:     push   %rbx
   0x00000000004012e2 <+4>:     mov    %rdi,%rbx
   0x00000000004012e5 <+7>:     mov    %rsi,%rbp
   0x00000000004012e8 <+10>:    call   0x4012c1 <string_length>
   0x00000000004012ed <+15>:    mov    %eax,%r12d
   0x00000000004012f0 <+18>:    mov    %rbp,%rdi
   0x00000000004012f3 <+21>:    call   0x4012c1 <string_length>
   0x00000000004012f8 <+26>:    mov    $0x1,%edx
   0x00000000004012fd <+31>:    cmp    %eax,%r12d
   0x0000000000401300 <+34>:    jne    0x401340 <strings_not_equal+98>
   0x0000000000401302 <+36>:    movzbl (%rbx),%eax
   0x0000000000401305 <+39>:    test   %al,%al
   0x0000000000401307 <+41>:    je     0x40132d <strings_not_equal+79>
   0x0000000000401309 <+43>:    cmp    0x0(%rbp),%al
   0x000000000040130c <+46>:    je     0x401317 <strings_not_equal+57>
   0x000000000040130e <+48>:    xchg   %ax,%ax
   0x0000000000401310 <+50>:    jmp    0x401334 <strings_not_equal+86>
   0x0000000000401312 <+52>:    cmp    0x0(%rbp),%al
   0x0000000000401315 <+55>:    jne    0x40133b <strings_not_equal+93>
   0x0000000000401317 <+57>:    add    $0x1,%rbx
   0x000000000040131b <+61>:    add    $0x1,%rbp
   0x000000000040131f <+65>:    movzbl (%rbx),%eax
   0x0000000000401322 <+68>:    test   %al,%al
   0x0000000000401324 <+70>:    jne    0x401312 <strings_not_equal+52>
   0x0000000000401326 <+72>:    mov    $0x0,%edx
   0x000000000040132b <+77>:    jmp    0x401340 <strings_not_equal+98>
--Type <RET> for more, q to quit, c to continue without paging--RET
   0x000000000040132d <+79>:    mov    $0x0,%edx
   0x0000000000401332 <+84>:    jmp    0x401340 <strings_not_equal+98>
   0x0000000000401334 <+86>:    mov    $0x1,%edx
   0x0000000000401339 <+91>:    jmp    0x401340 <strings_not_equal+98>
   0x000000000040133b <+93>:    mov    $0x1,%edx
   0x0000000000401340 <+98>:    mov    %edx,%eax
   0x0000000000401342 <+100>:   pop    %rbx
   0x0000000000401343 <+101>:   pop    %rbp
   0x0000000000401344 <+102>:   pop    %r12
   0x0000000000401346 <+104>:   ret
```

`<string_not_equal>` không gọi tới explode_bomb nên ta có thể bỏ qua nó. Hàm này sẽ trả về giá trị 0 nếu hai chuỗi bằng nhau và 1 nếu hai chuỗi khác nhau. Ta có thể thấy rằng hàm này sẽ so sánh hai chuỗi bằng cách so sánh từng ký tự trong chuỗi. Nếu hai chuỗi có độ dài khác nhau thì hàm sẽ trả về 1. Nếu hai chuỗi có độ dài bằng nhau thì hàm sẽ so sánh từng ký tự trong chuỗi. Nếu hai ký tự khác nhau thì hàm sẽ trả về 1. Nếu hai ký tự bằng nhau thì hàm sẽ tiếp tục so sánh ký tự tiếp theo. Nếu hai chuỗi bằng nhau thì hàm sẽ trả về 0. Cuối cùng, để vượt qua bài kiểm tra này, tất cả những gì bạn cần làm là nhập bất kỳ chuỗi nào có độ dài 46 ký tự không bắt đầu bằng số 0.

Sử dụng lệnh `ni 3` để di chuyển breakpoint tới dòng test `0x0000000000400efe <+14>: test %eax,%eax`.
Sau đó thực hiện lệnh `info register` để xem giá trị của các thanh ghi cho tới bước so sánh đó

```assembly
(gdb) ni 3
0x0000000000400efe in phase_1 ()
(gdb) disas phase_1
Dump of assembler code for function phase_1:
   0x0000000000400ef0 <+0>:     sub    $0x8,%rsp
   0x0000000000400ef4 <+4>:     mov    $0x401ae8,%esi
   0x0000000000400ef9 <+9>:     call   0x4012de <strings_not_equal>
=> 0x0000000000400efe <+14>:    test   %eax,%eax
   0x0000000000400f00 <+16>:    je     0x400f07 <phase_1+23>
   0x0000000000400f02 <+18>:    call   0x4016b3 <explode_bomb>
   0x0000000000400f07 <+23>:    add    $0x8,%rsp
   0x0000000000400f0b <+27>:    ret
End of assembler dump.
```

Ta được kết quả:

```assembly
(gdb) i r
rax            0x1                 1 ; sẽ gọi tới explode_bomb nếu giá trị của thanh ghi này khác 0
rbx            0x0                 0
rcx            0x3                 3
rdx            0x1                 1
```

Bây giờ chúng ta sẽ sử dụng chuỗi mà chúng ta tìm thấy lúc nãy và xem giá trị của thanh ghi %eax: `Science isn't about why, it's about why not?`

```assembly
(gdb) b phase_1
Breakpoint 1 at 0x400ef0
(gdb) r
Starting program: /home/k4zy/ktmt/bomb23/bomb
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Science isn't about why, it's about why not?

Breakpoint 1, 0x0000000000400ef0 in phase_1 ()
(gdb) disas
Dump of assembler code for function phase_1:
=> 0x0000000000400ef0 <+0>:     sub    $0x8,%rsp
   0x0000000000400ef4 <+4>:     mov    $0x401ae8,%esi
   0x0000000000400ef9 <+9>:     call   0x4012de <strings_not_equal>
   0x0000000000400efe <+14>:    test   %eax,%eax
   0x0000000000400f00 <+16>:    je     0x400f07 <phase_1+23>
   0x0000000000400f02 <+18>:    call   0x4016b3 <explode_bomb>
   0x0000000000400f07 <+23>:    add    $0x8,%rsp
   0x0000000000400f0b <+27>:    ret
End of assembler dump.
(gdb) ni 3
0x0000000000400efe in phase_1 ()
(gdb) i r
rax            0x0                 0 ; giá trị của thanh ghi này bằng 0 nên sẽ không gọi tới (jump pass) explode_bomb
rbx            0x0                 0
rcx            0x2c                44
rdx            0x0                 0
```

Vậy lời giải cho Phase 1 là `Science isn't about why, it's about why not?`

## Phase 2:

```assembly
Dump of assembler code for function phase_2:
   0x0000000000400f0c <+0>:     push   %r13
   0x0000000000400f0e <+2>:     push   %r12
   0x0000000000400f10 <+4>:     push   %rbp
   0x0000000000400f11 <+5>:     push   %rbx
   0x0000000000400f12 <+6>:     sub    $0x28,%rsp ; mở rộng ngăn xếp 56 bytes tương đương với 14 ngăn xếp 4 bytes
   0x0000000000400f16 <+10>:    mov    %rsp,%rsi ; lưu địa chỉ của thanh ghi %rsp vào thanh ghi %rsi
   0x0000000000400f19 <+13>:    call   0x4016d5 <read_six_numbers> ; Chúng ta có thể thấy yêu cầu của phase này chính là nhập vào 6 số
   0x0000000000400f1e <+18>:    mov    %rsp,%rbx ; lưu giá trị của thanh ghi %rsp (số thứ nhất) vào thanh ghi %rbx
   0x0000000000400f21 <+21>:    lea    0xc(%rsp),%r13 ; lưu giá trị của thanh ghi %rsp + 12 vào thanh ghi %r13
   0x0000000000400f26 <+26>:    mov    $0x0,%ebp ; lưu giá trị 0 vào thanh ghi %ebp
   0x0000000000400f2b <+31>:    mov    %rbx,%r12 ; lưu giá trị của thanh ghi %rbx vào thanh ghi %r12
   0x0000000000400f2e <+34>:    mov    0xc(%rbx),%eax ; lưu giá trị của thanh ghi %rbx + 12 vào thanh ghi %eax
   0x0000000000400f31 <+37>:    cmp    %eax,(%rbx) ; so sánh giá trị của thanh ghi %rbx + 12 với giá trị của thanh ghi %rbx
   0x0000000000400f33 <+39>:    je     0x400f3a <phase_2+46> ; nếu giá trị của thanh ghi %rbx + 12 bằng giá trị của thanh ghi %rbx thì nhảy tới 0x400f3a
   0x0000000000400f35 <+41>:    call   0x4016b3 <explode_bomb> ; gọi hàm explode_bomb
   0x0000000000400f3a <+46>:    add    (%r12),%ebp ; cộng giá trị của thanh ghi %r12 với giá trị của thanh ghi %ebp
   0x0000000000400f3e <+50>:    add    $0x4,%rbx ; cộng giá trị của thanh ghi %rbx + 4
   0x0000000000400f42 <+54>:    cmp    %r13,%rbx ; so sánh giá trị của thanh ghi %r13 với giá trị của thanh ghi %rbx
   0x0000000000400f45 <+57>:    jne    0x400f2b <phase_2+31> ; nếu giá trị của thanh ghi %r13 khác giá trị của thanh ghi %rbx thì nhảy tới 0x400f2b
   0x0000000000400f47 <+59>:    test   %ebp,%ebp ; so sánh giá trị của thanh ghi %ebp với giá trị của thanh ghi %ebp
   0x0000000000400f49 <+61>:    jne    0x400f50 <phase_2+68> ; nếu giá trị của thanh ghi %ebp khác giá trị của thanh ghi %ebp thì nhảy tới 0x400f50
   0x0000000000400f4b <+63>:    call   0x4016b3 <explode_bomb>
   0x0000000000400f50 <+68>:    add    $0x28,%rsp ;
   0x0000000000400f54 <+72>:    pop    %rbx
   0x0000000000400f55 <+73>:    pop    %rbp
   0x0000000000400f56 <+74>:    pop    %r12
   0x0000000000400f58 <+76>:    pop    %r13
   0x0000000000400f5a <+78>:    ret
```

Nhập vào 6 số nguyên `1 2 3 4 5 6`và disas <read_six_number>:

```assembly
(gdb) disas read_six_numbers
Dump of assembler code for function read_six_numbers:
   0x00000000004016d5 <+0>:     sub    $0x18,%rsp ; mở rộng ngăn xếp 24 bytes tương đương với 6 ngăn xếp 4 bytes
   0x00000000004016d9 <+4>:     mov    %rsi,%rdx ; lưu giá trị của thanh ghi %rsi vào thanh ghi %rdx
   0x00000000004016dc <+7>:     lea    0x4(%rsi),%rcx ; lưu giá trị của thanh ghi %rsi + 4 vào thanh ghi %rcx
   0x00000000004016e0 <+11>:    lea    0x14(%rsi),%rax ; lưu giá trị của thanh ghi %rsi + 20 vào thanh ghi %rax
   0x00000000004016e4 <+15>:    mov    %rax,0x8(%rsp)
   0x00000000004016e9 <+20>:    lea    0x10(%rsi),%rax
   0x00000000004016ed <+24>:    mov    %rax,(%rsp)
   0x00000000004016f1 <+28>:    lea    0xc(%rsi),%r9
   0x00000000004016f5 <+32>:    lea    0x8(%rsi),%r8
   0x00000000004016f9 <+36>:    mov    $0x401e2e,%esi ; giá trị tại địa chỉ $0x401e2e được chuyển vào thanh ghi %esi được so sánh với chuỗi mà ta đưa vào ban đầu
   0x00000000004016fe <+41>:    mov    $0x0,%eax
   0x0000000000401703 <+46>:    call   0x400c00 <__isoc99_sscanf@plt>
   0x0000000000401708 <+51>:    cmp    $0x5,%eax
   0x000000000040170b <+54>:    jg     0x401712 <read_six_numbers+61> ; nếu giá trị của thanh ghi %eax lớn hơn 5 thì bom không nổ
   0x000000000040170d <+56>:    call   0x4016b3 <explode_bomb> ; đặt breakpoint tại đây để bomb không nổ
   0x0000000000401712 <+61>:    add    $0x18,%rsp
   0x0000000000401716 <+65>:    ret
```

Chúng ta kiểm tra xem 0x401e2e có dạng như thế nào:

```assembly
(gdb) x/s 0x401e2e
0x401e2e:       "%d %d %d %d %d %d"
```

Nhất định đó chính là dạng của câu trả lời ta đang cần tìm, chính là 6 số nguyên với khoảng trống ở giữa. Nhìn vào đoạn sau:

```assembly
   0x0000000000401708 <+51>:    cmp    $0x5,%eax
   0x000000000040170b <+54>:    jg     0x401712 <read_six_numbers+61>
   0x000000000040170d <+56>:    call   0x4016b3 <explode_bomb>
```

Chúng ta có thể thấy nó có thể so sánh định dạng đầu vào của chúng ta với định dạng trong thanh ghi %esi. Nếu chúng ta nhièu hơn 5 chữ số hay cụ thể là 6 thì chúng ta có thể vượt qua được việc gọi tới quả bom và định dạng đáp án là `%d %d %d %d %d %d`

Hãy xem câu lệnh so sánh đang so sánh điều gì

```assembly
0x0000000000400f31 in phase_2 ()
(gdb) disas
Dump of assembler code for function phase_2:
   0x0000000000400f0c <+0>:     push   %r13
   0x0000000000400f0e <+2>:     push   %r12
   0x0000000000400f10 <+4>:     push   %rbp
   0x0000000000400f11 <+5>:     push   %rbx
   0x0000000000400f12 <+6>:     sub    $0x28,%rsp
   0x0000000000400f16 <+10>:    mov    %rsp,%rsi
   0x0000000000400f19 <+13>:    call   0x4016d5 <read_six_numbers>
   0x0000000000400f1e <+18>:    mov    %rsp,%rbx
   0x0000000000400f21 <+21>:    lea    0xc(%rsp),%r13
   0x0000000000400f26 <+26>:    mov    $0x0,%ebp
   0x0000000000400f2b <+31>:    mov    %rbx,%r12
   0x0000000000400f2e <+34>:    mov    0xc(%rbx),%eax
=> 0x0000000000400f31 <+37>:    cmp    %eax,(%rbx)
   0x0000000000400f33 <+39>:    je     0x400f3a <phase_2+46>
   0x0000000000400f35 <+41>:    call   0x4016b3 <explode_bomb>
   0x0000000000400f3a <+46>:    add    (%r12),%ebp
   0x0000000000400f3e <+50>:    add    $0x4,%rbx
   0x0000000000400f42 <+54>:    cmp    %r13,%rbx
   0x0000000000400f45 <+57>:    jne    0x400f2b <phase_2+31>
   0x0000000000400f47 <+59>:    test   %ebp,%ebp
   0x0000000000400f49 <+61>:    jne    0x400f50 <phase_2+68>
   0x0000000000400f4b <+63>:    call   0x4016b3 <explode_bomb>
   0x0000000000400f50 <+68>:    add    $0x28,%rsp
   0x0000000000400f54 <+72>:    pop    %rbx
   0x0000000000400f55 <+73>:    pop    %rbp
   0x0000000000400f56 <+74>:    pop    %r12
   0x0000000000400f58 <+76>:    pop    %r13
   0x0000000000400f5a <+78>:    ret
--Type <RET> for more, q to quit, c to continue without paging--
End of assembler dump.
(gdb) i r
rax            0x4                 4 ; giá trị của thanh ghi %eax hay chính là giá trị của thanh ghi %rbx + 0xc (số thứ 4 nhập vào)
rbx            0x7fffffffde80      140737488346752 ; kiểm tra cái này
rcx            0x7fffffffde70      140737488346736
rdx            0x0                 0
rsi            0x6                 6
rdi            0x7fffffffd810      140737488345104
rbp            0x0                 0x0
rsp            0x7fffffffde80      0x7fffffffde80
r8             0x1999999999999999  1844674407370955161
r9             0x0                 0
r10            0x7ffff7f47ac0      140737353382592
r11            0x7ffff7f483c0      140737353384896
r12            0x7fffffffde80      140737488346752
r13            0x7fffffffde8c      140737488346764
r14            0x0                 0
r15            0x7ffff7ffd040      140737354125376
```

Chúng ta kiểm tra giá trị tại địa chỉ lưu trong thanh ghi rbx là bao nhiêu

```assembly
(gdb) x/d 140737488346752
0x7fffffffde80: 1
```

Ta có 2 kết luận:

- Giá trị của thanh ghi %eax hay chính là giá trị của thanh ghi %rbx + 0xc bằng 4, vậy tức là số thứ 4 nhập có địa chỉ nằm tại thanh ghi %rbx + 0xc
  ```assembly
  0x0000000000400f2e <+34>: mov 0xc(%rbx),%eax
  ```
- Giá trị thanh ghi %rbx bằng 1 => thanh ghi %rsp lưu địa chỉ có giá trị số thứ 1 ta nhập vào
  ```assembly
  0x0000000000400f1e <+18>: mov %rsp,%rbx
  ```

So sánh 2 số thứ 1 và 4 nhập vào nếu không bằng nhau sẽ nổ bom

```assembly
=> 0x0000000000400f31 <+37>:    cmp    %eax,(%rbx)
   0x0000000000400f33 <+39>:    je     0x400f3a <phase_2+46>
   0x0000000000400f35 <+41>:    call   0x4016b3 <explode_bomb>
```

Ta tiếp tục kiểm tra các dòng hợp ngữ phía sau:

```assembly
   0x0000000000400f3a <+46>:    add    (%r12),%ebp
   0x0000000000400f3e <+50>:    add    $0x4,%rbx ; thanh ghi %rbx + 4 = địa chỉ của ô chứa giá trị số thứ 2 được nhập vào
   0x0000000000400f42 <+54>:    cmp    %r13,%rbx ; so sánh giá trị số thứ 2 và số thứ 4
   0x0000000000400f45 <+57>:    jne    0x400f2b <phase_2+31>
   0x0000000000400f47 <+59>:    test   %ebp,%ebp
   0x0000000000400f49 <+61>:    jne    0x400f50 <phase_2+68>
   0x0000000000400f4b <+63>:    call   0x4016b3 <explode_bomb>
   0x0000000000400f50 <+68>:    add    $0x28,%rsp
   0x0000000000400f54 <+72>:    pop    %rbx
   0x0000000000400f55 <+73>:    pop    %rbp
   0x0000000000400f56 <+74>:    pop    %r12
   0x0000000000400f58 <+76>:    pop    %r13
   0x0000000000400f5a <+78>:    ret
```

```assembly
0x0000000000400f3e <+50>: add $0x4,%rbx
```

Địa chỉ thanh ghi %rbx được tăng thêm 4, chuyển tới trỏ vào ngăn xếp chứa giá trị thứ 2 được nhập vào

```assembly
0x0000000000400f42 <+54>: cmp %r13,%rbx
```

Ta thấy số thứ 2 được so sánh với giá trị tại ô nhớ lưu ở %r13 hay chính là địa chỉ của ô nhớ %rsp + 0xc (+12 / tương đương 3 hàng, từ việc trỏ tới hàng thứ nhất di chuyển qua trỏ hàng thứ 4 trong ngăn xếp) (số thứ 4 nhập vào)

```assembly
0x0000000000400f45 <+57>: jne 0x400f2b <phase_2+31>
```

So sánh 2 số thứ 2 và 4 nhập vào nếu không bằng nhau sẽ nhảy tới dòng 31

```assembly
0x0000000000400f2b <+31>:    mov    %rbx,%r12 ; chuyển giá trị số thứ 2 vào thanh ghi %r12
```

```assembly
0x0000000000400f2e <+34>: mov 0xc(%rbx),%eax
```

Di chuyển giá trị tại địa chỉ %rbx + 0xc (tương đương số thứ 5 nhập vào vì %rbx đang chứa địa chỉ trỏ vào số thứ 2) vào thanh ghi %eax

```assembly
0x0000000000400f31 <+37>: cmp %eax,(%rbx)
```

Lúc này so sánh giá trị tại 2 thanh ghi %eax và %rbx (tương đương số thứ 5 và số thứ 2 nhập vào)

```assembly
0x0000000000400f33 <+39>: je 0x400f3a <phase_2+46>
```

So sánh giá trị, nếu bằng sẽ nhảy tới dòng 46

```assembly
   0x0000000000400f3a <+46>:    add    (%r12),%ebp
   0x0000000000400f3e <+50>:    add    $0x4,%rbx
   0x0000000000400f42 <+54>:    cmp    %r13,%rbx
   0x0000000000400f45 <+57>:    jne    0x400f2b <phase_2+31>
```

Dễ nhận ra rằng chúng ta lại đang trong vòng lặp và tiếp tục để thanh ghi %rbx trỏ tới các địa chỉ chứa các số mình nhập vào, và so sánh các cặp số: số thứ nhất với số thứ 4, số thứ 2 với số thứ 5 và số thứ 3 với số thứ 6. Ngay khi 1 trong các cặp số trên khác giá trị nhau bom sẽ phát nổ.

Các chuỗi 6 số nguyên thỏa mãn là các chuỗi 6 số đối xứng với nhau, ví dụ: 1 2 3 3 2 1

Thử đáp án : 1 2 3 3 2 1

```assembly
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Science isn't about why, it's about why not?
Phase 1 defused. How about the next one?
1 2 3 1 2 3

Breakpoint 1, 0x0000000000400f0c in phase_2 ()
(gdb) c
Continuing.
That's number 2.  Keep going!
```

## Phase 3:

```assembly
Dump of assembler code for function phase_3:
=> 0x0000000000400f5b <+0>:     sub    $0x18,%rsp ; mở rộng ngăn xếp 24 bytes tương đương với 6 hàng
   0x0000000000400f5f <+4>:     lea    0x8(%rsp),%rcx
   0x0000000000400f64 <+9>:     lea    0xc(%rsp),%rdx
   0x0000000000400f69 <+14>:    mov    $0x401e3a,%esi ; Điều gì đó được chuyển qua thanh ghi %esi?
   0x0000000000400f6e <+19>:    mov    $0x0,%eax
   0x0000000000400f73 <+24>:    call   0x400c00 <__isoc99_sscanf@plt> ; hàm này có chức năng nhận đầu vào?
   0x0000000000400f78 <+29>:    cmp    $0x1,%eax ; so sánh 1 với kết quả từ input. Có tác dụng kiểm tra input đầu vào có đúng với dạng yêu cầu không?
   0x0000000000400f7b <+32>:    jg     0x400f82 <phase_3+39> ; nhảy tới dòng 39 / bỏ qua bomb nếu kết quả từ input lớn hơn 1
   0x0000000000400f7d <+34>:    call   0x4016b3 <explode_bomb>
   0x0000000000400f82 <+39>:    cmpl   $0x7,0xc(%rsp) ; so sánh 7 với giá trị tại địa chỉ %rsp + 0xc
   0x0000000000400f87 <+44>:    ja     0x400fed <phase_3+146> ; nhảy tới bomb nếu kết quả từ input nhỏ hơn 7
   0x0000000000400f89 <+46>:    mov    0xc(%rsp),%eax
   0x0000000000400f8d <+50>:    jmp    *0x401b50(,%rax,8)

   0x0000000000400f94 <+57>:    mov    $0x0,%eax
   0x0000000000400f99 <+62>:    jmp    0x400fa0 <phase_3+69>
   0x0000000000400f9b <+64>:    mov    $0x376,%eax

   0x0000000000400fa0 <+69>:    sub    $0x27b,%eax
   0x0000000000400fa5 <+74>:    jmp    0x400fac <phase_3+81>
   0x0000000000400fa7 <+76>:    mov    $0x0,%eax

   0x0000000000400fac <+81>:    add    $0x3a,%eax
   0x0000000000400faf <+84>:    jmp    0x400fb6 <phase_3+91>
   0x0000000000400fb1 <+86>:    mov    $0x0,%eax

   0x0000000000400fb6 <+91>:    sub    $0x193,%eax
   0x0000000000400fbb <+96>:    jmp    0x400fc2 <phase_3+103>
   0x0000000000400fbd <+98>:    mov    $0x0,%eax

   0x0000000000400fc2 <+103>:   add    $0x3da,%eax
   0x0000000000400fc7 <+108>:   jmp    0x400fce <phase_3+115>
   0x0000000000400fc9 <+110>:   mov    $0x0,%eax
--Type <RET> for more, q to quit, c to continue without paging--RET
   0x0000000000400fce <+115>:   sub    $0xfd,%eax
   0x0000000000400fd3 <+120>:   jmp    0x400fda <phase_3+127>
   0x0000000000400fd5 <+122>:   mov    $0x0,%eax

   0x0000000000400fda <+127>:   add    $0xfd,%eax
   0x0000000000400fdf <+132>:   jmp    0x400fe6 <phase_3+139>
   0x0000000000400fe1 <+134>:   mov    $0x0,%eax

   0x0000000000400fe6 <+139>:   sub    $0x395,%eax
   0x0000000000400feb <+144>:   jmp    0x400ff7 <phase_3+156>
   0x0000000000400fed <+146>:   call   0x4016b3 <explode_bomb>
   0x0000000000400ff2 <+151>:   mov    $0x0,%eax
   0x0000000000400ff7 <+156>:   cmpl   $0x5,0xc(%rsp)
   0x0000000000400ffc <+161>:   jg     0x401004 <phase_3+169>
   0x0000000000400ffe <+163>:   cmp    0x8(%rsp),%eax
   0x0000000000401002 <+167>:   je     0x401009 <phase_3+174>
   0x0000000000401004 <+169>:   call   0x4016b3 <explode_bomb>
   0x0000000000401009 <+174>:   add    $0x18,%rsp
   0x000000000040100d <+178>:   ret
End of assembler dump.
```

Từ đoạn mã ở trên chúng ta có thể thấy đây là một câu lệnh `switch statement`. Chúng ta tách từng trường hợp (case) bằng một khoảng trắng ở giữa.

Việc đầu tiên chúng ta cần làm chính là xác định dạng của đáp án. Chúng ta sẽ để ý tới dòng <+14>.

```assembly
   0x0000000000400f69 <+14>:    mov    $0x401e3a,%esi

   (gdb) x/s 0x401e3a
   0x401e3a:       "%d %d"
```

Như vậy đáp án của chúng ta có dạng là 2 số nguyên, cách nhau bởi 1 khoảng trắng. Thử với đáp án là 1 2.

```assembly
   0x0000000000400f82 <+39>:    cmpl   $0x7,0xc(%rsp) ; so sánh 7 với giá trị tại địa chỉ %rsp + 0xc
   0x0000000000400f87 <+44>:    ja     0x400fed <phase_3+146> ; nhảy tới bomb nếu kết quả từ input nhỏ hơn 7
   0x0000000000400f89 <+46>:    mov    0xc(%rsp),%eax
   0x0000000000400f8d <+50>:    jmp    *0x401b50(,%rax,8)
```

Để kiểm tra xem giá trị nào đang lưu ở địa chỉ trong thanh ghi (%rsp + 0xc) được so sánh với 7 ta chhuyển breakpoint tới dòng <+39> và thực hiện lệnh `info register` để xem giá trị của các thanh ghi.

```assembly
rsp            0x7fffffffdeb0      0x7fffffffdeb0
```

Vì địa chỉ ta cần xem giá trị là `%rsp + 0xc = 0x7fffffffdebc` nên ta dùng lệnh `x/d 0x7fffffffdebc` để xem giá trị của nó.

```assembly
   (gdb) x/d 0x7fffffffdebc
   0x7fffffffdebc: 1
```

Hoặc đơn giản hơn có thể trực tiếp gọi lệnh `x/d $rsp + 0xc` để xem giá trị của nó.

```assembly
   (gdb) x/d $rsp + 0xc
   0x7fffffffdebc: 1
```

Ta sẽ thấy %rsp+0xc có địa chỉ lưu trữ số nguyên thứ nhất

Ta được kết luận, số đang được so sánh với 7 chính là số nguyên thứ nhất mà ta nhập vào từ bàn phím.

```assembly
=> 0x0000000000400f82 <+39>:    cmpl   $0x7,0xc(%rsp)
   0x0000000000400f87 <+44>:    ja     0x400fed <phase_3+146> ; ja = jump if above (nhảy nếu lớn hơn)

   0x0000000000400fed <+146>:   call   0x4016b3 <explode_bomb>
```

Như vậy số thứ nhất nhập vào từ bàn phím không được lớn hơn 7 nếu không bom sẽ nổ.

```assembly
   0x0000000000400f89 <+46>:    mov    0xc(%rsp),%eax
   0x0000000000400f8d <+50>:    jmp    *0x401b50(,%rax,8) ; địa chỉ nhảy tới = 0x401b50 + %rax(giá trị) * 8
   0x0000000000400f94 <+57>:    mov    $0x0,%eax
```

Tiếp đó, giá trị số thứ nhất được lưu vào thanh ghi `%eax`.
Thay vì tính tay ta có thể dùng lệnh `x/x` để tính địa chỉ mà chương trình sẽ nhảy tới.

```assembly
   (gdb) x/x 0x401b50 + 1 * 8
   0x401b58:       0x0000000000400f94
```

Xác định được địa chỉ mà chương trình sẽ nhảy tới là dòng <+57>. Với những số khác (từ 0 - 6) chúng ta sẽ có một kết quả khác nhau của địa chỉ được nhảy tới.

```assembly
   0x0000000000400f94 <+57>:    mov    $0x0,%eax ; chuyển giá trị 0 vào thanh ghi %eax
   0x0000000000400f99 <+62>:    jmp    0x400fa0 <phase_3+69>

   0x0000000000400fa0 <+69>:    sub    $0x27b,%eax ;
   0x0000000000400fa5 <+74>:    jmp    0x400fac <phase_3+81>

   0x0000000000400fac <+81>:    add    $0x3a,%eax ;
   0x0000000000400faf <+84>:    jmp    0x400fb6 <phase_3+91>

   0x0000000000400fb6 <+91>:    sub    $0x193,%eax ;
   0x0000000000400fbb <+96>:    jmp    0x400fc2 <phase_3+103>

   0x0000000000400fc2 <+103>:   add    $0x3da,%eax ;
   0x0000000000400fc7 <+108>:   jmp    0x400fce <phase_3+115>

   0x0000000000400fce <+115>:   sub    $0xfd,%eax ;
   0x0000000000400fd3 <+120>:   jmp    0x400fda <phase_3+127>

   0x0000000000400fda <+127>:   add    $0xfd,%eax ;
   0x0000000000400fdf <+132>:   jmp    0x400fe6 <phase_3+139>

   0x0000000000400fe6 <+139>:   sub    $0x395,%eax ;
   0x0000000000400feb <+144>:   jmp    0x400ff7 <phase_3+156>

   0x0000000000400ff7 <+156>:   cmpl   $0x5,0xc(%rsp) ; so sánh giá trị của thanh ghi %rsp + 0xc với 0x5
   0x0000000000400ffc <+161>:   jg     0x401004 <phase_3+169> ; jg = jump if greater (nhảy nếu lớn hơn)
   0x0000000000400ffe <+163>:   cmp    0x8(%rsp),%eax ; so sánh số thứ 2 với giá trị của thanh ghi %eax
   0x0000000000401002 <+167>:   je     0x401009 <phase_3+174> ; je = jump if equal (nhảy nếu bằng)
   0x0000000000401004 <+169>:   call   0x4016b3 <explode_bomb>
   0x0000000000401009 <+174>:   add    $0x18,%rsp
   0x000000000040100d <+178>:   ret
```

Sau hàng loạt những phép cộng trừ, ta có giá trị cuối cùng tại địa chỉ nằm trong thanh ghi %eax =

Lúc này ta cần biết được xem 0x8(%rsp) có giá trị như thế nào và dễ thấy đấy chính là số thứ 2 mà ta đã nhập từ bàn phím.

```assembly
   (gdb) x/d $rsp + 0x8
   0x7fffffffdeb8: 2
```

Ta được giới hạn thêm giá trị của số thứ nhất được nhập vào, khi này chỉ còn trong khoảng (0 - 4) vì nếu từ 5 sẽ nhảy tới bom.

Tại dòng <+167> ta có câu lệnh so sánh giá trị của thanh ghi %eax với số thứ 2 được nhập vào. Nếu bằng nhau thì nhảy tới dòng <+174> và ngược lại sẽ nhảy tới dòng <+169> và gọi hàm explode_bomb. Như vậy chỉ cần biết được giá trị của thanh ghi %eax nữa thôi là ta có thể giải được bài này.

```assembly
   0x0000000000400ffe <+163>:   cmp    0x8(%rsp),%eax
   0x0000000000401002 <+167>:   je     0x401009 <phase_3+174> ; je = jump if equal (nhảy nếu bằng)
   0x0000000000401004 <+169>:   call   0x4016b3 <explode_bomb>
   0x0000000000401009 <+174>:   add    $0x18,%rsp
```

Chuyển breakpoint tới sau dòng so sánh (dòng <+163>) và thực hiện lệnh `info register` để check giá trị

```assembly
   (gdb) u*0x0000000000401002
   0x0000000000401002 in phase_3 ()
   (gdb) i r
   rax            0xfffffc71          4294966385
   rbx            0x0                 0
   rcx            0x20                32
   rdx            0x0                 0
   rsi            0x2                 2
   rdi            0x7fffffffd860      140737488345184
   rbp            0x1                 0x1
   rsp            0x7fffffffdeb0      0x7fffffffdeb0
```

Có được giá trị tại địa chỉ lưu trong thanh ghi %eax = 4294966385.

**Kết luận:**

- Ta có được 1 cặp 2 số có thể nhập là đáp số của bài này là: 1 và 4294966385. Với mỗi giá trị số thứ nhất khác (2, 3, 4) sẽ có giá trị của số thứ 2 khác.
- Với việc thử từng số từ 1 tới 4 thì ta có được 4 cặp số đáp án của Phase 3 là:

`[a, b] = [1, 4294966385], [2, 4294967020], [3, 4294966962], [4, 69]`

## Phase 4:

```assembly
Dump of assembler code for function phase_4:
=> 0x0000000000401028 <+0>:     sub    $0x18,%rsp ; mở rộng ngăn xếp 24 bytes tương đương với 6 hàng ô nhớ
   0x000000000040102c <+4>:     lea    0xc(%rsp),%rdx
   0x0000000000401031 <+9>:     mov    $0x401e3d,%esi ; input có dạng "%d"
   0x0000000000401036 <+14>:    mov    $0x0,%eax ; đặt giá trị 0 cho thanh ghi %eax
   0x000000000040103b <+19>:    call   0x400c00 <__isoc99_sscanf@plt>
   0x0000000000401040 <+24>:    cmp    $0x1,%eax ; xác nhận đầu vào đúng dạng là 1 số nguyên không thì nhảy tới bomb dòng <+36>
   0x0000000000401043 <+27>:    jne    0x40104c <phase_4+36>
   0x0000000000401045 <+29>:    cmpl   $0x0,0xc(%rsp) ; 0x0 = 0
   0x000000000040104a <+34>:    jg     0x401051 <phase_4+41> ; jg = jump if greater (nhảy nếu lớn hơn) so sánh số nhập vào với 0, lớn hơn sẽ chuyển qua dòng <+41>
   0x000000000040104c <+36>:    call   0x4016b3 <explode_bomb>
   0x0000000000401051 <+41>:    mov    0xc(%rsp),%edi ; chuyển giá trị nhập vào tại địa chỉ 0xc(%rsp) sang thanh ghi %edi
   0x0000000000401055 <+45>:    call   0x40100e <func4>
   0x000000000040105a <+50>:    cmp    $0x18,%eax ; %eax từ hàm <func4>
   0x000000000040105d <+53>:    je     0x401064 <phase_4+60> ; so sánh %eax với 24, nếu bằng nhảy tới dòng <+60>
   0x000000000040105f <+55>:    call   0x4016b3 <explode_bomb>
   0x0000000000401064 <+60>:    add    $0x18,%rsp ; cân bằng stack
   0x0000000000401068 <+64>:    ret
End of assembler dump.
```

Dễ nhận thấy được một sự quen thuộc so với phase 3.
Ta tua nhanh các bước:

- Cần biết được định dạng đáp án nhâp vào là gì? Ở đây lần này là một số nguyên

```assembly
   0x0000000000401031 <+9>:     mov    $0x401e3d,%esi
   (gdb) x/s 0x401e3d
   0x401e3d:       "%d"
```

- Nhập thử kết quả là 3 và chạy lại chương trình. Cần xác định số được nhập vào sẽ được lưu ở vị trí chứa ở thanh ghi nào? Đặt breakpoint ngay trước khi hàm `<func4>` được gọi và sử dụng lệnh `info register` để xem giá trị của thanh ghi %rsp + 0xc và nhận ra đây chính là địa chỉ có giá trị là số ta nhập vào

```assembly
(gdb) u*0x0000000000401051
0x0000000000401031 in phase_4 ()
(gdb) disas
Dump of assembler code for function phase_4:
   0x000000000040104c <+36>:    call   0x4016b3 <explode_bomb>
=> 0x0000000000401051 <+41>:    mov    0xc(%rsp),%edi
   0x0000000000401055 <+45>:    call   0x40100e <func4>

   (gdb) x/d 0x7fffffffdee0 + 0xc
   0x7fffffffdeec: 3
```

Khi này %edi chứa địa chỉ trỏ tới ô nhớ có địa chỉ là số ta nhập vào = 3
Ta đi sâu hơn vào hàm `<func4>` để xem nó làm gì và đặc biệt quan tâm tới giá trị có địa chỉ lưu ở thanh ghi %eax sau khi hàm này được gọi

```assembly
(gdb) disas func4
Dump of assembler code for function func4:
   0x000000000040100e <+0>:     push   %rbx ; %edi lưu địa chỉ trỏ ô giá trị = 3 là số ta nhập vào
   0x000000000040100f <+1>:     mov    %edi,%ebx ; chuyển giá trị 3 sang thanh ghi %ebx
   0x0000000000401011 <+3>:     mov    $0x1,%eax ; đặt giá trị 1 cho thanh ghi %eax
   0x0000000000401016 <+8>:     cmp    $0x1,%edi ; so sánh giá trị số ta nhập vào với 1
   0x0000000000401019 <+11>:    jle    0x401026 <func4+24> ; jle = jump if less or equal (nhảy nếu nhỏ hơn hoặc bằng) nếu số ta nhập vào nhỏ hơn hoặc bằng 1 thì nhảy tới dòng <+24>
   0x000000000040101b <+13>:    lea    -0x1(%rdi),%edi
   0x000000000040101e <+16>:    call   0x40100e <func4>
   0x0000000000401023 <+21>:    imul   %ebx,%eax ; nhân %eax với %ebx
   0x0000000000401026 <+24>:    pop    %rbx
   0x0000000000401027 <+25>:    ret
```

Số ta nhập vào nếu lớn hơn 1, phép di chuyển được thực hiện

```assembly
   0x000000000040101b <+13>:    lea    -0x1(%rdi),%edi ; giảm giá trị số ta nhập vào đi 1 và lưu vào thanh ghi %edi
```

Sau đó gọi lại chính hàm `<func4>` với giá trị mới của thanh ghi %edi. Điều này sẽ tiếp tục thực hiện cho đến khi giá trị có địa chỉ tại thanh ghi %edi bằng 1.
Mỗi lần như vậy, giá trị có địa chỉ tại thanh ghi %eax sẽ được nhân với giá trị có địa chỉ tại thanh ghi %ebx. Và cuối cùng, giá trị của thanh ghi %eax sẽ là kết quả của hàm `<func4>`

Giá trị có địa chỉ tại %ebx sẽ lần lượt có giá trị là 3, 2, 1. Và giá trị có địa chỉ tại %eax sẽ lần lượt có giá trị là 1, 3, 6.

| %ebx | %eax | %eax-new |
| ---- | ---- | -------- |
| 3    | 1    | 3        |
| 2    | 3    | 6        |
| 1    | 6    | 6        |

Vậy giá trị tại địa chỉ lưu trong thanh ghi %eax sau khi hàm `<func4>` được gọi là 6. Ta tiếp tục xem hàm `<phase_4>` tiếp tục thực hiện như thế nào

```assembly
   0x0000000000401055 <+45>:    call   0x40100e <func4>
   0x000000000040105a <+50>:    cmp    $0x18,%eax ; so sánh giá trị tại địa chỉ lưu trong thanh ghi %eax với 24
   0x000000000040105d <+53>:    je     0x401064 <phase_4+60> ; je = jump if equal (nhảy nếu bằng) nếu giá trị tại địa chỉ lưu trong thanh ghi %eax bằng 24 thì nhảy tới dòng <+60> không thì bomb nổ
   0x000000000040105f <+55>:    call   0x4016b3 <explode_bomb>
   0x0000000000401064 <+60>:    add    $0x18,%rsp
   0x0000000000401068 <+64>:    ret
```

**Kết luận:**

- Ta cần phải nhập vào 1 số để hàm `<func4>` trả về giá trị 24. Ta lập bảng và thử các số và được **đáp án là 4**.
  | %ebx | %eax | %eax-new |
  |------|------|----------|
  | 4 | 1 | 4 |
  | 3 | 4 | 12 |
  | 2 | 12 | 24 |
  | 1 | 24 | 24 |

## Phase 5:

```objdump
0000000000401069 <phase_5>:
  401069:	53                   	push   %rbx
  40106a:	48 83 ec 10          	sub    $0x10,%rsp
  40106e:	48 89 fb             	mov    %rdi,%rbx
  401071:	e8 4b 02 00 00       	call   4012c1 <string_length>
  401076:	83 f8 06             	cmp    $0x6,%eax
  401079:	74 3f                	je     4010ba <phase_5+0x51>
  40107b:	e8 33 06 00 00       	call   4016b3 <explode_bomb>
  401080:	eb 38                	jmp    4010ba <phase_5+0x51>
  401082:	0f b6 14 03          	movzbl (%rbx,%rax,1),%edx
  401086:	83 e2 0f             	and    $0xf,%edx
  401089:	0f b6 92 90 1b 40 00 	movzbl 0x401b90(%rdx),%edx
  401090:	88 14 04             	mov    %dl,(%rsp,%rax,1)
  401093:	48 83 c0 01          	add    $0x1,%rax
  401097:	48 83 f8 06          	cmp    $0x6,%rax
  40109b:	75 e5                	jne    401082 <phase_5+0x19>
  40109d:	c6 44 24 06 00       	movb   $0x0,0x6(%rsp)
  4010a2:	be 3e 1b 40 00       	mov    $0x401b3e,%esi
  4010a7:	48 89 e7             	mov    %rsp,%rdi
  4010aa:	e8 2f 02 00 00       	call   4012de <strings_not_equal>
  4010af:	85 c0                	test   %eax,%eax
  4010b1:	74 0f                	je     4010c2 <phase_5+0x59>
  4010b3:	e8 fb 05 00 00       	call   4016b3 <explode_bomb>
  4010b8:	eb 08                	jmp    4010c2 <phase_5+0x59>
  4010ba:	b8 00 00 00 00       	mov    $0x0,%eax
  4010bf:	90                   	nop
  4010c0:	eb c0                	jmp    401082 <phase_5+0x19>
  4010c2:	48 83 c4 10          	add    $0x10,%rsp
  4010c6:	5b                   	pop    %rbx
  4010c7:	c3                   	ret
```

## Phase 6:

```objdump
00000000004010c8 <phase_6>:
  4010c8:	41 55                	push   %r13
  4010ca:	41 54                	push   %r12
  4010cc:	55                   	push   %rbp
  4010cd:	53                   	push   %rbx
  4010ce:	48 83 ec 58          	sub    $0x58,%rsp
  4010d2:	48 8d 74 24 30       	lea    0x30(%rsp),%rsi
  4010d7:	e8 f9 05 00 00       	call   4016d5 <read_six_numbers>
  4010dc:	4c 8d 6c 24 30       	lea    0x30(%rsp),%r13
  4010e1:	41 bc 00 00 00 00    	mov    $0x0,%r12d
  4010e7:	4c 89 ed             	mov    %r13,%rbp
  4010ea:	41 8b 45 00          	mov    0x0(%r13),%eax
  4010ee:	83 e8 01             	sub    $0x1,%eax
  4010f1:	83 f8 05             	cmp    $0x5,%eax
  4010f4:	76 05                	jbe    4010fb <phase_6+0x33>
  4010f6:	e8 b8 05 00 00       	call   4016b3 <explode_bomb>
  4010fb:	41 83 c4 01          	add    $0x1,%r12d
  4010ff:	41 83 fc 06          	cmp    $0x6,%r12d
  401103:	75 07                	jne    40110c <phase_6+0x44>
  401105:	be 00 00 00 00       	mov    $0x0,%esi
  40110a:	eb 42                	jmp    40114e <phase_6+0x86>
  40110c:	44 89 e3             	mov    %r12d,%ebx
  40110f:	48 63 c3             	movslq %ebx,%rax
  401112:	8b 44 84 30          	mov    0x30(%rsp,%rax,4),%eax
  401116:	39 45 00             	cmp    %eax,0x0(%rbp)
  401119:	75 05                	jne    401120 <phase_6+0x58>
  40111b:	e8 93 05 00 00       	call   4016b3 <explode_bomb>
  401120:	83 c3 01             	add    $0x1,%ebx
  401123:	83 fb 05             	cmp    $0x5,%ebx
  401126:	7e e7                	jle    40110f <phase_6+0x47>
  401128:	49 83 c5 04          	add    $0x4,%r13
  40112c:	eb b9                	jmp    4010e7 <phase_6+0x1f>
  40112e:	48 8b 52 08          	mov    0x8(%rdx),%rdx
  401132:	83 c0 01             	add    $0x1,%eax
  401135:	39 c8                	cmp    %ecx,%eax
  401137:	75 f5                	jne    40112e <phase_6+0x66>
  401139:	eb 05                	jmp    401140 <phase_6+0x78>
  40113b:	ba 20 33 60 00       	mov    $0x603320,%edx
  401140:	48 89 14 74          	mov    %rdx,(%rsp,%rsi,2)
  401144:	48 83 c6 04          	add    $0x4,%rsi
  401148:	48 83 fe 18          	cmp    $0x18,%rsi
  40114c:	74 15                	je     401163 <phase_6+0x9b>
  40114e:	8b 4c 34 30          	mov    0x30(%rsp,%rsi,1),%ecx
  401152:	83 f9 01             	cmp    $0x1,%ecx
  401155:	7e e4                	jle    40113b <phase_6+0x73>
  401157:	b8 01 00 00 00       	mov    $0x1,%eax
  40115c:	ba 20 33 60 00       	mov    $0x603320,%edx
  401161:	eb cb                	jmp    40112e <phase_6+0x66>
  401163:	48 8b 1c 24          	mov    (%rsp),%rbx
  401167:	48 8d 44 24 08       	lea    0x8(%rsp),%rax
  40116c:	48 8d 74 24 30       	lea    0x30(%rsp),%rsi
  401171:	48 89 d9             	mov    %rbx,%rcx
  401174:	48 8b 10             	mov    (%rax),%rdx
  401177:	48 89 51 08          	mov    %rdx,0x8(%rcx)
  40117b:	48 83 c0 08          	add    $0x8,%rax
  40117f:	48 39 f0             	cmp    %rsi,%rax
  401182:	74 05                	je     401189 <phase_6+0xc1>
  401184:	48 89 d1             	mov    %rdx,%rcx
  401187:	eb eb                	jmp    401174 <phase_6+0xac>
  401189:	48 c7 42 08 00 00 00 	movq   $0x0,0x8(%rdx)
  401190:	00
  401191:	bd 05 00 00 00       	mov    $0x5,%ebp
  401196:	48 8b 43 08          	mov    0x8(%rbx),%rax
  40119a:	8b 00                	mov    (%rax),%eax
  40119c:	39 03                	cmp    %eax,(%rbx)
  40119e:	7d 05                	jge    4011a5 <phase_6+0xdd>
  4011a0:	e8 0e 05 00 00       	call   4016b3 <explode_bomb>
  4011a5:	48 8b 5b 08          	mov    0x8(%rbx),%rbx
  4011a9:	83 ed 01             	sub    $0x1,%ebp
  4011ac:	75 e8                	jne    401196 <phase_6+0xce>
  4011ae:	48 83 c4 58          	add    $0x58,%rsp
  4011b2:	5b                   	pop    %rbx
  4011b3:	5d                   	pop    %rbp
  4011b4:	41 5c                	pop    %r12
  4011b6:	41 5d                	pop    %r13
  4011b8:	c3                   	ret
```
