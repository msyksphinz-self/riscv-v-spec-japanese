## 14. ベクトル浮動小数点命令

標準的なベクトル浮動小数点命令は、16ビット、32ビット、64ビット、128ビットのIEEE-754/2008互換の浮動小数点を扱う。現在のSEWがこれらのIEEE浮動小数点の型に合致しなければ、不正命令例外が発生する。

> サポートされる浮動小数点のビット幅はそのプラットフォームに依存する。

> 16ビット半精度浮動小数点の型をサポートするプラットフォームはスカラ側も半精度浮動小数点命令を`f`レジスタにおいて実装しなければならない。

ベクトル浮動小数点命令はNaNの取り扱いについてスカラの浮動小数点命令と同様の動きをする。

ベクトル ー スカラ命令におけるスカラの値の取り扱いについては、標準的な`f`スカラレジスタからオペランドを取り込む。

> Zfinx命令仕様バリエーションにより、スカラの浮動小数点値は整数レジスタ`x`からも取り込むことができるようになる予定である。

### 14.1. ベクトル浮動小数点例外フラグ

ベクトル浮動小数点の例外は任意のアクティブな浮動小数点ベクトルの要素で例外が発生した場合、標準的なFP例外フラグである`fflags`に設定される。インアクティブな要素では、浮動小数点例外は発生しない。

### 14.2. 単一ビット幅ベクトル浮動小数点加減算命令

```
    # 浮動小数点加算命令
    vfadd.vv vd, vs2, vs1, vm   # ベクトル ー ベクトル
    vfadd.vf vd, vs2, rs1, vm   # ベクトル ー スカラ

    # 浮動小数点減算命令
    vfsub.vv vd, vs2, vs1, vm   # ベクトル ー ベクトル
    vfsub.vf vd, vs2, rs1, vm   # ベクトル ー スカラ vd[i] = vs2[i] - f[rs1]
    vfrsub.vf vd, vs2, rs1, vm  # スカラ ー ベクトル vd[i] = f[rs1] - vs2[i]
```

### 14.3. ビット幅拡張ベクトル浮動小数点加減算命令

```
# ビット幅拡張ベクトル浮動小数点加減算命令 2*SEW = SEW +/- SEW
vfwadd.vv vd, vs2, vs1, vm  # ベクトル ー ベクトル
vfwadd.vf vd, vs2, rs1, vm  # ベクトル ー スカラ
vfwsub.vv vd, vs2, vs1, vm  # ベクトル ー ベクトル
vfwsub.vf vd, vs2, rs1, vm  # ベクトル ー スカラ

# ビット幅拡張ベクトル浮動小数点加減算命令 2*SEW = 2*SEW +/- SEW
vfwadd.wv  vd, vs2, vs1, vm  # ベクトル ー ベクトル
vfwadd.wf  vd, vs2, rs1, vm  # ベクトル ー スカラ
vfwsub.wv  vd, vs2, vs1, vm  # ベクトル ー ベクトル
vfwsub.wf  vd, vs2, rs1, vm  # ベクトル ー スカラ
```

### 14.4. 単一ビット幅ベクトル浮動小数点乗除算命令

```
    # 浮動小数点乗算命令
    vfmul.vv vd, vs2, vs1, vm   # ベクトル ー ベクトル
    vfmul.vf vd, vs2, rs1, vm   # ベクトル ー スカラ

    # 浮動小数点除算命令
    vfdiv.vv vd, vs2, vs1, vm   # ベクトル ー ベクトル
    vfdiv.vf vd, vs2, rs1, vm   # ベクトル ー スカラ

    # オペランドの形式が逆の浮動小数点除算命令。divide vector = scalar / vector
    vfrdiv.vf vd, vs2, rs1, vm  # スカラ ー ベクトル, vd[i] = f[rs1]/vs2[i]
```

### 14.5. ビット幅拡張浮動小数点乗算命令

```
# ビット幅拡張浮動小数点乗算命令
vfwmul.vv    vd, vs2, vs1, vm # ベクトル ー ベクトル
vfwmul.vf    vd, vs2, rs1, vm # ベクトル ー スカラ
```

### 14.6. 単一ビット幅浮動小数点Fused Multiply-Add命令

Fused Multiply-Addの4つのバリエーションについて命令を提供する。入力オペランドを破壊する形式については2種類を用意しており、1つは加算のオペランドを上書きするもの、もう一つは乗算の項の一つ目のオペランドを破壊するものである。

```
# 浮動小数点 Multiply-Accumulate, 加算のオペランドを上書き
vfmacc.vv vd, vs1, vs2, vm    # vd[i] = +(vs1[i] * vs2[i]) + vd[i]
vfmacc.vf vd, rs1, vs2, vm    # vd[i] = +(f[rs1] * vs2[i]) + vd[i]

# 浮動小数点 negate-(multiply-accumulate), 減算のオペランドを上書き
vfnmacc.vv vd, vs1, vs2, vm   # vd[i] = -(vs1[i] * vs2[i]) - vd[i]
vfnmacc.vf vd, rs1, vs2, vm   # vd[i] = -(f[rs1] * vs2[i]) - vd[i]

# 浮動小数点 multiply-subtract-accumulator, 減算のオペランドを上書き
vfmsac.vv vd, vs1, vs2, vm    # vd[i] = +(vs1[i] * vs2[i]) - vd[i]
vfmsac.vf vd, rs1, vs2, vm    # vd[i] = +(f[rs1] * vs2[i]) - vd[i]

# 浮動小数点 negate-(multiply-subtract-accumulator), 減算のオペランドを上書き
vfnmsac.vv vd, vs1, vs2, vm   # vd[i] = -(vs1[i] * vs2[i]) + vd[i]
vfnmsac.vf vd, rs1, vs2, vm   # vd[i] = -(f[rs1] * vs2[i]) + vd[i]

# 浮動小数点 multiply-add, 乗算のオペランドを上書き
vfmadd.vv vd, vs1, vs2, vm    # vd[i] = +(vs1[i] * vd[i]) + vs2[i]
vfmadd.vf vd, rs1, vs2, vm    # vd[i] = +(f[rs1] * vd[i]) + vs2[i]

# 浮動小数点 negate-(multiply-add), 乗算のオペランドを上書き
vfnmadd.vv vd, vs1, vs2, vm   # vd[i] = -(vs1[i] * vd[i]) - vs2[i]
vfnmadd.vf vd, rs1, vs2, vm   # vd[i] = -(f[rs1] * vd[i]) - vs2[i]

# 浮動小数点 multiply-sub, 乗算のオペランドを上書き
vfmsub.vv vd, vs1, vs2, vm    # vd[i] = +(vs1[i] * vd[i]) - vs2[i]
vfmsub.vf vd, rs1, vs2, vm    # vd[i] = +(f[rs1] * vd[i]) - vs2[i]

# 浮動小数点 negate-(multiply-sub), 乗算のオペランドを上書き
vfnmsub.vv vd, vs1, vs2, vm   # vd[i] = -(vs1[i] * vd[i]) + vs2[i]
vfnmsub.vf vd, rs1, vs2, vm   # vd[i] = -(f[rs1] * vd[i]) + vs2[i]
```

> スカラの浮動小数点FMAエンコーディングの中で、使用していない丸めモードのビットパタンを使用してオペランドを破壊しないFMAを定義することも可能であるが、その場合はマスクができない形式での3入力1出力の命令となる。

### 14.7. ビット幅拡張ベクトル浮動小数点Fused Multiply-Add命令

ビット幅拡張を行うFused Multiply-Add命令はビット幅拡張された加算の項に結果を書き込む。乗算の入力値はSEWのビットサイズであり、加算の項と結果を書き込む項は2*SEWのビット幅である。

```
# ビット幅拡張浮動小数点 multiply-accumulate, 加算の項を上書き
vfwmacc.vv vd, vs1, vs2, vm    # vd[i] = +(vs1[i] * vs2[i]) + vd[i]
vfwmacc.vf vd, rs1, vs2, vm    # vd[i] = +(f[rs1] * vs2[i]) + vd[i]

# ビット幅拡張浮動小数点 negate-(multiply-accumulate), 加算の項を上書き
vfwnmacc.vv vd, vs1, vs2, vm   # vd[i] = -(vs1[i] * vs2[i]) - vd[i]
vfwnmacc.vf vd, rs1, vs2, vm   # vd[i] = -(f[rs1] * vs2[i]) - vd[i]

# ビット幅拡張浮動小数点 multiply-subtract-accumulator, 加算の項を上書き
vfwmsac.vv vd, vs1, vs2, vm    # vd[i] = +(vs1[i] * vs2[i]) - vd[i]
vfwmsac.vf vd, rs1, vs2, vm    # vd[i] = +(f[rs1] * vs2[i]) - vd[i]

# ビット幅拡張浮動小数点 negate-(multiply-subtract-accumulator), 加算の項を上書き
vfwnmsac.vv vd, vs1, vs2, vm   # vd[i] = -(vs1[i] * vs2[i]) + vd[i]
vfwnmsac.vf vd, rs1, vs2, vm   # vd[i] = -(f[rs1] * vs2[i]) + vd[i]
```

### 14.8. ベクトル浮動小数点Square-Root命令

この命令は単一項のベクトル － ベクトル命令である。

```
    # 浮動小数点 square root
    vfsqrt.v vd, vs2, vm   # ベクトル ー ベクトル square root
```

### 14.9. ベクトル浮動小数点 MIN/MAX命令

The vector floating-point `vfmin` and `vfmax` instructions have the same behavior as the corresponding scalar floating-point instructions in version 2.2 of the RISC-V F/D/Q extension.

ベクトル浮動小数点`vfmin`, `vfmax`命令はスカラの浮動小数点命令 Version 2.2における RISC-V F/D/Q拡張の命令と同じ動きをする。

```
    # 浮動小数点 Minimum
    vfmin.vv vd, vs2, vs1, vm   # ベクトル ー ベクトル
    vfmin.vf vd, vs2, rs1, vm   # ベクトル ー スカラ

    # 浮動小数点 Maximum
    vfmax.vv vd, vs2, vs1, vm   # ベクトル ー ベクトル
    vfmax.vf vd, vs2, rs1, vm   # ベクトル ー スカラ
```

### 14.10. ベクトル浮動小数点符号ビット挿入命令

スカラの符号ビット挿入命令のベクトル版である。結果は符号ビットを除いて`vs2`オペランドと同じものが書き込まれる。

```
    vfsgnj.vv vd, vs2, vs1, vm   # ベクトル ー ベクトル
    vfsgnj.vf vd, vs2, rs1, vm   # ベクトル ー スカラ

    vfsgnjn.vv vd, vs2, vs1, vm   # ベクトル ー ベクトル
    vfsgnjn.vf vd, vs2, rs1, vm   # ベクトル ー スカラ

    vfsgnjx.vv vd, vs2, vs1, vm   # ベクトル ー ベクトル
    vfsgnjx.vf vd, vs2, rs1, vm   # ベクトル ー スカラ
```

### 14.11. ベクトル浮動小数点比較命令

ベクトル浮動小数点比較命令は2つのソースオペランドを比較し、その結果をマスクレジスタに書き込む。書き込み先マスクレジスタは常に1つのベクトルレジスタを使用し、[Mask Register Layout](https://riscv.github.io/documents/riscv-v-spec/#sec-mask-register-layout)で説明したレイアウトに基づいて結果を書き込む。

ベクトル浮動小数点比較命令はスカラの浮動小数点比較命令に基づいて設計されている。`vmfeq`と`vmfne`はSignaling NaN入力を受け付けた場合にのみ例外を発生する。`vmflt`, `vmfle`, `vmfgt`, `vmfge`命令はSignaling NaNとQuiet NaNのどちらでも例外を発生する。

`vmfne`はどちらかのオペランドがNaNである場合は1を書き込む。一方で他の比較命令ではどちらかのオペランドがNaNである場合は0を書き込む。

すべての比較命令では、LMUL > 1であり結果の書き込みレジスタはソースオペランドとオーバラップすることはできない。

```
    # Compare equal
    vmfeq.vv vd, vs2, vs1, vm  # ベクトル ー ベクトル
    vmfeq.vf vd, vs2, rs1, vm  # ベクトル ー スカラ

    # Compare not equal
    vmfne.vv vd, vs2, vs1, vm  # ベクトル ー ベクトル
    vmfne.vf vd, vs2, rs1, vm  # ベクトル ー スカラ

    # Compare less than
    vmflt.vv vd, vs2, vs1, vm  # ベクトル ー ベクトル
    vmflt.vf vd, vs2, rs1, vm  # ベクトル ー スカラ

    # Compare less than or equal
    vmfle.vv vd, vs2, vs1, vm  # ベクトル ー ベクトル
    vmfle.vf vd, vs2, rs1, vm  # ベクトル ー スカラ

    # Compare greater than
    vmfgt.vf vd, vs2, rs1, vm  # ベクトル ー スカラ

    # Compare greater than or equal
    vmfge.vf vd, vs2, rs1, vm  # ベクトル ー スカラ

比較演算         アセンブリ命令               アセンブリの疑似命令

va < vb         vmflt.vv vd, va, vb, vm
va <= vb        vmfle.vv vd, va, vb, vm
va > vb         vmflt.vv vd, vb, va, vm    vmfgt.vv vd, va, vb, vm
va >= vb        vmfle.vv vd, vb, va, vm    vmfge.vv vd, va, vb, vm

va < f          vmflt.vf vd, va, f, vm
va <= f         vmfle.vf vd, va, f, vm
va > f          vmfgt.vf vd, va, f, vm
va >= f         vmfge.vf vd, va, f, vm

va, vbはベクトルレジスタグループ
f     はスカラの浮動小数点レジスタ
```

> NaNのアンオーダーな比較も正しく処理するためには、すべての形式を定義する必要がある。

> C99の浮動小数点のQuietな比較操作は、どちらかの入力オペランドがNaNである場合にSignalingの比較をマスクすることで実装できる。この例を以下に示す。比較オペランドがNaNでない定数であれば、真ん中の2命令は除去することができる。

```
    # isgreater()の実装例
    vmfeq.vv v0, va, va        # AがNaNでない場合のみマスクが設定される。
    vmfeq.vv v1, vb, vb        # BがNanでない場合のみマスクが設定される。
    vmand.mm v0, v0, v1        # AとBが比較可能である場合のみマスクが設定される。
    vmfgt.vv v0, va, vb, v0.t  #  したがって、比較可能な値のみが比較される。
```

> 上記の命令列では、2番目の`vmfeq`命令をマスクし、`vmand`命令を除去することもできるが、このより効率的なコードでは`va`にQuiet NaNが含まれており、該当する`vb`の要素にSignaling NaNが含まれてえいる場合に誤って不正値例外を発生してしまう。

### 14.12. ベクトル浮動小数点分類命令

この命令はスカラの命令と同一であり、単一オペランドのベクトル － ベクトル命令である。

```
    vfclass.v vd, vs2, vm   # ベクトル ー ベクトル
```

オペランドの種類によって書き込み要素の下位の10ビットにマスクが生成される。SEW=16ビット以上の場合のみ命令が定義され、結果の値は常に書き込み要素のビット幅にフィットするようになっている。

### 14.13. ベクトル浮動小数点マージ命令

ベクトル － スカラ 浮動小数点マージ命令は`start`から始まり現在の`vl`長までのすべてのボディー要素に対して、マスクの値依存してマージ操作を行う。

`vfmerge.vfm`命令は常にマスクされている(`vm=0`)。各要素はマスクの値0である場合は最初のオペランドが書き込み要素にコピーされ、そうでない場合は浮動小数点レジスタの値が書き込み要素にコピーされる。

```
vfmerge.vfm vd, vs2, rs1, v0  # vd[i] = v0[i].LSB ? f[rs1] : vs2[i]
```

> 浮動小数点算術演算のように、FLEN > SEWであれば`vfmerge.vfm`命令は`f[rs1]`が正しくNaN-Boxedされていない場合にはCanonicalなNaNが代入される。

### 14.14. ベクトル浮動小数点移動命令

ベクトル浮動小数点移動命令は、浮動小数点スカラオペランドをベクトルレジスタグループに移動する命令である。この命令はスカラレジスタ`f`の値を、ベクトルレジスタグループのすべてのアクティブな要素にコピーする。この命令はつねにマスクはされない(`vm=1`)。この命令は`vs2`フィールドに`v0`を設定され、ほかのすべての`vs2`フィールドが予約されている必要がある(xxx意味不明)。

```
vfmv.v.f vd, rs1  # vd[i] = f[rs1]
```

> `vfmv.v.f`命令は`vfmerge.vfm`命令とエンコーディングを共有しているが、`vm=1`かつ`vs2=v0`であるところが異なる。

> 浮動小数点算術演算のように、FLEN > SEWであれば`vfmv.v.f`命令は`f[rs1]`が正しくNaN-Boxedされていない場合にはCanonicalなNaNが代入される。

### 14.15. 単一ビット幅浮動小数点 整数変換命令

型の返還命令については、浮動小数点の値と、符号なし整数あるいは符号付き整数の変換命令が定義されている。ソースオペランドも書き込み先オペランドもSEWのビット幅である必要がある。

```
vfcvt.xu.f.v vd, vs2, vm   # 浮動小数点から符号なし整数への変換
vfcvt.x.f.v  vd, vs2, vm   # 浮動小数点から符号付整数への変換

vfcvt.f.xu.v vd, vs2, vm   # 符号なし整数から浮動小数点への変換
vfcvt.f.x.v  vd, vs2, vm   # 符号付き整数から浮動小数点への変換
```

例外の発生条件については、スカラの返還命令に準ずる。変換では常に動的な丸めモード`frm`に基づいて丸めが行われる。

### 14.16. ビット幅拡張浮動小数点 整数変換命令

ビット幅の小さな整数と、浮動小数点の値を変換する命令が定義されている。

```
vfwcvt.xu.f.v vd, vs2, vm   # 浮動小数点から、倍のビット幅の符号なし整数に変換する。
vfwcvt.x.f.v  vd, vs2, vm   # 浮動小数点から、倍のビット幅の符号付き整数に変換する。

vfwcvt.f.xu.v vd, vs2, vm   # 符号なし整数から倍のビット幅の浮動小数点に変換する。
vfwcvt.f.x.v  vd, vs2, vm   # 符号付整数からバイトビット幅の浮動小数点に変換する。

vfwcvt.f.f.v vd, vs2, vm   # 1ワード分の浮動小数点の値から倍のサイズの浮動小数点に変換する。
```

これらの命令は他のビット幅を拡張する命令と同うように、ベクトルレジスタのオーバラップに関する制約が存在する([Widening Vector Arithmetic Instructions](https://riscv.github.io/documents/riscv-v-spec/#sec-widening)を参照のこと)。

> 倍のビット幅のIEEE浮動小数点は、常に元のビット幅の整数の値を正確に表現することができる。

> 倍のビット幅の浮動小数点の値は元のビット幅のIEEE浮動小数点を正確に表現できる。

すべての浮動小数点のビット幅拡張変換命令は単一の命令ではサポートされないが、複数の変換命令を使用することにより例外を発生させずに正確に同じ操作を表現することができる。

### 14.17. ビット幅を縮小するベクトル浮動小数点変換命令

幅の広い整数もしくは浮動小数点の値から、幅の狭い型に対して変換する命令を定義する。

```
vfncvt.xu.f.w vd, vs2, vm   # 倍のサイズの浮動小数点から符号なし整数へ変換する。
vfncvt.x.f.w  vd, vs2, vm   # 倍のサイズの浮動小数点から符号付き整数へ変換する。

vfncvt.f.xu.w vd, vs2, vm   # 倍のサイズの符号なし整数から浮動小数点へ変換する。
vfncvt.f.x.w  vd, vs2, vm   # 倍のサイズの符号付き整数から浮動小数点へ変換する。

vfncvt.f.f.w vd, vs2, vm   # 倍のサイズの浮動小数点から元のサイズの浮動小数点に変換する。
vfncvt.rod.f.f.w vd, vs2, vm  # 倍のサイズの浮動小数点の値から元のサイズの浮動小数点に変換する。
                              #  丸めモードはoddである。
```

これらの命令はビット幅を縮小する他のベクトル命令と同じレジスタオーバラップの制約を持っている([Narrowing Vector Arithmetic Instructions](https://riscv.github.io/documents/riscv-v-spec/#sec-narrowing)を参照のこと)

> すべての浮動小数点の変換を単一の命令でサポートすることはできない。変換処理はいくつかのシーケンスを踏む。結果は同等に丸められ、最後のステップ以外ですべてが奇数への丸め（ `vfncvt.rod.f.f.w`）を使用する場合、同じ例外フラグが発生する。 最終ステップでのみ、目的の丸めモードを使用する必要がある。

> 整数値のビット幅を半分にする処理は、ビット幅を縮小する整数シフト命令において、シフト量を0に設定することで実現できる。

