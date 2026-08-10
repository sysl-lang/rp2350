# rp2350

The RP2350's register map, as sysl constants — the chip on a Raspberry Pi Pico 2.

Base addresses, register offsets, field masks and shifts, and the vendor's own names for the values a
field takes. One module per peripheral, 54 of them, 32,980 constants.

```
dependencies {
  rp2350 { git = "github.com/sysl-lang/rp2350", version = "0.1.0" }
}
```

```sysl
import sh.sysl.rp2350.io_bank0

// Point GPIO 15 at the single-cycle IO block, which is what a plain output pin is.
val ctrl = io_bank0.BASE + io_bank0.GPIO15_CTRL_OFFSET
val funcsel = (read(ctrl) & io_bank0.GPIO15_CTRL_FUNCSEL_MASK) >> io_bank0.GPIO15_CTRL_FUNCSEL_SHIFT

if funcsel == io_bank0.GPIO15_CTRL_FUNCSEL_SIO_15 then ...
```

## It is generated, and must not be edited

Every `.sysl` file here comes from Raspberry Pi's own `RP2350.svd` — the CMSIS description shipped
inside the pico-sdk — by way of [sysl-lang/svd](https://github.com/sysl-lang/svd):

```
svd constants RP2350.svd -o . -m sh.sysl.rp2350
```

A hand edit is lost the next time the chip is regenerated. A correction belongs in the generator, or
in a newer SVD.

**`sh/sysl/rp2350/tests.sysl` is the one exception** — the only hand-written file, and the guard on
regeneration. It checks a handful of addresses and field positions against the **datasheet** rather
than against the SVD, so that a generator change or a filter left switched on has something
independent to fail against. `sysl test .` runs it.

It is deliberately small. Pinning all 32,980 constants would only assert that the generator agrees
with itself, which the generator's own golden tests already say.

## What it does not do

**It knows nothing about a board.** No pin assignments, no "the LED is on GPIO 25". A board's wiring
is a board's business.

**It knows nothing about drivers.** It does not know what SPI *is*; it knows where `SPI0_BASE` is and
which bits of `SSPCR0` mean what. A driver is a different package.

**It is not `sh.sysl.pico2`.** That one reaches the chip through the pico-sdk's C, and is how a
program using the SDK's build gets a working board. This one is the silicon, with no C at all and no
SDK — for code that would rather write the registers itself.

## What it costs

**Nothing in the binary.** Constants nobody names do not reach the executable: a program compiled
against this package is byte-identical to one compiled without it, verified against sysl 0.0.38. So
the map being complete costs a chip's flash nothing, which is why nothing here is filtered out —
including `OTP_DATA_RAW`'s 307 registers of fuse layout.

**At compile time, the whole package rather than the module you import.** Measured 2026-08-10, and it
is worth stating because the opposite is the natural assumption:

| this package, on `--lib` | added to each compile |
|---|---|
| as a source root | **+2.7 s** |
| as a prebuilt `.syslib` | **+0.3 s** |

Both are flat whether a program imports one module or all 54. One module per peripheral is a
readability decision, and a good one, but it does not make an unused peripheral free. Build the
artifact once if the wait matters.

## Capabilities

None. It declares constants — no function, no storage, no C, no allocator, no operating system — so it
is usable from a freestanding target, which is the only kind that wants it.

## Licence

ISC for the sysl in this repository. The register names, addresses and descriptions are Raspberry
Pi's, from `RP2350.svd` in the pico-sdk, which is BSD-3-Clause.
