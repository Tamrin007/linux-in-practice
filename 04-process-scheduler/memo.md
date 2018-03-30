# 第 4 章：プロセススケジューラ

## 実験 4-A, 4-B, 4-C

論理 CPU 上ではプロセスを任意の時間で区切って順番に動かしている

## コンテキストスイッチ

論理 CPU 上で動作するプロセスの切り替え

```c
void main(void)
{
    foo()
    bar()
}
```

上記コードでは `foo()` の呼び出しの後に別プロセスが動く可能性がある

## プロセスの状態

- 実行状態：論理 CPU を使用中
- 実行待ち状態：CPU 時間が割り当てられるのを待っている
- スリープ状態：イベント発生待ち
- ゾンビ状態：親プロセスが終了状態を得るのを待っている

```
$ ps ax
  PID TTY      STAT   TIME COMMAND
    1 ?        Ss     0:15 /sbin/init
                .
                .
                .
 1050 ?        Ss     0:00 /usr/sbin/atd -f
 1055 ?        Ss     0:00 /usr/sbin/sshd -D
 1060 ?        Ss     0:00 /usr/sbin/acpid
 1066 ?        Ssl    0:00 /usr/sbin/rsyslogd -n
 1072 ?        Ss     0:00 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation
 1076 ?        Ss     0:00 /usr/sbin/cron -f
 1079 ?        Ssl    0:00 /usr/lib/snapd/snapd
 1100 ?        Ss     0:00 /sbin/mdadm --monitor --pid-file /run/mdadm/monitor.pid --daemonise --scan --syslog
 1106 ?        Ssl    0:00 /usr/lib/policykit-1/polkitd --no-debug
 1157 ?        Ss     0:00 /usr/sbin/irqbalance --pid=/var/run/irqbalance.pid
 1194 ?        Sl     0:00 /usr/sbin/VBoxService
 1211 tty1     Ss+    0:00 /sbin/agetty --noclear tty1 linux
 1212 ttyS0    Ss+    0:00 /sbin/agetty --keep-baud 115200 38400 9600 ttyS0 vt220
 1488 ?        Ss     0:00 sshd: vagrant [priv]
 1490 ?        Ss     0:00 /lib/systemd/systemd --user
 1492 ?        S      0:00 (sd-pam)
 1525 ?        R      0:00 sshd: vagrant@pts/0
 1526 pts/0    Ss     0:00 -bash
 1565 pts/0    R+     0:00 ps ax
```

## 状態遷移

プロセスは「実行待ち状態」「実行状態」「スリープ状態」を遷移する

## アイドル状態

- 「何もしない」というプロセスが動作している
- 通常は CPU の特殊な命令によって CPU を休止状態にする
- `sar` コマンドの `%idle`

### 疑問

図 04-16 において、ファイル読み出し中は CPU がアイドル状態になるとされているが、そうすると何がファイルを読み出すのか？（別プロセスが読み出すからこのプロセスは CPU 時間を使わないということ？）

## スループットとレイテンシ

- スループット = 完了したプロセスの数 / 経過時間
    - 論理 CPU がアイドル状態にならない場合はプロセスを増やしてもスループットは変わらない（厳密にはコンテキストスイッチのオーバーヘッドで下がる）
- レイテンシ = 処理終了時刻 - 処理開始時刻
    - プロセスの増加でレイテンシは悪化

## 論理 CPU が複数ある場合のスケジューリング

ロードバランサ（グローバルスケジューラ）で複数の論理 CPU 間の負荷分散を行う（ラウンドロビンとか）

## 優先度の変更

- `nice()` システムコールで特定のプロセスに実行優先度を付与できる
- -19 から 20 まで
- `sar` コマンドの `%nice` は優先度を変更したプログラムの実行時間
