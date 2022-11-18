# Analysis of A Binary Bomb Lab / Giải bomb nhị phân

## Thang Long University - Computer Architecture - CS212 - Semester I/2022

```
Start working on 11/17/2022 - During the period of the final exam of the first term in 2022
Please feel free to fork or star if helpful! (^^ゞ
Created by: A41316 - Nguyễn Hữu Khoa
```

_The article is referenced from:_

- [https://github.com/sc2225/Bomb-Lab](https://github.com/sc2225/Bomb-Lab)

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
Mỗi lần như vậy, giá trị có địa chỉ tại thanh ghi %eax sẽ được nhân với giá trị có địa chỉ tại thanh ghi %ebx. Và cuối cùng, giá trị có địa chỉ tại thanh ghi %eax sẽ là kết quả của hàm `<func4>`

Giá trị có địa chỉ tại %ebx sẽ lần lượt có giá trị là 3, 2, 1. Và giá trị có địa chỉ tại %eax sẽ lần lượt có giá trị là 1, 3, 6.

| %ebx | %eax | %eax-new |
| ---- | ---- | -------- |
| 3    | 1    | 3        |
| 2    | 3    | 6        |
| 1    | 6    | 6        |

Dễ thấy được công thức của hàm `<func4>` là

```math
f(x) = x! = x * (x - 1) * (x - 2) * ... * 1
```

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
Ta cần phải nhập vào 1 số để hàm `<func4>` trả về giá trị 24. Dựa vào công thức ở trên ta tìm được **đáp án là 4**.
| %ebx | %eax | %eax-new |
|------|------|----------|
| 4 | 1 | 4 |
| 3 | 4 | 12 |
| 2 | 12 | 24 |
| 1 | 24 | 24 |

## Phase 5:

```assembly
Dump of assembler code for function phase_5:
=> 0x0000000000401069 <+0>:     push   %rbx
   0x000000000040106a <+1>:     sub    $0x10,%rsp
   0x000000000040106e <+5>:     mov    %rdi,%rbx ; lưu địa chỉ của chuỗi nhập vào vào thanh ghi %rbx
   0x0000000000401071 <+8>:     call   0x4012c1 <string_length> ;
   0x0000000000401076 <+13>:    cmp    $0x6,%eax ; so sánh giá trị tại địa chỉ lưu trong thanh ghi %eax với 6
   0x0000000000401079 <+16>:    je     0x4010ba <phase_5+81> ; nếu bằng 6 thì nhảy tới dòng <+81> không thì bomb nổ
   0x000000000040107b <+18>:    call   0x4016b3 <explode_bomb>
   0x0000000000401080 <+23>:    jmp    0x4010ba <phase_5+81>
   0x0000000000401082 <+25>:    movzbl (%rbx,%rax,1),%edx
   0x0000000000401086 <+29>:    and    $0xf,%edx
   0x0000000000401089 <+32>:    movzbl 0x401b90(%rdx),%edx
   0x0000000000401090 <+39>:    mov    %dl,(%rsp,%rax,1)
   0x0000000000401093 <+42>:    add    $0x1,%rax
   0x0000000000401097 <+46>:    cmp    $0x6,%rax
   0x000000000040109b <+50>:    jne    0x401082 <phase_5+25>
   0x000000000040109d <+52>:    movb   $0x0,0x6(%rsp)
   0x00000000004010a2 <+57>:    mov    $0x401b3e,%esi
   0x00000000004010a7 <+62>:    mov    %rsp,%rdi
   0x00000000004010aa <+65>:    call   0x4012de <strings_not_equal>
   0x00000000004010af <+70>:    test   %eax,%eax
   0x00000000004010b1 <+72>:    je     0x4010c2 <phase_5+89>
   0x00000000004010b3 <+74>:    call   0x4016b3 <explode_bomb>
   0x00000000004010b8 <+79>:    jmp    0x4010c2 <phase_5+89>
   0x00000000004010ba <+81>:    mov    $0x0,%eax
   0x00000000004010bf <+86>:    nop
   0x00000000004010c0 <+87>:    jmp    0x401082 <phase_5+25>
   0x00000000004010c2 <+89>:    add    $0x10,%rsp
   0x00000000004010c6 <+93>:    pop    %rbx
   0x00000000004010c7 <+94>:    ret
```

Kiểm tra hàm `<string_length>`:

```assembly
Dump of assembler code for function string_length:
   0x00000000004012c1 <+0>:     cmpb   $0x0,(%rdi) ; rdi = string input. Kiểm tra xem có gì trong chuỗi nhập vào hay không
   0x00000000004012c4 <+3>:     je     0x4012d8 <string_length+23>
   0x00000000004012c6 <+5>:     mov    %rdi,%rdx ; di chuyển chuỗi nhập vào qua thanh ghi %rdx
   0x00000000004012c9 <+8>:     add    $0x1,%rdx ; string + 1
   0x00000000004012cd <+12>:    mov    %edx,%eax; eax = string + 1
   0x00000000004012cf <+14>:    sub    %edi,%eax ; eax = string + 1 - string
   0x00000000004012d1 <+16>:    cmpb   $0x0,(%rdx) ; so sánh xem có gì trong chuỗi nhập vào hay không
   0x00000000004012d4 <+19>:    jne    0x4012c9 <string_length+8>
   0x00000000004012d6 <+21>:    repz ret
   0x00000000004012d8 <+23>:    mov    $0x0,%eax ; chuỗi rỗng sẽ trả về 0 vào thanh ghi %eax
   0x00000000004012dd <+28>:    ret
```

Có vẻ hàm sẽ trả lại độ dài của chuỗi. Với việc muốn chuỗi nhập vào có giá trị trả về từ hàm trên là 6 ta thử chuỗi `ibcghk` để xem chúng ta có thể vượt qua bomb đầu tiên hay không

```assembly
   0x0000000000401071 <+8>:     call   0x4012c1 <string_length>
=> 0x0000000000401076 <+13>:    cmp    $0x6,%eax
   0x0000000000401079 <+16>:    je     0x4010ba <phase_5+81>
   0x000000000040107b <+18>:    call   0x4016b3 <explode_bomb>

(gdb) i r
rax            0x6                 6
rbx            0x603b80            6306688
rcx            0x6                 6
```

Chúng ta có thể thấy tại điểm này, %eax từ hàm `<string_length>` đã có giá trị là 6. Vì vậy hàm `<string_length>` chắc chắn cho ta độ dài của một chuỗi và đảm bảo rằng đầu vào dài 6 ký tự

Khi có được chuỗi là 6 kí tự, nhảy tới dòng `<+81>` và trở lại với dòng `<+25>`

```assembly
   0x0000000000401079 <+16>:    je     0x4010ba <phase_5+81>

   0x00000000004010ba <+81>:    mov    $0x0,%eax ; đặt giá trị 0 vào ô nhớ có địa chỉ tại thanh ghi %eax

   0x00000000004010c0 <+87>:    jmp    0x401082 <phase_5+25> ; nhảy tới dòng <+25>

   0x0000000000401082 <+25>:    movzbl (%rbx,%rax,1),%edx ; lấy giá trị tại ô nhớ có địa chỉ là (%rbx,%rax,1) và đặt vào thanh ghi %edx
```

<!-- movbzl means: move a byte with zero extension into 32-bit (“l”) register -->

Tại dòng `<+81>` giá trị tại địa chỉ (%rbx, %rax, 1) được chuyển vào thanh ghi `%edx`. Nhưng điều ta quan tâm thì là giá trị được truyền vào thanh ghi `%edx` là gì. Để xem được giá trị này ta đặt breakpoit tại dòng `<+29>` với lệnh `u*0x0000000000401086` và dùng lệnh `info register` để xem giá trị của thanh ghi `%edx`

```assembly
(gdb) u*0x0000000000401086
0x0000000000401086 in phase_5 ()
(gdb) i r
rax            0x0                 0
rbx            0x603b80            6306688
rcx            0x6                 6
rdx            0x69                105
```

`105` trong bảng mã ASCII là kí tự `i` hay chính là kí tự đầu tiên của chuỗi ta đã nhập vào `ibcghk`

```assembly
   0x0000000000401086 <+29>:    and    $0xf,%edx ; thực hiện phép AND với 0xf và đặt kết quả vào thanh ghi %edx
   0x0000000000401089 <+32>: movzbl 0x401b90(%rdx),%edx ; lấy giá trị tại ô nhớ có địa chỉ là 0x401b90(%rdx) và đặt vào thanh ghi %edx
```

Tiếp tục thực hiện tương tự như trên và biết được tại địa chỉ (0x401b90)%rdx có giá trị `98` (là kí tự `b` trong bảng mã ASCII) và chính là kí tự thứ 2 trong chuỗi ta nhập vào bàn phím `ibcghk`

```assembly
(gdb) i r
rax            0x0                 0
rbx            0x603b80            6306688
rcx            0x6                 6
rdx            0x62                98
```

Tiếp đó là lệnh chuyển giá trị tại thanh ghi %dl qua địa chỉ chứa kí tự thứ nhất của chuỗi nhập vào.

```assembly
   0x0000000000401090 <+39>: mov %dl,(%rsp,%rax,1) ; đặt giá trị tại thanh ghi %dl vào ô nhớ có địa chỉ (%rsp,%rax,1)
   0x0000000000401093 <+42>: add $0x1,%rax ; thực hiện phép cộng với 1 và đặt kết quả vào thanh ghi %rax
   0x0000000000401097 <+46>: cmp $0x6,%rax ; so sánh giá trị tại thanh ghi %rax với 6
   0x000000000040109b <+50>: jne 0x401082 <phase_5+25>
```

Nhận thấy được chuỗi đang có sự thay đổi về kí tự, kí tự đầu tiên của chuỗi sẽ không còn là `i` nữa. Sử dụng lệnh `i r` để check giá trị sau khi thực hiện lệnh `mov %dl,(%rsp,%rax,1)` và ta có kết quả như sau

```assembly
(gdb) i r
rax            0x0                 0
rbx            0x603b80            6306688
rcx            0x6                 6
rdx            0x62                98
```

Vậy là kí tự đầu tiên của chuỗi đã được thay đổi thành `b`. Ta tiếp tục với những dòng sau đó:

Có thể dự đoán rằng sẽ có sự thay đổi kí tự của cả chuỗi. Ta bỏ qua tất cả để tới dòng `<+62>` để xem điều gì được đưa vào hàm `<strings_not_equal>`

```assembly
   0x00000000004010a2 <+57>:    mov    $0x401b3e,%esi
=> 0x00000000004010a7 <+62>:    mov    %rsp,%rdi
   0x00000000004010aa <+65>:    call   0x4012de <strings_not_equal>
   0x00000000004010af <+70>:    test   %eax,%eax

   (gdb) i r
   rax            0x6                 6
   rbx            0x603b80            6306688
   rcx            0x6                 6
   rdx            0x6e                110
   rsi            0x401b3e            4201278
   rdi            0x603b80            6306688

   (gdb) x/s 6306688
   0x603b80:       "ibdghk"

   (gdb) x/s 4201278
   0x401b3e:       "giants"
```

```assembly
   (gdb) i r
   rax            0x6                 6
   rbx            0x603b80            6306688
   rcx            0x6                 6
   rdx            0x6e                110
   rsi            0x401b3e            4201278
   rdi            0x7fffffffdee0      140737488346848

   (gdb) x/s 140737488346848
   0x7fffffffdee0: "brehon"
```

Chuỗi nhập vào là `ibcghk` và chuỗi sau khi được mã hóa lại `brehon`. Như vậy ta cần nhập vào 1 chuỗi để khi kết thúc mã hóa sẽ có giá trị là `giants`

Sau hàng loạt lần chạy thử ta có bảng mã hóa như sau:
| Kí tự | **Mã hóa** | Kí tự | **Mã hóa** | Kí tự | **Mã hóa** |
|:-----:|:----------:|:-----:|:----------:|:-----:|:----------:|
| a | **s** | b | **r** | c | **e** |
| d | **e** | e | **a** | f | **w** |
| g | **h** | h | **o** | i | **b** |
| j | **p** | k | **n** | l | **u** |
| m | **t** | n | **f** | o | **g** |
| p | **i** | q | **s** | r | **r** |
| s | **v** | t | **e** | u | **a** |
| v | **w** | z | **p** | | |

Dựa vào bảng trên dễ dàng tìm ra chuỗi cần nhập vào là `opukmq`

```assembly
opukmq
Breakpoint 1, 0x0000000000401069 in phase_5 ()
(gdb) c
Continuing.
Congratulations! You've (mostly) defused the bomb!
Hit Control-C to escape phase 6 (for free!), but if you want to
try phase 6 for extra credit, you can continue.  Just beware!
```

## Phase 6:

Khi mình làm xong Phase 5 cũng là lúc 1h sáng =))
Từ sau này chừa cái tật để dành tới trước hôm thi mới xem (￣ y▽ ￣)╭ Ohohoho.....

```assembly
Dump of assembler code for function phase_6:
=> 0x00000000004010c8 <+0>:     push   %r13
   0x00000000004010ca <+2>:     push   %r12
   0x00000000004010cc <+4>:     push   %rbp
   0x00000000004010cd <+5>:     push   %rbx
   0x00000000004010ce <+6>:     sub    $0x58,%rsp
   0x00000000004010d2 <+10>:    lea    0x30(%rsp),%rsi
   0x00000000004010d7 <+15>:    call   0x4016d5 <read_six_numbers>
   0x00000000004010dc <+20>:    lea    0x30(%rsp),%r13
   0x00000000004010e1 <+25>:    mov    $0x0,%r12d
   0x00000000004010e7 <+31>:    mov    %r13,%rbp
   0x00000000004010ea <+34>:    mov    0x0(%r13),%eax
   0x00000000004010ee <+38>:    sub    $0x1,%eax
   0x00000000004010f1 <+41>:    cmp    $0x5,%eax
   0x00000000004010f4 <+44>:    jbe    0x4010fb <phase_6+51>
   0x00000000004010f6 <+46>:    call   0x4016b3 <explode_bomb>
   0x00000000004010fb <+51>:    add    $0x1,%r12d
   0x00000000004010ff <+55>:    cmp    $0x6,%r12d
   0x0000000000401103 <+59>:    jne    0x40110c <phase_6+68>
   0x0000000000401105 <+61>:    mov    $0x0,%esi
   0x000000000040110a <+66>:    jmp    0x40114e <phase_6+134>
   0x000000000040110c <+68>:    mov    %r12d,%ebx
   0x000000000040110f <+71>:    movslq %ebx,%rax
   0x0000000000401112 <+74>:    mov    0x30(%rsp,%rax,4),%eax
   0x0000000000401116 <+78>:    cmp    %eax,0x0(%rbp)
   0x0000000000401119 <+81>:    jne    0x401120 <phase_6+88>
   0x000000000040111b <+83>:    call   0x4016b3 <explode_bomb>
   0x0000000000401120 <+88>:    add    $0x1,%ebx
   0x0000000000401123 <+91>:    cmp    $0x5,%ebx
   0x0000000000401126 <+94>:    jle    0x40110f <phase_6+71>
   0x0000000000401128 <+96>:    add    $0x4,%r13
--Type <RET> for more, q to quit, c to continue without paging--
   0x000000000040112c <+100>:   jmp    0x4010e7 <phase_6+31>
   0x000000000040112e <+102>:   mov    0x8(%rdx),%rdx
   0x0000000000401132 <+106>:   add    $0x1,%eax
   0x0000000000401135 <+109>:   cmp    %ecx,%eax
   0x0000000000401137 <+111>:   jne    0x40112e <phase_6+102>
   0x0000000000401139 <+113>:   jmp    0x401140 <phase_6+120>

   0x000000000040113b <+115>:   mov    $0x603320,%edx
   0x0000000000401140 <+120>:   mov    %rdx,(%rsp,%rsi,2)
   0x0000000000401144 <+124>:   add    $0x4,%rsi
   0x0000000000401148 <+128>:   cmp    $0x18,%rsi
   0x000000000040114c <+132>:   je     0x401163 <phase_6+155>
   0x000000000040114e <+134>:   mov    0x30(%rsp,%rsi,1),%ecx
   0x0000000000401152 <+138>:   cmp    $0x1,%ecx
   0x0000000000401155 <+141>:   jle    0x40113b <phase_6+115>
   0x0000000000401157 <+143>:   mov    $0x1,%eax
   0x000000000040115c <+148>:   mov    $0x603320,%edx
   0x0000000000401161 <+153>:   jmp    0x40112e <phase_6+102>
   0x0000000000401163 <+155>:   mov    (%rsp),%rbx
   0x0000000000401167 <+159>:   lea    0x8(%rsp),%rax
   0x000000000040116c <+164>:   lea    0x30(%rsp),%rsi
   0x0000000000401171 <+169>:   mov    %rbx,%rcx
   0x0000000000401174 <+172>:   mov    (%rax),%rdx
   0x0000000000401177 <+175>:   mov    %rdx,0x8(%rcx)
   0x000000000040117b <+179>:   add    $0x8,%rax
   0x000000000040117f <+183>:   cmp    %rsi,%rax
   0x0000000000401182 <+186>:   je     0x401189 <phase_6+193>
   0x0000000000401184 <+188>:   mov    %rdx,%rcx
   0x0000000000401187 <+191>:   jmp    0x401174 <phase_6+172>
   0x0000000000401189 <+193>:   movq   $0x0,0x8(%rdx)
   0x0000000000401191 <+201>:   mov    $0x5,%ebp
   0x0000000000401196 <+206>:   mov    0x8(%rbx),%rax
--Type <RET> for more, q to quit, c to continue without paging--
   0x000000000040119a <+210>:   mov    (%rax),%eax
   0x000000000040119c <+212>:   cmp    %eax,(%rbx)
   0x000000000040119e <+214>:   jge    0x4011a5 <phase_6+221>
   0x00000000004011a0 <+216>:   call   0x4016b3 <explode_bomb>
   0x00000000004011a5 <+221>:   mov    0x8(%rbx),%rbx
   0x00000000004011a9 <+225>:   sub    $0x1,%ebp
   0x00000000004011ac <+228>:   jne    0x401196 <phase_6+206>
   0x00000000004011ae <+230>:   add    $0x58,%rsp
   0x00000000004011b2 <+234>:   pop    %rbx
   0x00000000004011b3 <+235>:   pop    %rbp
   0x00000000004011b4 <+236>:   pop    %r12
   0x00000000004011b6 <+238>:   pop    %r13
   0x00000000004011b8 <+240>:   ret
```

Dòng này sẽ đảm bảo mọi số mình nhập vào không có số nào lớn hơn 6

```assembly
   0x00000000004010f1 <+41>:    cmp    $0x5,%eax
```

Dòng này sẽ đảm bảo không có 2 số nào trùng nhau bằng sử dụng cả vòng lặp 6 lần

```assembly
   0x0000000000401116 <+78>:    cmp    %eax,0x0(%rbp)
   0x0000000000401119 <+81>:    jne    0x401120 <phase_6+88>
   0x000000000040111b <+83>:    call   0x4016b3 <explode_bomb>
   0x0000000000401120 <+88>:    add    $0x1,%ebx
   0x0000000000401123 <+91>:    cmp    $0x5,%ebx
   0x0000000000401126 <+94>:    jle    0x40110f <phase_6+71>
```

Để ý tới địa chỉ xuất hiện ở dòng `<+115>`

```assembly
   0x000000000040113b <+115>:   mov    $0x603320,%edx

   (gdb) x/44x 0x603320
   0x603320 <node1>:       0x000001bb      0x00000001      0x00603330      0x00000000
   0x603330 <node2>:       0x00000227      0x00000002      0x00603340      0x00000000
   0x603340 <node3>:       0x00000361      0x00000003      0x00603350      0x00000000
   0x603350 <node4>:       0x0000020f      0x00000004      0x00603360      0x00000000
   0x603360 <node5>:       0x00000142      0x00000005      0x00603370      0x00000000
   0x603370 <node6>:       0x0000025a      0x00000006      0x00000000      0x00000000
   0x603380 <lab_id>:      0x73637775      0x31353365      0x7335312d      0x616c2d70
   0x603390 <lab_id+16>:   0x00003262      0x00000000      0x00000000      0x00000000
   0x6033a0 <lab_id+32>:   0x00000000      0x00000000      0x00000000      0x00000000
   0x6033b0 <lab_id+48>:   0x00000000      0x00000000      0x00000000      0x00000000
   0x6033c0 <lab_id+64>:   0x00000000      0x00000000      0x00000000      0x00000000
```

0x000001bb = 443 ứng với số 1
0x00000227 = 551 ứng với số 2
0x00000361 = 865 ứng với số 3
0x0000020f = 527 ứng với số 4
0x00000142 = 322 ứng với số 5
0x0000025a = 602 ứng với số 6

Sắp xếp giá trị theo từ lớn tới bé ta có 865, 602, 551, 527, 443, 322 hay 3, 6, 2, 4, 1, 5

**Kết quả:**

```assembly
3 6 2 4 1 5

Breakpoint 1, 0x00000000004010c8 in phase_6 ()
(gdb) c
Continuing.
Congratulations! You've defused the bomb! Again!
```
