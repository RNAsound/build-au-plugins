---
name: dsp-filters-dynamics
description: Use when designing or debugging plugin filters, modulation-safe coefficient updates, compressors, limiters, detectors, gain reduction, lookahead, latency, and true-peak behavior.
---

# DSP Filters And Dynamics

Use this skill for filters, modulation behavior, compressors, limiters, and gain processing in audio plugins.

## Filter guidance
- Distinguish static filtering from heavily modulated filtering.
- Cascaded biquads are usually fine for mostly static filters.
- Use TPT or state-variable structures for continuously modulated filters.
- Avoid naive linear interpolation of biquad coefficients under heavy modulation.
- Smooth the user parameter, then update coefficients at controlled boundaries.
- Compute sensitive coefficients in higher precision when needed.
- Clamp and sanity-check externally influenced values.

## Filter decision rules
- Static EQ:
  Use biquads or vDSP where appropriate.
- Fast cutoff modulation:
  Prefer state-variable or topology-preserving structures.
- Resonance near Nyquist:
  Test stability and coefficient conditioning aggressively.
- Nonlinear filters:
  Choose structure for stability, not only tone.
- UI-controlled frequency sweeps:
  Smooth parameter motion and regression-test rapid sweeps.

## Dynamics guidance
- Use peak or true-peak detection when protection is the product promise.
- Use RMS or energy detection when loudness shaping is the goal.
- Use feedforward designs when predictability and stability matter.
- Add lookahead only when the gain computer needs preemption.
- Use soft knee for smoother threshold transitions.
- Use hard or brickwall behavior when peak containment is the primary requirement.
- Smooth meters independently from gain computation.

## Limiter and detector choices
| Style | Latency | Protection | Sonic character | Best use |
|---|---:|---:|---|---|
| Brickwall peak limiter | Low to medium | High on sample peaks | Assertive | Track protection, live-safe ceilings |
| Brickwall true-peak limiter | Medium to high | Highest for delivery ceilings | Cleaner but costlier | Master-bus/export protection |
| Soft limiter | Low | Moderate | Forgiving, colored | Musical saturation/protection |
| RMS leveler | Low to medium | Lower for fast transients | Smooth | Program leveling, bus control |
| Peak compressor | Low | Strong transient control | Reactive | Drums, transient management |
| RMS compressor | Medium | Weaker transient catch | Smooth | Vocals, buses, leveling |

## Safety rules
- Declare plugin latency equal to lookahead plus oversampling or linear-phase latency that affects the rendered signal.
- Provide a soft bypass or gain handoff path.
- Keep gain-reduction meter smoothing separate from detector timing.
- Use inter-sample-aware peak measurement for mastering limiters or ceiling tools.
- Reset or quarantine NaN and Inf states immediately.
- Add denormal handling for long silence and high-feedback paths.

## Implementation checklist
1. Choose detector type before tuning attack and release.
2. Choose feedforward or feedback topology intentionally.
3. Define latency and tail behavior before host validation.
4. Add bypass tests for clicks and discontinuities.
5. Add rapid automation tests for threshold, ratio, makeup, mix, and ceiling.
6. Validate true peak for delivery-oriented limiters.
7. Run silence tests long enough to expose denormal behavior.
