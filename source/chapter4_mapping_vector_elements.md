## 4. ベクトル要素のベクトルレジスタへの割り付け

以下の図は、ELEN, VLENの実装と同様に、現在のSEWおよびLMULの設定に基づいてベクトルレジスタのバイトをどのようにして異なるサイズのベクトル要素にパックするかについて説明している。要素は、最下位のビットの中で最も重要度の低いバイトを持つ各ベクトルレジスタにパックされる。

Elements are packed into each vector register with the least-significant byte in the lowest-numbered bits.

> 以前のRISC-V Vector Proposal(<0.6)ではこのマッピングをソフトウェアから隠していたが、今回のProposalではすべての構成について具体的なマッピングについて説明している。これにより実装の柔軟性が失われるが、コンフィグレーションが変更されることによりゼロ埋めの必要性が除去される。マッピングを明示することにより、明示的なコンテキストの保存・回復コードを簡単化することができる。つまりコンフィグレーションを読み込まずにすべてのベクトルレジスタを保存するのではなく、`vl`,`vtype`を保存して、`vtype`を便利な値にリセットする(例えば、LMUL=8, SEW=ELENとする4つのベクトルグループ)ことができる。この逆の処理を行えば、ベクトルの状態を回復させることができる。

### 4.1 LMUL=1のマッピング

LMUL=1の場合、要素はベクトルレジスタに対して単純に最下位から最上位に向けてパッキングされる。

>可読性を向上させるために、ベクトルレジスタのレイアウトはバイトサイズで右から左にバイトアドレスを増加させる形式でく述する。エレメント内のビットはリトルエンディアンの形式で記述し、ビットインデックスは右から左へと増加していく。

```
要素のインデックは16進数で記述し、格納されている要素のうち最も下位の位置に記述している。ELEN<=128かつLMUL=1である。

 VLEN=32b

 Byte         3 2 1 0

 SEW=8b       3 2 1 0
 SEW=16b        1   0
 SEW=32b            0

 VLEN=64b

 Byte        7 6 5 4 3 2 1 0

 SEW=8b      7 6 5 4 3 2 1 0
 SEW=16b       3   2   1   0
 SEW=32b           1       0
 SEW=64b                   0

 VLEN=128b

 Byte        F E D C B A 9 8 7 6 5 4 3 2 1 0

 SEW=8b      F E D C B A 9 8 7 6 5 4 3 2 1 0
 SEW=16b       7   6   5   4   3   2   1   0
 SEW=32b           3       2       1       0
 SEW=64b                   1               0
 SEW=128b                                  0

 VLEN=256b

 Byte     1F1E1D1C1B1A19181716151413121110 F E D C B A 9 8 7 6 5 4 3 2 1 0

 SEW=8b   1F1E1D1C1B1A19181716151413121110 F E D C B A 9 8 7 6 5 4 3 2 1 0
 SEW=16b     F   E   D   C   B   A   9   8   7   6   5   4   3   2   1   0
 SEW=32b         7       6       5       4       3       2       1       0
 SEW=64b                 3               2               1               0
 SEW=128b                                1                               0
```

### 4.2. LMUL > 1のマッピング

ベクトルレジスタがグループ化されているとき、ベクトルレジスタグループの要素はベクトルレジスタでストライプして配置される。ストライプの距離はSLENビットで表現され、グループ内で1つのベクトルレジスタ内で何ビットが1つの要素としてパッキングされているかを示している。

例えば、SLEN=128のとき、ストライプのパタンは128ビットの倍数である。最初の128/SEW個の要素はグループ内で最初のベクトルレジスタのスタート位置から連続してパッキングされる。次の128/SEW要素は連続してグループ内で次のベクトルレジスタのスタート位置から連続してパッキングされる。最初のLMUL\*128/SEW個の要素をパッキングすると、次のLMUL\*128/SEW個の要素のグループは次の128ビットのセグメントとしてグループ内のベクトルレジスタでパッキングされる。これが続く。

```
 例1: VLEN=32b, SEW=16b, LMUL=2

 Byte         3 2 1 0
 v2*n           1   0
 v2*n+1         3   2

 例2: VLEN=64b, SEW=32b, LMUL=2

 Byte         7 6 5 4 3 2 1 0
 v2*n               1       0
 v2*n+1             3       2

 例3: VLEN=128b, SEW=32b, LMUL=2

 Byte        F E D C B A 9 8 7 6 5 4 3 2 1 0
 v2*n              3       2       1       0
 v2*n+1            7       6       5       4

 例4: VLEN=256b, SEW=32b, LMUL=2

 Byte     1F1E1D1C1B1A19181716151413121110 F E D C B A 9 8 7 6 5 4 3 2 1 0
 v2*n            B       A       9       8       3       2       1       0
 v2*n+1          F       E       D       C       7       6       5       4
```

SEW> SLENの場合、ストライピングパターンは、グループ内の次のベクトルレジスタに移動する前に、グループ内の各ベクトルレジスタに1つの要素を配置する。 したがって、LMUL=2の場合、偶数ベクトルレジスタにはベクトルの偶数要素が含まれ、奇数ベクトルレジスタにはベクトルの奇数要素が含まれます。

> ほとんどの実装では、SLEN≥ELENである。

```
 例: VLEN=256b, SEW=256b, LMUL=2

 Byte     1F1E1D1C1B1A19181716151413121110 F E D C B A 9 8 7 6 5 4 3 2 1 0
 v2*n                                                                    0
 v2*n+1                                                                  1
```

LMUL=4では、ベクトルレジスタは以下のように要素を格納する。

```
 例1: VLEN=32b, SLEN=32b, SEW=16b, LMUL=4,

 Byte         3 2 1 0
 v4*n           1   0
 v4*n+1         3   2
 v4*n+2         5   4
 v4*n+3         7   6

 例2: VLEN=64b, SLEN=64b, SEW=32b, LMUL=4

 Byte         7 6 5 4 3 2 1 0
 v4*n               1       0
 v4*n+1             3       2
 v4*n+2             5       4
 v4*n+3             7       6


 例3: VLEN=128b, SLEN=64b, SEW=32b, LMUL=4

 Byte          F E D C B A 9 8 7 6 5 4 3 2 1 0
 v4*n                9       8       1       0   32b elements
 v4*n+1              B       A       3       2
 v4*n+2              D       C       5       4
 v4*n+3              F       E       7       6

 例4: VLEN=128b, SLEN=128b, SEW=32b, LMUL=4

 Byte          F E D C B A 9 8 7 6 5 4 3 2 1 0
 v4*n                3       2       1       0   32b elements
 v4*n+1              7       6       5       4
 v4*n+2              B       A       9       8
 v4*n+3              F       E       D       C

 例5: VLEN=256b, SLEN=128b, SEW=32b, LMUL=4

 Byte     1F1E1D1C1B1A19181716151413121110 F E D C B A 9 8 7 6 5 4 3 2 1 0
 v4*n           13      12      11      10       3       2       1       0
 v4*n+1         17      16      15      14       7       6       5       4
 v4*n+2         1B      1A      19      18       B       A       9       8
 v4*n+3         1F      1E      1D      1C       F       E       D       C

 例6: VLEN=256b, SLEN=128b, SEW=256b, LMUL=4

 Byte     1F1E1D1C1B1A19181716151413121110 F E D C B A 9 8 7 6 5 4 3 2 1 0
 v4*n                                                                    0
 v4*n+1                                                                  1
 v4*n+2                                                                  2
 v4*n+3                                                                  3
```

似たようなパタンで、LMUL=8の場合である。

```
 例: VLEN=256b, SLEN=128b, SEW=32b, LMUL=8

 Byte   1F1E1D1C1B1A19181716151413121110 F E D C B A 9 8 7 6 5 4 3 2 1 0
 v8*n         23      22      21      20       3       2       1       0
 v8*n+1       27      26      25      24       7       6       5       4
 v8*n+2       2B      2A      29      28       B       A       9       8
 v8*n+3       2F      2E      2D      2C       F       E       D       C
 v8*n+4       33      32      31      30      13      12      11      10
 v8*n+5       37      36      35      34      17      16      15      14
 v8*n+6       3B      3A      39      38      1B      1A      19      18
 v8*n+7       3F      3E      3D      3C      1F      1E      1D      1C
```

アーキテクチャ上はさまざまなストライピングパターンが表示されるが、ストライピングパターンに関係なく同じ結果を生成するソフトウェアを作成できる。 主な制約は、ベクトルレジスタグループに保持されている値にアクセスするために使用されるLMULを変更しないことである(つまり、グループに値を書き込むために使用されるものとは異なるLMULで値を読み取らない)。

> 実装のストライピング長SLENは、幅が広いユニットのストライドメモリアクセスをベクターレジスタファイルの並列アクセスにコーナーターンするために必要な、幅の異なる操作のデータパス配線とバッファリングのトレードオフを最適化するように設定される。

> 以前の明示的なコンフィグレーションデザインでは、これらのトレードオフをマイクロアーキテクチャレベルで管理し、構成ごとに最適化することができた。

### 4.3. Mixed-Width演算へのマッピング

ベクトルレジスタグループ内の要素をマップするために使用されるパターンは、複数の要素幅にわたる操作をサポートするときにデータパスの配線を減らすように設計されている。 この場合の推奨されているソフトウェア戦略は、 `vtype`を動的に変更して、SEW / LMULを一定に(つまりVLMAXを一定に)維持することである。

次の例は、VLEN=256b / SLEN=128bの実装における4つの異なるパッキング要素の幅(8b,16b,32b,64b)を示している。 ベクトルレジスタグループ化係数(LMUL)は、各要素が同じ数のベクトル要素(この例では32)を保持できるように相対的な要素サイズによって増加し、ストリップマイニングコードを簡素化する。 同じインデックスを持つ要素間の操作は、データパスの同じ128b部分にあるオペランドビットにのみ影響する。

```
 VLEN=256b, SLEN=128b
 Byte     1F1E1D1C1B1A19181716151413121110 F E D C B A 9 8 7 6 5 4 3 2 1 0

 SEW=8b, LMUL=1, VLMAX=32

 v1       1F1E1D1C1B1A19181716151413121110 F E D C B A 9 8 7 6 5 4 3 2 1 0

 SEW=16b, LMUL=2, VLMAX=32

 v2*n       17  16  15  14  13  12  11  10   7   6   5   4   3   2   1   0
 v2*n+1     1F  1E  1D  1C  1B  1A  19  18   F   E   D   C   B   A   9   8

 SEW=32b, LMUL=4, VLMAX=32

 v4*n           13      12      11      10       3       2       1       0
 v4*n+1         17      16      15      14       7       6       5       4
 v4*n+2         1B      1A      19      18       B       A       9       8
 v4*n+3         1F      1E      1D      1C       F       E       D       C

 SEW=64b, LMUL=8, VLMAX=32

 v8*n                   11              10               1               0
 v8*n+1                 13              12               3               2
 v8*n+2                 15              14               5               4
 v8*n+3                 17              16               7               6
 v8*n+4                 19              18               9               8
 v8*n+5                 1B              1A               B               A
 v8*n+6                 1D              1C               D               C
 v8*n+7                 1F              1E               F               E
```

LMULに大きな値を設定することは、論理的なベクトルレジスタ長を削減しなければならないときに、単純にベクトル長を増加させ、命令フェッチ幅を削減し、ディスパッチのオーバヘッドを削減するために役に立つ。

以下の表は、混合幅動作について、可能なSEW/LMULの動作ポイントについて示している。

```
       横軸はLMULの値を示し、各軸はSEW/MULの動作ポイントを示している。

 SEW/LMUL    1   2   4   8  16  32  64 128 256 512 1024

      SEW
        8    8   4   2   1
       16        8   4   2   1
       32            8   4   2   1
       64                8   4   2   1
      128                    8   4   2   1
      256                        8   4   2   1
      512                            8   4   2   1
     1024                                8   4   2   1
```

>SLENが空間データパス幅よりも小さい場合、LMUL値を大きくすると、短いベクトルのデータパス使用率が低くなる。 VLEN=256b、SLEN=128b、およびLMUL=8を使用した上記の例では、実装が256b幅のベクトルデータパスを持つ純粋な空間である場合、アプリケーションベクトル長が17未満の場合、データパスの半分のみがアクティブになる。 以下の「vsetvl」命令には、必要なアプリケーションベクトル長(AVL)と要素幅の範囲に従って適切なLMULを動的に選択する機能を追加できる。

> 狭いマシンでは、SLENが少なくともデータパスの空間幅と同じ大きさに設定されるため、LMULを減らす必要ない。 幅の広いマシンでは、SLENを空間データパスの幅よりも小さく設定して、幅が混在する操作(例えば、width=1024、ELEN=32、SLEN=128)の配線を減らすことができる。

### 4.4. マスクレジスタのレイアウト

ベクトルのマスクは、SEWとLMULの値に関係なく1つのベクトルレジスタにのみ適用される。各ベクトル操作に使用されるマスクビットは現在のSEWとLMUL設定に依存する。

ベクトルオペランドの最大エレメント長は以下になる。

```
               VLMAX = LMUL * VLEN/SEW
```

マスクは、マスクレジスタをVLEN/LVMAXフィールドで分割することにより各エレメントに割り当てられる。各マスクエレメントのサイズ**MLEN**は以下で計算できる。

```
                MLEN = VLEN/VLMAX
                     = VLEN/(LMUL * VLEN/SEW)
                     = SEW/LMUL
```

MLENはELEN(SEW=ELEN, LMUL=1)から1(SEW=8b, LMUL=8)までの値を取ることができ、したがって単一のベクトルレジスタは常に全体のマスクレジスタを保持することができる。

マスクビットの要素*i*はマスクレジスタのビット[MLEN*i+(MLEN-1):MLEN*i]である。比較命令によりマスクの要素に書き込みが行われた場合、マスクの要素の再開ビットに比較結果が書き込まれ、上位ビットはゼロが設定される。その値がマスクとして読みだされた場合、マスク要素の再開ビットのみがコントロースマスクとして使用され、上位のビットは無視される。現在のベクトル長を超えるマスク要素はゼロとなる。

パターンは、一定のSEW/LMUL値の場合、有効なプレディケートビットがマスクベクトルレジスタの同じビットに配置されるため、幅の異なる要素を含むループでのマスキングの使用が簡単になる。

```
VLEN=32b

          Byte    3   2   1   0
 LMUL=1,SEW=8b
                  3   2   1   0  Element
                [24][16][08][00] Mask bit position in decimal

 LMUL=2,SEW=16b
                      1       0
                    [08]    [00]
                      3       2
                    [24]    [16]

 LMUL=4,SEW=32b               0
                            [00]
                              1
                            [08]
                              2
                            [16]
                              3
                            [24]
```

```
 LMUL=2,SEW=8b
                  3   2   1   0
                [12][08][04][00]
                  7   6   5   4
                [28][24][20][16]

 LMUL=8,SEW=32b
                              0
                            [00]
                              1
                            [04]
                              2
                            [08]
                              3
                            [12]
                              4
                            [16]
                              5
                            [20]
                              6
                            [24]
                              7
                            [28]

 LMUL=8,SEW=8b
                  3   2   1   0
                [03][02][01][00]
                  7   6   5   4
                [07][06][05][04]
                  B   A   9   8
                [11][10][09][08]
                  F   E   D   C
                [15][14][13][12]
                 13  12  11  10
                [19][18][17][16]
                 17  16  15  14
                [23][22][21][20]
                 1B  1A  19  18
                [27][26][25][24]
                 1F  1E  1D  1C
                [31][30][29][28]
```

```
 VLEN=256b, SLEN=128b
 Byte     1F1E1D1C1B1A19181716151413121110 F E D C B A 9 8 7 6 5 4 3 2 1 0

 SEW=8b, LMUL=1, VLMAX=32

 v1       1F1E1D1C1B1A19181716151413121110 F E D C B A 9 8 7 6 5 4 3 2 1 0
        [248]          ...            [128] ...[96] ...[64] ...[32] ... [0] Mask bit positions in decimal

 SEW=16b, LMUL=2, VLMAX=32

 v2*n       17  16  15  14  13  12  11  10   7   6   5   4   3   2   1   0
          [184]          ...          [128]    ...     [32]    ...      [0]
 v2*n+1     1F  1E  1D  1C  1B  1A  19  18   F   E   D   C   B   A   9   8
          [248]          ...          [196]    ...     [96]    ...     [64]

 SEW=32b, LMUL=4, VLMAX=32

 v4*n           13      12      11      10       3       2       1       0
              [152]        ...        [128]    [24]        ...          [0]
 v4*n+1         17      16      15      14       7       6       5       4
              [184]        ...        [160]    [56]        ...         [32]
 v4*n+2         1B      1A      19      18       B       A       9       8
              [116]        ...        [192]    [88]        ...         [64]
 v4*n+3         1F      1E      1D      1C       F       E       D       C
              [248]        ...        [224]   [120]        ...         [96]

 SEW=64b, LMUL=8, VLMAX=32

 v8*n                   11              10               1               0
                      [136]           [128]             [8]             [0]
 v8*n+1                 13              12               3               2
                      [152]           [144]            [24]            [16]
 v8*n+2                 15              14               5               4
                      [168]           [160]            [40]            [32]
 v8*n+3                 17              16               7               6
                      [184]           [176]            [56]            [48]
 v8*n+4                 19              18               9               8
                      [200]           [192]            [72]            [64]
 v8*n+5                 1B              1A               B               A
                      [216]           [208]            [88]            [80]
 v8*n+6                 1D              1C               D               C
                      [232]           [224]           [104]            [96]
 v8*n+7                 1F              1E               F               E
                      [248]           [240]           [120]           [112]
```

## 5. ベクトル命令フォーマット

ベクトル拡張命令は既存の4つの命令オペコード(LOAD-FP, STORE-FP, AMO)および新しいメジャーオペコード(OP-V)を使用している。

ベクトルロードとストア命令は浮動小数点巣からロードストアのメジャーオペコード(LOAD-FP/STORE-FP)でエンコーディングされている。ベクトルロードとストアは標準的なスカラの浮動小数点ロードストア命令の12ビットの即値フィールドを使って、より多くの命令エンコーディングを提供している。ビット25は標準的なベクトルマスクビットを示している(マスクエンコーディングの章を参照のこと)。

```
LOAD-FPメジャーオペコードを使用したベクトルロード命令のフォーマット
31 29 28 26  25  24      20 19       15 14   12 11      7 6     0
 nf  | mop | vm |  lumop   |    rs1    | width |    vd   |0000111| VL*  unit-stride
 nf  | mop | vm |   rs2    |    rs1    | width |    vd   |0000111| VLS* strided
 nf  | mop | vm |   vs2    |    rs1    | width |    vd   |0000111| VLX* indexed
  3     3     1      5           5         3         5       7

STORE-FPメジャーオペコードを使用を使用したベクトルストア命令のフォーマット
31 29 28 26  25  24      20 19       15 14   12 11      7 6     0
 nf  | mop | vm |  sumop   |    rs1    | width |   vs3   |0100111| VS*  unit-stride
 nf  | mop | vm |   rs2    |    rs1    | width |   vs3   |0100111| VSS* strided
 nf  | mop | vm |   vs2    |    rs1    | width |   vs3   |0100111| VSX* indexed
  3     3     1      5           5         3         5        7
```

```
AMOメジャーオペコードを使用したベクトルAMO命令のフォーマット
31    27 26  25  24      20 19       15 14   12 11      7 6     0
 amoop  |wd| vm |   vs2    |    rs1    | width | vs3/vd  |0101111| VAMO*
   5      1   1      5           5         3        5        7
```

```
OP-Vメジャーオペコードを使用したベクトル算術演算のフォーマット

31       26  25   24      20 19      15 14   12 11      7 6     0
  funct6   | vm  |   vs2    |    vs1   | 0 0 0 |    vd   |1010111| OP-V (OPIVV)
  funct6   | vm  |   vs2    |    vs1   | 0 0 1 |  vd/rd  |1010111| OP-V (OPFVV)
  funct6   | vm  |   vs2    |    vs1   | 0 1 0 |  vd/rd  |1010111| OP-V (OPMVV)
  funct6   | vm  |   vs2    |   simm5  | 0 1 1 |    vd   |1010111| OP-V (OPIVI)
  funct6   | vm  |   vs2    |    rs1   | 1 0 0 |    vd   |1010111| OP-V (OPIVX)
  funct6   | vm  |   vs2    |    rs1   | 1 0 1 |    vd   |1010111| OP-V (OPFVF)
  funct6   | vm  |   vs2    |    rs1   | 1 1 0 |  vd/rd  |1010111| OP-V (OPMVX)
     6        1        5          5        3        5        7
```

```
OP-Vメジャーオペコードを使用したベクトルコンフィグレーションのフォーマット

 31 30         25 24      20 19      15 14   12 11      7 6     0
 0 |        zimm[10:0]      |    rs1   | 1 1 1 |    rd   |1010111| vsetvli
 1 |   000000    |   rs2    |    rs1   | 1 1 1 |    rd   |1010111| vsetvl
 1        6            5          5        3        5        7
```

ベクトル命令はスカラオペランドとベクトルソースオペランドを持つことができ、スカラとベクトルの両方の結果を出力することができる。ほとんどのベクトル命令は無条件で動作するか、マスクに基づいて条件的に動作する。

ベクトルロードストア命令はレジスタ要素とメモリの間でビットパタンを移動する。ベクトル算術演算はベクトルレジスタ要素の値を演算する。

### 5.1. スカラオペランド

スカラオペランドは、即値、`X`整数レジスタ、`f`浮動小数点レジスタ、ベクトルの要素0レジスタを取ることができる。スカラの結果は`X`, `f, もしくはベクトルレジスタの要素0番目に書き込まれる。LMULのセッティングに関係なく、どのようなベクトルレジスタもスカラの値を保持することができる。

> 0.6からの仕様変更として、浮動小数点レジスタはベクトルレジスタとオーバレイしなくなった点と、スカラの要素嘘として整数と浮動小数点レジスタを取ることができるということが挙げられる。`f`レジスタとのオーバレイの仕様を削除したことにより、ベクトルレジスタのレジスタプレッシャの問題を削減し、標準的な呼び出し規約での相互干渉を避けることができる。また、高性能スカラ浮動小数点演算命令において実装を簡単化する。また、Zfinx ISAとの互換性をサポートする。`f`レジスタを`v`レジスタとオーバレイすることのメリットはいくつかの実装で状態ビットの数を削減することができるが、高性能な設計では実装が複雑化し、Zfinxオプションでの処理が複雑化する。

### 5.2. ベクトルオペランド

ベクトルオペランド・ベクトル演算結果はLMULに基づいて1つ以上のベクトルレジスタを使用するが、常にグループ内の最小ベクトルレジスタアドレスを使用する。最小ベクトルレジスタアドレス以外を使用すると、不定命令例外を発生する。

いくつかのベクトル命令はより幅の長い要素を読み込むか書き込みを行い、`vlmul`で指定されたレジスタグループよりもより多くのベクトルレジスタグループを使用することがある。最大のベクトルレジスタグループは8を超える事は無く、もしベクトル命令が8よりも大きなレジスタグループを必要とするならば、その命令は不定命令例外が発生する。例えば、LMUL=8でより幅の広いレジスタを使用する命令を実行した場合、不定命令例外が発生する。

### 5.3 ベクトルマスク

多くのベクトル命令ではマスクイングをサポートしている。マスクされた要素に対する演算は、書き込みレジスタ要素の値を更新せず、例外も発生しない。

ベースレジスタ拡張では、マスクの値はマスク付きベクトル命令によって使用され、マスクの値は常にベクトルレジスタ`v0`から与えられる。各要素の再開ビットが制御命令のマスクベクタとして使用される。

> 将来のベクトル拡張では、マスクレジスタとしてすべての空間を利用できるような長い命令エンコーディングを提供する予定である。

マスク付きベクトル命令の書き込みレジスタグループは、LMUL=1の時のみおーばラップすることができる。それ以外の場合は不定命令例外が発生する。

> この制約では、非ゼロの`vstart`値による再開がサポートされる。

他のベクトルレジスタは計算中のマスクを保持することができ、プレディケートの計算のためにベクトル論理演算などを適用できる。

#### 5.3.1. マスクのエンコーディング

マスクが使用可能な場合、命令中の`inst[25]`に配置されている`vm`1ビットとしてエンコードされる。

| vm   | 説明                                    |
| :--- | :-------------------------------------- |
| 0    | vector result, only where v0[i].LSB = 1 |
| 1    | unmasked                                |

> 初期の仕様では、`vm`は2ビットであり`vm[1:0]`とし、スカラ命令と同様に`v0`を使用して双方とも1もしくは排他的なマスクをサポートしていた。

ベクトルのマスクはアセンブラコードでは追加のベクトルオペランドとして表現される。`.t`は`v0[i].LSB`が1である場合に演算が発生する。マスクが指定されていないと、マスク無しベクトル命令(`vm=1`)として演算が実行される。

```
    vop.v*    v1, v2, v3, v0.t  # enabled where v0[i].LSB=1, m=0
    vop.v*    v1, v2, v3        # unmasked vector operation, m=1
```

> `v0`によるTrueの形のプレディケーションをベクトルマスクレジスタをサポートしているにも関わらず、アセンブリの構文ではマスクレジスタの指定としてすべてのベクトルレジスタを指定でき、Both Trueと排他的なマスクをサポートできるように設計している。`.t`はマスクの仕様を視覚的にエンコードすることを助ける。

### 5.4 プリスタート・アクティブ・インアクティブ・ボディー・テールの要素定義

ベクトル命令を実行中に、演算が適用される要素は以下の4つの別々の状態に分類される。

- **プリスタート(prestart)** 要素は、その要素のインデックが`vstart`に格納されている初期値よりも小さいものを指す。プリスタート状態の要素は例外を発生させず、書き込みレジスタに指定されても要素のアップデートは行われない。
- **アクティブ(active)**要素は現在のベクトル長の設定に含まれているベクトル要素であり、現在のマスク設定により有効化されている要素である。アクティブな要素は例外を発生する可能性があり、書き込みベクトルレジスタグループにより書き込みが発生する可能性がある。
- **インアクティブ(inactive)**な要素は現在のベクトル長の設定に含まれているベクトル要素であるが、現在のマスクそっていにより有効化されていないベクトル要素である。インアクティブな要素は例外を発生させず、書き込みレジスタに指定されても書き込みが行われない。
- **テール(tail)**要素はベクトル命令の実行中に、現在のベクトル長の設定よりも後ろのベクトル要素である。テールのベクトル要素は例外を発生させないが、ベクトルレジスタグループにおいて書き込み時はゼロが書き込まれる。
- 加えて、他の要素として**ボディー(body)**要素は、アクティブ要素とインアクティブ要素の両方を示したものである。つまり、プリスタート以降で、テール以前の要素を指す。  

```
    ベクトル要素のインデックス xについて、
    プリスタート     = (0 <= x < vstart)
    mask(x)        = unmasked || v0[x].LSB == 1
    アクティブ(x)    = (vstart <= x < vl) && mask(x)
    インアクティブ(x) = (vstart <= x < vl) && !mask(x)
    ボディー(x)      = active(x) || inactive(x)
    テール(x)        = (vl <= x < VLMAX)
```

All regular vector instructions place zeros in the tail elements of the destination vector register group. Some vector arithmetic instructions are not maskable, so have no inactive elements, but still zero the tail elements.

すべての通常ベクトル命令は、書き込みレジスタのテール要素に対してゼロを書き込む。いくつかのベクトル算術演算はマスク不可能であり、したがってインアクティブな要素は存在しないが、テール要素についてはゼロを書き込む。

> インアクティブとテール要素のアップデートルールは、実装の要求とベクトルレジスタのECCの問題およびリネーミングに関する問題の妥協案として設計されている。

> `vl`以降のレジスタをゼロ埋めしないと、リネーミングを行う実装において悪影響があり、すべての命令においてVL以降のすべての要素をコピーする必要がある。一方でリネーミングを行わない実装ではテールのゼロ化を行うことのペナルティは小さい。リネーミングを行う実装では、リネーミングを行わないことですべてのベクトルレジスタのコピーを避けることができ、各レジスタの演算が、任意のベクトル長に対してすべての帯域を占有してしまうだけに十分深くなってしまうことが問題である。`vl`以降の値をすべてゼロで埋めてしまうことにより、ほとんどのソフトウェアの影響を削減するが、リダクション捜査の場合にも若干のコストが発生する。

> For zeroing tail updates, implementations with temporally long vector registers, either with or without register renaming, will be motivated to add microarchitectural state to avoid actually writing zeros to all tail elements, but this is a relatively simple microarchitectural optimization. For example, one bit per element group or a quantized VL can be used to track the extent of zeroing. An element group is the set of elements comprising the smallest atomic unit of execution in the microarchitecture (often equivalent to the width of the physical datapath in the machine). The microarchitectural state for an element group indicates that zero should be returned for the element group on a read, and that zero should be substituted in for any masked-off elements in the group on the first write to that element group (after which the element group zero bit can be cleared).
>
> テール意向をゼロに埋めるために、一時的に長いベクトルレジスタを使用する実装では、レジスタのリネーミングの有無にかかわらず、実際にゼロを書き込む操作を割くるために新たなマイクロアーキテクチャの状態を追加することも考えられるが、これは比較的単純なマイクロアーキテクチャの最適化である。例えば、要素グループ毎に1ビット、もしくは量子化されたVLによりゼロ埋めを行う範囲を追跡することができる。要素グループは、マイクロアーキテクチャの実行の最小アトミックユニット単位(多くの場合、マシンの物理データパスのサイズと同じ)を構成する要素のセットとなる。要素グループのマイクロアーキテクチャの状態は、読み出し時には要素グループにゼロが返され、マスクされた要素に対する書き込みは、最初の書き込み時にグループ内のマスクされた要素にゼロを代入されることを示す(その後、要素グループのゼロビットはクリアできる)。

>Providing merging predication instead of zeroing inactive elements on a masked operation reduces code path length for many code blocks, and reduces register pressure by allowing different code paths to use disjoint sets of elements in the same vector register. Implementations with vector register ECC or renaming will have to perform read-update-write on the destination register value to preserve inactive elements on arithmetic instructions, so would appear to need an extra vector register read port. However, the arithmetic instructions are designed such that the largest read-port requirement is for fused multiply-add instructions that are destructive and overwrite one source, and hence do not need an extra read port to preserve inactive elements. Given that linear algebra is one of the more important applications for vector units, and that fused multiply-add is the dominant operation in linear algebra routines, microarchitectures will be optimized for fused multiply-add operations and so should be able to preserve inactive elements on other arithmetic operations without large additional cost. However, masked vector load instructions incur the cost of an additional read port on their destination register. The need to support resumable vector loads with non-zero `vstart` values also drives the need to preserve vector load destination register values. The AMOs have been defined to be destructive in their source operand to reduce the maximum read port requirement for the memory pipe. An option that was considered was to have loads behave differently from arithmetic instructions and to zero any masked-off elements. However, this would require additional instructions and increase register pressure, and vector loads must in any case still cope with non-zero `vstart` values through some mechanism.

> マスクされたインアクティブな要素をゼロ埋めする方法の代わりに、マージプレディケートを実装することにより、多くのコードブロックのコードパスの長さが短くなり、異なるコードパスが同じベクトルレジスタ内の要素の異なる要素の集合を使用できるため、レジスタプレッシを軽減することができる。ベクトルレジスタのECC、もしくはリネーミングを使用する実装では、算術命令のインアクティブな要素を保持するために、読み取り/更新/書き込みを実行する必要あり、ベクトルレジスタの読み取りポートを増やす必要がある。ただし、算術命令は、最大の読み取りポート数が1つのソースを上書きするFused積和命令であるように設計されているため、インアクティブな要素を保持するための追加の読み取りポートは必要ない。線形代数はベクトル単位のより重要なアプリケーションの1つであり、線形積和が線形代数ルーチンの主要な演算であるため、マイクロアーキテクチャはFused積和演算に対して最適化され、インアクティブな要素を保持できるはずである。大きな追加コストなしで他の算術演算も実装できる。ただし、マスクされたベクトルロード命令では、デスティネーションレジスタに追加の読み取りポートのコストが発生する。ゼロ以外の`vstart`値でリジューム可能なベクトルロードをサポートする必要性があり、ベクトルロードデスティネーションレジスタ値を保持する必要性も発生する。 AMOは、メモリパイプの最大リードポートの数を減らすために、ソースオペランドを上書きするように設計されている。ほかの方法として考えられたものは、算術命令とは異なる動作をロードに与え、マスクされた要素をゼロにすることであった。しかしこの方法では追加の命令が必要であり、レジスターのプレッシャーが増加する。ベクタロードは、いずれの場合でも、何らかのメカニズムを介してゼロ以外の`vstart`値に対処する必要がある。







