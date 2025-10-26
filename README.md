# QPoland-Hackathon
# [REPORT](https://drive.google.com/file/d/1Zz4xTW4QkSV2XL6isgQ_von2vUobnW_5/view?usp=sharing)

Quantum Kernel Methods for Molecular Classification 
# [video explanation](https://drive.google.com/file/d/1rYQtFwgPOplyPN3nsYAjceQRI8MYTxLE/view?usp=sharing)


#[PPT](https://drive.google.com/file/d/1c76_NXuMKMVjhqJ0eWfoCSk6JRkooxtf/view?usp=drive_link)

## Hierarchical Topological Quantum Graph Kernels

---

## 1. Mathematical Foundation

### 1.1 Persistent Homology Extraction

For a molecular graph G = (V, E) with vertices V (atoms) and edges E (bonds), we construct a filtration based on chemical properties:

**Filtration Construction:**
```
F₀ ⊆ F₁ ⊆ ... ⊆ Fₙ = G
```

Where Fᵢ includes edges with bond strength ≤ threshold τᵢ:
- τ₀: hydrogen bonds
- τ₁: single bonds
- τ₂: double bonds  
- τ₃: triple bonds
- τ₄: aromatic bonds

**Betti Numbers at Each Scale:**
- β₀(Fᵢ): number of connected components
- β₁(Fᵢ): number of independent cycles (rings)
- β₂(Fᵢ): number of voids/cavities

**Persistent Homology Features:**
For each homology dimension k ∈ {0,1,2}, we extract:
```
PH_k = {(birth_i, death_i) | i ∈ features_k}
```

This gives us persistent diagrams encoding topological features across scales.

### 1.2 Feature Vectorization

From persistence diagrams, we construct feature vectors using:

**1. Statistical Summaries:**
```python
μₖ = mean(death - birth)  # Average persistence
σₖ = std(death - birth)   # Persistence variability  
εₖ = entropy(death - birth)  # Topological entropy
```

**2. Persistence Images:**
Map (birth, death) pairs to a 2D grid with Gaussian weighting:
```
PI(x,y) = Σᵢ w(pᵢ) · φ(x,y | pᵢ)
```
where w(pᵢ) = (death_i - birth_i)^γ weights by persistence.

**3. Betti Curves:**
```
β_k(t) = |{f ∈ PH_k : birth_f ≤ t ≤ death_f}|
```

For HTQ, we use the statistical summaries as our primary encoding.

### 1.3 Quantum State Encoding

Given topological features h = [β₀, β₁, β₂, μ₀, μ₁, μ₂, σ₀, σ₁, σ₂], we map to quantum state:

**Angle Computation:**
```python
θᵢ = 2π · normalize(hᵢ)  # Rotation angles
φᵢ = π · atan2(hᵢ, h_{i+1})  # Phase angles
```

**Quantum Circuit:**
```
|ψ(G)⟩ = U_ent · U_rot · |0⟩^⊗n

where:
U_rot = ⊗ᵢ RY(θᵢ) ⊗ RZ(φᵢ)
U_ent = Π_{i<j} CNOT(i,j)  [progressive entanglement]
```

**State Representation:**
For n=6 qubits, we have a 64-dimensional complex Hilbert space:
```
|ψ(G)⟩ = Σ_{s∈{0,1}^6} αₛ|s⟩
```
where |αₛ|² = probability of basis state s.

### 1.4 Quantum Fidelity Kernel

The kernel between two graphs G₁ and G₂ is:

```
K(G₁, G₂) = |⟨ψ(G₁)|ψ(G₂)⟩|²

Explicitly:
K(G₁, G₂) = |Σₛ α*ₛ(G₁) · αₛ(G₂)|²
```

**Properties:**
- Symmetric: K(G₁, G₂) = K(G₂, G₁)
- Positive semi-definite: Gram matrix is PSD
- Bounds: 0 ≤ K(G₁, G₂) ≤ 1
- Geometric interpretation: squared cosine of angle in Hilbert space

**Kernel Matrix:**
For dataset D = {G₁, ..., Gₙ}, we construct:
```
K_{ij} = |⟨ψ(Gᵢ)|ψ(Gⱼ)⟩|²


