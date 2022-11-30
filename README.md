# interflop-backend-mcaquad

# Arguments
The MCA backends implement Montecarlo Arithmetic.
- `libinterflop_mca.so`: uses floating point types to represent stochastic
  noise.  It uses quad type to compute MCA operations on doubles and double
  type to compute MCA operations on floats.

```bash
VFC_BACKENDS="libinterflop_mca.so --help" ./test
test: verificarlo loaded backend libinterflop_mca.so
Usage: libinterflop_mca.so [OPTION...]

  -m, --mode=MODE            select MCA mode among {ieee, mca, pb, rr}
      --precision-binary32=PRECISION
                             select precision for binary32 (PRECISION >= 0)
      --precision-binary64=PRECISION
                             select precision for binary64 (PRECISION >= 0)
      --error-mode=ERR_MODE  select error mode among (rel, abs, all)
      --max-abs-error-exponent=ERR_EXPONENT
                             select the magnitude of the maximum allowed
                             absolute error (this option is only used when
                             error-mode={abs, all})
  -d, --daz                  denormals-are-zero: sets denormals inputs to zero
  -f, --ftz                  flush-to-zero: sets denormal output to zero
  -s, --seed=SEED            fix the random generator seed
  -?, --help                 Give this help list
      --usage                Give a short usage message
```

Two options control the behavior of the MCA backend.

The option `--mode=MODE` controls the arithmetic error mode. It accepts the
following case insensitive values:

 * `mca`: (default mode) Montecarlo Arithmetic with inbound and outbound errors
 * `ieee`: the program uses standard IEEE arithmetic, no errors are introduced
 * `pb`: Precision Bounding inbound errors only
 * `rr`: Random Rounding outbound errors only

The option `--precision-binary64=PRECISION` controls the virtual
precision used for the floating point operations in double precision
(respectively for single precision with --precision-binary32). It
accepts an integer value that represents the virtual precision at
which MCA operations are performed. Its default value is 53 for
binary64 and 24 for binary32. A precise definition of the
virtual precision is given [here](https://hal.archives-ouvertes.fr/hal-01192668).

One should note when using the QUAD backend, that the round operations during
MCA computation always use round-to-zero mode.

In Random Round mode, the exact operations in given virtual precision are
preserved.

The option `--error-mode=ERR_MODE` controls the way in which the error is
interpreted. It accepts the following modes:

 * `rel`: (default mode) the error is specified relative to the magnitude of
 the floating-point number
 * `abs`: the error threshold is specified as an absolute value, independent of
the value of the floating-point number, to be interpreted as 2<sup>ERR_EXPONENT</sup>
 * `all`: both relative and absolute modes are active simultaneously

The option `--max-abs-error-exponent=ERR_EXPONENT` is used only when the option
`--error-mode=ERR_MODE` is active and controls the magnitude of the error
threshold, when in absolute error mode or all mode. The error thershold is set
to 2<sup>ERR_EXPONENT</sup>.

The options `--daz` and `--ftz` flush subnormal numbers to 0.
The `--daz` (**Denormals-Are-Zero**) flushes subnormal inputs to 0.
The `--ftz` (**Flush-To-Zero**) flushes subnormal output to 0.

```bash
   $ VFC_BACKENDS="libinterflop_mca.so --mode=ieee" ./test
   0x0.fffffep-126 +0x1.000000p-149 = 0x1.000000p-126
   $ VFC_BACKENDS="libinterflop_mca.so --mode=ieee --daz" ./test
   0x0.fffffep-126 +0x1.000000p-149 = 0x0
   $ VFC_BACKENDS="libinterflop_mca.so --mode=ieee --ftz" ./test
   0x0.fffffep-126 +0x1.000000p-149 = 0x1.000000p-126
```

The option `--seed` fixes the random generator seed. It should not generally be used
except if one to reproduce a particular MCA trace.
