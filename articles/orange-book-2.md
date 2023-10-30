---
title: "30日後にosを完成させる。(2日目)"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["os"]
published: true
---

## 10/30のアウトプット
- バイナリエディタではなく普通のエディタ(vim)でcファイルを作成しコンパイルとリンクを行い、hello worldを表示させた(p51まで)

- メモリマップをファイルへ保存した(2章)

:::details 実際のメモリマップファイルの中身
```
Index, Type, Type(name), PhysicalStart, NumberOfPages, Attribute
0, 3, EfiBootServicesCode, 00000000, 1, F // ブートサービスドライバの実行コードが0番地から4KiB(4096バイト)だけある
1, 7, EfiConventionalMemory, 00001000, 9F, F // 空き容量が4,096バイト目から(9 * 16 + 15 * 1) * 4KiB(651,264バイト)だけある
2, 7, EfiConventionalMemory, 00100000, 700, F // 空き容量が1,048,576バイト目から7 * 256 * 4KiB(7,340,032バイト)だけある
3, A, EfiACPIMemoryNVS, 00800000, 8, F
4, 7, EfiConventionalMemory, 00808000, 8, F
5, A, EfiACPIMemoryNVS, 00810000, F0, F
6, 4, EfiBootServicesData, 00900000, B00, F
7, 7, EfiConventionalMemory, 01400000, 3AB36, F
8, 4, EfiBootServicesData, 3BF36000, 20, F
9, 7, EfiConventionalMemory, 3BF56000, 270B, F
10, 1, EfiLoaderCode, 3E661000, 2, F
11, 4, EfiBootServicesData, 3E663000, 10F, F
12, 7, EfiConventionalMemory, 3E772000, 1, F
13, 4, EfiBootServicesData, 3E773000, 10A, F
14, 3, EfiBootServicesCode, 3E87D000, B7, F
15, A, EfiACPIMemoryNVS, 3E934000, 12, F
16, 0, EfiReservedMemoryType, 3E946000, 1C, F
17, 3, EfiBootServicesCode, 3E962000, 10A, F
18, 6, EfiRuntimeServicesData, 3EA6C000, 5, F
19, 5, EfiRuntimeServicesCode, 3EA71000, 5, F
20, 6, EfiRuntimeServicesData, 3EA76000, 5, F
21, 5, EfiRuntimeServicesCode, 3EA7B000, 5, F
22, 6, EfiRuntimeServicesData, 3EA80000, 5, F
23, 5, EfiRuntimeServicesCode, 3EA85000, 7, F
24, 6, EfiRuntimeServicesData, 3EA8C000, 8F, F
25, 4, EfiBootServicesData, 3EB1B000, 4DA, F
26, 7, EfiConventionalMemory, 3EFF5000, 4, F
27, 4, EfiBootServicesData, 3EFF9000, 6, F
28, 7, EfiConventionalMemory, 3EFFF000, 1, F
29, 4, EfiBootServicesData, 3F000000, A1B, F
30, 7, EfiConventionalMemory, 3FA1B000, 1, F
31, 3, EfiBootServicesCode, 3FA1C000, 17F, F
32, 5, EfiRuntimeServicesCode, 3FB9B000, 30, F
33, 6, EfiRuntimeServicesData, 3FBCB000, 24, F
34, 0, EfiReservedMemoryType, 3FBEF000, 4, F
35, 9, EfiACPIReclaimMemory, 3FBF3000, 8, F
36, A, EfiACPIMemoryNVS, 3FBFB000, 4, F
37, 4, EfiBootServicesData, 3FBFF000, 201, F
38, 7, EfiConventionalMemory, 3FE00000, 8D, F
39, 4, EfiBootServicesData, 3FE8D000, 20, F
40, 3, EfiBootServicesCode, 3FEAD000, 20, F
41, 4, EfiBootServicesData, 3FECD000, 9, F
42, 3, EfiBootServicesCode, 3FED6000, 1E, F
43, 6, EfiRuntimeServicesData, 3FEF4000, 84, F
44, A, EfiACPIMemoryNVS, 3FF78000, 88, F
45, 6, EfiRuntimeServicesData, FFC00000, 400, 1
```
:::

## 理解したこと
- 0xが16進数表記。
- 0bが2進数表記。
- cpuはデジタル回路。電圧が低いか高いかという２値で動作するような回路。そのため2進数を使う。
- 64bitCPUとは一回の計算で64桁の２進数を扱えるということ。
例:
(1 * 2 ** 63 + 1 * 2 ** 62 + 1 * 2 ** 61 + 1 * 2 ** 60 + 1 * 2 ** 59 + 1 * 2 ** 58 + 1 * 2 ** 57 + 1 * 2 ** 56 + 1 * 2 ** 55 + 1 * 2 ** 54 + 1 * 2 ** 53 + 1 * 2 ** 52 + 1 * 2 ** 51 + 1 * 2 ** 50 + 1 * 2 ** 49 + 1 * 2 ** 48 + 1 * 2 ** 47 + 1 * 2 ** 46 + 1 * 2 ** 45 + 1 * 2 ** 44 + 1 * 2 ** 43 + 1 * 2 ** 42 + 1 * 2 ** 41 + 1 * 2 ** 40 + 1 * 2 ** 39 + 1 * 2 ** 38 + 1 * 2 ** 37 + 1 * 2 ** 36 + 1 * 2 ** 35 + 1 * 2 ** 34 + 1 * 2 ** 33 + 1 * 2 ** 32 + 1 * 2 ** 31 + 1 * 2 ** 30 + 1 * 2 ** 29 + 1 * 2 ** 28 + 1 * 2 ** 27 + 1 * 2 ** 26 + 1 * 2 ** 25 + 1 * 2 ** 24 + 1 * 2 ** 23 + 1 * 2 ** 22 + 1 * 2 ** 21 + 1 * 2 ** 20 + 1 * 2 ** 19 + 1 * 2 ** 18 + 1 * 2 ** 17 + 1 * 2 ** 16 + 1 * 2 ** 15 + 1 * 2 ** 14 + 1 * 2 ** 13 + 1 * 2 ** 12 + 1 * 2 ** 11 + 1 * 2 ** 10 + 1 * 2 ** 9 + 1 * 2 ** 8 + 1 * 2 ** 7 + 1 * 2 ** 6 + 1 * 2 ** 5 + 1 * 2 ** 4 + 1 * 2 ** 3 + 1 * 2 ** 2 + 1 * 2 ** 1 + 1 * 2 ** 0) = 1844京6744兆737億955万2000

- コンピュータで扱う数値は２種類。機械語命令かそれ以外か

- 1日目でなぜhello world!文字列が出たのか？実行可能ファイルをUSBメモリに指定された名前で保存されており、pcに内蔵しているUEFI BIOSがそのファイルを読み取ったから。

- usbメモリとはよく見るusb。フラッシュメモリというものでデータの長期保存には適正ていないが電源を消してもデータは消えない。

- BIOS(basic input output system)はpcに電源を入れて最初に実行されるファームウェアと呼ばれるプログラム。osをストレージから読み出す。UEFI(unified extensible firmware interface) という仕様に従って作られたBIOSをUEFI BIOSという。

- UEFI BIOSが実行してくれるアプリケーションをUEFIアプリケーションと呼ぶ。

- メモリには1バイトごとに番地が振られている
- CPUはアドレスを使って機械語命令を読み、データを読み書きする。

- メインメモリにはPysicalStart, Type, NumberOfPagesがある。
    - 番地, 番地の使われ方、何ページあるか(ページは4KiB単位)
    - メモリマップの例
    
| Type値 | Type名 | 意味 |
| ---- | ---- | ---- |
| 1 | EfiLoaderCode | UEFIアプリケーションの実行コード |
| 2 | EfiLoaderData | UEFIアプリケーションが使うデータ領域 |
| 3 | EfiBootServicesCode | ブートサービスドライバの実行コード |
| 4 | EfiLoaderData | ブートサービスドライバが使うデータ領域 |
| 7 | EfiConventionalMemory | 空き容量 |

## 感想
みかん本のgithubのソースコードを見てるだけで実際に手を動かしていないことに気づいた。
これではいかんとgithubのリポジトリを作った。
https://github.com/satouyuuki/mikanos

ただ写経してるだけだとつまらないので以下のURLを参考にqemuのデバッグを試みたが失敗。(uefiアプリのビルドが全然終わらない)
https://blog.rkarsnk.jp/post/2021/12/10/debugefiapp/
詰まって色々試してるうちに時間が経ってしまったので今日はここまで
