## 12. Vector Integer Arithmetic Instructions

A set of vector integer arithmetic instructions are provided.

### 12.1. Vector Single-Width Integer Add and Subtract

Vector integer add and subtract are provided. Reverse-subtract instructions are also provided for the ベクトル - スカラ forms.

```
# Integer adds.
vadd.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
vadd.vx vd, vs2, rs1, vm   # ベクトル - スカラ
vadd.vi vd, vs2, imm, vm   # ベクトル - 即値

# Integer subtract
vsub.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
vsub.vx vd, vs2, rs1, vm   # ベクトル - スカラ

# Integer reverse subtract
vrsub.vx vd, vs2, rs1, vm   # vd[i] = rs1 - vs2[i]
vrsub.vi vd, vs2, imm, vm   # vd[i] = imm - vs2[i]
```

### 12.2. Vector Widening Integer Add/Subtract

The widening add/subtract instructions are provided in both signed and unsigned variants, depending on whether the narrower source operands are first sign- or zero-extended before forming the double-width sum.

```
# Widening unsigned integer add/subtract, 2*SEW = SEW +/- SEW
vwaddu.vv  vd, vs2, vs1, vm  # ベクトル - ベクトル
vwaddu.vx  vd, vs2, rs1, vm  # ベクトル - スカラ
vwsubu.vv  vd, vs2, vs1, vm  # ベクトル - ベクトル
vwsubu.vx  vd, vs2, rs1, vm  # ベクトル - スカラ

# Widening signed integer add/subtract, 2*SEW = SEW +/- SEW
vwadd.vv  vd, vs2, vs1, vm  # ベクトル - ベクトル
vwadd.vx  vd, vs2, rs1, vm  # ベクトル - スカラ
vwsub.vv  vd, vs2, vs1, vm  # ベクトル - ベクトル
vwsub.vx  vd, vs2, rs1, vm  # ベクトル - スカラ

# Widening unsigned integer add/subtract, 2*SEW = 2*SEW +/- SEW
vwaddu.wv  vd, vs2, vs1, vm  # ベクトル - ベクトル
vwaddu.wx  vd, vs2, rs1, vm  # ベクトル - スカラ
vwsubu.wv  vd, vs2, vs1, vm  # ベクトル - ベクトル
vwsubu.wx  vd, vs2, rs1, vm  # ベクトル - スカラ

# Widening signed integer add/subtract, 2*SEW = 2*SEW +/- SEW
vwadd.wv  vd, vs2, vs1, vm  # ベクトル - ベクトル
vwadd.wx  vd, vs2, rs1, vm  # ベクトル - スカラ
vwsub.wv  vd, vs2, vs1, vm  # ベクトル - ベクトル
vwsub.wx  vd, vs2, rs1, vm  # ベクトル - スカラ
```

|      | An integer value can be doubled in width using the widening add instructions with a scalar operand of `x0`. Can define assembly pseudoinstructions `vwcvt.x.x.v vd,vs,vm = vwadd.vx vd,vs,x0,vm` and `vwcvtu.x.x.v vd,vs,vm = vwaddu.vx vd,vs,x0,vm`. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 12.3. Vector Integer Add-with-Carry / Subtract-with-Borrow Instructions

To support multi-word integer arithmetic, instructions that operate on a carry bit are provided. For each operation (add or subtract), two instructions are provided: one to provide the result (SEW width), and the second to generate the carry output (single bit encoded as a mask boolean).

These instructions are encoded as unmasked instructions (vm=1) and operate on all body elements. Encodings corresponding to the masked versions (vm=0) of these instructions are reserved.

The carry inputs and outputs are represented using the mask register layout as described in Section [Mask Register Layout](https://riscv.github.io/documents/riscv-v-spec/#sec-mask-register-layout). Due to encoding constraints, the carry input must come from the implicit `v0` register, but carry outputs can be written to any vector register that respects the source/destination overlap restrictions below.

```
 # Produce sum with carry.

# vd[i] = vs2[i] + vs1[i] + v0[i].LSB
 vadc.vvm   vd, vs2, vs1, v0  # ベクトル - ベクトル

 # vd[i] = vs2[i] + x[rs1] + v0[i].LSB
 vadc.vxm   vd, vs2, rs1, v0  # ベクトル - スカラ

 # vd[i] = vs2[i] + imm + v0[i].LSB
 vadc.vim   vd, vs2, imm, v0  # ベクトル - 即値

 # Produce carry out in mask register format

# vd[i] = carry_out(vs2[i] + vs1[i] + v0[i].LSB)
 vmadc.vvm   vd, vs2, vs1, v0  # ベクトル - ベクトル

 # vd[i] = carry_out(vs2[i] + x[rs1] + v0[i].LSB)
 vmadc.vxm   vd, vs2, rs1, v0  # ベクトル - スカラ

 # vd[i] = carry_out(vs2[i] + imm + v0[i].LSB)
 vmadc.vim   vd, vs2, imm, v0  # ベクトル - 即値
```

Because implementing a carry propagation requires executing two instructions with unchanged inputs, destructive accumulations will require an additional move to obtain correct results.

```
  # Example multi-word arithmetic sequence, accumulating into v4
  vmadc.vvm v1, v4, v8, v0  # Get carry into temp register v1
  vadc.vvm v4, v4, v8, v0   # Calc new sum
  vmcpy.m v0, v1             # Move temp carry into v0 for next word
```

The subtract with borrow instruction `vsbc` performs the equivalent function to support long word arithmetic for subtraction. There are no subtract with immediate instructions.

```
 # Produce difference with borrow.

# vd[i] = vs2[i] - vs1[i] - v0[i].LSB
 vsbc.vvm   vd, vs2, vs1, v0  # ベクトル - ベクトル

 # vd[i] = vs2[i] - x[rs1] - v0[i].LSB
 vsbc.vxm   vd, vs2, rs1, v0  # ベクトル - スカラ

 # Produce borrow out in mask register format

 # vd[i] = borrow_out(vs2[i] - vs1[i] - v0[i].LSB)
 vmsbc.vvm   vd, vs2, vs1, v0  # ベクトル - ベクトル

 # vd[i] = borrow_out(vs2[i] - x[rs1] - v0[i].LSB)
 vmsbc.vxm   vd, vs2, rs1, v0  # ベクトル - スカラ
```

For `vmsbc`, the borrow is defined to be 1 iff the difference, prior to truncation, is negative.

For `vadc` and `vsbc`, an illegal instruction exception is raised if the destination vector register is `v0` and LMUL > 1.

|      | This constraint corresponds to the constraint on masked vector operations that overwrite the mask register. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

For `vmadc` and `vmsbc`, an illegal instruction exception is raised if the destination vector register overlaps a source vector register group and LMUL > 1.

### 12.4. Vector Bitwise Logical Instructions

```
# Bitwise logical operations.
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

|      | With an immediate of -1, scalar-immediate forms of the `vxor` instruction provide a bitwise NOT operation. This can be provided as an assembler pseudoinstruction `vnot.v`. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 12.5. Vector Single-Width Bit Shift Instructions

A full complement of vector shift instructions are provided, including logical shift left, and logical (zero-extending) and arithmetic (sign-extending) shift right.

```
# Bit shift operations
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

Only the low lg2(SEW) bits are read to obtain the shift amount.

The immediate is treated as an unsigned shift amount, with a maximum shift amount of 31.

### 12.6. Vector Narrowing Integer Right Shift Instructions

The narrowing right shifts extract a smaller field from a wider operand and have both zero-extending (`srl`) and sign-extending (`sra`) forms. The shift amount can come from a vector or a scalar `x` register or a 5-bit immediate. The low lg2(2*SEW) bits of the vector or scalar shift amount value are used (e.g., the low 6 bits for a SEW=64-bit to SEW=32-bit narrowing operation). The unsigned immediate form supports shift amounts up to 31 only.

```
 # Narrowing shift right logical, SEW = (2*SEW) >> SEW
 vnsrl.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
 vnsrl.vx vd, vs2, rs1, vm   # ベクトル - スカラ
 vnsrl.vi vd, vs2, uimm, vm   # ベクトル - 即値

 # Narrowing shift right arithmetic, SEW = (2*SEW) >> SEW
 vnsra.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
 vnsra.vx vd, vs2, rs1, vm   # ベクトル - スカラ
 vnsra.vi vd, vs2, uimm, vm   # ベクトル - 即値
```

|      | It could be useful to add support for `n4` variants, where the destination is 1/4 width of source. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 12.7. Vector Integer Comparison Instructions

The following integer compare instructions write 1 to the destination mask register element if the comparison evaluates to true, and 0 otherwise. The destination mask vector is always held in a single vector register, with a layout of elements as described in Section [Mask Register Layout](https://riscv.github.io/documents/riscv-v-spec/#sec-mask-register-layout).

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

The following table indicates how all comparisons are implemented in native machine code.

```
Comparison      Assembler Mapping             Assembler Pseudoinstruction

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

|      | The immediate forms of `vmslt{u}.vi` are not provided as the immediate value can be decreased by 1 and the `vmsle{u}.vi` variants used instead. The `vmsle.vi` range is -16 to 15, resulting in an effective `vmslt.vi` range of -15 to 16. The `vmsleu.vi` range is 0 to 15 (and `(~0)-15` to `~0`), giving an effective `vmsltu.vi` range of 1 to 16 (Note, `vmsltu.vi` with immediate 0 is not useful as it is always false). Similarly, `vmsge{u}.vi` is not provided and the comparison is implemented using `vmsgt{u}.vi` with the immediate decremented by one. The resulting effective `vmsge.vi` range is -15 to 16, and the resulting effective `vmsgeu.vi` range is 1 to 16 (Note, `vmsgeu.vi` with immediate 0 is not useful as it is always true). |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | The `vmsgt` forms for register scalar and immediates are provided to allow a single comparison instruction to provide the correct polarity of mask value without using additional mask logical instructions. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

To reduce encoding space, the `vmsge{u}.vx` form is not directly provided, and so the `va ≥ x` case requires special treatment.

|      | The `vmsge{u}.vx` could potentially be encoded in a non-orthogonal way under the unused OPIVI variant of `vmslt{u}`. These would be the only instructions in OPIVI that use a scalar `x`register however. Alternatively, a further two funct6 encodings could be used, but these would have a different operand format (writes to mask register) than others in the same group of 8 funct6 encodings. The current PoR is to omit these instructions and to synthesize where needed as described below. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

The `vmsge{u}.vx` operation can be synthesized by reducing the value of `x` by 1 and using the `vmsgt{u}.vx` instruction, when it is known that this will not underflow the representation in `x`.

```
Sequences to synthesize `vmsge{u}.vx` instruction

va >= x,  x > minimum

   addi t0, x, -1; vmsgt{u}.vx vd, va, t0, vm
```

The above sequence will usually be the most efficient implementation, but assembler pseudoinstructions can be provided for cases where the range of `x` is unknown.

```
unmasked va >= x

  pseudoinstruction: vmsge{u}.vx vd, va, x
  expansion: vmslt{u}.vx vd, va, x; vmnand.mm vd, vd, vd

masked va >= x, vd != v0

  pseudoinstruction: vmsge{u}.vx vd, va, x, v0.t
  expansion: vmslt{u}.vx vd, va, x, v0.t; vmxor.mm vd, vd, v0

masked va >= x, any vd

  pseudoinstruction: vmsge{u}.vx vd, va, x, v0.t, vt
  expansion: vmslt{u}.vx vt, va, x;  vmandnot.mm vd, vd, vt

  The vt argument to the pseudoinstruction must name a temporary vector register that is
  not same as vd and which will be clobbered by the pseudoinstruction
```

Comparisons effectively AND in the mask, e.g,

```
    # (a < b) && (b < c) in two instructions
    vmslt.vv    v0, va, vb        # All body elements written
    vmslt.vv    v0, vb, vc, v0.t  # Only update at set mask
```

For all comparison instructions, an illegal instruction exception is raised if the destination vector register overlaps a source vector register group and LMUL > 1.

### 12.8. Vector Integer Min/Max Instructions

Signed and unsigned integer minimum and maximum instructions are supported.

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

### 12.9. Vector Single-Width Integer Multiply Instructions

The single-width multiply instructions perform a SEW-bit*SEW-bit multiply and return an SEW-bit-wide result. The `**mulh**` versions write the high word of the product to the destination register.

```
# Signed multiply, returning low bits of product
vmul.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
vmul.vx vd, vs2, rs1, vm   # ベクトル - スカラ

# Signed multiply, returning high bits of product
vmulh.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
vmulh.vx vd, vs2, rs1, vm   # ベクトル - スカラ

# Unsigned multiply, returning high bits of product
vmulhu.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
vmulhu.vx vd, vs2, rs1, vm   # ベクトル - スカラ

# Signed(vs2)-Unsigned multiply, returning high bits of product
vmulhsu.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
vmulhsu.vx vd, vs2, rs1, vm   # ベクトル - スカラ
```

|      | There is no `vmulhus` opcode to return high half of unsigned-vector * signed-scalar product. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | The current `vmulh*` opcodes perform simple fractional multiplies, but with no option to scale, round, and/or saturate the result. Can consider changing definition of `vmulh`, `vmulhu`, `vmulhsu` to use `vxrm` rounding mode when discarding low half of product. There is no possibility of overflow in this case. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 12.10. Vector Integer Divide Instructions

The divide and remainder instructions are equivalent to the RISC-V standard scalar integer multiply/divides, with the same results for extreme inputs.

```
    # Unsigned divide.
    vdivu.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
    vdivu.vx vd, vs2, rs1, vm   # ベクトル - スカラ

    # Signed divide
    vdiv.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
    vdiv.vx vd, vs2, rs1, vm   # ベクトル - スカラ

    # Unsigned remainder
    vremu.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
    vremu.vx vd, vs2, rs1, vm   # ベクトル - スカラ

    # Signed remainder
    vrem.vv vd, vs2, vs1, vm   # ベクトル - ベクトル
    vrem.vx vd, vs2, rs1, vm   # ベクトル - スカラ
```

|      | The decision to include integer divide and remainder was contentious. The argument in favor is that without a standard instruction, software would have to pick some algorithm to perform the operation, which would likely perform poorly on some microarchitectures versus others. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | There is no instruction to perform a "scalar divide by vector" operation. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 12.11. Vector Widening Integer Multiply Instructions

The widening integer multiply instructions return the full 2*SEW-bit product from an SEW-bit*SEW-bit multiply.

```
# Widening signed-integer multiply
vwmul.vv  vd, vs2, vs1, vm# ベクトル - ベクトル
vwmul.vx  vd, vs2, rs1, vm # ベクトル - スカラ

# Widening unsigned-integer multiply
vwmulu.vv vd, vs2, vs1, vm # ベクトル - ベクトル
vwmulu.vx vd, vs2, rs1, vm # ベクトル - スカラ

# Widening signed-unsigned integer multiply
vwmulsu.vv vd, vs2, vs1, vm # ベクトル - ベクトル
vwmulsu.vx vd, vs2, rs1, vm # ベクトル - スカラ
```

### 12.12. Vector Single-Width Integer Multiply-Add Instructions

The integer multiply-add instructions are destructive and are provided in two forms, one that overwrites the addend or minuend (`vmacc`, `vnmsac`) and one that overwrites the first multiplicand (`vmadd`, `vnmsub`).

The low half of the product is added or subtracted from the third operand.

|      | "sac" is intended to be read as "subtract from accumulator". The opcode is "vnmsac" to match the (unfortunately counterintuitive) floating-point `fnmsub` instruction definition. Similarly for the "vnmsub" opcode. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

```
# Integer multiply-add, overwrite addend
vmacc.vv vd, vs1, vs2, vm    # vd[i] = +(vs1[i] * vs2[i]) + vd[i]
vmacc.vx vd, rs1, vs2, vm    # vd[i] = +(x[rs1] * vs2[i]) + vd[i]

# Integer multiply-sub, overwrite minuend
vnmsac.vv vd, vs1, vs2, vm    # vd[i] = -(vs1[i] * vs2[i]) + vd[i]
vnmsac.vx vd, rs1, vs2, vm    # vd[i] = -(x[rs1] * vs2[i]) + vd[i]

# Integer multiply-add, overwrite multiplicand
vmadd.vv vd, vs1, vs2, vm    # vd[i] = (vs1[i] * vd[i]) + vs2[i]
vmadd.vx vd, rs1, vs2, vm    # vd[i] = (x[rs1] * vd[i]) + vs2[i]

# Integer multiply-sub, overwrite multiplicand
vnmsub.vv vd, vs1, vs2, vm    # vd[i] = -(vs1[i] * vd[i]) + vs2[i]
vnmsub.vx vd, rs1, vs2, vm    # vd[i] = -(x[rs1] * vd[i]) + vs2[i]
```

### 12.13. Vector Widening Integer Multiply-Add Instructions

The widening integer multiply-add instructions add a SEW-bit*SEW-bit multiply result to (from) a 2*SEW-bit value and produce a 2*SEW-bit result. All combinations of signed and unsigned multiply operands are supported.

```
# Widening unsigned-integer multiply-add, overwrite addend
vwmaccu.vv vd, vs1, vs2, vm    # vd[i] = +(vs1[i] * vs2[i]) + vd[i]
vwmaccu.vx vd, rs1, vs2, vm    # vd[i] = +(x[rs1] * vs2[i]) + vd[i]

# Widening signed-integer multiply-add, overwrite addend
vwmacc.vv vd, vs1, vs2, vm    # vd[i] = +(vs1[i] * vs2[i]) + vd[i]
vwmacc.vx vd, rs1, vs2, vm    # vd[i] = +(x[rs1] * vs2[i]) + vd[i]

# Widening signed-unsigned-integer multiply-add, overwrite addend
vwmaccsu.vv vd, vs1, vs2, vm    # vd[i] = +(signed(vs1[i]) * unsigned(vs2[i])) + vd[i]
vwmaccsu.vx vd, rs1, vs2, vm    # vd[i] = +(signed(x[rs1]) * unsigned(vs2[i])) + vd[i]

# Widening unsigned-signed-integer multiply-add, overwrite addend
vwmaccus.vx vd, rs1, vs2, vm    # vd[i] = +(unsigned(x[rs1]) * signed(vs2[i])) + vd[i]
```

### 12.14. Vector Integer Merge Instructions

The vector integer merge instructions combine two source operands based on the mask field. Unlike regular arithmetic instructions, the merge operates on all body elements (i.e., the set of elements from `vstart` up to the current vector length in `vl`).

The `vmerge` instructions are always masked (`vm=0`). The instructions combine two sources as follows. At elements where the mask value is zero, the first operand is copied to the destination element, otherwise the second operand is copied to the destination element. The first operand is always a vector register group specified by `vs2`. The second operand is a vector register group specified by `vs1` or a scalar `x` register specified by `rs1` or a 5-bit sign-extended immediate.

```
vmerge.vvm vd, vs2, vs1, v0  # vd[i] = v0[i].LSB ? vs1[i] : vs2[i]
vmerge.vxm vd, vs2, rs1, v0  # vd[i] = v0[i].LSB ? x[rs1] : vs2[i]
vmerge.vim vd, vs2, imm, v0  # vd[i] = v0[i].LSB ? imm    : vs2[i]
```

### 12.15. Vector Integer Move Instructions

The vector integer move instructions copy a source operand to a vector register group. These instructions are always unmasked (`vm=1`). The first operand specifier (`vs2`) must contain `v0`, and any other vector register number in `vs2` is *reserved*. This instruction copies the `vs1`, `rs1`, or immediate operand to the first `vl` locations of the destination vector register group.

```
vmv.v.v vd, vs1 # vd[i] = vs1[i]
vmv.v.x vd, rs1 # vd[i] = rs1
vmv.v.i vd, imm # vd[i] = imm
```

|      | Mask values can be widened into SEW-width elements using a sequence `vmv.v.i vd, 0; vmerge.vim vd, vd, 1, v0`. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | The vector integer move instructions share the encoding with the vector merge instructions, but with `vm=1` and `vs2=v0`. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

