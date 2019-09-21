## 16. ベクトルマスク命令

ベクトルレジスタに保持されたマスクの値を操作するためのいくつかの命令が定義されている。

### 16.1. ベクトルマスクレジスタ論理演算命令

ベクトルマスクレジスタの論理演算は、マスクレジスタ上で実行される。マスクレジスタの1要素のサイズはSEW/LMULであり、したがってこれらの命令は`vtype`レジスタの`vlmul`フィールドの設定に関係なく、すべて単一ベクトルレジスタに対して適用される。

他のベクトル命令と同様に、`vstart`よりも小さなインデックスの要素は変更されず、これらの命令の実行後には`vstart`は0に戻される。ベクトルマスク論理演算命令はマスクされず、従ってインアクティブな要素は存在しない。`vl`以降のマスク要素、つまりテール要素はすべてゼロに設定される。

マスク要素では、これらの命令は常に再開ビットを使用して演算が実行され、その結果をゼロ拡張して書き込みレジスタマスク要素に書き込む。

```
    vmand.mm vd, vs2, vs1     # vd[i] =   vs2[i].LSB &&  vs1[i].LSB
    vmnand.mm vd, vs2, vs1    # vd[i] = !(vs2[i].LSB &&  vs1[i].LSB)
    vmandnot.mm vd, vs2, vs1  # vd[i] =   vs2[i].LSB && !vs1[i].LSB
    vmxor.mm  vd, vs2, vs1    # vd[i] =   vs2[i].LSB ^^  vs1[i].LSB
    vmor.mm  vd, vs2, vs1     # vd[i] =   vs2[i].LSB ||  vs1[i].LSB
    vmnor.mm  vd, vs2, vs1    # vd[i] = !(vs2[i[.LSB ||  vs1[i].LSB)
    vmornot.mm  vd, vs2, vs1  # vd[i] =   vs2[i].LSB || !vs1[i].LSB
    vmxnor.mm vd, vs2, vs1    # vd[i] = !(vs2[i].LSB ^^  vs1[i].LSB)
```

いくつかのアセンブラ疑似命令では、マスク論理演算を簡単に実行するための共通の命令が定義されている。

```
    vmcpy.m vd, vs  => vmand.mm vd, vs, vs  # マスクレジスタのコピー
    vmclr.m vd     => vmxor.mm vd, vd, vd   # マスクレジスタのクリア
    vmset.m vd     => vmxnor.mm vd, vd, vd  # マスクレジスタの設定
    vmnot.m vd, vs => vmnand.mm vd, vs, vs  # マスクレジスタの反転
```

> vmcpy.m命令はvmmvとは呼ばない。アーキテクチャ内では、mvという用語はビット全体を意味を持たずコピーするという意味を持つからである。vmcpy.m命令は書き込みマスクレジスタの上位ビットをクリアし、これらのビットの値にか買わざる、マスクレジスタの上位ビットをゼロに設定する。

8つのマスク論理命令は、2つの入力マスクによって16個のマスク関数を生成することができる:

| inputs |      |      |      |      |
| :----- | :--- | ---- | ---- | ---- |
| 0      | 0    | 1    | 1    | src1 |
| 0      | 1    | 0    | 1    | src2 |

| output |      |      |      | instruction                | 疑似命令         |
| :----- | :--- | :--- | ---- | -------------------------- | ---------------- |
| 0      | 0    | 0    | 0    | vmxor.mm vd, vd, vd        | vmclr.m vd       |
| 1      | 0    | 0    | 0    | vmnor.mm vd, src1, src2    |                  |
| 0      | 1    | 0    | 0    | vmandnot.mm vd, src2, src1 |                  |
| 1      | 1    | 0    | 0    | vmnand.mm vd, src1, src1   | vmnot.m vd, src1 |
| 0      | 0    | 1    | 0    | vmandnot.mm vd, src1, src2 |                  |
| 1      | 0    | 1    | 0    | vmnand.mm vd, src2, src2   | vmnot.m vd, src2 |
| 0      | 1    | 1    | 0    | vmxor.mm vd, src1, src2    |                  |
| 1      | 1    | 1    | 0    | vmnand.mm vd, src1, src2   |                  |
| 0      | 0    | 0    | 1    | vmand.mm vd, src1, src2    |                  |
| 1      | 0    | 0    | 1    | vmxnor.mm vd, src1, src2   |                  |
| 0      | 1    | 0    | 1    | vmand.mm vd, src2, src2    | vmcpy.m vd, src2 |
| 1      | 1    | 0    | 1    | vmornot.mm vd, src2, src1  |                  |
| 0      | 0    | 1    | 1    | vmand.mm vd, src1, src1    | vmcpy.m vd, src1 |
| 1      | 0    | 1    | 1    | vmornot.mm vd, src1, src2  |                  |
| 1      | 1    | 1    | 1    | vmxnor.mm vd, vd, vd       | vmset.m vd       |

> ベクトルマスク命令は後続のマスクベクトル操作と簡単に結合できるように設計されている。使用する前に、`v0`レジスタに値を移動することでプレディケートレジスタの数を拡張することができる。

### 16.2. ベクトルマスクPopulation Count命令` vpopc`

```
    vpopc.m rd, vs2, vm
```

ソースオペランドは単一のベクトルレジスタであり、マスクレジスタの値を[Mask Register Layout](https://riscv.github.io/documents/riscv-v-spec/#sec-mask-register-layout)の形式で格納している。

`vpopc.m`命令はベクトルソースマスクレジスタのアクティブな要素の中からマスク要素の数を数え(つまりマスク要素の再開ビットが設定されている要素の数を数え)、スカラレジスタ`x`に格納する。

演算はマスクの下で実行され、マスクされた要素のみがカウントの対象となる。

```
 vpopc.m rd, vs2, v0.t # x[rd] = sum_i ( vs2[i].LSB && v0[i].LSB )
```

`vpopc.m`命令の例外は常に`vstart`が0であるものとして通知される。`vpopc`命令は`vstart`が0でない場合に不正命令例外を発生させる。

### 16.3. `vfirst`先頭マスク設定ビットの探索命令

```
    vfirst.m rd, vs2, vm
```

`vfirst`命令はソースマスクベクトルの中から、アクティブ要素の中でLSBが1に設定されている最も小さな要素のインデックスを探し、そのインデックス値をGPRに格納する。もしLSBが1に設定されている要素が1つも存在しなければ、GPRには-1が格納される。

|      | Software can assume that any negative value (highest bit set) corresponds to no element found, as vector lengths will never exceed 231 on any implementation. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

> ソフトウェアは、`vfirst`命令の結果、整数レジスタの値が負の数である(再上位ビットが1に設定されている)ならば、該当する要素が見つからなかったと考えることができる。これは任意の実装でベクトル長が231を超えることがないという環境での想定である。

`vfirst`命令の例外は常に`vstart`が0であるものとして通知される。`vfirst`命令は`vstart`が0でない場合に不正命令例外を発生させる。

### 16.4. `vmsbf.m` 最初のマスク設定ビットよりも前のビットを設定する命令(set-before-first命令)

```
    vmsbf.m vd, vs2, vm

 # 例

     7 6 5 4 3 2 1 0   要素の番号

     1 0 0 1 0 1 0 0   v3 contents
                       vmsbf.m v2, v3
     0 0 0 0 0 0 1 1   v2 contents

     1 0 0 1 0 1 0 1   v3 contents
                       vmsbf.m v2, v3
     0 0 0 0 0 0 0 0   v2

     0 0 0 0 0 0 0 0   v3 contents
                       vmsbf.m v2, v3
     1 1 1 1 1 1 1 1   v2

     1 1 0 0 0 0 1 1   v0 vcontents
     1 0 0 1 0 1 0 0   v3 contents
                       vmsbf.m v2, v3, v0.t
     0 1 x x x x 1 1   v2 contents
```

`vmsbf.m`命令はマスクレジスタを入力ソースオペランドとして取り、マスクレジスタに結果を書き込む。この命令はソースベクトルレジスタの要素のうち最初にLSBに1が設定されている要素よりも前のすべての要素に対して1を書き込み、それ以降の要素についてはゼロを設定する。もしソースベクトルのうち1が設定されている要素が存在しなければ、すべてのアクティブな要素に対して1が書き込まれる。

書き込みマスクレジスタのテール要素はすべてクリアされる。

`vmsbf.m`命令の例外は常に`vstart`が0であるものとして通知される。`vmsbf`命令は`vstart`が0でない場合に不正命令例外を発生させる。

### 16.5. `vmsif.m` 最初のマスクビットを含むマスクビットセット命令(set-including-first命令)

最初のマスクビットを含むマスクビットセット命令は、set-before-firstと似ているが、マスクビットの最初に1が設定されているビットも1を設定するビットとして含めるところが異なる。

```
    vmsif.m vd, vs2, vm

 # 例

     7 6 5 4 3 2 1 0   要素の番号

     1 0 0 1 0 1 0 0   v3 contents
                       vmsif.m v2, v3
     0 0 0 0 0 1 1 1   v2 contents

     1 0 0 1 0 1 0 1   v3 contents
                       vmsif.m v2, v3
     0 0 0 0 0 0 0 1   v2

     1 1 0 0 0 0 1 1   v0 vcontents
     1 0 0 1 0 1 0 0   v3 contents
                       vmsif.m v2, v3, v0.t
     1 1 x x x x 1 1   v2 contents
```

空きこみマスクレジスタのテール意向の要素はすべてクリアされる。

`vmsif.m`命令の例外は常に`vstart`が0であるものとして通知される。`vmsif`命令は`vstart`が0でない場合に不正命令例外を発生させる。

### 16.6. `vmsof.m` 最初のマスクビットのみ設定する命令(set-only-first mask bit)

最初のマスクビットのみを設定する命令は、set-before-first命令と似ているが、マスクビットを設定されている最初の要素のみビットをセットするところが異なる。

```
    vmsof.m vd, vs2, vm

 # 例

     7 6 5 4 3 2 1 0   要素の番号

     1 0 0 1 0 1 0 0   v3 contents
                       vmsof.m v2, v3
     0 0 0 0 0 1 0 0   v2 contents

     1 0 0 1 0 1 0 1   v3 contents
                       vmsof.m v2, v3
     0 0 0 0 0 0 0 1   v2

     1 1 0 0 0 0 1 1   v0 vcontents
     1 1 0 1 0 1 0 0   v3 contents
                       vmsof.m v2, v3, v0.t
     0 1 x x x x 0 0   v2 contents
```

The tail elements in the destination mask register are cleared.

書き込みマスクレジスタのテール要素はすべて0にクリアされる。

`vmsof.m`命令の例外は常に`vstart`が0であるものとして通知される。`vmsof`命令は`vstart`が0でない場合に不正命令例外を発生させる。

### 16.7. ベクトルマスク命令を使用したサンプルプログラム

以下のサンプルプログラムはデータに依存して脱出を決めるループをベクトル化した例である。

```
  # char* strcpy(char *dst, const char* src)
strcpy:
      mv a2, a0             # 書き込み先アドレスをコピーする
loop:
    vsetvli x0, x0, e8      # 最大ベクトル長(バイト単位)を取得する
    vlbuff.v v1, (a1)       # コピー元データを取得する
      csrr t1, vl           # フェッチしたバイト数を取得する
    vmseq.vi v0, v1, 0      # フェッチしたバイト数に0(NULL)が含まれているか？
    vfirst.m a3, v0         # ゼロを見つけたか？
      add a1, a1, t1        # ポインタを進める。
    vmsif.m v0, v0          # ゼロバイトが含まれている要素よりも先のベクトル要素をマスクする。
    vsb.v v1, (a2), v0.t    # バイトをメモリに書き出す。
      add a2, a2, t1        # ポインタを進める
      bltz a3, loop         # 0を含む要素を見つけなければ、ループを続ける。

      ret

  # char* strncpy(char *dst, const char* src, size_t n)
strncpy:
      mv a3, a0             # 書き込み先アドレスをコピーする
loop:
    vsetvli x0, a2, e8      # 最大ベクトル長(バイト単位)を取得する。
    vlbuff.v v1, (a1)       # コピー元データを取得する
    vmseq.vi v0, v1, 0      # フェッチしたバイト数に0(NULL)が含まれているか？
    vfirst.m a4, v0         # ゼロを見つけたか？
    vmsif.m v0, v0          # ゼロバイトが含まれている要素よりも先のベクトル要素をマスクする。
    vsb.v v1, (a3), v0.t    # バイトをメモリに書き出す。
      bgez a4, exit         # 終了
      csrr t1, vl           # フェッチしたバイト数を取得する
      add a1, a1, t1        # ポインタを進める。
      sub a2, a2, t1        # カウンタをデクリメントする。
      add a3, a3, t1        # ポインタを進める。
      bnez a2, loop         # これ以上データがあるか？

exit:
      ret
```

### 16.8. ベクトルIota命令

`viota.m`命令はソースベクトルマスクレジスタを読み取り、すべてのベクトルレジスタに対して、そのベクトル要素よりも小さい要素の最下位ビットの総和を格納する命令である。つまり、マスク値の並列プレフィックス加算である。

この命令はマスク可能であり、有効な要素のみを加算し、有効な要素にのみ書き込みを実行できる。

```
 viota.m vd, vs2, vm

 # 例

     7 6 5 4 3 2 1 0   要素の番号

     1 0 0 1 0 0 0 1   v2 contents
                       viota.m v4, v2 # Unmasked
     2 2 2 1 1 1 1 0   v4 result

     1 1 1 0 1 0 1 1   v0 contents
     1 0 0 1 0 0 0 1   v2 contents
     2 3 4 5 6 7 8 9   v4 contents
                       viota.m v4, v2, v0.t # Masked
     1 1 1 5 1 7 1 0   v4 results
```

SEWの設定値が結果の値よりも大きい場合はこの結果の値はゼロ拡張される。もし結果の値が書き込みレジスタのSEWのビット数よりも大きくオーバフローした場合には、下位のSEWビットの値のみが保持される。

`viota.m`命令の例外が発生した場合はvstart`は0として通知される。。トラップハンドラから復帰しした後は、この命令は最初から再開される。

書き込み先のベクトルレジスタグループがソースベクトルマスクレジスタとオーバラップしていた場合に不正命令例外が発生する。命令がマスクされている場合は、書き込みレジスタグループが`v0`とオーバラップしている場合に不正命令例外が発生する。

> これらの制約は2つの理由で付けられている。1つ目は、一時的な長いベクトルレジスタと、ベクトルレジスタリネーミングを実行しない場合のWARハザードを簡単に避けるためのものである。2番目は、トラップ後の命令の実行再開を簡単にするためである。

`viota.m`命令はメモリスキャッタ命令(インデック付きストア命令)と併用して、ベクトル圧縮関数として使用できる。

```
    # 入力メモリの配列の中から非ゼロの値を拾って圧縮し、配列にストアする。
    #
    # size_t compact_non_zero(size_t n, const int* in, int* out)
    # {
    #   size_t i;
    #   size_t count = 0;
    #   int *p = out;
    #
    #   for (i=0; i<n; i++)
    #   {
    #       const int v = *in++;
    #       if (v != 0)
    #           *p++ = v;
    #   }
    #
    #   return (size_t) (p - out);
    # }
    #
    # a0 = n
    # a1 = &in
    # a2 = &out

compact_non_zero:
    li a6, 0                      # 非ゼロの要素の数をクリアする。
loop:
    vsetvli a5, a0, e32,m8        # 32ビット整数として処理する。
    vlw.v v8, (a1)                # 入力ベクトルをロードする。
      sub a0, a0, a5              # 入力した数だけ減算する。
      slli a5, a5, 2              # バイト単位に変換する。
    vmsne.vi v0, v8, 0            # 非ゼロの値を検査する。
      add a1, a1, a5              # 入力ポインタを進める。
    vpopc.m a5, v0                # v0に設定する要素の値をカウントする。
    viota.m v16, v0               # アクティブ要素の書き込みオフセットを計算する。
      add a6, a6, a5              # 要素の数を加算する。
    vsll.vi v16, v16, 2, v0.t     # オフセットをバイト単位に変換する。
      slli a5, a5, 2              # 非ゼロの要素の数をバイト単位に変換する。
    vsuxw.v v8, (a2), v16, v0.t   # バイト単位に拡大した結果を、マスクの下でスキャッタする。
      add a2, a2, a5              # 出力ポインタを進める。
      bnez a0, loop               # これ以上要素が存在するか？

      mv a0, a6                   # カウント値を返す。
      ret
```

### 16.9. ベクトル要素インデックス命令

`vid.v`命令は各要素のインデックスを書き込みベクトルレジスタグループに0から`vl`-1まで書き込む。

```
    vid.v vd, vm  # 書き込みレジスタに要素のIDを書き込む。
```

この命令はマスクを適用することができる。

この命令の`vs2`フィールドは`v0`に設定しなければならず、それ以外の場合はエンコーディングは予約となる。

SEWの値が結果の値よりも大きい場合には、値はゼロ拡張される。もし結果の値がSEWのビット幅よりもオーバフローした場合には、SEWビットの下位ビットが保持される。

>この制約は、リネーミングを行わない長いベクトルの実装においてWARハザードを避けるためと、命令の実行再開をサポートするためである。

> マイクロアーキテクチャとして、`vid.v`命令は`viota.m`命令と同じデータパスを使用して実装できるが、暗黙的にマスクのソースを指定する必要がある。

