# Analysis of A Binary Bomb Lab / Giải bomb nhị phân

## Thang Long University - Computer Architecture - CS212 - Semester I/2022

```
Thực hiện trong kì thi cuối kì môn Kiến trúc máy tính - CS212 - Kì I/2022 được tham khảo từ nhiều tài liệu có ghi nguồn.
Tặng sao nếu thấy hữu ích nhe (^^ゞ

Performed in the final exam of Computer Architecture - CS212 - Semester I/2022 - Thang Long University with references from many sources attested.
Please feel free to fork or star if helpful! (^^ゞ

```

_The article is referenced from: [https://john.coffee/pages/binary-bomb-lab](https://john.coffee/pages/binary-bomb-lab)_

## Overview / Tổng quan về giải bomb

Read in this [post](https://github.com/kr4zym3nvn/Analysis-of-Binary-Bomb-Lab/blob/master/Analysis%20of%20CME%20bomb%20lab%20program.md)

## Getting Strings and Objdump

Run the following commands to create text files which we will look at later:

_Chạy các dòng lệnh sau để tạo file text mà chúng ta sẽ cần sau này_

```
strings bomb > strings.txt
objdump -d bomb > assembly.txt
```

You should now have two files: strings.txt and assembly.txt. Now let’s get started with Phase 1!

_Bây giờ chúng ta sẽ có 2 tệp strings.txt và assembly.txt. Bây giờ chúng ta sẽ bắt đầu với Phase 1!_

_Vietnamese is below_

## Phase 1:

```objdump
0000000000400ef0 <phase_1>:
  400ef0:	48 83 ec 08          	sub    $0x8,%rsp // building stack frame with 8 more bytes
  400ef4:	be e8 1a 40 00       	mov    $0x401ae8,%esi // what is this being moved to esi? (0x401ae8)
  400ef9:	e8 e0 03 00 00       	call   4012de <strings_not_equal> // call strings_not_equal function
  400efe:	85 c0                	test   %eax,%eax // test eax register
  400f00:	74 05                	je     400f07 <phase_1+0x17> // jump if equal
  400f02:	e8 ac 07 00 00       	call   4016b3 <explode_bomb> // call explode_bomb function
  400f07:	48 83 c4 08          	add    $0x8,%rsp // add 8 bytes to stack frame
  400f0b:	c3                   	ret
```

Chúng ta có thể thấy rằng hàm <strings_not_equal> đang được gọi với hàm ý so sánh hai chuỗi.
Việc đầu tiên là cần xác định được bối cảnh thủ tục hay chính là các biến được truyền vào hàm là gì.
Dễ dàng nhận thấy biến sử dụng trong so sánh là giá trị của địa chỉ lưu trong thanh ghi $eax. Ngay trước khi gọi hàm, ở trên có xuất hiện thanh ghi $esi cũng liên quan. Ta sử dụng địa chỉ đó trong bộ nhớ và xem nó chứa gì dưới dạng chuỗi.

    ```objdump
    400ef4:	be e8 1a 40 00       	mov    $0x401ae8,%esi
    ```

Lets examine what is being moved from address 0x401ae8. We know it has to be a string of some sort so we use '/s'.

```objdump
(gdb) x/s 0x401ae8
0x401ae8:       "Science isn't about why, it's about why not?"
```

## Phase 2:

```objdump
0000000000400f0c <phase_2>:
  400f0c:	41 55                	push   %r13
  400f0e:	41 54                	push   %r12
  400f10:	55                   	push   %rbp
  400f11:	53                   	push   %rbx
  400f12:	48 83 ec 28          	sub    $0x28,%rsp
  400f16:	48 89 e6             	mov    %rsp,%rsi
  400f19:	e8 b7 07 00 00       	call   4016d5 <read_six_numbers>
  400f1e:	48 89 e3             	mov    %rsp,%rbx
  400f21:	4c 8d 6c 24 0c       	lea    0xc(%rsp),%r13
  400f26:	bd 00 00 00 00       	mov    $0x0,%ebp
  400f2b:	49 89 dc             	mov    %rbx,%r12
  400f2e:	8b 43 0c             	mov    0xc(%rbx),%eax
  400f31:	39 03                	cmp    %eax,(%rbx)
  400f33:	74 05                	je     400f3a <phase_2+0x2e>
  400f35:	e8 79 07 00 00       	call   4016b3 <explode_bomb>
  400f3a:	41 03 2c 24          	add    (%r12),%ebp
  400f3e:	48 83 c3 04          	add    $0x4,%rbx
  400f42:	4c 39 eb             	cmp    %r13,%rbx
  400f45:	75 e4                	jne    400f2b <phase_2+0x1f>
  400f47:	85 ed                	test   %ebp,%ebp
  400f49:	75 05                	jne    400f50 <phase_2+0x44>
  400f4b:	e8 63 07 00 00       	call   4016b3 <explode_bomb>
  400f50:	48 83 c4 28          	add    $0x28,%rsp
  400f54:	5b                   	pop    %rbx
  400f55:	5d                   	pop    %rbp
  400f56:	41 5c                	pop    %r12
  400f58:	41 5d                	pop    %r13
  400f5a:	c3                   	ret
```

## Phase 3:

```objdump
0000000000400f5b <phase_3>:
  400f5b:	48 83 ec 18          	sub    $0x18,%rsp
  400f5f:	48 8d 4c 24 08       	lea    0x8(%rsp),%rcx
  400f64:	48 8d 54 24 0c       	lea    0xc(%rsp),%rdx
  400f69:	be 3a 1e 40 00       	mov    $0x401e3a,%esi
  400f6e:	b8 00 00 00 00       	mov    $0x0,%eax
  400f73:	e8 88 fc ff ff       	call   400c00 <__isoc99_sscanf@plt>
  400f78:	83 f8 01             	cmp    $0x1,%eax
  400f7b:	7f 05                	jg     400f82 <phase_3+0x27>
  400f7d:	e8 31 07 00 00       	call   4016b3 <explode_bomb>
  400f82:	83 7c 24 0c 07       	cmpl   $0x7,0xc(%rsp)
  400f87:	77 64                	ja     400fed <phase_3+0x92>
  400f89:	8b 44 24 0c          	mov    0xc(%rsp),%eax
  400f8d:	ff 24 c5 50 1b 40 00 	jmp    *0x401b50(,%rax,8)
  400f94:	b8 00 00 00 00       	mov    $0x0,%eax
  400f99:	eb 05                	jmp    400fa0 <phase_3+0x45>
  400f9b:	b8 76 03 00 00       	mov    $0x376,%eax
  400fa0:	2d 7b 02 00 00       	sub    $0x27b,%eax
  400fa5:	eb 05                	jmp    400fac <phase_3+0x51>
  400fa7:	b8 00 00 00 00       	mov    $0x0,%eax
  400fac:	83 c0 3a             	add    $0x3a,%eax
  400faf:	eb 05                	jmp    400fb6 <phase_3+0x5b>
  400fb1:	b8 00 00 00 00       	mov    $0x0,%eax
  400fb6:	2d 93 01 00 00       	sub    $0x193,%eax
  400fbb:	eb 05                	jmp    400fc2 <phase_3+0x67>
  400fbd:	b8 00 00 00 00       	mov    $0x0,%eax
  400fc2:	05 da 03 00 00       	add    $0x3da,%eax
  400fc7:	eb 05                	jmp    400fce <phase_3+0x73>
  400fc9:	b8 00 00 00 00       	mov    $0x0,%eax
  400fce:	2d fd 00 00 00       	sub    $0xfd,%eax
  400fd3:	eb 05                	jmp    400fda <phase_3+0x7f>
  400fd5:	b8 00 00 00 00       	mov    $0x0,%eax
  400fda:	05 fd 00 00 00       	add    $0xfd,%eax
  400fdf:	eb 05                	jmp    400fe6 <phase_3+0x8b>
  400fe1:	b8 00 00 00 00       	mov    $0x0,%eax
  400fe6:	2d 95 03 00 00       	sub    $0x395,%eax
  400feb:	eb 0a                	jmp    400ff7 <phase_3+0x9c>
  400fed:	e8 c1 06 00 00       	call   4016b3 <explode_bomb>
  400ff2:	b8 00 00 00 00       	mov    $0x0,%eax
  400ff7:	83 7c 24 0c 05       	cmpl   $0x5,0xc(%rsp)
  400ffc:	7f 06                	jg     401004 <phase_3+0xa9>
  400ffe:	3b 44 24 08          	cmp    0x8(%rsp),%eax
  401002:	74 05                	je     401009 <phase_3+0xae>
  401004:	e8 aa 06 00 00       	call   4016b3 <explode_bomb>
  401009:	48 83 c4 18          	add    $0x18,%rsp
  40100d:	c3                   	ret
```

## Phase 4:

```objdump
000000000040100e <func4>:
  40100e:	53                   	push   %rbx
  40100f:	89 fb                	mov    %edi,%ebx
  401011:	b8 01 00 00 00       	mov    $0x1,%eax
  401016:	83 ff 01             	cmp    $0x1,%edi
  401019:	7e 0b                	jle    401026 <func4+0x18>
  40101b:	8d 7f ff             	lea    -0x1(%rdi),%edi
  40101e:	e8 eb ff ff ff       	call   40100e <func4>
  401023:	0f af c3             	imul   %ebx,%eax
  401026:	5b                   	pop    %rbx
  401027:	c3                   	ret

0000000000401028 <phase_4>:
  401028:	48 83 ec 18          	sub    $0x18,%rsp
  40102c:	48 8d 54 24 0c       	lea    0xc(%rsp),%rdx
  401031:	be 3d 1e 40 00       	mov    $0x401e3d,%esi
  401036:	b8 00 00 00 00       	mov    $0x0,%eax
  40103b:	e8 c0 fb ff ff       	call   400c00 <__isoc99_sscanf@plt>
  401040:	83 f8 01             	cmp    $0x1,%eax
  401043:	75 07                	jne    40104c <phase_4+0x24>
  401045:	83 7c 24 0c 00       	cmpl   $0x0,0xc(%rsp)
  40104a:	7f 05                	jg     401051 <phase_4+0x29>
  40104c:	e8 62 06 00 00       	call   4016b3 <explode_bomb>
  401051:	8b 7c 24 0c          	mov    0xc(%rsp),%edi
  401055:	e8 b4 ff ff ff       	call   40100e <func4>
  40105a:	83 f8 18             	cmp    $0x18,%eax
  40105d:	74 05                	je     401064 <phase_4+0x3c>
  40105f:	e8 4f 06 00 00       	call   4016b3 <explode_bomb>
  401064:	48 83 c4 18          	add    $0x18,%rsp
  401068:	c3                   	ret
```

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
