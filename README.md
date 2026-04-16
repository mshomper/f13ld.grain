# F13LD.grain

**Part of the [F13LD](https://f13ld.app) tool suite by [Not a Robot Engineering](https://notarobot-eng.com)**

**[Open the Tool](https://mshomper.github.io/f13ld.grain)**

A browser-based implicit field explorer for generating directionally biased stochastic metamaterial scaffolds. Three mathematically distinct field types share a common design pipeline: interactive 2D section view, WebGL raymarched 3D preview, MIL-HS mechanical homogenization, and JSON export for downstream rendering and validation.

All geometry is fully implicit — no meshes, no STL intermediates, no skeleton graphs. Structure is defined as a scalar field evaluated on demand at any point in space.

---

## Overview

Traditional periodic lattices (TPMS, BCC, FCC) offer predictable but rigid geometry. Real biological structures — trabecular bone, cartilage, insect cuticle — are stochastic: they achieve mechanical efficiency through statistical isotropy or anisotropy, not periodicity. This tool lets you explore that design space.

The three field types occupy distinct positions in the stochastic geometry landscape:

| Field type | Mechanism | Natural topology | Beam character | Directional control |
|---|---|---|---|---|
| Spinodoid | VMF wave superposition | Sheet / half-solid | Emergent at low VF + high κ | Strong — VMF concentration |
| Gaussian random field | Spectral envelope on magnitude | Sheet / half-solid | Moderate | Moderate — per-axis frequency |
| Hyperuniform kernel | Oriented Gaussian kernels on jittered grid | Half-solid | Explicit — set by aspect ratio | Good — VMF on kernel axes |

---

## Usage

Open `spinodoid_tool.html` directly in a browser. No server, no build step, no dependencies.

**Recommended workflow:**

1. Select a field type and set directional bias
2. Explore parameters in **Section 2D** — fast, interactive, full resolution
3. Switch to **Field 3D** to read the overall spatial character
4. Run **homogenization** to get effective mechanical properties
5. **Export JSON** — carries all parameters and homogenization results
6. Feed the JSON to your offline renderer or FFT homogenization pipeline

---

## Field Types

### Spinodoid (VMF wave superposition)

```
φ(x) = (1/√N) · Σᵢ cos(kᵢ · x + θᵢ)
```

Wave vectors **kᵢ** are drawn from a von Mises-Fisher distribution concentrated around your principal direction(s). This is the same class of fields produced by spinodal decomposition in materials science, and the closest mathematical analog to real trabecular bone microstructure.

At low volume fraction and high κ, the minority phase forms beam-like rods. At mid volume fraction, it forms interpenetrating sheets. The field is statistically homogeneous but not periodic — each seed produces a different valid realization of the same structural family.

**Parameters:**
- **N waves** — number of superposed cosines. More waves → more statistically converged, less sensitive to seed. 32–64 is the practical range.
- **Frequency** — spatial frequency of features (features per unit domain). Lower = coarser structure.
- **RNG seed** — selects one realization from the statistical family. Different seeds produce different geometry with identical statistical properties.

### Gaussian Random Field (spectral)

```
φ̂(k) = √S(k) · ξ(k)    [spectral domain]
```

Wave vector **magnitudes** are drawn from a Gaussian spectral envelope centered at `k₀ = 2π·freq`, rather than being approximately uniform as in spinodoids. This creates a preferred **length scale** (consistent pore size) rather than a preferred **direction**. Directional anisotropy is introduced by making `k₀` depend on the projection of the wave direction onto your principal axes.

The practical difference from spinodoid is visible in section view: GRF produces more consistently sized features with less directional streaking at the same κ. Both are valid — the choice depends on whether length-scale uniformity or directional alignment is the design priority.

**Additional parameter:**
- **Bandwidth σ** — spectral width as a fraction of k₀. Low σ → near-periodic cells of consistent size. High σ → broad irregular spectrum, more organic character.

### Hyperuniform Kernel Field

```
φ(x) = Σᵢ exp(-‖Rᵢᵀ(x - xᵢ)‖²_ellipsoid) - 0.3
```

Kernel centers **xᵢ** are placed via a stratified jittered grid (hyperuniform distribution — suppressed long-range density fluctuations, no clustering, guaranteed whole-domain coverage). Each kernel gets a VMF-sampled orientation **θᵢ** and an elongated Gaussian shape. The isosurface of the summed field produces smooth beam-like geometry whose character is directly set by the kernel shape.

This is the most direct route to beam-like implicit geometry without an explicit skeleton. Unlike spinodoid, where beam character emerges statistically from field topology at low VF, the hyperuniform field is geometrically beam-like by construction.

**Additional parameters:**
- **Aspect ratio** — kernel length-to-width ratio. 1 = spherical blobs (closed-cell foam). 4–8 = distinct organic beam-like struts. 10+ = near-cylindrical rods.
- **Kernel width** — cross-section diameter of each kernel in normalized coordinates. Controls strut thickness independently of length.
- **N kernels** — number of kernel centers. More kernels → denser, more connected network. The jittered grid placement guarantees uniform spatial distribution at any N.

> **3D render performance:** Each kernel requires an `exp()` evaluation per ray step. N > 80 may be slow in the 3D view. Use Section 2D for high-N exploration; the Python renderer handles production voxelization offline.

---

## Directional Bias

Three modes, shared across all field types:

**Single** — one principal direction specified by polar angle θ and azimuth φ. θ=0 is +Z (aligned columns). Sweep θ to tilt the beam direction relative to the build axis.

**Orthotropic** — three independent VMF components along X, Y, Z with separate weight sliders. Produces transversely isotropic, cubic, or fully orthotropic symmetry classes depending on weight ratios.

**Isotropic** — κ is ignored, wave vectors / kernel orientations are drawn uniformly from the full sphere. Produces isotropic stochastic foam. Statistically equivalent to your noise tool at the same topology.

**Concentration κ** — controls how tightly the directional distribution clusters around the principal direction(s).
- κ = 0 → isotropic (equivalent to Isotropic mode)
- κ ≈ 4–8 → organic trabecular character
- κ ≥ 12 → strongly aligned, near-columnar

---

## Scaffold Geometry

**Topology** controls how the scalar field is converted to a solid/void geometry:

| Mode | SDF expression | Character |
|---|---|---|
| Sheet | `abs(φ - center) < half-width` | Two surfaces bounding a hollow channel |
| Half-solid | `φ > center` | One solid domain with open pore network — the natural spinodoid topology |
| Solid | `abs(φ - center) > half-width` (inverted) | Filled slab |

- **Center** — shifts the isosurface threshold. In half-solid mode this directly controls volume fraction: negative = more solid, positive = less.
- **Half-width** — slab thickness in sheet and solid modes. Hidden in half-solid mode.
- **Smoothing** — separable box-filter blur applied before isosurface extraction. Rounds sharp junctions and reduces noise in low-N fields.

---

## Homogenization (MIL-HS)

Estimates effective elastic properties from the voxelized scalar field using the Mean Intercept Length method coupled to the Hashin-Shtrikman upper bound.

**Outputs:**
- **Volume fraction ρ** — solid phase fraction
- **Ex, Ey, Ez** — directional Young's moduli (GPa)
- **Gxy** — shear modulus (GPa)
- **νeff** — effective Poisson's ratio
- **Zener A** — anisotropy ratio. A = 1 is perfectly isotropic. A < 1 is axially dominant (stiffer along principal axis). A > 1 is diagonally dominant.
- **Stiffness ellipsoid** — rotating WebGL visualization of the directional stiffness surface

> MIL-HS assumes statistical homogeneity. Results improve with higher grid resolution and higher N waves / N kernels. For spinodoid and GRF fields, higher frequency also improves convergence.

**E_s (GPa)** — solid material stiffness. Set to your material of interest (Ti6Al4V ≈ 114 GPa, PEEK ≈ 3.6 GPa, stainless 316L ≈ 193 GPa).

---

## Export JSON

The exported JSON carries the complete parameter set and, if homogenization has been run, the mechanical results. This file is the handoff point to the offline pipeline.

```json
{
  "meta": { "version": "0.1.0", "tool": "spinodoid-field-explorer", "timestamp": "..." },
  "field": {
    "type": "spinodoid | gaussian | hyperuniform",
    "n_waves": 48,
    "kappa": 6.0,
    "frequency": 0.15,
    "rng_seed": 42,
    "dir_mode": "single | ortho | iso",
    "principal_direction": [0, 0, 1],
    "ortho_weights": null
  },
  "geometry": {
    "center": 0.0,
    "half_width": 0.15,
    "smoothing": 0.0,
    "topology": "sheet | half | solid"
  },
  "homogenization": {
    "grid": 48,
    "E_solid_GPa": 100,
    "volume_fraction": 28.4,
    "Ex_GPa": 3.21,
    ...
  }
}
```

---

## Part of the F13LD Suite

| Tool | Field type | Topology | Directional control |
|---|---|---|---|
| [TPMS Builder](https://mshomper.github.io/tpms-builder) | Periodic trigonometric | Sheet / strut | Crystallographic symmetry |
| Noise Explorer | Isotropic stochastic | Sheet / beam foam | None |
| **Spinodoid Explorer** | Anisotropic stochastic | Sheet / half-solid / solid | VMF + orthotropic |

---

## Technical Notes

**Fully implicit principle** — all geometry is defined as a scalar field evaluated pointwise. No mesh generation, no STL intermediates, no explicit surface representation exists until marching cubes is intentionally invoked in the offline renderer.

**3D view** — WebGL1 raymarcher. Wave-based fields (spinodoid, GRF) use sphere tracing with a Lipschitz-estimated step size. The hyperuniform kernel field uses fixed-step marching calibrated to kernel width, since the Gaussian kernel sum is not a true SDF. Internal render resolution is capped at 420px for performance; CSS upscaling handles display.

**Section 2D** — CPU rasterization of the scalar field at the chosen axis/position. Always accurate regardless of field type or N. Recommended for parameter exploration, especially at high N kernels.

**Browser compatibility** — Chrome and Firefox. WebGL1 required for 3D view. Section 2D works without WebGL.

---

## License

MIT — see LICENSE file.

*Not a Robot Engineering · notarobot-eng.com*
