In this post, I will analyze *Kakaotalk*, a popular messaging application in Korea. The objective is finding Kakaotalk login credentials in memory.

## 1. Scenario
I created 5 memory dumps for this scenario. 

The first memory dump is named *kakaotalk_passwdlogin_keeploggedin.raw*. It was created by selecting "keep me logged in" then logging in with my email and password.

![[image1_login.png]]

The following is a diagram showing how the remaining 4 memory dumps were created.

![[Image Creation.png]]

*state1.raw* was dumped before logging in. *state2.raw* was dumped right after logging in using my password. *state3.raw* and *state4.raw* were each created after opening a chatroom and sending a message, respectively.

*state1.raw* capture moment:
![[state1.png]]

*state2.raw* capture moment:
![[state2.png]]

*state3.raw* capture moment:
![[state3.png]]

*state4.raw* capture moment:
![[state4.png]]

## 2. Analysis

Before we begin, as always, let's find out the PIDs that will be used. 

``` sh
$ vol -f kakaotalk_passwdlogin_keeploggedin.raw -q windows.pslist | grep Kakao
1832    3692    KakaoTalk.exe   0x86861c07b2c0  26      -       1       False   2026-05-02 23:46:05.000000 UTC  N/A     Disabled
$ vol -f state1.raw -q windows.pslist | grep Kakao
8172    4420    KakaoTalk.exe   0x8709aaedb080  14      -       1       False   2026-05-05 00:02:32.000000 UTC  N/A     Disabled
```

The PID is 1832 for *kakaotalk_passwdlogin_keeploggedin.raw* and 8172 for *stateX.raw* files.

We want to extract credentials for logging in, which means email or phone number and the password. 

### finding email in memory
Let's start with the email first. I will use *state4.raw* since it was created last and if the email is supposed to be overwritten(removed), state4 has the highest chance of it having happened.

``` sh
$ vol -f state4.raw -q windows.vadyarascan --wide --yara-string "0101087ssk@gmail.com" --pid 8172
[REMOVED]
0x548eb70       8172    2026-05-05 00:02:32.000000 UTC  4420    KakaoTalk.exe   1       26      r1      $a
30 00 31 00 30 00 31 00 30 00 38 00 37 00 73 00 0.1.0.1.0.8.7.s.
73 00 6b 00 40 00 67 00 6d 00 61 00 69 00 6c 00 s.k.@.g.m.a.i.l.
2e 00 63 00 6f 00 6d 00                         ..c.o.m.
0x548f2f0       8172    2026-05-05 00:02:32.000000 UTC  4420    KakaoTalk.exe   1       26      r1      $a
30 00 31 00 30 00 31 00 30 00 38 00 37 00 73 00 0.1.0.1.0.8.7.s.
73 00 6b 00 40 00 67 00 6d 00 61 00 69 00 6c 00 s.k.@.g.m.a.i.l.
2e 00 63 00 6f 00 6d 00                         ..c.o.m.
0x4f7d600       8172    2026-05-05 00:02:32.000000 UTC  4420    KakaoTalk.exe   1       26      r1      $a
30 00 31 00 30 00 31 00 30 00 38 00 37 00 73 00 0.1.0.1.0.8.7.s.
73 00 6b 00 40 00 67 00 6d 00 61 00 69 00 6c 00 s.k.@.g.m.a.i.l.
2e 00 63 00 6f 00 6d 00                         ..c.o.m.
0x5a0e251       8172    2026-05-05 00:02:32.000000 UTC  4420    KakaoTalk.exe   1       26      r1      $a
30 31 30 31 30 38 37 73 73 6b 40 67 6d 61 69 6c 0101087ssk@gmail
2e 63 6f 6d                                     .com
0x5a0e2cc       8172    2026-05-05 00:02:32.000000 UTC  4420    KakaoTalk.exe   1       26      r1      $a
30 31 30 31 30 38 37 73 73 6b 40 67 6d 61 69 6c 0101087ssk@gmail
2e 63 6f 6d                                     .com
```

There are 5 results for the email address in state4. Let's look up their references in state4 as well. 

``` sh
$ vol -f state4.raw -q windows.vadyarascan --yara-string "{ (70 eb 48 05 | f0 f2 48 05 | 00 d6 f7 04 | cc e2 a0 05) }" --pid 8172
[REMOVED]
0x56ae100       8172    2026-05-05 00:02:32.000000 UTC  4420    KakaoTalk.exe   1       26      r1      $a
70 eb 48 05                                     p.H.
0x56ae3c0       8172    2026-05-05 00:02:32.000000 UTC  4420    KakaoTalk.exe   1       26      r1      $a
f0 f2 48 05                                     ..H.
0x5ac1dc8       8172    2026-05-05 00:02:32.000000 UTC  4420    KakaoTalk.exe   1       26      r1      $a
00 d6 f7 04                                     ....
```

There are 3 pointers to the email addresses. They each point to a different email address string. Next, I try to find a pattern where the email string pointers are stored. 

``` sh
$ vol -f state4.raw -q windows.vadinfo --pid 8172 --address 0x56ae3c0 --dump
[REMOVED]
$ vol -f state4.raw -q windows.vadinfo --pid 8172 --address 0x5ac1dc8 --dump
[REMOVED]
# rename for consistency; state4 => *-3.dmp
$ mv pid.8172.vad.0x5930000-0x68fffff.dmp pid.8172.vad.0x5930000-0x68fffff-3.dmp
$ xxd pid.8172.vad.0x5130000-0x592ffff-3.dmp > xxd_state4_0x5130000.txt
$ xxd pid.8172.vad.0x5930000-0x68fffff-3.dmp > xxd_state4_0x5930000.txt
$ cat xxd_state4_0x5130000.txt | grep -C 5 -e "57e100"
0057e0b0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0057e0c0: d06c af05 0000 0000 cbad 79b2 fea2 5808  .l........y...X.
0057e0d0: c81d 6542 0100 0000 0000 0000 0000 0000  ..eB............
0057e0e0: 0412 3400 212b 0000 0000 0000 0000 0000  ..4.!+..........
0057e0f0: 0200 0000 0000 0000 0001 00c4 0870 0001  .............p..
0057e100: 70eb 4805 0000 0000 0000 0000 0000 0000  p.H.............
0057e110: 1400 0000 0000 0000 1700 0000 0000 0000  ................
0057e120: 707d f804 0000 0000 d07d f804 0000 0000  p}.......}......
0057e130: d07d f804 0000 0000 b333 f969 0103 0712  .}.......3.i....
0057e140: d0bf 4405 0000 0000 f0bf 4405 0000 0000  ..D.......D.....
0057e150: f0bf 4405 0000 0000 b333 f969 08ca 0001  ..D......3.i....
$ cat xxd_state4_0x5130000.txt | grep -C 5 -e "57e3c0"
0057e370: 0f00 0000 0000 0000 0000 0000 0000 0000  ................
0057e380: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0057e390: 0f00 0000 0000 0000 303f 9505 0000 0000  ........0?......
0057e3a0: e0be c204 0000 0000 0000 0000 0000 0000  ................
0057e3b0: 2700 0000 0000 0000 2f00 0000 0000 0000  '......./.......
0057e3c0: f0f2 4805 0000 0000 0000 0000 0000 0000  ..H.............
0057e3d0: 1400 0000 0000 0000 1700 0000 0000 0000  ................
0057e3e0: 6900 6f00 7300 0000 0000 0000 0000 0000  i.o.s...........
0057e3f0: 0300 0000 0000 0000 0700 0000 0000 0000  ................
0057e400: 3200 3600 2e00 3300 2e00 3500 0000 0000  2.6...3...5.....
0057e410: 0600 0000 0000 0000 0700 0000 0000 0000  ................
$ cat xxd_state4_0x5930000.txt | grep -C 5 -e "191dc0"
00191d70: 2803 7b42 0100 0000 0000 0000 0000 0000  (.{B............
00191d80: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00191d90: 0700 0000 0000 0000 20cd 3505 0000 0000  ........ .5.....
00191da0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00191db0: 0000 0000 0000 00a5 2600 0000 6800 0000  ........&...h...
00191dc0: af00 0000 7700 0000 00d6 f704 0000 0000  ....w...........
00191dd0: 0000 0001 0101 0100 0000 0000 0900 0000  ................
00191de0: 7273 006f 0000 0000 0000 0000 0000 636f  rs.o..........co
00191df0: 0000 0000 0000 0000 0100 0000 0a73 dc00  .............s..
00191e00: 0a73 dc00 9dc7 f100 3d0b 1004 0000 0000  .s......=.......
00191e10: 0100 0000 6e70 7574 0000 0000 0000 0000  ....nput........
```

Of course, with just 1 sample it is almost impossible to find a pattern. Time to try the other *.raw* file. 

``` sh
# find email address in memory
$ vol -q -f kakaotalk_passwdlogin_keeploggedin.raw windows.vadyarascan --pid 1832 --yara-string "0101087ssk@gmail.com" --wide
[REMOVED]
0x524c750       1832    2026-05-02 23:46:05.000000 UTC  3692    KakaoTalk.exe   1       26      r1      $a
30 00 31 00 30 00 31 00 30 00 38 00 37 00 73 00 0.1.0.1.0.8.7.s.
73 00 6b 00 40 00 67 00 6d 00 61 00 69 00 6c 00 s.k.@.g.m.a.i.l.
2e 00 63 00 6f 00 6d 00                         ..c.o.m.
0x524cb50       1832    2026-05-02 23:46:05.000000 UTC  3692    KakaoTalk.exe   1       26      r1      $a
30 00 31 00 30 00 31 00 30 00 38 00 37 00 73 00 0.1.0.1.0.8.7.s.
73 00 6b 00 40 00 67 00 6d 00 61 00 69 00 6c 00 s.k.@.g.m.a.i.l.
2e 00 63 00 6f 00 6d 00                         ..c.o.m.
0x56f0e20       1832    2026-05-02 23:46:05.000000 UTC  3692    KakaoTalk.exe   1       26      r1      $a
30 00 31 00 30 00 31 00 30 00 38 00 37 00 73 00 0.1.0.1.0.8.7.s.
73 00 6b 00 40 00 67 00 6d 00 61 00 69 00 6c 00 s.k.@.g.m.a.i.l.
2e 00 63 00 6f 00 6d 00                         ..c.o.m.
# find pointer to email address
$ vol -f kakaotalk_passwdlogin_keeploggedin.raw -q windows.vadyarascan --yara-string "{ (50 c7 24 05 | 50 cb 24 05 | 20 0e 6f 05) }" --pid 1832
[REMOVED]
0x51d60f0       1832    2026-05-02 23:46:05.000000 UTC  3692    KakaoTalk.exe   1       26      r1      $a
50 c7 24 05                                     P.$.
0x51d63b0       1832    2026-05-02 23:46:05.000000 UTC  3692    KakaoTalk.exe   1       26      r1      $a
50 cb 24 05                                     P.$.
0x55d17a8       1832    2026-05-02 23:46:05.000000 UTC  3692    KakaoTalk.exe   1       26      r1      $a
20 0e 6f 05
# extract VADs
$ vol -f kakaotalk_passwdlogin_keeploggedin.raw -q windows.vadinfo --pid 1832 --address 0x51d60f0 --dump
[REMOVED]
$ xxd pid.1832.vad.0x51b0000-0x59affff.dmp > xxd_other_0x51b0000.txt
# view data surrounding email pointer
$ cat xxd_other_0x51b0000.txt | grep -C 5 -e "000260f0"
000260a0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000260b0: b013 bc05 0000 0000 5c29 5525 c7f8 0308  ........\)U%....
000260c0: c81d 6542 0100 0000 0000 0000 0000 0000  ..eB............
000260d0: 0412 3400 212b 0000 0000 0000 0000 0000  ..4.!+..........
000260e0: 0200 0000 0000 0000 0000 0000 0000 0000  ................
000260f0: 50c7 2405 0000 0000 0000 0000 0000 0000  P.$.............
00026100: 1400 0000 0000 0000 1700 0000 0000 0000  ................
00026110: c0f1 fb04 0000 0000 20f2 fb04 0000 0000  ........ .......
00026120: 20f2 fb04 0000 0000 c88c f669 0000 0000   ..........i....
00026130: 80fc 2005 0000 0000 a0fc 2005 0000 0000  .. ....... .....
00026140: a0fc 2005 0000 0000 c88c f669 0000 0000  .. ........i....
$ cat xxd_other_0x51b0000.txt | grep -C 5 -e "000263b0"
00026360: 0f00 0000 0000 0000 0000 0000 0000 0000  ................
00026370: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00026380: 0f00 0000 0000 0000 60c2 1b05 0000 0000  ........`.......
00026390: 9069 3e05 0000 0000 0000 0000 0000 0000  .i>.............
000263a0: 2700 0000 0000 0000 2f00 0000 0000 0000  '......./.......
000263b0: 50cb 2405 0000 0000 0000 0000 0000 0000  P.$.............
000263c0: 1400 0000 0000 0000 1700 0000 0000 0000  ................
000263d0: 6900 6f00 7300 0000 0000 0000 0000 0000  i.o.s...........
000263e0: 0300 0000 0000 0000 0700 0000 0000 0000  ................
000263f0: 3200 3600 2e00 3300 2e00 3500 0000 0000  2.6...3...5.....
00026400: 0600 0000 0000 0000 0700 0000 0000 0000  ................
$ cat xxd_other_0x51b0000.txt | grep -C 5 -e "4217a0"
00421750: 2803 7b42 0100 0000 0000 0000 0000 0000  (.{B............
00421760: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00421770: 0700 0000 0000 0000 30ce db05 0000 0000  ........0.......
00421780: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00421790: 0000 0000 0000 00ba 2600 0000 6800 0000  ........&...h...
004217a0: af00 0000 7700 0000 200e 6f05 0000 0000  ....w... .o.....
004217b0: 0000 0001 0101 0156 0000 0000 0900 0000  .......V........
004217c0: 016e 0074 0000 0000 0000 0000 0000 0000  .n.t............
004217d0: 0000 0000 0000 0000 0100 0000 0a73 dc00  .............s..
004217e0: 0a73 dc00 9dc7 f100 6b0a 1004 0000 0000  .s......k.......
004217f0: 0100 0000 0000 0001 0000 0000 0000 0000  ................
```

It's still not easy to find a common pattern. Let's use *diff* to see if we can get a more clear view.

``` sh
# save each surrounding data to a text file
$ cat xxd_other_0x51b0000.txt | grep -C 5 -e "000260f0" | cut -d" " -f 2-9 > email_pointer_other.txt
$ cat xxd_other_0x51b0000.txt | grep -C 5 -e "000263b0" | cut -d" " -f 2-9 >> email_pointer_other.txt
$ cat xxd_other_0x51b0000.txt | grep -C 5 -e "4217a0" | cut -d" " -f 2-9 >> email_pointer_other.txt
$ cat xxd_state4_0x5130000.txt | grep -C 5 -e "57e100" | cut -d" " -f 2-9 > email_pointer_state4.txt
$ cat xxd_state4_0x5130000.txt | grep -C 5 -e "57e3c0" | cut -d" " -f 2-9 >> email_pointer_state4.txt
$ cat xxd_state4_0x5930000.txt | grep -C 5 -e "191dc0" | cut -d" " -f 2-9 >> email_pointer_state4.txt
# compare data
$ diff -y email_pointer_state4.txt email_pointer_other.txt
# first pointer
0000 0000 0000 0000 0000 0000 0000 0000                         0000 0000 0000 0000 0000 0000 0000 0000
d06c af05 0000 0000 cbad 79b2 fea2 5808                       | b013 bc05 0000 0000 5c29 5525 c7f8 0308
c81d 6542 0100 0000 0000 0000 0000 0000                         c81d 6542 0100 0000 0000 0000 0000 0000
0412 3400 212b 0000 0000 0000 0000 0000                         0412 3400 212b 0000 0000 0000 0000 0000
0200 0000 0000 0000 0001 00c4 0870 0001                       | 0200 0000 0000 0000 0000 0000 0000 0000
70eb 4805 0000 0000 0000 0000 0000 0000                       | 50c7 2405 0000 0000 0000 0000 0000 0000
1400 0000 0000 0000 1700 0000 0000 0000                         1400 0000 0000 0000 1700 0000 0000 0000
707d f804 0000 0000 d07d f804 0000 0000                       | c0f1 fb04 0000 0000 20f2 fb04 0000 0000
d07d f804 0000 0000 b333 f969 0103 0712                       | 20f2 fb04 0000 0000 c88c f669 0000 0000
d0bf 4405 0000 0000 f0bf 4405 0000 0000                       | 80fc 2005 0000 0000 a0fc 2005 0000 0000
f0bf 4405 0000 0000 b333 f969 08ca 0001                       | a0fc 2005 0000 0000 c88c f669 0000 0000
# second pointer
0f00 0000 0000 0000 0000 0000 0000 0000                         0f00 0000 0000 0000 0000 0000 0000 0000
0000 0000 0000 0000 0000 0000 0000 0000                         0000 0000 0000 0000 0000 0000 0000 0000
0f00 0000 0000 0000 303f 9505 0000 0000                       | 0f00 0000 0000 0000 60c2 1b05 0000 0000
e0be c204 0000 0000 0000 0000 0000 0000                       | 9069 3e05 0000 0000 0000 0000 0000 0000
2700 0000 0000 0000 2f00 0000 0000 0000                         2700 0000 0000 0000 2f00 0000 0000 0000
f0f2 4805 0000 0000 0000 0000 0000 0000                       | 50cb 2405 0000 0000 0000 0000 0000 0000
1400 0000 0000 0000 1700 0000 0000 0000                         1400 0000 0000 0000 1700 0000 0000 0000
6900 6f00 7300 0000 0000 0000 0000 0000                         6900 6f00 7300 0000 0000 0000 0000 0000
0300 0000 0000 0000 0700 0000 0000 0000                         0300 0000 0000 0000 0700 0000 0000 0000
3200 3600 2e00 3300 2e00 3500 0000 0000                         3200 3600 2e00 3300 2e00 3500 0000 0000
0600 0000 0000 0000 0700 0000 0000 0000                         0600 0000 0000 0000 0700 0000 0000 0000
# third pointer
2803 7b42 0100 0000 0000 0000 0000 0000                         2803 7b42 0100 0000 0000 0000 0000 0000
0000 0000 0000 0000 0000 0000 0000 0000                         0000 0000 0000 0000 0000 0000 0000 0000
0700 0000 0000 0000 20cd 3505 0000 0000                       | 0700 0000 0000 0000 30ce db05 0000 0000
0000 0000 0000 0000 0000 0000 0000 0000                         0000 0000 0000 0000 0000 0000 0000 0000
0000 0000 0000 00a5 2600 0000 6800 0000                       | 0000 0000 0000 00ba 2600 0000 6800 0000
af00 0000 7700 0000 00d6 f704 0000 0000                       | af00 0000 7700 0000 200e 6f05 0000 0000
0000 0001 0101 0100 0000 0000 0900 0000                       | 0000 0001 0101 0156 0000 0000 0900 0000
7273 006f 0000 0000 0000 0000 0000 636f                       | 016e 0074 0000 0000 0000 0000 0000 0000
0000 0000 0000 0000 0100 0000 0a73 dc00                         0000 0000 0000 0000 0100 0000 0a73 dc00
0a73 dc00 9dc7 f100 3d0b 1004 0000 0000                       | 0a73 dc00 9dc7 f100 6b0a 1004 0000 0000
0100 0000 6e70 7574 0000 0000 0000 0000                       | 0100 0000 0000 0001 0000 0000 0000 0000
```

The pipe(|) indicates lines that are different. Surprisingly, there are quite a few lines that are common, despite having different virtual addresses.

This seems to be enough to create a plugin, especially since we can filter out false positives by dereferencing the pointer to see if actually does point to an email address. 

However, bruteforce scanning all VADs is going to take too long. I wonder if we can find a way to filter out VADs. 

``` sh
$ head xxd_other_0x51b0000.txt
00000000: 0000 0000 0000 0000 1b29 5562 faf8 0101  .........)Ub....
00000010: eeff eeff 0200 0000 1800 9b05 0000 0000  ................
00000020: 1800 c604 0000 0000 0000 4700 0000 0000  ..........G.....
[REMOVED]
$ head xxd_state4_0x5130000.txt
00000000: 0000 0000 0000 0000 8cad 79f5 c3a2 0101  ..........y.....
00000010: eeff eeff 0200 0000 1800 9305 0000 0000  ................
00000020: 1800 be04 0000 0000 0000 4100 0000 0000  ..........A.....
[REMOVED]
$ head xxd_state4_0x5930000.txt
00000000: 0000 0000 0000 0000 8cad 79f5 c3a2 0101  ..........y.....
00000010: eeff eeff 0200 0000 1800 a409 0000 0000  ................
00000020: 1800 1305 0000 0000 0000 4100 0000 0000  ..........A.....
```

The start of each VAD containing the email address pointer looks quite similar. They all start with 8 empty bytes and bytes 15 to 26 are in common. 

We can also check the VAD properties. 

``` sh
$ vol -q -f kakaotalk_passwdlogin_keeploggedin.raw windows.vadinfo --pid 1832 --address 0x524c750 | grep Kakao
1832    KakaoTalk.exe   0xffff86861c312850      0x51b0000       0x59affff       VadS    PAGE_READWRITE  2047    1       0xffff86861be2b380      N/A     Disabled
$ vol -q -f state4.raw windows.vadinfo --pid 8172 | grep -e "0x5130000" -e "0x5930000"
8172    KakaoTalk.exe   0xffff8709abadc5b0      0x5130000       0x592ffff       VadS    PAGE_READWRITE  2047    1       0xffff8709a427f100      N/A     Disabled
8172    KakaoTalk.exe   0xffff8709abaddd20      0x5930000       0x68fffff       VadS    PAGE_READWRITE  4047    1       0xffff8709aa5105c0      N/A     Disabled
```

All 3 VADs have tag VadS, PAGE_READWRITE permission, and private memory set. The high commit charge is notable as well. 

### finding password in memory
Now we have enough to find email addresses in memory, let's see if we can do the same for passwords. 

Again, I will start with *state4.raw* because it is most likely not to have the password in memory if it is supposed to be overwritten. 

Removed output here indicates nothing was found. 

``` sh
$ vol -f state4.raw -q windows.vadyarascan --wide --yara-string "[PASSWORD]" --pid 8172
[REMOVED]
$ vol -f state3.raw -q windows.vadyarascan --wide --yara-string "[PASSWORD]" --pid 8172
[REMOVED]
$ vol -f state2.raw -q windows.vadyarascan --wide --yara-string "[PASSWORD]" --pid 8172
Volatility 3 Framework 2.27.0

Offset  PID     CreateTime      PPID    ImageFileName   SessionId       Threads Rule    Component       Value

0x55128d0       8172    2026-05-05 00:02:32.000000 UTC  4420    KakaoTalk.exe   1       27      r1      $a
[WIDE_PASSWORD_BYTES]             [WIDE_PASSWORD]
0x55e0648       8172    2026-05-05 00:02:32.000000 UTC  4420    KakaoTalk.exe   1       27      r1      $a
[WIDE_PASSWORD_BYTES]             [WIDE_PASSWORD]
0x55e14b0       8172    2026-05-05 00:02:32.000000 UTC  4420    KakaoTalk.exe   1       27      r1      $a
[PASSWORD_BYTES]                               [PASSWORD]
```

The password seems to be overwritten in state4 and state3, despite it having been less than 10 seconds after login when the dumps were captured. This is not good for our purposes, but good from a security standpoint. 

As we did with the email address, let's find pointers to these password strings and view the data surrounding them. 

``` sh
$ vol -f state2.raw -q windows.vadyarascan --yara-string "{ d0 28 51 05 }" --pid 8172
[REMOVED]
$ vol -f state2.raw -q windows.vadyarascan --yara-string "{ 48 06 5e 05 }" --pid 8172
Volatility 3 Framework 2.27.0

Offset  PID     CreateTime      PPID    ImageFileName   SessionId       Threads Rule    Component       Value

0x5708070       8172    2026-05-05 00:02:32.000000 UTC  4420    KakaoTalk.exe   1       27      r1      $a
48 06 5e 05                                     H.^.
$ vol -f state2.raw -q windows.vadyarascan --yara-string "{ b0 14 5e 05 }" --pid 8172
[REMOVED]
```

There is only 1 pointer to the password strings. It is at offset `0x5708070`.

``` sh
# extract VAD
$ vol -f state2.raw -q windows.vadinfo --pid 8172 --address 0x55128d0 --dump
Volatility 3 Framework 2.27.0

PID     Process Offset  Start VPN       End VPN Tag     Protection      CommitCharge    PrivateMemory   Parent  File    File output

8172    KakaoTalk.exe   0xffff8709abadc5b0      0x5130000       0x592ffff       VadS    PAGE_READWRITE  2047    1       0xffff8709ac460340      N/A     pid.8172.vad.0x5130000-0x592ffff.dmp
# rename for consistency; state2 => *-1.dmp
$ mv pid.8172.vad.0x5130000-0x592ffff.dmp pid.8172.vad.0x5130000-0x592ffff-1.dmp
# 0x5d8070 = 0x5708070 - 0x5130000
$ cat xxd_state2_0x5130000.txt | grep -C 5 -e "5d8070"
005d8020: 0000 0000 0000 0000 0000 0000 0000 0000  ................
005d8030: f00f 751e ff7f 0000 0000 0000 0000 0000  ..u.............
005d8040: 0000 0000 0000 0000 0000 0000 0000 0000  ................
005d8050: 0000 0000 0000 0000 0000 0000 0000 0000  ................
005d8060: 0000 0000 0000 0000 68ff 5d05 0000 0000  ........h.].....
005d8070: 4806 5e05 0000 0000 0000 0000 0000 0000  H.^.............
005d8080: 0000 0000 0000 0000 50da 7105 0000 0000  ........P.q.....
005d8090: a0a7 5505 0000 0000 00a5 5505 0000 0000  ..U.......U.....
005d80a0: 708a 9805 0000 0000 0100 0000 0000 0000  p...............
005d80b0: 7aeb 8942 0100 0000 e094 7c05 0000 0000  z..B......|.....
005d80c0: 0300 0000 0000 0000 1879 7005 0000 0000  .........yp.....
```

Now we have 1 sample of where password pointers are stored. Let's get the other sample from the other file. 

``` sh
$ vol -q -f kakaotalk_passwdlogin_keeploggedin.raw windows.vadyarascan --pid 1832 --yara-string "[PASSWORD]" --wide
Volatility 3 Framework 2.27.0

Offset  PID     CreateTime      PPID    ImageFileName   SessionId       Threads Rule    Component       Value
```

Unfortunately, it seems the memory dump was captured too late. I have no choice but to create another memory dump file. 

I created a memory dump called *phone_number_login.raw*. In it, I logged in with my phone number + password instead of using my email. Perhaps we can find the phone number as well. 

Let's use the new memory dump to extract sample 2 of password pointer storage. 

``` sh
$ vol -f phone_number_login.raw windows.vadyarascan --wide --pid 5040 --yara-string "[PASSWORD]"
Volatility 3 Framework 2.27.0
Progress:  100.00               PDB scanning finished
Offset  PID     CreateTime      PPID    ImageFileName   SessionId       Threads Rule    Component       Value

0x54ee4d8       5040    2026-05-05 07:52:51.000000 UTC  3308    KakaoTalk.exe   1       26      r1      $a
[WIDE_PASSWORD_BYTES]             [WIDE_PASSWORD]
0x54f01f0       5040    2026-05-05 07:52:51.000000 UTC  3308    KakaoTalk.exe   1       26      r1      $a
[PASSWORD_BYTES]                               [PASSWORD]
0x55b2fa0       5040    2026-05-05 07:52:51.000000 UTC  3308    KakaoTalk.exe   1       26      r1      $a
[WIDE_PASSWORD_BYTES]             [WIDE_PASSWORD]
0x5641e10       5040    2026-05-05 07:52:51.000000 UTC  3308    KakaoTalk.exe   1       26      r1      $a
[WIDE_PASSWORD_BYTES]             [WIDE_PASSWORD]
0x7ff8d2059400  5040    2026-05-05 07:52:51.000000 UTC  3308    KakaoTalk.exe   1       26      r1      $a
[WIDE_PASSWORD_BYTES]             [WIDE_PASSWORD]
```

There are 5 password strings in memory.

``` sh
$ vol -f phone_number_login.raw windows.vadyarascan --pid 5040 --yara-string "{ (d8 e4 4e 05 | f0 01 4f 05 | a0 2f 5b 05 | 10 1e 64 05 | 00 94 05 d2 f8 7f )}"
Volatility 3 Framework 2.27.0
Progress:  100.00               PDB scanning finished
Offset  PID     CreateTime      PPID    ImageFileName   SessionId       Threads Rule    Component       Value

0x3170e58       5040    2026-05-05 07:52:51.000000 UTC  3308    KakaoTalk.exe   1       26      r1      $a
a0 2f 5b 05                                     ./[.
0x513bd90       5040    2026-05-05 07:52:51.000000 UTC  3308    KakaoTalk.exe   1       26      r1      $a
d8 e4 4e 05                                     ..N.
```

There are 2 pointers to the password strings in memory.

``` sh
$ vol -f phone_number_login.raw -q windows.vadinfo --pid 5040 --address 0x3170e58 --dump
[REMOVED]
$ vol -f phone_number_login.raw -q windows.vadinfo --pid 5040 --address 0x513bd90 --dump
[REMOVED]
$ xxd pid.5040.vad.0x3170000-0x326ffff.dmp > xxd_phone_0x3170000.txt
$ xxd pid.5040.vad.0x4c10000-0x540ffff.dmp > xxd_phone_0x4c10000.txt
$ cat xxd_phone_0x3170000.txt | grep -C 5 -e "00000e50"
00000e00: 0300 0000 0000 0000 e0d6 b603 0000 0000  ................
00000e10: 0300 0000 0000 0000 a08f 2205 0000 0000  ..........".....
00000e20: 0300 0000 0000 0000 2060 5700 0000 0000  ........ `W.....
00000e30: 0300 0000 0000 0000 a083 1a05 0000 0000  ................
00000e40: 0300 0000 0000 0000 b0bf 1305 0000 0000  ................
00000e50: 0300 0000 0000 0000 a02f 5b05 0000 0000  ........./[.....
00000e60: 0300 0000 0000 0000 600f b303 0000 0000  ........`.......
00000e70: 0380 0100 0000 0000 401f 7c03 0000 0000  ........@.|.....
00000e80: 0300 0000 0000 0000 6018 b103 0000 0000  ........`.......
00000e90: 0380 0100 0000 0000 801f 7c03 0000 0000  ..........|.....
00000ea0: 0300 0000 0000 0000 f0c4 2705 0000 0000  ..........'.....
$ cat xxd_phone_0x4c10000.txt | grep -C 5 -e "52bd90"
0052bd40: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0052bd50: f00f dacb f87f 0000 0000 0000 0000 0000  ................
0052bd60: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0052bd70: 0000 0000 0000 0000 0000 0000 0000 0000  ................
0052bd80: 0000 0000 0000 0000 58bc 2a05 0000 0000  ........X.*.....
0052bd90: d8e4 4e05 0000 0000 0000 0000 0000 0000  ..N.............
0052bda0: 0041 6e04 0000 0000 e043 2805 0000 0000  .An......C(.....
0052bdb0: 502a 0f05 0000 0000 8033 0f05 0000 0000  P*.......3......
0052bdc0: 1054 1005 0000 0000 0100 0000 0000 0000  .T..............
0052bdd0: 7aeb 8942 0100 0000 7aeb 8942 0100 0000  z..B....z..B....
0052bde0: 0300 0000 0000 0000 38b6 1305 0000 0000  ........8.......
```

Neither of them look very similar password pointer storage sample 1 from state2. The second entry has the closer address so let's compare that one with sample 1 first. 

``` sh
$ cat xxd_state2_0x5130000.txt | grep -C 10 -e "5d8070" | cut -d" " -f 2-9 > state2_password_pointer.txt
$ cat xxd_phone_0x4c10000.txt | grep -C 10 -e "52bd90" | cut -d" " -f 2-9 > phone_password_pointer.txt
$ diff -y state2_password_pointer.txt phone_password_pointer.txt
0000 0000 0000 0000 0000 0000 0000 0000                         0000 0000 0000 0000 0000 0000 0000 0000
10c1 5a42 0100 0000 f8c1 5a42 0100 0000                         10c1 5a42 0100 0000 f8c1 5a42 0100 0000
0000 0000 ffff ffff ffff ffff 0000 0000                         0000 0000 ffff ffff ffff ffff 0000 0000
0000 0000 0000 0000 0000 0000 0000 0000                         0000 0000 0000 0000 0000 0000 0000 0000
0000 0000 0000 0000 0000 0000 0000 0000                         0000 0000 0000 0000 0000 0000 0000 0000
0000 0000 0000 0000 0000 0000 0000 0000                         0000 0000 0000 0000 0000 0000 0000 0000
f00f 751e ff7f 0000 0000 0000 0000 0000                       | f00f dacb f87f 0000 0000 0000 0000 0000
0000 0000 0000 0000 0000 0000 0000 0000                         0000 0000 0000 0000 0000 0000 0000 0000
0000 0000 0000 0000 0000 0000 0000 0000                         0000 0000 0000 0000 0000 0000 0000 0000
0000 0000 0000 0000 68ff 5d05 0000 0000                       | 0000 0000 0000 0000 58bc 2a05 0000 0000
4806 5e05 0000 0000 0000 0000 0000 0000                       | d8e4 4e05 0000 0000 0000 0000 0000 0000
0000 0000 0000 0000 50da 7105 0000 0000                       | 0041 6e04 0000 0000 e043 2805 0000 0000
a0a7 5505 0000 0000 00a5 5505 0000 0000                       | 502a 0f05 0000 0000 8033 0f05 0000 0000
708a 9805 0000 0000 0100 0000 0000 0000                       | 1054 1005 0000 0000 0100 0000 0000 0000
7aeb 8942 0100 0000 e094 7c05 0000 0000                       | 7aeb 8942 0100 0000 7aeb 8942 0100 0000
0300 0000 0000 0000 1879 7005 0000 0000                       | 0300 0000 0000 0000 38b6 1305 0000 0000
0000 0000 0000 0000 0000 0000 0000 0000                         0000 0000 0000 0000 0000 0000 0000 0000
a007 8605 0000 0000 0000 0000 0000 0000                       | 80eb 0605 0000 0000 0000 0000 0000 0000
f0fb 4305 0000 0000 6004 4405 0000 0000                       | 008e 2505 0000 0000 7096 2505 0000 0000
1018 4405 0000 0000 0001 4405 0000 0000                       | 20e0 2505 0000 0000 f0d9 2505 0000 0000
30fe 4305 0000 0000 20f9 4305 0000 0000                       | 80e3 2505 0000 0000 c0e5 2505 0000 0000
```

Fortunately, if we go far enough up, there is plenty common ground that we can use. 

Let's also look at the start of the VADs so we can filter out VADs that we don't have to scan. 

``` sh
$ head xxd_state2_0x5130000.txt | cut -d" " -f 2-9 > state2_password_pointer_head.txt
$ head xxd_phone_0x4c10000.txt | cut -d" " -f 2-9 > phone_password_pointer_head.txt
$ diff -y state2_password_pointer_head.txt phone_password_pointer_head.txt
0000 0000 0000 0000 8cad 79f5 c3a2 0101                       | 0000 0000 0000 0000 a44a 1b42 fa9e 0101
eeff eeff 0200 0000 1800 9305 0000 0000                       | eeff eeff 0200 0000 1800 4105 0000 0000
1800 be04 0000 0000 0000 4100 0000 0000                       | 1800 7703 0000 0000 0000 5600 0000 0000
0000 1305 0000 0000 ff07 0000 0000 0000                       | 0000 c104 0000 0000 ff07 0000 0000 0000
7000 1305 0000 0000 00f0 9205 0000 0000                       | 7000 c104 0000 0000 00f0 4005 0000 0000
0000 0000 0100 0000 0000 0000 0000 0000                         0000 0000 0100 0000 0000 0000 0000 0000
e0ef 9205 0000 0000 e0ef 9205 0000 0000                       | e0ef 4005 0000 0000 e0ef 4005 0000 0000
0000 0000 0000 0000 8aa9 71ff c4a2 0110                       | 0000 0000 0000 0000 a24e 1348 fd9e 0110
80c1 8a05 0000 0000 d07c fd04 0000 0000                       | 30a3 d104 0000 0000 30f0 6505 0000 0000
0e00 0000 c0d0 e0f0 8a81 79cf 0000 0000                       | 0e00 0000 c0d0 e0f0 f8af 3cc8 0000 0000
```

Even at first glance, the VADs containing password pointers seem to have a similar structure to the VADs that contained pointers to email addresses. This should make things slightly more convenient when creating the plugin. 

### finding phone number in memory

Now we know how to find the email and password, which are all we need to log in to *Kakaotalk*. However, I also want to try finding the phone number if we are able to. 

First, let's see if it is removed from memory like the password is. 

``` sh
$ vol -f phone_number_login.raw -q windows.vadyarascan --pid 5040 --yara-string "0103[REDACTED]"
Volatility 3 Framework 2.27.0

Offset  PID     CreateTime      PPID    ImageFileName   SessionId       Threads Rule    Component       Value

0x54ef660       5040    2026-05-05 07:52:51.000000 UTC  3308    KakaoTalk.exe   1       26      r1      $a
30 31 30 33 [REDACTED]                0103[REDACTED]
```

The number is found in our *phone_number_login.raw* dump, which is reassuring. Next up is another dump I created. I dumped the memory about a minute or so after logging in with my phone number.

``` sh
$ vol -f phone_number_login2.raw -q windows.vadyarascan --wide --pid 5796 --yara-string "0103[REDACTED]"
Volatility 3 Framework 2.27.0

Offset  PID     CreateTime      PPID    ImageFileName   SessionId       Threads Rule    Component       Value

```

Apparently the phone number does not persist in memory like the email address does. 

Using the same method that was used for finding email addresses and passwords in memory probably can get us the phone number, but with the email address being always available and phone number only being available for a short time when it is used to login makes it a lower priority. Thus, I will skip it. 

## 3. Plugin

First we need to filter out the VADs that we will not be using. 

``` python
    def find_pointer_vad(self, vad_node):
        kernel = self.context.modules[self.config['kernel']]
        protect_vals = vadinfo.VadInfo.protect_values(
            self.context,
            kernel.layer_name,
            kernel.symbol_table_name
        )
        winnt_vals = vadinfo.winnt_protections

        if vad_node.get_tag() != "VadS":
            return True

        if vad_node.get_protection(protect_vals, winnt_vals) != "PAGE_READWRITE":
            return True

        if vad_node.get_private_memory() != 1:
            return True

        # Keep everything else
        return False
```
``` python
	# filter out VADs that are not qualitifed
	if layer.read(start_address, 8, pad=False) != b"\x00"*8:
		continue
	if layer.read(start_address+14, 12, pad=False) != b"\x01\x01\xee\xff\xee\xff\x02\x00\x00\x00\x18\x00":
		continue

```

Then we can scan for email address pointers and password pointers in memory.

``` sh
def find_password_pointer(start_address, end_address, layer):
    password_pointer_rule = r"""
    rule pattern1 {
        strings:
            $a = /\x00{16}\x10\xc1\x5a\x42\x01\x00{3}\xf8\xc1\x5a\x42\x01\x00{7}\xff{8}\x00{52}.{16}\x00{32}/
        condition:
            $a
    }
    """

    if USE_YARA_X:
        import yara_x
        compiled_rules = yara_x.compile(password_pointer_rule)
    else:
        import yara
        compiled_rules = yara.compile(source=password_pointer_rule)

    scanner = yarascan.YaraScanner(rules=compiled_rules)
    chunk = layer.read(start_address, end_address-start_address+1, pad=True)

    password_list = []
    address_list = []

    for offset, rule_name, name, value in scanner(chunk, start_address):
        password_address_bytes = layer.read(offset+160, 8, pad=False)

        password_address = int.from_bytes(password_address_bytes, byteorder="little")
        password = get_password(password_address, layer)
        if password:
            password_list.append(password)
            address_list.append(password_address)

    return password_list, address_list

def find_email_pointer(start_address, end_address, layer):
    email_pointer_rule = r"""
    rule pattern1 {
        strings:
            $a = /\xc8\x1d\x65\x42\x01\x00{11}\x04\x12\x34\x00\x21\x2b\x00{10}.{32}\x14\x00{7}\x17\x00{7}/
        condition:
            $a
    }
    rule pattern2 {
        strings:
            $a = /\x0f\x00{31}.{32}\x27\x00{7}\x2f\x00{7}.{16}\x14\x00{7}\x17\x00{7}\x69\x00\x6f\x00\x73\x00{11}\x03\x00{7}\x07\x00{7}\x32\x00\x36\x00\x2e\x00\x33\x00\x2e\x00\x35\x00{5}\x06\x00{7}\x07\x00{7}/
        condition:
            $a
    }
    rule pattern3 {
        strings:
            $a = /\x28\x03\x7b\x42\x01\x00{27}.{16}\x00{16}.{64}\x00{8}\x01\x00{3}\x0a\x73\xdc\x00/
        condition:
            $a
    }
    """

    if USE_YARA_X:
        import yara_x
        compiled_rules = yara_x.compile(email_pointer_rule)
    else:
        import yara
        compiled_rules = yara.compile(source=email_pointer_rule)

    scanner = yarascan.YaraScanner(rules=compiled_rules)
    chunk = layer.read(start_address, end_address-start_address+1, pad=True)

    email_list = []
    address_list = []
    pattern_list = []

    for offset, rule_name, name, value in scanner(chunk, start_address):
        if rule_name == "pattern1":
            email_address_bytes = layer.read(offset+48, 8, pad=False)
            pattern = 1

        elif rule_name == "pattern2":
            email_address_bytes = layer.read(offset+80, 8, pad=False)
            pattern = 2

        else:
            email_address_bytes = layer.read(offset+88, 8, pad=False)
            pattern = 3

        email_address = int.from_bytes(email_address_bytes, byteorder="little")
        email = get_email(email_address, layer)
        if email:
            email_list.append(email)
            address_list.append(email_address)
            pattern_list.append(pattern)

    return email_list, address_list, pattern_list
```

### result

The plugin works! 

``` sh
$ vol -f state2.raw -q windows.kakaotalk_credentials --pid 8172
Volatility 3 Framework 2.27.0

Virtual Address Pattern Number  Content type    Content

0x548eb70       1       Email address   0101087ssk@gmail.com
0x548f2f0       2       Email address   0101087ssk@gmail.com
0x4f7ed30       3       Email address   0101087***@gmail.com
0x55e0648       -       password        [PASSWORD]
0x4f7d600       3       Email address   0101087ssk@gmail.com
```

You can see that it only shows the email address once the password is overwritten. 

``` sh
$ vol -f state4.raw -q windows.kakaotalk_credentials --pid 8172
Volatility 3 Framework 2.27.0

Virtual Address Pattern Number  Content type    Content

0x548eb70       1       Email address   0101087ssk@gmail.com
0x548f2f0       2       Email address   0101087ssk@gmail.com
0x4f7ed30       3       Email address   0101087***@gmail.com
0x4f7d600       3       Email address   0101087ssk@gmail.com
```

Also, the plugin can find email addresses in memory even when the phone number is used to login to *Kakaotalk*.

``` sh
$ vol -f phone_number_login.raw -q windows.kakaotalk_credentials --pid 5040
Volatility 3 Framework 2.27.0

Virtual Address Pattern Number  Content type    Content

0x4c4bea0       3       Email address   0101087ssk@gmail.com
0x52ad2c0       2       Email address   0101087ssk@gmail.com
0x4c4de90       3       Email address   0101087***@gmail.com
0x54ee4d8       -       password        [PASSWORD]
```

## 4. test system
``` sh
# Window version
$ vol -f state1.raw windows.info
[REMOVED]
Major/Minor     15.19041
MachineType     34404
KeNumberProcessors      4
SystemTime      2026-05-05 00:03:13+00:00
[REMOVED]
PE MajorOperatingSystemVersion  10
PE MinorOperatingSystemVersion  0
PE Machine      34404
PE TimeDateStamp        Fri May 20 08:24:42 2101
# Kakaotalk version
$ vol -f state1.raw windows.pslist --dump --pid 8172
[REMOVED]
8172    4420    KakaoTalk.exe   0x8709aaedb080  14      -       1       False   2026-05-05 00:02:32.000000 UTC  N/A     8172.KakaoTalk.exe.0x140000000.dmp
$ exiftool 8172.KakaoTalk.exe.0x140000000.dmp
[REMOVED]
Linker Version                  : 14.50
Code Size                       : 37903360
Initialized Data Size           : 13682176
Uninitialized Data Size         : 0
Entry Point                     : 0x467e058
OS Version                      : 6.0
Image Version                   : 0.0
Subsystem Version               : 6.0
Subsystem                       : Windows GUI
```

## 5. Reflection
While working on this post, I realized the task of finding common patterns between memory dumps can be automated. I can create a plugin to (1) search for (hex) string using yara (2) find pointers to that data (3) extract data surrounding that pointer (4) repeat for different dump for sample 2 (5) compare samples to find common pattern.

The plugins I have created so far may not work for different versions of Windows or *Notepad.exe* and *Kakaotalk*. But if I can automate this task, all I have to do is find the executable version used in the memory dump. Then I can create 2 sample dumps using the same version, extract pattern from these 2 dumps, and use pattern to find the data I need from the real memory dump. 

Of course, if a pattern can't be found for whatever reason, it's not usable, but so far this method seems to work most of the time so it should come in handy.

## 6. Tools
As always, tools I created can be found on my Github. 

- *kakaotalk_credentials.py*:
	- https://github.com/DeceptiveRat/Custom_Volatility_Utilities

The memory dumps used will not be shared this scenario because it may contain sensitive chat logs. 
