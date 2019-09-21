## 16. Vector Mask Instructions

Several instructions are provided to help operate on mask values held in a vector register.

### 16.1. Vector Mask-Register Logical Instructions

Vector mask-register logical operations operate on mask registers. The size of one element in a mask register is SEW/LMUL, so these instructions all operate on single vector registers regardless of the setting of the `vlmul` field in `vtype`. They do not change the value of `vlmul`.

As with other vector instructions, the elements with indices less than `vstart` are unchanged, and `vstart` is reset to zero after execution. Vector mask logical instructions are always unmasked so there are no inactive elements. Mask elements past `vl`, the tail elements, are zeroed.

Within a mask element, these instructions perform their operations using only the least-significant bit of each operand and zero-extend the single-bit result to fill the destination mask element.

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

Several assembler pseudoinstructions are defined as shorthand for common uses of mask logical operations:

```
    vmcpy.m vd, vs  => vmand.mm vd, vs, vs  # Copy mask register
    vmclr.m vd     => vmxor.mm vd, vd, vd   # Clear mask register
    vmset.m vd     => vmxnor.mm vd, vd, vd  # Set mask register
    vmnot.m vd, vs => vmnand.mm vd, vs, vs  # Invert bits
```

|      | The vmcpy.m instruction is not called vmmv as elsewhere in the architecture mv implies a bitwise copy without interpreting the bits. The vmcpy.m instruction will clear upper bits of the destination mask register to zero regardless of source values in these bits. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

The set of eight mask logical instructions can generate any of the 16 possibly binary logical functions of the two input masks:

| inputs |      |      |      |      |
| :----- | :--- | ---- | ---- | ---- |
| 0      | 0    | 1    | 1    | src1 |
| 0      | 1    | 0    | 1    | src2 |

| output | instruction | pseudoinstruction |      |                            |                  |
| :----- | :---------- | :---------------- | ---- | -------------------------- | ---------------- |
| 0      | 0           | 0                 | 0    | vmxor.mm vd, vd, vd        | vmclr.m vd       |
| 1      | 0           | 0                 | 0    | vmnor.mm vd, src1, src2    |                  |
| 0      | 1           | 0                 | 0    | vmandnot.mm vd, src2, src1 |                  |
| 1      | 1           | 0                 | 0    | vmnand.mm vd, src1, src1   | vmnot.m vd, src1 |
| 0      | 0           | 1                 | 0    | vmandnot.mm vd, src1, src2 |                  |
| 1      | 0           | 1                 | 0    | vmnand.mm vd, src2, src2   | vmnot.m vd, src2 |
| 0      | 1           | 1                 | 0    | vmxor.mm vd, src1, src2    |                  |
| 1      | 1           | 1                 | 0    | vmnand.mm vd, src1, src2   |                  |
| 0      | 0           | 0                 | 1    | vmand.mm vd, src1, src2    |                  |
| 1      | 0           | 0                 | 1    | vmxnor.mm vd, src1, src2   |                  |
| 0      | 1           | 0                 | 1    | vmand.mm vd, src2, src2    | vmcpy.m vd, src2 |
| 1      | 1           | 0                 | 1    | vmornot.mm vd, src2, src1  |                  |
| 0      | 0           | 1                 | 1    | vmand.mm vd, src1, src1    | vmcpy.m vd, src1 |
| 1      | 0           | 1                 | 1    | vmornot.mm vd, src1, src2  |                  |
| 1      | 1           | 1                 | 1    | vmxnor.mm vd, vd, vd       | vmset.m vd       |

|      | The vector mask logical instructions are designed to be easily fused with a following masked vector operation to effectively expand the number of predicate registers by moving values into `v0` before use. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 16.2. Vector mask population count `vpopc`

```
    vpopc.m rd, vs2, vm
```

The source operand is a single vector register holding mask register values as described in Section [Mask Register Layout](https://riscv.github.io/documents/riscv-v-spec/#sec-mask-register-layout).

The `vpopc.m` instruction counts the number of mask elements of the active elements of the vector source mask register that have their least-significant bit set, and writes the result to a scalar `x` register.

The operation can be performed under a mask, in which case only the masked elements are counted.

```
 vpopc.m rd, vs2, v0.t # x[rd] = sum_i ( vs2[i].LSB && v0[i].LSB )
```

Traps on `vpopc.m` are always reported with a `vstart` of 0. The `vpopc` instruction will raise an illegal instruction exception if `vstart` is non-zero.

### 16.3. `vfirst` find-first-set mask bit

```
    vfirst.m rd, vs2, vm
```

The `vfirst` instruction finds the lowest-numbered active element of the source mask vector that has its LSB set and writes that element’s index to a GPR. If no element has an LSB set, -1 is written to the GPR.

|      | Software can assume that any negative value (highest bit set) corresponds to no element found, as vector lengths will never exceed 231 on any implementation. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Traps on `vfirst` are always reported with a `vstart` of 0. The `vfirst` instruction will raise an illegal instruction exception if `vstart` is non-zero.

### 16.4. `vmsbf.m` set-before-first mask bit

```
    vmsbf.m vd, vs2, vm

 # Example

     7 6 5 4 3 2 1 0   Element number

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

The `vmsbf.m` instruction takes a mask register as input and writes results to a mask register. The instruction writes a 1 to all active mask elements before the first source element that has a set LSB, then writes a zero to that element and all following active elements. If there is no set bit in the source vector, then all active elements in the destination are written with a 1.

The tail elements in the destination mask register are cleared.

Traps on `vmsbf.m` are always reported with a `vstart` of 0. The `vmsbf` instruction will raise an illegal instruction exception if `vstart` is non-zero.

### 16.5. `vmsif.m` set-including-first mask bit

The vector mask set-including-first instruction is similar to set-before-first, except it also includes the element with a set bit.

```
    vmsif.m vd, vs2, vm

 # Example

     7 6 5 4 3 2 1 0   Element number

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

The tail elements in the destination mask register are cleared.

Traps on `vmsif.m` are always reported with a `vstart` of 0. The `vmsif` instruction will raise an illegal instruction exception if `vstart` is non-zero.

### 16.6. `vmsof.m` set-only-first mask bit

The vector mask set-only-first instruction is similar to set-before-first, except it only sets the first element with a bit set, if any.

```
    vmsof.m vd, vs2, vm

 # Example

     7 6 5 4 3 2 1 0   Element number

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

Traps on `vmsof.m` are always reported with a `vstart` of 0. The `vmsof` instruction will raise an illegal instruction exception if `vstart` is non-zero.

### 16.7. Example using vector mask instructions

The following is an example of vectorizing a data-dependent exit loop.

```
  # char* strcpy(char *dst, const char* src)
strcpy:
      mv a2, a0             # Copy dst
loop:
    vsetvli x0, x0, e8   # Max length vectors of bytes
    vlbuff.v v1, (a1)       # Get src bytes
      csrr t1, vl           # Get number of bytes fetched
    vmseq.vi v0, v1, 0      # Flag zero bytes
    vfirst.m a3, v0         # Zero found?
      add a1, a1, t1        # Bump pointer
    vmsif.m v0, v0          # Set mask up to and including zero byte.
    vsb.v v1, (a2), v0.t    # Write out bytes
      add a2, a2, t1        # Bump pointer
      bltz a3, loop         # Zero byte not found, so loop

      ret

  # char* strncpy(char *dst, const char* src, size_t n)
strncpy:
      mv a3, a0             # Copy dst
loop:
    vsetvli x0, a2, e8   # Vectors of bytes.
    vlbuff.v v1, (a1)       # Get src bytes
    vmseq.vi v0, v1, 0      # Flag zero bytes
    vfirst.m a4, v0         # Zero found?
    vmsif.m v0, v0          # Set mask up to and including zero byte.
    vsb.v v1, (a3), v0.t    # Write out bytes
      bgez a4, exit         # Done
      csrr t1, vl           # Get number of bytes fetched
      add a1, a1, t1        # Bump pointer
      sub a2, a2, t1        # Decrement count.
      add a3, a3, t1        # Bump pointer
      bnez a2, loop         # Anymore?

exit:
      ret
```

### 16.8. Vector Iota Instruction

The `viota.m` instruction reads a source vector mask register and writes to each element of the destination vector register group the sum of all the least-significant bits of elements in the mask register whose index is less than the element, e.g., a parallel prefix sum of the mask values.

This instruction can be masked, in which case only the enabled elements contribute to the sum and only the enabled elements are written.

```
 viota.m vd, vs2, vm

 # Example

     7 6 5 4 3 2 1 0   Element number

     1 0 0 1 0 0 0 1   v2 contents
                       viota.m v4, v2 # Unmasked
     2 2 2 1 1 1 1 0   v4 result

     1 1 1 0 1 0 1 1   v0 contents
     1 0 0 1 0 0 0 1   v2 contents
     2 3 4 5 6 7 8 9   v4 contents
                       viota.m v4, v2, v0.t # Masked
     1 1 1 5 1 7 1 0   v4 results
```

The result value is zero-extended to fill the destination element if SEW is wider than the result. If the result value would overflow the destination SEW, the least-significant SEW bits are retained.

Traps on `viota.m` are always reported with a `vstart` of 0, and execution is always restarted from the beginning when resuming after a trap handler. An illegal instruction exception is raised if `vstart` is non-zero.

An illegal instruction exception is raised if the destination vector register group overlaps the source vector mask register. If the instruction is masked, an illegal instruction exception is issued if the destination vector register group overlaps `v0`.

|      | These constraints exist for two reasons. First, to simplify avoidance of WAR hazards in implementations with temporally long vector registers and no vector register renaming. Second, to enable resuming execution after a trap simpler. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

The `viota.m` instruction can be combined with memory scatter instructions (indexed stores) to perform vector compress functions.

```
    # Compact non-zero elements from input memory array to output memory array
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
    li a6, 0                      # Clear count of non-zero elements
loop:
    vsetvli a5, a0, e32,m8  # 32-bit integers
    vlw.v v8, (a1)                # Load input vector
      sub a0, a0, a5              # Decrement number done
      slli a5, a5, 2              # Multiply by four bytes
    vmsne.vi v0, v8, 0            # Locate non-zero values
      add a1, a1, a5              # Bump input pointer
    vpopc.m a5, v0                # Count number of elements set in v0
    viota.m v16, v0               # Get destination offsets of active elements
      add a6, a6, a5              # Accumulate number of elements
    vsll.vi v16, v16, 2, v0.t     # Multiply offsets by four bytes
      slli a5, a5, 2              # Multiply number of non-zero elements by four bytes
    vsuxw.v v8, (a2), v16, v0.t   # Scatter using scaled viota results under mask
      add a2, a2, a5              # Bump output pointer
      bnez a0, loop               # Any more?

      mv a0, a6                   # Return count
      ret
```

### 16.9. Vector Element Index Instruction

The `vid.v` instruction writes each element’s index to the destination vector register group, from 0 to `vl`-1.

```
    vid.v vd, vm  # Write element ID to destination.
```

The instruction can be masked.

The `vs2` field of the instruction must be set to `v0`, otherwise the encoding is *reserved*.

The result value is zero-extended to fill the destination element if SEW is wider than the result. If the result value would overflow the destination SEW, the least-significant SEW bits are retained.

|      | This constraint is to avoid WAR hazards in long vector implementations without register renaming, and to support restart. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | Microarchitectures can implement `vid.v` instruction using the same datapath as `viota.m` but with an implicit set mask source. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |


  