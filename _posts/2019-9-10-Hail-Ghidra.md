---
layout: post
title: Hail Ghidra
---
## Bypassing the login system on a binary

### Introduction
___

As a student, one of the courses I enjoyed the most was one about understanding the deep structure and workflow inside an operating system with the help of [Modern Operating Systems](https://en.wikipedia.org/wiki/Modern_Operating_Systems) book by Andrew Tanenbaum.

The coding part was the most funny for me and my friends.
In those coding labs the student tries to reveal hidden information in the different UNIX system **shared resources** like unix sockets for example.

Today we will try to analyze one of those programs particularly an exam.

The exam program hides certain information within the Operating System for students to work to reveal that data, if so, the program reveals a number called **secret**, the **more** secrets the **highest** the grade.

Before access the program first a login is done via the student identification and a password.

> In this blog I will try to get inside the exam using simple **reverse engineering**  techniques



(the info showed in this blog was gathered **long time ago**)

(this is not a tutorial of how to decompile or break a binary file)

### Let's jump right into it
___

The binary program that contains the exam is called *monitor*:

``` shell
$ file monitor 
monitor: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/l, for GNU/Linux 2.6.24, BuildID[sha1]=***68984e4bcc8dbe5a70d81*****8cccd9f82a5, with debug_info, not stripped

```

The file is a **ELF** (executable) dynamically linked for **64 bit** and for GNU linux using GCC.

*readelf* gives us more information about the file, in this case searching for info like **passwords** or students **identification** numbers (spanish identification card number or DNI)

``` shell
$ arraiz@mintVM:~/Desktop/ghidra$ readelf monitor -all | grep DNI
   243: 0000000000401f30    77 FUNC    GLOBAL DEFAULT   13 DNItol

```

``` shell
$ arraiz@mintVM:~/Desktop/ghidra$ readelf monitor -a | grep pas
   126: 0000000000402510   209 FUNC    GLOBAL DEFAULT   13 check_passwd

```

Here we see some declarations to **C functions** in the program, this information here is really helpful when switching to [**IDA**](https://www.hex-rays.com/products/ida/) or **Ghidra** for code analysis.



10 minutes later collecting data about functions definitions and strings reveal some **HARDCODED** strings inside the binary.

``` shell
$ arraiz@mintVM:~/Desktop/ghidra$ strings monitor
14******F
....
....
omitted lines
....
....
100**01*E
100**01*E
```

Holly cows

Those are the **Identification numbers** (+40) of all students, remember that those IDs **are the program password** as well.

There are also some 'testing' identification numbers with fake IDs  for development maybe.

The point here is that the IDs are not even encrypted or hashed.

The last try to get more hardcoded strings in the binary gives **this**. 

``` shell
$ arraiz@mintVM:~/Desktop/ghidra$ strings monitor | grep key
-----BEGIN RSA PRIVATE KEY-----
MIIEowI*********************************************************
II/UIZAUuRpYigRbjbWI8gb9bF0SWKWOoP9a+fmhdQ7Stkl79i2SsFyzU67zIYHg
x74ORxr4CU9GE6VnRp6y61nFmR6qTko/fOIFoik5McdkeyW2cezYvVtOafokOhTr
tl09k2Udi1Kl6+NxA4B+mgvRGJ9KM7IhCNWxOsn4G7W1bWjKYuqTUQ0DaE7b394W
XLCCbeh41Zfeyy3EIN2ipD3gsoePxx3URomeRFS0MCyH9vuJVblmrg/JrYlgpOQC+nueHe1OzfWU97s0jE4vSVgttTPmOYPqfMjsMQIDAQABAoIBAGB2cJnNeGQRkJcN
5LHFEOHfcT/8OuwhvOm9IRCDr3qsumL/vEfW8ge8GaCPbzBmRsO/KNdOytZi7yKm
jVvReCs89ooS3c4SowfQHvUWfKLA487cLX1sznPYTiMovICPeKcel7QqIWsH8PWX
Bd7FN7497GZs/c+wxdkDIngGYSiPbAzSI58n6X7sM8+EPFl4ESPfRzG9IWjXba3u
WsuUidhDAuldBYJrQbAM6thPXWDv85948iMpyNe18eT31z9Jnpq4OceG6Hc51A1i
o75YfoH5xQgmHFC7eY8hm5QNw+pgQMYREsvJBrBKgSUysciXhXhxglNDJaNbBMYH
3Iuo5b0CgYEA4ap1LLHAFq9ytL8iTbMZ7ibucwy6fA4QJl5OliJev4Oh/ObA0hB9
Vdbj334VEogZIHR4je/eAUq1IZyMx7dyba0MBrVHSwDW97ejjvFtTn8aAg9X83Z7
RsQIhBNGtGX+zlB8cCVuvrTwivbHBaIDwrZCtclKY7nHBTPBzLl7lM8CgYEA2Ypp
eIOimxuYfx4M+3eVDTOd4QVn+4fGykXoiqzxtAN7mtoCapXPedUxvSANrbQJvq94
j59qAZaCfvNzz9Y7N9q1++JI67O03tr8fp4UZGBBIB3+tBW2s1kkd339Kl54lf4S
Gf00zGPsQKPA1fVYqRI97YYhZbJVM5NQxNSFrv8CgYBRFIizPU5SGEmzbXUqy64G
ZlCIX8tlJTxiPMIpqUG3t9js4A/pqekOfX40X728gc/dXFuwS73NYwU/hVsDqwLf
KyzGAD4UUcHrET0f79ihOoOit9aW8DwMygRxR
***************************
fh3r6N3khOxgDx+TqhUf+wKBgQCr/8T+lU22x63eK/tlxBnkc0BMD0M03BiwC3Ae
**********************
XTlJ9EsBl4e7kDGYCZmnCDXodmYSD5kKLafaE4+gIosZ9C+kLNggjLzNJ6xFW+2x
ivlil7xGUZD2AAkRatTraYEGw+Uh6t2TEOFzDTpZrV+li7QLEbJHH/s99i9pdPuy
dLoyLwKBgD1CFDmKye8SnqfA+BOjLWXpoZ8kfWJc5KhRmxNK3g357iQBmIzz7kRP
/MisZoI+cArEy7ChlsBhQUTKmi/HWd5kZ6pED3hkkzztU9Rh9SfI7mlUplmZBtvf
ZvlYkArbn0TUbn4dsKfCtfsqt3ontIshsLcxDdFRGcc87i7xbACJ
-----END RSA PRIVATE KEY-----
```

Mother of horses.

That is a private key perhaps used to connect to the server used to process the marks of all students doing some **data encription** ?.

Because this post is intended to break the login, the key founded here will be used in another chapter (?).

Now time to check if the login still available with the data founded in the binary.

The ID numbers are working fine but the program asks for another code, an 'access code'.

``` shell
$ arraiz@mintVM:~/Desktop/ghidra$ ./monitor 
Introduce tu DNI (con letra).
1********

Introduce el codigo de acceso:

```

No other codes or password were found in the 'string analysis' so is time to take [King Ghidorah](https://github.com/NationalSecurityAgency/ghidra) out.

Once **decompiled** (buzzword_counter++) all code inside the binary **Ghidra** gives a **C implementation** of the functions inside the binary (some of them discovered in the string analysis), so is time to find some 'access code' o 'pass' functions.



**Bingo**

```c
  int check_passwd(void)

  {
    long lVar1;
    int iVar2;
    long lVar3;
    byte *pbVar4;
    byte *pbVar5;
    long in_FS_OFFSET;
    bool bVar6;
    bool bVar7;
    bool bVar8;
    byte bVar9;
    char buf [256];
    
    bVar9 = 0;
    lVar1 = *(long *)(in_FS_OFFSET + 0x28);
    bVar6 = false;
    bVar8 = true;
    __printf_chk(1,msg_monitor[(long)lang][5]);
    get_pwd(buf,7);
    lVar3 = 7;
    pbVar4 = (byte *)buf;
    pbVar5 = (byte *)"asipwd";
    do {
      if (lVar3 == 0) break;
      lVar3 = lVar3 + -1;
      bVar6 = *pbVar4 < *pbVar5;
      bVar8 = *pbVar4 == *pbVar5;
      pbVar4 = pbVar4 + (ulong)bVar9 * -2 + 1;
      pbVar5 = pbVar5 + (ulong)bVar9 * -2 + 1;
    } while (bVar8);
    iVar2 = 1;
    bVar7 = (!bVar6 && !bVar8) < bVar6;
    bVar6 = (!bVar6 && !bVar8) == bVar6;
    if (!bVar6) {
      lVar3 = 7;
      pbVar4 = (byte *)buf;
      pbVar5 = (byte *)"isapwd";
      do {
        if (lVar3 == 0) break;
        lVar3 = lVar3 + -1;
        bVar7 = *pbVar4 < *pbVar5;
        bVar6 = *pbVar4 == *pbVar5;
        pbVar4 = pbVar4 + (ulong)bVar9 * -2 + 1;
        pbVar5 = pbVar5 + (ulong)bVar9 * -2 + 1;
      } while (bVar6);
      if (!bVar6) {
        lVar3 = 7;
        pbVar4 = (byte *)buf;
        pbVar5 = (byte *)"prfasi";
        do {
          if (lVar3 == 0) break;
          lVar3 = lVar3 + -1;
          bVar7 = *pbVar4 < *pbVar5;
          bVar6 = *pbVar4 == *pbVar5;
          pbVar4 = pbVar4 + (ulong)bVar9 * -2 + 1;
          pbVar5 = pbVar5 + (ulong)bVar9 * -2 + 1;
        } while (bVar6);
        if (!bVar6) {
          lVar3 = 7;
          pbVar4 = (byte *)buf;
          pbVar5 = (byte *)"devasi";
          do {
            if (lVar3 == 0) break;
            lVar3 = lVar3 + -1;
            bVar7 = *pbVar4 < *pbVar5;
            bVar6 = *pbVar4 == *pbVar5;
            pbVar4 = pbVar4 + (ulong)bVar9 * -2 + 1;
            pbVar5 = pbVar5 + (ulong)bVar9 * -2 + 1;
          } while (bVar6);
          iVar2 = 0;
          if ((!bVar7 && !bVar6) == bVar7) {
            developer = 1;
            iVar2 = 1;
          }
        }
      }
    }
    if (lVar1 == *(long *)(in_FS_OFFSET + 0x28)) {
      return iVar2;
    }
                      /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
```

There are some values **'asipwd' , 'isapwd', 'prfasi', 'devasi'** hardcoded inside the C function, lets give them a shot...
```
Introduce tu DNI (con letra).
1********(fake ID founded)

Introduce el codigo de acceso:
******

...........
omitted lines
...........

1.- EJERCICIO 1: ******
2.- EJERCICIO 2: ******
3.- EJERCICIO 3: ******
4.- EJERCICIO 4: ******
5.- EJERCICIO 5: ******
6.- EJERCICIO 6: ******
7.- EJERCICIO 7: ******
0.- FIN DE LA PRUEBA (necesario)

-------
```
**Now we are IN!**

**NOW I CAN LEARN ABOUT IPCs!**

In other posts I will try to deconstruct the function that generate the secrets or use the private key to make some *h4ck1ng* or maybe not. 

### Conclusions
___

When writing software that is going to be for public use, it must be taken into account that there are tools and techniques to analyze the code within it and it is not necessary to have high knowledge about software to perform an analysis.

This blog was intended to demonstrate that with 3 tools (2 of them Linux commands) you can perform a simple analysis of a binary file.

### Regards to jtpfevaa