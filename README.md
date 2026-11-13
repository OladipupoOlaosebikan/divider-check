# DividerCheck

DividerCheck is a Fe library for checking whether a two-resistor voltage divider stays at or below an ADC input limit across input-voltage and resistor tolerances.

## Reproduce the demo

Install [Fe 26.6.0](https://github.com/argotorg/fe/releases/tag/v26.6.0). From this directory, run:

```text
fe --version
fe check .
fe test .
```

The test named `nominally_safe_divider_can_exceed_limit` is the demo. A 4.95 V input with 10 kΩ above the tap and 20 kΩ below it gives exactly 3.30 V at nominal values. With independent ±1% resistor tolerances, the enclosing output interval is 3.277926 V to 3.321927 V. The assessment flags the upper limit as exceeded.

## How it works

`src/lib.fe` exports `Voltage`, `Resistor`, `Assessment`, and `assess`. Voltages are in integer microvolts, nominal resistances in integer ohms, and tolerances in basis points (100 basis points is 1%). The types distinguish voltage from resistance at compile time.

For the ideal divider, Vout = Vin × Rlower / (Rupper + Rlower). The lower bound uses minimum input voltage, maximum upper resistance, and minimum lower resistance. The upper bound uses the opposite extremes. Keeping scaled resistances as integers preserves the ratio, and cross-multiplication checks the exact upper bound before rounding. The returned bounds round outward to whole microvolts. The calculation widens inputs to `u256` before multiplying so even the maximum `u64` inputs fit.

`assess` returns `None` for a reversed input range, zero nominal resistance, or tolerance of 100% or greater. The limit is inclusive. The model assumes DC, an unloaded output, and independent resistor tolerances. ADC input loading, sampling transients, and temperature effects beyond the supplied tolerance are outside the model.

## Contest disclosure

Built from scratch for the Fe Builders Challenge. The source has no external dependencies or reused third-party implementation. OpenAI Codex helped propose the project and draft the Fe implementation, test cases, and documentation.

Licensed under MIT.
