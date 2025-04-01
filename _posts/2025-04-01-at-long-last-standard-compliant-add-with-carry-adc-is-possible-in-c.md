---
layout: post
title: Standard compliant add-with-carry (adc) in C23
date: 2025-04-01 11:32 -0400
---

The official release of C23 five months ago (October 31, 2024) brought a ... of syntactical and standard library changes improvements.

The most interesting being in the form of '<stdbit.h>' and '<stdckdint.h>', which provide standardized bit functions and checked integer operations respectively.

The nonstandard method for implementing add with carry has been to use compiler-specific instrinsic functions/builtins, such as `__builtin_add_overflow`, which is supported by both Clang and GCC.

These compilers also include `__builtin_addc()`, which they both implement through something like this:

```c
  ({ __typeof__ (a) s; \
      __typeof__ (a) c1 = __builtin_add_overflow (a, b, &s); \
      __typeof__ (a) c2 = __builtin_add_overflow (s, carry_in, &s); \
      *(carry_out) = c1 | c2; \
      s; })
```

Now, with '<stdckdint.h>' under C23, we can leaverage the `ckd_add()` function, which is functionally equivalent to `__builtin_add_overflow`:

```c
#include <stdckdint.h>

#define stdc_addc( result, a, b, cin ) \
    ( ckd_add(result, a, b) | ckd_add(result, *result, (typeof(*result))cin) )

bool add_words(unsigned int n, unsigned *result, unsigned *a, unsigned *b) {
    bool carry = false;
    for(unsigned i = 0; i < n; i++) {
        carry = stdc_addc(&result[i], a[i], b[i], carry);
    }
    return carry;
}
```
