## 12. ベクトル整数算術演算命令

ベクトル整数演算命令について説明する。

### 12.1. 単一ビット幅の整数加算と減算命令

ベクトル整数加算命令と減算命令について説明する。逆減算命令についても、ベクトル - スカラの形式で定義されている。

```
# 整数加算.
vadd.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
vadd.vx vd, vs2, rs1, vm   # ベクトル - スカラ
vadd.vi vd, vs2, imm, vm   # ベクトル - 即値

# 整数減算
vsub.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
vsub.vx vd, vs2, rs1, vm   # ベクトル - スカラ

# 整数逆減算
vrsub.vx vd, vs2, rs1, vm   # vd[i] = rs1 - vs2[i]
vrsub.vi vd, vs2, imm, vm   # vd[i] = imm - vs2[i]
```

### 12.2. ベクトルビット幅拡張整数加算・減算命令

ビット幅を拡張する整数加算・減算命令は、符号付きと符号なしのバリエーションが存在し、元のビット幅のソースオペランドを最初に符号付かゼロ拡張することで倍のビット幅で演算を行う。

```
# ビット幅拡張 符号なし整数加減算命令、 2*SEW = SEW +/- SEW
vwaddu.vv  vd, vs2, vs1, vm  # ベクトル - ベクトル
vwaddu.vx  vd, vs2, rs1, vm  # ベクトル - スカラ
vwsubu.vv  vd, vs2, vs1, vm  # ベクトル - ベクトル
vwsubu.vx  vd, vs2, rs1, vm  # ベクトル - スカラ

# ビット幅拡張 符号あり整数加減算命令 2*SEW = SEW +/- SEW
vwadd.vv  vd, vs2, vs1, vm  # ベクトル - ベクトル
vwadd.vx  vd, vs2, rs1, vm  # ベクトル - スカラ
vwsub.vv  vd, vs2, vs1, vm  # ベクトル - ベクトル
vwsub.vx  vd, vs2, rs1, vm  # ベクトル - スカラ

# ビット幅拡張 符号なし整数加減算命令 2*SEW = 2*SEW +/- SEW
vwaddu.wv  vd, vs2, vs1, vm  # ベクトル - ベクトル
vwaddu.wx  vd, vs2, rs1, vm  # ベクトル - スカラ
vwsubu.wv  vd, vs2, vs1, vm  # ベクトル - ベクトル
vwsubu.wx  vd, vs2, rs1, vm  # ベクトル - スカラ

# ビット幅拡張 符号あり整数加減算命令 2*SEW = 2*SEW +/- SEW
vwadd.wv  vd, vs2, vs1, vm  # ベクトル - ベクトル
vwadd.wx  vd, vs2, rs1, vm  # ベクトル - スカラ
vwsub.wv  vd, vs2, vs1, vm  # ベクトル - ベクトル
vwsub.wx  vd, vs2, rs1, vm  # ベクトル - スカラ
```

> 整数の値はスカラオペランド`x0`の加算命令を使用してビット幅を倍に拡張することができる。疑似命令として`vwcvt.x.x.v vd,vs,vm = vwadd.vx vd,vs,x0,vm`と`vwcvtu.x.x.v vd,vs,vm = vwaddu.vx vd,vs,x0,vm`を定義することができる。

### 12.3. キャリー付きベクトル整数加算命令、ボロー付きベクトル整数減算命令

複数ワードの整数演算をサポートそるために、キャリービットを使用する命令を定義する。加減算操作において、2つの命令が定義されている: 1つは結果をSEWの長さで生成し、2つ目はキャリー出力(マスクのBoolean値の1ビットとして生成される)命令である。

キャリーの入力と出力はマスクレジスタレイアウトの章で説明した[マスクレジスタのレイアウト](https://riscv.github.io/documents/riscv-v-spec/#sec-mask-register-layout)として記述されている。エンコーディングの制約により、キャリー入力は`v0`レジスタより入力される必要があるが、キャリーの出力は任意のベクトルレジスタに書き込んでよい。ただし、ソースレジスタ・書き込みレジスタのオーバラップの条件に従う必要がある。

`vadc`と`vsbc`命令はソースオペランドとキャリーイン・ボローインの加減算を行い、結果をベクトルレジスタ`vd`に書き込む。これらの命令はマスク命令(`vm=0`)としてエンコードされるが、この命令ではすべてのエレメントに対して演算が行われて、書き込みも発生する。アンマスクの場合のエンコーディングは予約されている。

`vmadc`および`vmsbc`命令はソースオペランドと、オプションでマスクがされている場合(`vm=0`)キャリーイン・ボローインの加減算を行い、結果をマスクレジスタ`vd`に書き込む。

マスクされていない場合(`vm=1`)はキャリーインとボローインは使用されない。これらの命令はすべての要素に対して演算が実行され、マスクがされていたとしても書き込みが行われる。

```
 # キャリー付き加算命令

 # vd[i] = vs2[i] + vs1[i] + v0[i].LSB
 vadc.vvm   vd, vs2, vs1, v0  # ベクトル - ベクトル

 # vd[i] = vs2[i] + x[rs1] + v0[i].LSB
 vadc.vxm   vd, vs2, rs1, v0  # ベクトル - スカラ

 # vd[i] = vs2[i] + imm + v0[i].LSB
 vadc.vim   vd, vs2, imm, v0  # ベクトル - 即値

 # マスクレジスタのフォーマットに従ってキャリーを生成する命令

 # vd[i] = carry_out(vs2[i] + vs1[i] + v0[i].LSB)
 vmadc.vvm   vd, vs2, vs1, v0  # ベクトル - ベクトル

 # vd[i] = carry_out(vs2[i] + x[rs1] + v0[i].LSB)
 vmadc.vxm   vd, vs2, rs1, v0  # ベクトル - スカラ

 # vd[i] = carry_out(vs2[i] + imm + v0[i].LSB)
 vmadc.vim   vd, vs2, imm, v0  # ベクトル - 即値
 
 # vd[i] = carry_out(vs2[i] + vs1[i])
 vmadc.vv    vd, vs2, vs1      # Vector-vector, no carry-in

 # vd[i] = carry_out(vs2[i] + x[rs1])
 vmadc.vx    vd, vs2, rs1      # Vector-scalar, no carry-in

 # vd[i] = carry_out(vs2[i] + imm)
 vmadc.vi    vd, vs2, imm      # Vector-immediate, no carry-in
```

キャリー伝搬を実装するためには、既存の2つの命令に加えて、正しい答えを得るために破壊的なアキュムレータが必要になる。

```
  # 複数ワードの算術演算命令を実現する例。v4に答えを累積する。
  vmadc.vvm v1, v4, v8, v0  # v1にキャリーを一時的に格納する。
  vadc.vvm v4, v4, v8, v0   # 加算を実行する。
  vmcpy.m v0, v1            # v0にキャリーを移動して、次の演算の準備をする。
```

ボロー付きの減算命令`vsbc`はワード長の大きな値の減算命令と同じ働きをする。この命令には即値オペランドの命令は定義されていない。

```
 # ボローにより差分を計算する。

# vd[i] = vs2[i] - vs1[i] - v0[i].LSB
 vsbc.vvm   vd, vs2, vs1, v0  # ベクトル - ベクトル

 # vd[i] = vs2[i] - x[rs1] - v0[i].LSB
 vsbc.vxm   vd, vs2, rs1, v0  # ベクトル - スカラ

 # マスクレジスタフォーマットにボローアウトを出力する。
 
 # vd[i] = borrow_out(vs2[i] - vs1[i] - v0[i].LSB)
 vmsbc.vvm   vd, vs2, vs1, v0  # ベクトル - ベクトル

 # vd[i] = borrow_out(vs2[i] - x[rs1] - v0[i].LSB)
 vmsbc.vxm   vd, vs2, rs1, v0  # ベクトル - スカラ
 
 # vd[i] = borrow_out(vs2[i] - vs1[i])
 vmsbc.vv    vd, vs2, vs1      # Vector-vector, no borrow-in

 # vd[i] = borrow_out(vs2[i] - x[rs1])
 vmsbc.vx    vd, vs2, rs1      # Vector-scalar, no borrow-in
```

`vmsbc`では、差分が発生した場合にはtruncationが発生する前に1が生成される、これは負の数である(xxx 訳者注: 意味不明)。

`vadc`と`vsbc`命令は書き込み先ベクトルレジスタが`v0`である場合、かつ`LMUL>1`である場合には不正命令例外が発生する。

> この制約は、マスクベクトル演算に関して、マスクレジスタを上書きする場合の制約に相当する。

`vmadc`および`vmsbc`では、書き込み先ベクトルレジスタがソースベクトルレジスタグループとオーバラップし、かつ`LMUL > 1`である場合に不正命令例外が発生する。

### 12.4. ベクトルビット演算命令

```
# 論理演算命令
vand.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
vand.vx vd, vs2, rs1, vm   # ベクトル - スカラ
vand.vi vd, vs2, imm, vm   # ベクトル - 即値

vor.vv vd, vs2, vs1, vm    # ベクトル - ベクトル
vor.vx vd, vs2, rs1, vm    # ベクトル - スカラ
vor.vi vd, vs2, imm, vm    # ベクトル - 即値

vxor.vv vd, vs2, vs1, vm    # ベクトル - ベクトル
vxor.vx vd, vs2, rs1, vm    # ベクトル - スカラ
vxor.vi vd, vs2, imm, vm    # ベクトル - 即値
```

> スカラの即値を持つ`xvor`命令で、即値オペランドを-1とすることで論理NOT命令として使用することができる。これは疑似アセンブラ命令`vnot.v`として使用できる。

### 12.5. 同一幅ベクトルビットシフト命令

すべての形式のベクトルシフト命令を定義している。論理左シフト、ゼロ拡張の論理右シフト、符号拡張の論理右シフトである。

```
# ビットシフト操作

vsll.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
vsll.vx vd, vs2, rs1, vm   # ベクトル - スカラ
vsll.vi vd, vs2, uimm, vm   # ベクトル - 即値

vsrl.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
vsrl.vx vd, vs2, rs1, vm   # ベクトル - スカラ
vsrl.vi vd, vs2, uimm, vm   # ベクトル - 即値

vsra.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
vsra.vx vd, vs2, rs1, vm   # ベクトル - スカラ
vsra.vi vd, vs2, uimm, vm   # ベクトル - 即値
```

シフト量は、オペランドの下位lg2(SEW)ビットのみ有効である。

即値は符号なしシフト量として扱われ、最大のシフト量は31である。

### 12.6. ベクトル幅を縮退する整数右シフト命令

大きなビット幅のオペランドから、右シフト命令によってより小さなフィールドに縮退する命令は2種類定義されている。ゼロ拡張を行う命令(`srl`)と符号拡張を行う命令(`sra`)である。シフト量はスカラの整数レジスタもしくは5ビットの即値である。ベクトルもしくはスカラレジスタの下位lg2(2*SEW)ビットがシフト量として使用される(例えば、SEW=64-bitからSEW=32-bitへのビット縮退のシフト命令であれば、下位の6ビットがシフト量として使用される。

```
 # ビット縮退論理右シフト命令, SEw = (2*SEW) >> SEW
 
 vnsrl.wv vd, vs2, vs1, vm   # ベクトル - ベクトル
 vnsrl.wx vd, vs2, rs1, vm   # ベクトル - スカラ
 vnsrl.wi vd, vs2, uimm, vm   # ベクトル - 即値

 # ビット縮退算術右シフト命令, SEW = (2*SEW) >> SEW
 vnsra.wv vd, vs2, vs1, vm   # ベクトル - ベクトル
 vnsra.wx vd, vs2, rs1, vm   # ベクトル - スカラ
 vnsra.wi vd, vs2, uimm, vm   # ベクトル - 即値
```

> バリエーションとして、1/4のサイズまでビット縮小を行うn4命令を定義することもできる。

### 12.7. 整数ベクトル比較命令

以下の整数比較命令は、比較結果新であれば書き込みレジスタのマスク要素に1を書き込み、そうでなければ0を書き込む。書き込みマスクレジスタは常に1つのベクトルレジスタであり、要素のレイアウトは[マスクレジスタのレイアウト](https://riscv.github.io/documents/riscv-v-spec/#sec-mask-register-layout)に示した通りである。

```
# Set if equal
vmseq.vv vd, vs2, vs1, vm  # ベクトル - ベクトル
vmseq.vx vd, vs2, rs1, vm  # ベクトル - スカラ
vmseq.vi vd, vs2, imm, vm  # ベクトル - 即値

# Set if not equal
vmsne.vv vd, vs2, vs1, vm  # ベクトル - ベクトル
vmsne.vx vd, vs2, rs1, vm  # ベクトル - スカラ
vmsne.vi vd, vs2, imm, vm  # ベクトル - 即値

# Set if less than, unsigned
vmsltu.vv vd, vs2, vs1, vm  # ベクトル - ベクトル
vmsltu.vx vd, vs2, rs1, vm  # ベクトル - スカラ

# Set if less than, signed
vmslt.vv vd, vs2, vs1, vm  # ベクトル - ベクトル
vmslt.vx vd, vs2, rs1, vm  # ベクトル - スカラ

# Set if less than or equal, unsigned
vmsleu.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
vmsleu.vx vd, vs2, rs1, vm   # ベクトル - スカラ
vmsleu.vi vd, vs2, imm, vm   # ベクトル - 即値

# Set if less than or equal, signed
vmsle.vv vd, vs2, vs1, vm  # ベクトル - ベクトル
vmsle.vx vd, vs2, rs1, vm  # ベクトル - スカラ
vmsle.vi vd, vs2, imm, vm  # ベクトル - 即値

# Set if greater than, unsigned
vmsgtu.vx vd, vs2, rs1, vm   # ベクトル - スカラ
vmsgtu.vi vd, vs2, imm, vm   # ベクトル - 即値

# Set if greater than, signed
vmsgt.vx vd, vs2, rs1, vm    # ベクトル - スカラ
vmsgt.vi vd, vs2, imm, vm    # ベクトル - 即値

# Following two instructions are not provided directly
# Set if greater than or equal, unsigned
# vmsgeu.vx vd, vs2, rs1, vm    # ベクトル - スカラ
# Set if greater than or equal, signed
# vmsge.vx vd, vs2, rs1, vm    # ベクトル - スカラ
```

以下の表は、すべての比較演算がどのようにマシンコードにマッピングされるかを示している。

```
比較演算         アセンブラのマッピング            アセンブラの疑似命令

va < vb         vmslt{u}.vv vd, va, vb, vm
va <= vb        vmsle{u}.vv vd, va, vb, vm
va > vb         vmslt{u}.vv vd, vb, va, vm    vmsgt{u}.vv vd, va, vb, vm
va >= vb        vmsle{u}.vv vd, vb, va, vm    vmsge{u}.vv vd, va, vb, vm

va < x          vmslt{u}.vx vd, va, x, vm
va <= x         vmsle{u}.vx vd, va, x, vm
va > x          vmsgt{u}.vx vd, va, x, vm
va >= x         see below

va < i          vmsle{u}.vi vd, va, i-1, vm    vmslt{u}.vi vd, va, i, vm
va <= i         vmsle{u}.vi vd, va, i, vm
va > i          vmsgt{u}.vi vd, va, i, vm
va >= i         vmsgt{u}.vi vd, va, i-1, vm    vmsge{u}.vi vd, va, i, vm

va, vb vector register groups
x      scalar integer register
i      immediate
```

> `vmslt{u}.vi`は即値オペランドの形式は定義されない代わりに、`vmsle{i}.vi`のバリエーションから1を減算して使用することができる。`vmsle.vi`のオペランドの範囲は-16から15までであり、`vmslt.vi`の範囲は-15から16となる。`vmsleu.vi`のオペランドの範囲は0から15(かつ(~0)-15から~0まで)、`vmsltu.vi`の範囲は1から16となる(ここで、`vmsltu.vi`で即値が0の場合は常に偽であるためあまり使い物にならない)。同様に、`vmsge{i}.vi`は定義されず、その代わりに`vmsge{u}.vi`の即値を1つ減算して使用すること。`vmsge.vi`のオペランドの有効範囲は-15から16であり、`vmsgeu.vi`の有効範囲は1から16である(ここで、`vmsgeu.vi`の即値0のオペランドは常に真であるためあまり役に立たない)。

> `vmsgt`の形式ではレジスタオペランドはスカラレジスタと即値の形式が与えられており、余分なマスクの論理演算を必要とせず、1つの比較命令によりマスクの値を制御できるように設計されている。

エンコーディング領域を節約するために、`vmsge{u}.vx`の形式は直接は定義されず、`va ≥ x`の場合に特殊な処理を行う。

> `vmsge{}.vx`の形式は`vmslt{u}`形式の使用されないOPIVIバリエーションの中で、直行性を持たずに定義されている。これらの命令はOPIVI命令の中でスカラの整数レジスタを使用する唯一の命令となる予定である。その代わりに、さらなる2つのfunct6エンコーディングを使用することができるが、これらの命令形式では、同じfunct6の8つのグループと同じエンコーディングではなく、さらに異なるオペランドのフォーマットを使用する予定である(マスクレジスタへの書き込みなど)。現在のPoRではこれらの命令は除外されており、以下で説明する。

`vmsge{u}.vx`命令は`vmsgt{u}.vx`の命令のオペランドを1つ減算することにより合成することができる。この場合に、整数レジスタはアンダーフローは発生しないことが知られている。

```
`vmsge{u}.vx`命令の生成シーケンス

va >= x,  x > minimum

   addi t0, x, -1; vmsgt{u}.vx vd, va, t0, vm
```

上記のシーケンスは通常は最も効率的な実装であるが、整数レジスタ`x`が道の場合にはアセンブラによる疑似命令により生成することができる。

```
マスクのない va >= x

  疑似命令: vmsge{u}.vx vd, va, x
  展開: vmslt{u}.vx vd, va, x; vmnand.mm vd, vd, vd

マスクされた va >= x, vd != v0

  疑似命令: vmsge{u}.vx vd, va, x, v0.t
  展開: vmslt{u}.vx vd, va, x, v0.t; vmxor.mm vd, vd, v0

マスクされた va >= x, any vd

  疑似命令: vmsge{u}.vx vd, va, x, v0.t, vt
  展開: vmslt{u}.vx vt, va, x;  vmandnot.mm vd, vd, vt

  疑似命令においてvtレジスタは一時使用レジスタとして指定しなければならず、vdと同一で経なく疑似命令において破壊可能なレジスタでなければならない。
```

比較命令は、マスクにおいてAND演算を効率的に実行できる。

```
    # (a < b) && (b < c) in two instructions
    vmslt.vv    v0, va, vb        # すべてのボディー要素に対して書き込み。
    vmslt.vv    v0, vb, vc, v0.t  # マスクがセットされている要素に対してのみ書き込み。
```

すべての比較命令について、LMUL > 1かつ書き込みレジスタグループがソースレジスタグループとオーバラップしている場合に不正命令例外が発生する。

### 12.8. ベクトル整数Min/Max命令

符号付、符号なしの整数のMax/Min命令が定義されている。

```
# Unsigned minimum
vminu.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
vminu.vx vd, vs2, rs1, vm   # ベクトル - スカラ

# Signed minimum
vmin.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
vmin.vx vd, vs2, rs1, vm   # ベクトル - スカラ

# Unsigned maximum
vmaxu.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
vmaxu.vx vd, vs2, rs1, vm   # ベクトル - スカラ

# Signed maximum
vmax.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
vmax.vx vd, vs2, rs1, vm   # ベクトル - スカラ
```

### 12.9. ベクトル単一ビット幅整数乗算命令

単一ビット幅の乗算命令はSEWビット*SEWビットの乗算を行い、SEWビットの結果を返す。`**mulh**`版は、書き込みレジスタに乗算結果の上位ワードを書き込む。

```
# 符号付乗算, 積の下位ビットを返す。
vmul.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
vmul.vx vd, vs2, rs1, vm   # ベクトル - スカラ

# 符号付乗算, 席の上位ビットを返す。
vmulh.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
vmulh.vx vd, vs2, rs1, vm   # ベクトル - スカラ

# 符号なし乗算, 積の上位ビットを返す。
vmulhu.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
vmulhu.vx vd, vs2, rs1, vm   # ベクトル - スカラ

# 符号付(vs2)-符号なし乗算, 積の上位ビットを返す。
vmulhsu.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
vmulhsu.vx vd, vs2, rs1, vm   # ベクトル - スカラ
```

> 符号なしベクトル * 符号付スカラの乗算結果を返す`vmulhus`オペコードは定義されていない。

> 現在の`vmulh*`オペコードは簡単な乗算結果を返す命令であるが、結果をスケール、丸め、Saturateする機能は備えていない。`vmulh`, `vmulhu`, `vmulhsu`の命令定義を変えて、`vxrm`丸めモードを使用して下位のハーフ積を捨てるように仕様を変更することも考えられる。この場合ではオーバフローは発生しない。

### 12.10. ベクトル整数除算命令

除算命令と剰余命令はRISC-Vの標準スカラ整数乗算・剰余命令と同一である。ベクトル値を入力すること以外、仕様は同一である。

```
    # 符号なし除算
    vdivu.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
    vdivu.vx vd, vs2, rs1, vm   # ベクトル - スカラ

    # 符号付除算
    vdiv.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
    vdiv.vx vd, vs2, rs1, vm   # ベクトル - スカラ

    # 符号なし剰余
    vremu.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
    vremu.vx vd, vs2, rs1, vm   # ベクトル - スカラ

    # 符号付剰余
    vrem.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
    vrem.vx vd, vs2, rs1, vm   # ベクトル - スカラ
```

> 整数除算命令と剰余命令を含めるかどうかについては議論があった。標準的な命令を定義しなくても、ソフトウェアにより同様のアルゴリズムを選択することができるという考え方と、それではいくつかのマイクロアーキテクチャと比較して性能が落ちてしまうという議論があった。

> スカラをベクトルで除算するという命令は定義されていない。

### 12.11. ビット幅拡張ベクトル整数乗算命令

整数乗算命令において、SEWビットとSEWビットのオペランドを入力し演算結果を2SEWビットの乗算結果として返す命令を定義する。

```
# ビット幅拡張符号付整数乗算
vwmul.vv  vd, vs2, vs1, vm# ベクトル - ベクトル
vwmul.vx  vd, vs2, rs1, vm # ベクトル - スカラ

# ビット幅拡張符号なし整数乗算命令
vwmulu.vv vd, vs2, vs1, vm # ベクトル - ベクトル
vwmulu.vx vd, vs2, rs1, vm # ベクトル - スカラ

# ビット幅拡張符号付、符号なし整数乗算命令
vwmulsu.vv vd, vs2, vs1, vm # ベクトル - ベクトル
vwmulsu.vx vd, vs2, rs1, vm # ベクトル - スカラ
```

### 12.12. 同一ビット幅ベクトル乗算加算命令

整数の乗算加算命令はレジスタを破壊する命令であり、2種類の形式で定義される。1つ目は加減算を行うオペランドに対して値を上書きする命令(`vmacc`, `vnmsac`)であり、もう1つは乗算の最初のオペランドのレジスタを破壊するものである(`vmadd`, `vnmsub`)。

加減算の項では、3番目のオペランドと積の下位半分のビットに対して演算が実行される。

> "sac"命令は"subtract from accumulator"の意味である。オペコードは"vnmsac"であり、(残念ながら直観に反するが)浮動小数点の`fnmsub`命令の定義にマッチする。"vnmsub"命令も同様である。

```
# 整数乗算加算、加算のオペランドを上書き
vmacc.vv vd, vs1, vs2, vm    # vd[i] = +(vs1[i] * vs2[i]) + vd[i]
vmacc.vx vd, rs1, vs2, vm    # vd[i] = +(x[rs1] * vs2[i]) + vd[i]

# 整数乗算減算、減算のオペランドを上書き
vnmsac.vv vd, vs1, vs2, vm    # vd[i] = -(vs1[i] * vs2[i]) + vd[i]
vnmsac.vx vd, rs1, vs2, vm    # vd[i] = -(x[rs1] * vs2[i]) + vd[i]

# 整数乗算加算、乗算のオペランドを上書き
vmadd.vv vd, vs1, vs2, vm    # vd[i] = (vs1[i] * vd[i]) + vs2[i]
vmadd.vx vd, rs1, vs2, vm    # vd[i] = (x[rs1] * vd[i]) + vs2[i]

# 整数乗算減算、乗算のオペランドを上書き
vnmsub.vv vd, vs1, vs2, vm    # vd[i] = -(vs1[i] * vd[i]) + vs2[i]
vnmsub.vx vd, rs1, vs2, vm    # vd[i] = -(x[rs1] * vd[i]) + vs2[i]
```

### 12.13. ビット幅拡張ベクトル整数乗算加算命令

乗算加算命令において、SEWビットとSEWビットのオペランドを入力して、2SEWビット幅の演算結果を返す命令を定義する。すべての命令の組み合わせで、符号付と符号なしの乗算オペランドをサポートする。

```
# ビット幅拡張整数乗算加算、加算のオペランドを上書き
vwmaccu.vv vd, vs1, vs2, vm    # vd[i] = +(vs1[i] * vs2[i]) + vd[i]
vwmaccu.vx vd, rs1, vs2, vm    # vd[i] = +(x[rs1] * vs2[i]) + vd[i]

# ビット幅拡張整数乗算減算、加算のオペランドを上書き
vwmacc.vv vd, vs1, vs2, vm    # vd[i] = +(vs1[i] * vs2[i]) + vd[i]
vwmacc.vx vd, rs1, vs2, vm    # vd[i] = +(x[rs1] * vs2[i]) + vd[i]

# ビット幅拡張整数乗算加算、乗算のオペランドを上書き
vwmaccsu.vv vd, vs1, vs2, vm    # vd[i] = +(signed(vs1[i]) * unsigned(vs2[i])) + vd[i]
vwmaccsu.vx vd, rs1, vs2, vm    # vd[i] = +(signed(x[rs1]) * unsigned(vs2[i])) + vd[i]

# ビット幅拡張整数乗算減算、乗算のオペランドを上書き
vwmaccus.vx vd, rs1, vs2, vm    # vd[i] = +(unsigned(x[rs1]) * signed(vs2[i])) + vd[i]
```

### 12.13 4倍ビット幅拡張ベクトル整数乗算加算命令

4倍ビット幅整数乗算加算命令はSEWビット\*SEWビットの乗算の結果に4\*SEWビット幅の値を加算し、4\*SEWビット幅の結果を生成する。符号あり、符号なしのすべての組み合わせのオペランドをサポートしている。

> これらの命令は現在"V"ベース命令への取り込みは計画されていない。

> ELEN=32のマシンでは、8b * 8b = 16bの結果を32bのアキュムレータに書き込むモードしかサポートされていない。ELEN=64の場合には16b * 16b = 32b の結果を、64bにアキュムレートする。

```
# Quad-widening unsigned-integer multiply-add, overwrite addend
# 4倍ビット幅拡張符号なし整数乗算加算命令、加算のオペランドは上書き
vqmaccu.vv vd, vs1, vs2, vm    # vd[i] = +(vs1[i] * vs2[i]) + vd[i]
vqmaccu.vx vd, rs1, vs2, vm    # vd[i] = +(x[rs1] * vs2[i]) + vd[i]

# Quad-widening signed-integer multiply-add, overwrite addend
# 4倍ビット幅符号拡張あり整数乗算加算命令、加算のオペランドを上書き
vqmacc.vv vd, vs1, vs2, vm    # vd[i] = +(vs1[i] * vs2[i]) + vd[i]
vqmacc.vx vd, rs1, vs2, vm    # vd[i] = +(x[rs1] * vs2[i]) + vd[i]

# Quad-widening signed-unsigned-integer multiply-add, overwrite addend
# 4倍ビット幅符号拡張あり・符号拡張なし整数乗算加算命令、加算のオペランドを上書き。
vqmaccsu.vv vd, vs1, vs2, vm    # vd[i] = +(signed(vs1[i]) * unsigned(vs2[i])) + vd[i]
vqmaccsu.vx vd, rs1, vs2, vm    # vd[i] = +(signed(x[rs1]) * unsigned(vs2[i])) + vd[i]

# Quad-widening unsigned-signed-integer multiply-add, overwrite addend
# 4倍ビット幅符号拡張なし・符号拡張あり整数乗算加算命令、加算のオペランドを上書き。
vqmaccus.vx vd, rs1, vs2, vm    # vd[i] = +(unsigned(x[rs1]) * signed(vs2[i])) + vd[i]
```

### 12.14. ベクトル整数マージ命令

ベクトル整数マージ命令は、2つのソースオペランドをマスクフィールドに基づいてマージする命令である。通常の算術演算命令と違い、マージの操作はすべてのボディー要素に対して適用される(つまり、`vstart`から`vl`までのすべてのベクトル要素に対して適用される)。

`vmerge`命令は常にマスクされる(`vm=0`)。この命令は2つのソースオペランドを以下に従ってマージする。マスクの値がゼロである場合は、最初のオペランドが書き込み要素に対してコピーされ、そうでなければ2番目のオペランドが書き込み要素に対してコピーされる。最初のオペランドは常に`vs2`で指定されるベクトルレジスタグループであり、2番目のオペランドは`vs1`で指定されるベクトルレジスタグループか、`rs1`で指定される整数スカラレジスタ`x`か、5ビットの符号拡張された即値である。

```
vmerge.vvm vd, vs2, vs1, v0  # vd[i] = v0[i].LSB ? vs1[i] : vs2[i]
vmerge.vxm vd, vs2, rs1, v0  # vd[i] = v0[i].LSB ? x[rs1] : vs2[i]
vmerge.vim vd, vs2, imm, v0  # vd[i] = v0[i].LSB ? imm    : vs2[i]
```

### 12.15. ベクトル整数移動命令

ベクトル整数移動命令はソースオペランドからベクトルレジスタグループへの値のコピーを行う。この命令は常にマスクされない(`vm=1`)。最初のオペランド(`vs2`)は`v0`でなければならず、他のベクトルレジスタを`vs2`に指定する形式は予約されている。この命令hあ`vs1`, `rs1`もしくは即値のオペランドを書き込みベクトルレジスタグループの最初の`vl`の場所にコピーする。

```
vmv.v.v vd, vs1 # vd[i] = vs1[i]
vmv.v.x vd, rs1 # vd[i] = rs1
vmv.v.i vd, imm # vd[i] = imm
```

> マスクの値は`vmv.v.i vd, 0; vmerge.vim vd, vd, 1, v0`の命令列を使用してSEWビット幅まで拡張することができる。

> ベクトル整数移動命令はベクトルマージ命令とエンコーディングを共有している。`vm=1`かつ`vs2=v0`であるところだけが異なる。

