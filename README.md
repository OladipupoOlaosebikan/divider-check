# DividerCheck

DividerCheck is a Fe library for checking a voltage divider against an ADC input limit across input-voltage and component tolerances. It can model both an unloaded divider and the divider loaded by an ADC's finite input resistance.

## Reproduce the demo

Install [Fe 26.6.0](https://github.com/argotorg/fe/releases/tag/v26.6.0). From this directory, run:

```text
fe --version
fe check .
fe test .
```

The test named `nominally_safe_divider_can_exceed_limit` is the demo. A 4.95 V input with 10 kΩ above the tap and 20 kΩ below it gives exactly 3.30 V at nominal values. With independent ±1% resistor tolerances, the enclosing output interval is 3.277926 V to 3.321927 V. The assessment flags the upper limit as exceeded. For the loaded model, 10 kΩ upper, lower, and ADC input resistances make the effective lower leg 5 kΩ, so a 10 V input produces 3.333333… V.

## How it works

`src/lib.fe` exports `Voltage`, `Resistor`, `Load`, `Assessment`, `assess`, and `assess_loaded`. Voltages are in integer microvolts, nominal resistances in integer ohms, and tolerances in basis points (100 basis points is 1%). `Load` represents the ADC's effective DC input resistance. The types distinguish voltage from resistance at compile time.

For the ideal divider, Vout = Vin × Rlower / (Rupper + Rlower). For the loaded divider, Vout = Vin × (Rlower × Rload) / (Rupper × (Rlower + Rload) + Rlower × Rload). The lower bound combines minimum input voltage with the component extremes that minimize output. The upper bound uses the opposite extremes. Keeping scaled resistances as integers preserves the ratio, and cross-multiplication checks the exact upper bound before rounding. The returned bounds round outward to whole microvolts. The calculation widens inputs to `u256` before multiplying so even the maximum `u64` inputs fit.

Both assessment functions return `None` for a reversed input range, zero nominal resistance, or tolerance of 100% or greater. The limit is inclusive. The loaded model assumes a DC-equivalent ADC input resistance and independent component tolerances. It does not model ADC sampling transients, leakage variation beyond the supplied tolerance, or temperature effects beyond the supplied tolerances.

## Contest disclosure

Built from scratch for the Fe Builders Challenge. The source has no external dependencies or reused third-party implementation. OpenAI Codex helped propose the project and draft the Fe implementation, test cases, and documentation.

Licensed under MIT.
