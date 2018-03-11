# 第 2 章：ユーザモードで実現する機能

## システムコール

- カーネルへの処理依頼
- システムコール処理中は CPU がカーネルモードに移行

strace - trace system calls and signals

```
$ strace ./hello
（略）
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 0), ...}) = 0
brk(NULL)                               = 0x1edb000
brk(0x1efc000)                          = 0x1efc000
write(1, "hello world\n", 12)           = 12
exit_group(0)                           = ?
+++ exited with 0 +++
```

`-T` オプションで各システムコールの実行時間を測定できる

```
$ strace -T ./hello
（略）
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 0), ...}) = 0 <0.000106>
brk(NULL)                               = 0x142e000 <0.000071>
brk(0x144f000)                          = 0x144f000 <0.000085>
write(1, "hello world\n", 12)           = 12 <0.001023>
exit_group(0)                           = ?
+++ exited with 0 +++
```

### 実行時間の実験

- ユーザモードで実行している時間：%user, %nice
- カーネルモードで実行している時間：%system
- プロセスもカーネルも動いていない時間：%idle

```
$ ./loop &
$ sar -P ALL 1 1
Linux 4.4.0-116-generic (ubuntu-xenial)         03/11/2018      _x86_64_      (2 CPU)

11:53:35 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
11:53:36 AM     all     45.05      0.00      0.00      0.00      0.00     54.95
11:53:36 AM       0    100.00      0.00      0.00      0.00      0.00      0.00
11:53:36 AM       1      0.99      0.00      0.00      0.00      0.00     99.01

Average:        CPU     %user     %nice   %system   %iowait    %steal     %idle
Average:        all     45.05      0.00      0.00      0.00      0.00     54.95
Average:          0    100.00      0.00      0.00      0.00      0.00      0.00
Average:          1      0.99      0.00      0.00      0.00      0.00     99.01
```

```
$ ./ppidloop &
$ sar -P ALL 1 1
Linux 4.4.0-116-generic (ubuntu-xenial)         03/11/2018      _x86_64_      (2 CPU)

11:58:58 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
11:58:59 AM     all     19.37      0.00     28.27      0.00      0.00     52.36
11:58:59 AM       0      0.00      0.00      0.00      0.00      0.00    100.00
11:58:59 AM       1     40.66      0.00     59.34      0.00      0.00      0.00

Average:        CPU     %user     %nice   %system   %iowait    %steal     %idle
Average:        all     19.37      0.00     28.27      0.00      0.00     52.36
Average:          0      0.00      0.00      0.00      0.00      0.00    100.00
Average:          1     40.66      0.00     59.34      0.00      0.00      0.00
```

## 標準 C ライブラリ

- GNU プロジェクトの標準 C ライブラリ：glibc
    - システムコールのラッパー関数
    - POSIX で定義されている関数

Go も libc をリンクしている

```
$ ldd /usr/bin/go
    linux-vdso.so.1 =>  (0x00007ffc275de000)
    libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f8804f37000)
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f8804b6d000)
    /lib64/ld-linux-x86-64.so.2 (0x00007f8805154000)
```

