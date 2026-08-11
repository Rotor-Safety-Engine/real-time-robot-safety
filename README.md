# Real-Time Robot Safety — Sub-Millisecond Physics Validation

**Real-time robot safety** systems for collaborative robots, humanoid robots, and VLA-based automation. When a VLA model outputs an action, you need a safety check that finishes before the next control cycle — and it needs to be **100% deterministic**.

---

## Why Real-Time Safety Matters

Traditional industrial safety relies on fixed fences and interlocks. But collaborative robots, humanoid robots, and **VLA-controlled systems** operate in unstructured environments where hazards can't be pre-defined. The safety system has to react in real time to each individual action.

### The latency problem with cloud-based VLA safety

If your safety check requires a round-trip to the cloud (like calling a VLA API to "verify safety"), the latency chain looks like:

```
Action generation → Network → Cloud inference → Network → Controller → Actuator
= 500ms – 2000ms+ total
```

In 500ms, a robot arm moving at 1m/s travels **50cm** — enough to cause serious injury before the safety check even returns.

**Real-time safety requires local, deterministic, sub-millisecond execution.**

---

## What Makes a Safety System "Real-Time"?

### 1. Sub-millisecond latency
Per-check latency well under 1ms, ideally under 100μs, so it fits comfortably within robot control cycles (typically 1ms – 10ms).

### 2. Deterministic execution
Same input → same output, always. No probability, no statistical variance. This is a **core requirement of ISO 10218** for safety-rated functions — a probabilistic model with 99% accuracy is not acceptable as a primary safety mechanism.

### 3. Local execution
Runs on the same device as the controller — no network dependency, no single point of failure.

### 4. Zero dependency
The safety layer should not depend on complex runtime environments, ML frameworks, or external services. Safety is the last thing that should fail.

### Comparison of approaches

| Approach | Latency | Deterministic | Local | Zero Dep | ISO 10218 Aligned |
|----------|---------|--------------|-------|----------|-------------------|
| Cloud VLA safety check | 500-2000ms | ❌ No | ❌ No | ❌ No | ❌ Not for safety-rated |
| ML-based on-device | 10-100ms | ❌ No | ✅ Yes | ❌ No | ❌ Not verifiable |
| Rule-based controller | 0.1-1ms | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Traditional approach |
| **Physics-based deterministic** | **<50μs** | **✅ Yes** | **✅ Yes** | **✅ Yes** | **✅ Fully aligned** |

---

## Real-Time Physics Safety Architecture

A modern real-time safety system for VLA and humanoid robots typically has these layers:

### Layer 1: Semantic Plausibility Check
Fast rejection of physically impossible actions (e.g., "grasp water" — semantic verb-object conflict). Microseconds to reject.

### Layer 2: Safety Parameter Mapping
Maps the action type and context to the appropriate safety thresholds.

### Layer 3: Dynamic Physics Analysis
The core computational layer — calculates forces, pressures, impulses, and stability in real time:

- **Dynamic contact area** — real-time contact patch size based on force and stiffness
- **Impulse boundary** — mass × velocity constraints to prevent high-energy impacts
- **Reaction force stability** — chassis friction and weight limits to prevent tipping

### Layer 4: Comprehensive Decision
Aggregates all checks, outputs risk level (not just binary safe/unsafe), and provides retreat parameters for safe recovery.

---

## Rotor Safety Engine — Production-Grade Real-Time Safety

→ **[Rotor Safety Engine](https://github.com/Rotor-Safety-Engine/safety-engine)**

Rotor is a **deterministic, physics-based real-time safety layer** designed for VLA models, humanoid robots, and collaborative robots. It implements the full 4-layer architecture described above.

### Key performance:

- **~17μs per full safety check** (Python, single-threaded)
- **~60,000 checks per second** throughput
- **100% deterministic** — same input always produces same output
- **Single Python file, zero dependencies** — drop into any project
- **349 test cases, 100% pass rate**

### How it integrates with VLA:

```
VLA Model (action generation)
    ↓
Rotor Safety Engine (local, ~17μs)
    ↓ ── safe → Controller → Robot
    ↓
    └── unsafe → Block / Modify + over_ratio feedback
```

The VLA model continues to handle perception, planning, and high-level decision-making. Rotor sits between the VLA output and the physical robot as a **deterministic safety gate** — analogous to how industrial robots have separate safety-rated controllers.

### Use cases:

- **Collaborative robot safety** — PFL verification aligned with ISO 10218 / ISO/TS 15066
- **Humanoid robot safety** — whole-body contact and stability analysis
- **VLA safety layer** — physical validation for every VLA-generated action
- **Edge safety inference** — runs on the robot itself, no cloud dependency
- **Embodied AI safety** — grounding AI actions in physical reality

---

## Performance Benchmarks

| Metric | Value | Notes |
|--------|-------|-------|
| Single check latency | ~17μs | Python 3.10, x86_64 |
| Throughput | ~60,000/s | Single thread |
| Memory footprint | ~140KB | Single file |
| Determinism | 100% | Same input → same output |
| Dependencies | 0 | Standard library only |

---

## Related Concepts

- Deterministic safety
- Physics-based safety
- ISO 10218
- ISO/TS 15066
- Dynamic contact area
- Impulse safety boundary
- Reaction force stability
- VLA safety
- Humanoid robot safety
- Embodied AI safety
- Collision detection
- Cobot safety
- Power and Force Limiting (PFL)

## License

MIT — educational and reference use.
