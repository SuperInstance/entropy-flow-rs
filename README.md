# Entropy Flow (Rust)

**Entropy Flow (Rust)** is a Rust library implementing information-theoretic measures — Shannon entropy, KL divergence, Jensen-Shannon divergence, and mutual information — for real-time conservation-law monitoring in the SuperInstance fleet.

## Why It Matters

Entropy measures quantify uncertainty, which is the mathematical foundation for understanding agent behavior diversity. In the SuperInstance conservation framework, Shannon entropy over the ternary action distribution {Avoid, Unknown, Choose} measures how "decided" a population is. Low entropy means most agents choose the same action (consensus); high entropy means actions are uniformly distributed (uncertainty). The conservation law (Law 5) predicts that the entropy of the action distribution should be stable across scales — the Rust implementation provides the real-time, low-latency computation needed for live fleet monitoring, complementing the Python implementation's use in offline analysis.

## How It Works

**Shannon entropy:**
```
H(X) = −Σ pᵢ × log₂(pᵢ)
```

For ternary action distribution {p_avoid, p_unknown, p_choose}:
- Minimum H = 0 (all agents choose the same action — pure consensus)
- Maximum H = log₂(3) ≈ 1.585 bits (uniform — maximum uncertainty)

**KL divergence:**
```
D(P‖Q) = Σ pᵢ × ln(pᵢ / qᵢ)
```

Used to compare observed action distribution P against the theoretical conservation prediction Q (294:1 avoidance-to-choose ratio). If D(P‖Q) exceeds threshold, conservation is violated.

**Jensen-Shannon divergence:**
```
JS(P‖Q) = ½ D(P‖M) + ½ D(Q‖M),  M = ½(P + Q)
```

Symmetric and bounded — preferred over KL for comparative analysis because it doesn't require absolute continuity (Qᵢ > 0 wherever Pᵢ > 0).

**Mutual information:**
```
I(X; Y) = H(X) − H(X | Y)
```

Measures the reduction in uncertainty about X when Y is known. In the fleet, this quantifies how much the γ-layer (observed actions) tells us about the η-layer (internal model state).

**Complexity:**

| Measure | Time | Space |
|---------|------|-------|
| Shannon entropy | O(n) | O(k) for k bins |
| KL divergence | O(n) | O(k) |
| JS divergence | O(n) | O(k) |
| Mutual information | O(n × bins²) | O(bins²) |

## Quick Start

```rust
fn main() {
    println!("Entropy Flow: information-theoretic measures for fleet conservation.");
    // In the fleet:
    // 1. Collect action distribution from agents
    // 2. Compute Shannon entropy to measure population uncertainty
    // 3. Compute KL divergence vs expected conservation distribution
    // 4. Alert if divergence exceeds threshold (conservation violation)
}
```

## API

| Function | Description |
|----------|-------------|
| Shannon entropy | H(X) in bits or nats |
| KL divergence | D(P‖Q), asymmetric |
| JS divergence | Symmetric, bounded |
| Mutual information | I(X; Y) reduction in uncertainty |

## Architecture Notes

Entropy Flow (Rust) provides the **real-time entropy monitoring** for γ + η = C. It runs in the η-layer, continuously computing entropy of the γ-layer's action stream. When entropy deviates beyond conservation tolerance, the fleet's conservation-law verification triggers. The Rust implementation ensures sub-millisecond computation for fleet-scale monitoring.

See [ARCHITECTURE.md](https://github.com/SuperInstance/SuperInstance/blob/main/ARCHITECTURE.md).

**Numerical stability:** Computing KL divergence requires evaluating pᵢ × log(pᵢ/qᵢ). When qᵢ is very small (near zero) but pᵢ is non-zero, the ratio pᵢ/qᵢ explodes, causing floating-point overflow. This implementation uses the numerically stable identity: p × log(p/q) = p × log(p) − p × log(q), with the convention 0 × log(0) = 0 (by L'Hôpital's rule, lim_{x→0+} x log x = 0). This avoids division and handles zero-probability events gracefully.

**Connection to Law 5 (Conservation):** The conservation law states that the avoidance ratio A = n_avoid / n_total is invariant across population scales. Entropy-wise, this means H({A, U, C}) should also be scale-invariant, since H depends only on the proportions. The Rust implementation computes H in O(n) per observation window, enabling sliding-window entropy monitoring at fleet scale. When H deviates beyond σ = 0.001 from its theoretical value (H(294/295, ε, 1/295) ≈ 0.012 bits), the conservation violation is flagged.

## References

1. Shannon, C.E. (1948). "A Mathematical Theory of Communication." *Bell System Technical Journal*, 27.
2. Cover, T.M. & Thomas, J.A. (2006). *Elements of Information Theory*. 2nd ed. Wiley.
3. MacKay, D.J.C. (2003). *Information Theory, Inference, and Learning Algorithms*. Cambridge University Press. Chapter 2: Probability, Entropy, and Inference.

## License

MIT
