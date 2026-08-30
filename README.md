# Probable 8-Ball Nudge Simulator

A billiards/pool-shot simulator where the cut angle on each shot is generated
by measuring a biased qubit on a quantum circuit (via
[Qiskit](https://www.ibm.com/quantum/qiskit)) instead of a classical PRNG.

What it does

`simulate(trials=N)` fires `N` virtual shots at a stationary target ball.
Each shot's cut angle comes from `nudge()`, a small quantum circuit that:

1. Puts several qubits into an equal superposition (`H` gates) to generate
   a roughly uniform random magnitude.
2. Applies an `RY` rotation to one "significant" qubit, biasing the shot
   toward small cut angles (gentle nudges) more often than large ones.
3. Uses a second, unbiased qubit as a fair coin to pick the sign
   (left/right cut).
4. Rejection-samples any draw that falls outside the target range.

The physics then applies the standard 90° rule for an equal-mass elastic
collision (cue and target angles are complementary) and a constant-friction
deceleration model to place both balls at their final resting positions.

`plot_nudge_distribution()` generates a validation histogram (see
`assets/nudge_distribution.png`) showing the measured `nudge()` output
actually matches the intended bias, rather than just asserting it in a
comment.

Why quantum randomness (and an honest caveat)

The circuit currently runs on **`AerSimulator`**, which does *not* produce
true quantum randomness. It's a classical simulator that samples from the
circuit's probability distribution using an ordinary PRNG under the hood.
Statistically it's indistinguishable from generating the same biased
distribution with `numpy.random`.

`QiskitRuntimeService` and `SamplerV2` are already imported to make it
straightforward to add a real-IBM-QPU code path later (see Roadmap) — that's
the point at which the randomness would become genuinely quantum-sourced.

Physics assumptions

| Quantity | Value | Notes |
|---|---|---|
| Ball mass | 0.17 kg | Standard pool ball |
| Coefficient of friction | 0.125 | Cloth-on-ball |
| Incoming cue speed | 2 m/s | |
| Gravity | 10 m/s² | |
| Deceleration | `μ·g` = 1.25 m/s² | Derived, not assumed |

Post-collision, the target ball takes the cut angle `θ` and the cue ball
deflects at `90° − θ` (or `90° + θ` for the opposite sign), per the
standard elastic-collision rule for equal masses. Distance traveled after
the collision follows `d = v² / (2a)` under constant friction deceleration.

Setup

```bash
pip install -r requirements.txt
```

Everything runs locally on `AerSimulator`, so no IBM account needed.

Usage

Open `notebook/probable_8_ball_nudge_simulator.ipynb` and run all cells, or:

```python
simulate(trials=10)          # plot final ball positions
plot_nudge_distribution()    # validate the nudge() output shape
```

Roadmap

- [ ] Wire up `QiskitRuntimeService` / `SamplerV2` to optionally run `nudge()`
      on a real IBM Quantum backend, for genuinely quantum-sourced randomness
      instead of Aer's classical simulation.
- [ ] Batch draws into one job instead of one job per shot, since real
      hardware meters QPU time per job — this matters a lot once real
      hardware is wired in.
## License

MIT — see `LICENSE`.
