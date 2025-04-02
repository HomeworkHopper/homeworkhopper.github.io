---
layout: post
title: Standard compliant add-with-carry in C23
date: 2025-04-01 11:32 -0400
categories: [Programming, C]
tags: [adc, c23, assembly]
description: Short summary of the post.
---

The release of C23 has brought a

[n2683.pdf](/assets/pdf/n2683.pdf)

of syntactical and standard library changes improvements.

The most interesting being in the form of '<stdbit.h>' and '<stdckdint.h>', which provide standardized bit functions and checked integer operations respectively.

The nonstandard method for implementing add with carry has been to use compiler-specific intrinsic functions/builtins, such as `__builtin_add_overflow()`, which is supported by both Clang and GCC.

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

static unsigned stdc_addcu (
	unsigned a, unsigned b, bool c_in, bool *c_out
) {
	unsigned result;
	const bool c1 = ckd_add(&result, a, b);
	const bool c2 = ckd_add(&result, result, (unsigned)c_in);
	*c_out = c1 | c2;
	return result;
}
```

If we examine the assembly produced by Clang 20.1.0, it's clear that no ADC instruction is present:

However, this makes perfect sense: `c_in` could have come from anywhere -- in this way, it is clear that a true add-with-carry can only occur within context.

```c
void add_uwords(unsigned *result, unsigned *a, unsigned *b, int n)  {
	bool carry = false;
	for(int i = 0; i < n; i++) {
		result[i] = stdc_addcu(a[i], b[i], carry, &carry);
	}
}
```
