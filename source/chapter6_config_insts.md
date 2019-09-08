## 6. コンフィグレーション設定命令

アプリケーションの要求に応じて`vl`と`vtype`を簡単に設定できるような命令群が定義されている。

### 6.1. `vsetvli`/`vsetvl` 命令

```
 vsetvli rd, rs1, vtypei # rd = 新しいvlの値, rs1 = AVL, vtypei = 新しいvtypeの値
                         # rs1 = x0の場合、最大のベクトル長が使用される。
 vsetvl  rd, rs1, rs2    # rd = 新しいvlの値, rs1 = AVL, rs2 = 新しいvtypeの値
                         # rs1 = x0の場合、最大のベクトル長が使用される。
```

The `vsetvli` instruction sets the `vtype` and `vl` CSRs based on its arguments, and writes the new value of `vl` into `rd`.

`vsetvli`命令は`vtype`と`vl`のCSRレジスタをその引数に応じて設定し、新しい`vl`の値を`rd`に書き込む。

The new `vtype` setting is encoded in the immediate fields of `vsetvli` and in the `rs2` register for `vsetvl`.

`vtype`の新しい設定値は`vsetvli`のフィールドの即値として設定され、`vsetvl`命令の場合は`rs2`レジスタで指定される。

```
OP-Vメジャーオペコードを使用したベクトルコンフィグレーション命令のフォーマット。

 31 30         25 24      20 19      15 14   12 11      7 6     0
 0 |        zimm[10:0]      |    rs1   | 1 1 1 |    rd   |1010111| vsetvli
 1 |   000000    |   rs2    |    rs1   | 1 1 1 |    rd   |1010111| vsetvl
 1        6            5          5        3        5        7
```

| ビット   | 名前       | 説明                                           |
| -------- | :--------- | :--------------------------------------------- |
| XLEN-1   | vill       | 不正な値が設定されたことを示す。               |
| XLEN-2:7 |            | 予約領域(0が書き込まれる)                      |
| 6:5      | vediv[1:0] | EDIV拡張に使用される。                         |
| 4:2      | vsew[2:0]  | Standard element width (SEW) の設定。          |
| 1:0      | vlmul[1:0] | Vector register group multiplier (LMUL) の設定 |

```
 vtypei設定に使用される値のアセンブラで使用される名前。
 
 e8    #   8b elements
 e16   #  16b elements
 e32   #  32b elements
 e64   #  64b elements
 e128  # 128b elements

 m1   # Vlmul x1, mの指定がない場合はこの値が使用される。
 m2   # Vlmul x2
 m4   # Vlmul x4
 m8   # Vlmul x8

 d1   # EDIV 1, dの指定がない場合はこの値が使用される。
 d2   # EDIV 2
 d4   # EDIV 4
 d8   # EDIV 8

例:
    vsetvli t0, a0, e8          # SEW= 8, LMUL=1, EDIV=1
    vsetvli t0, a0, e8,m2       # SEW= 8, LMUL=2, EDIV=1
    vsetvli t0, a0, e32,m2,d4   # SEW=32, LMUL=2, EDIV=4
```

`vtype`の設定値が実装によってサポートされていない場合、`vtype`レジスタ内の`vill`ビットが設定され、それ以外の`vtype`レジスタのビットフィールドはゼロが設定され、`vl`レジスタの値もゼロに設定される。

アプリケーションにより必要となるベクトル長(Application Vector Length : AVL)は`rs1`レジスタ内に、符号なし整数として設定される。`rs1`のレジスタ指定として`x0`を設定することで、無限長の`AVL`を指定することになり、これは実装可能なベクトル長の最大値を設定したことになる。

> `vsetvl{i}`において、AVL=`x0`を指定することで現在のVLMAX値を取得することができる。この命令では、`vl`の変更は発生しない。

> `vsetvl{i}`の動きは`rd`=`x0`であるかに依存しない。`rd`=`x0`では、アプリケーションがすでに`vl`の値がなんであるかを仮定している場合に使用できる。例えば、SEWとLMULをAVLの定数に置き換える場合に使用できる。

> 初期の仕様では`vtype`に不正な値を設定すると例外が発生する仕様になっていた。ISAへのCSR書き込みに最初にデータ依存の例外を追加することになる。現在のスキームでは、特定の設定に対して`vill`がクリアされているかどうかを確認することによってベクトルユニットの設定の問い合わせを軽量化する手法をサポートしている。

### 6.2.  `vl`への設定条件

The `vsetvl{i}` instructions first set VLMAX according to the `vtype` argument, then set `vl` obeying the following constraints:

`vsetvl{i}`命令は、最初に引数の`vtype`に基づいてVLMAXを設定し、以下が成り立つ。

1. `AVL ≤ VLMAX`である場合、`vl = AVL`である。
2. `AVL < (2 * VLMAX)`である場合、`vl ≥ ceil(AVL / 2)` を設定する。
3. `AVL ≥ (2 * VLMAX)`である場合、`vl = VLMAX`を設定する。
4. AVLとVLMAXの値が同じならば、どのような実装でも同じ値が書き込まれる。
5. 以下の制約は、上記の設定に基づいて常に成立する。
   1. `AVL = 0`の場合`vl = 0`が設定される。
   2. `AVL > 0`の場合、`vl > 0`である。
   3. `vl ≤ VLMAX`
   4. `vl ≤ AVL`
   5. `vsetvl {i}`のAVL引数として使用されたときに `vl`から読み取られた値は、結果のVLMAXが` vl`が読み取られた時点のVLMAXの値に等しい場合、 `vl`で同じ値になる。

> `vl`の設定制約は、`AVL ≤ VLMAX`のレジスタスピルとコンテキストスワップ全体で`vl`の動作を維持するのに十分に厳格でありながら、`AVL >  VLMAX`のベクターレーン使用率を実装できるように十分に柔軟に設計されている。例えば、これにより、実装が`VLMAX < AVL < 2 * VLMAX`の場合に`vl = ceil（AVL / 2）`を設定して、ストリップマインループの最後の2回の繰り返しで作業を均等に分散できる。 要件2は、リダクションループの最初のストリップマイン反復が、`AVL < 2 * VLMAX`の場合でも、すべての反復の最大ベクトル長を使用することを保証する。 これにより、ソフトウェアはストリップループ中に観測されたベクトル長の最大値を明示的に計算する必要がなくなる。

### 6.3. `vsetvl`命令

`vsetvl`命令は`vsetvli`命令と同様な動作をするが、`vtype`の設定が`rs2`レジスタの内容に基づいて設定されることが異なる。コンテキストのリストアに使用さたり、`vtypei`の即値フィールドが所望の設定に対して小さすぎる場合に使用される。

> いくつかのアクティブな複雑な型では、`x`レジスタに異なる値を設定する必要があり、必要に応じて`vsetvl`命令を使用してスワップする必要がある。

### 6.4. 例

SEWとLMULの設定値は、単一ループ中で動的に切り替えることが可能で、これによりデータ型の幅を混合させても高いスループットを達成することが可能となる。

```
# Example: 16ビットの値をロードし、32ビットの乗算を行い、その結果を3ビット右シフトし、32ビット値をストアする。

# 最大幅のエレメントのみ使用するループ。

loop:
    vsetvli a3, a0, e32,m8  # 32-bitの要素のみ使用する。
    vlh.v v8, (a1)          # 16ビットの符号付ロード値を32ビットのレジスタ要素に格納する。
      sll t1, a3, 1         # 2バイト分要素の幅を計算する。
      add a1, a1, t1        # ポインタを進める。
    vmul.vx  v8, v8, x10    # 32bの乗算結果を得る。
    vsrl.vi  v8, v8, 3      # 要素をシフトする。
    vsw.v v8, (a2)          # 32ビットの値をストアする。
      sll t1, a3, 2         # 4バイト分要素の幅を計算する。
      add a2, a2, t1        # ポインタを進める。
      sub a0, a0, a3        # カウンタをデクリメントする。
      bnez a0, loop         # 要素が残っているか？

# 要素の幅をスイッチするタイプのループ

loop:
    vsetvli a3, a0, e16,m4  # `vtype`を16ビットの整数に設定する。
    vlh.v v4, (a1)          # 16ビットのデータをロードする。
      slli t1, a3, 1        # 2バイト分要素の幅を計算する。
      add a1, a1, t1        # ポインタを進める。
    vwmul.vx v8, v4, x10    # v8-v15を使って、32ビットの乗算を実行する。
    
    vsetvli x0, a0, e32,m8  # 32ビットのデータ幅に変更する。
    vsrl.vi v8, v8, 3
    vsw.v v8, (a2)          # 32ビットの値をストアする。
      slli t1, a3, 2        # 4バイト分要素の幅を計算する。
      add a2, a2, t1        # ポインタを進める。
      sub a0, a0, a3        # カウンタをデクリメントする。
      bnez a0, loop         # 要素が残っているか？
```

2番目のループの方がより複雑であるが、16ビット幅の乗算命令を使用することで32ビットの整数乗算よりも高速で、高い性能を得ることができる。また、ベクトルレジスタへの書き込み幅が小さいことで、16ビットのベクトルロードの方が高速である。
