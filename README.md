# NetSpectra

**Higher‑order statistics for hybrid (farm + region) dynamic movement networks.**  
A compact simulator and analysis scaffold for *temporal edge/node appearance*, *random‑walk mass*, and *directed spectral signatures* (magnetic Laplacian).

[![Version](https://img.shields.io/badge/version-v0.10--alpha-3b82f6)](#)
![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)
[![Last Commit](https://img.shields.io/github/last-commit/EvoLandEco/netspectra)](https://github.com/EvoLandEco/netspectra)

---

## What is NetSpectra?

NetSpectra proposes a **hybrid dynamic graph representation** for livestock movement networks:

- Keep **farm-level** detail inside one *focal* region (e.g., a COROP region in NL),
- Contract everything outside into **regional supernodes** and **superflows**,
- Analyze day-by-day dynamics with **higher‑order statistics** that go beyond static degree counts.

This lets you preserve local detail where you care most, while still retaining a coherent national “context backbone”.

For formal definitions and interpretation guides, see:  
**https://qtj.me/blog-hybrid-corop-farm.html**

Network simulator             |  Higher-order statistics
:-------------------------:|:-------------------------:
![Screenshot 1](assets/screenshots/UI-A.png)  |  ![Screenshot 2](assets/screenshots/UI-B.png)

---

## What’s included

- An interactive D3 simulator that generates a 10‑day hybrid network on the NL COROP map and visualizes:
  - **TEA / TNA**: temporal edge / node appearance (new / reactivated / persistent / churn)
  - **π‑mass**: stationary random-walk mass on Farms vs Regions (LIC-restricted)
  - **Magnetic Laplacian spectra**: directed spectral signatures (with quality + weighting toggles)
- COROP boundaries + label points (`nl_corop.geojson`, `nl_corop_labelpoint.geojson`)

> If you fork this for a different country, replace the two GeoJSONs in `assets/files/` and preserve the `statcode` / `statnaam` fields, or update the property lookups in the HTML.

---

## Quickstart

### 1) Clone
```bash
git clone https://github.com/EvoLandEco/netspectra.git
cd netspectra
```

### 2) Serve the repository root
Browsers block `fetch()` from `file://`, so run a tiny local server from the repo root:

```bash
python -m http.server 8000
```

### 3) Open in browser
```text
http://localhost:8000/hybrid_simulator.html
```

Keep the bundled asset paths in place (or update the paths inside the HTML):
- `assets/files/nl_corop.geojson`
- `assets/files/nl_corop_labelpoint.geojson`

The simulator currently loads D3, Turf, numeric.js, and iframe-resizer from public CDNs, so the browser session needs internet access unless you vendor those dependencies locally.

---

## How the simulator works

Each run builds a 10-day directed hybrid network. The selected focal COROP is represented at farm level: `nFarms` farm nodes are sampled inside the focal polygon, while every non-focal COROP becomes one regional supernode positioned at its label point.

Eligible directed pairs are split into four blocks: farm→farm, farm→region, region→farm, and region→region. On each day, yesterday’s edges persist with probability `p_keep`; all remaining eligible pairs can appear as new edges with probability `pBirthFF` for farm→farm pairs and `pBirthX` for every cross-region block. Each new edge receives a simulated positive trade volume, and persisted edges keep their volume with a small random perturbation.

Statistics are computed from each day’s directed graph as follows:

- **TEA / TNA** compare the current edge set or active-node set with the previous day and report persistent, reactivated, new, and churned activity, plus average novelty and persistence ratios across the 10 days.
- **π-mass** builds a lazy row-stochastic walk, restricts it to the day’s largest irreducible / strongly connected component, computes the stationary distribution, and sums that mass over farm nodes versus region nodes.
- **Magnetic spectra** build a normalized magnetic Laplacian from the directed adjacency matrix. `Fast` computes every other day and linearly interpolates skipped days, `Balanced` computes daily on the active induced subgraph, and `Full` computes daily on the full hybrid node set.
- **Weight mode** affects π-mass and magnetic spectra only: `Unweighted` treats every edge as weight 1, while `Weighted` uses the simulated trade volumes.

---

## How to use

1. **Pick a focal region** and **farm count** (5-100).
2. Tune network dynamics:
   - **p_keep** controls persistence of yesterday’s edges,
   - **Birth prob inside FF** controls new farm→farm edges,
   - **Birth prob (FR/RF/RR)** controls new farm↔region and region→region edges.
3. Use the **Day** slider to step through time.
4. Use the **Unweighted / Weighted** switch to decide whether:
   - π‑mass and magnetic spectra treat each edge equally (**unweighted**), or
   - they incorporate simulated trade volume (**weighted**).
5. Use **Spectrum quality** to trade speed for fidelity:
   - **Fast** samples spectra every other day and interpolates the skipped days,
   - **Balanced** computes spectra daily on the active induced subgraph,
   - **Full** computes spectra daily on the full hybrid node set.

The status badge in the map panel shows progress; hover it to reveal **Cancel** while heavy computations run.

---

## Embedding in a blog

The simulator supports:
- automatic height resizing using **iframe-resizer**, and
- dark-mode sync with a site-level `#darkmode` checkbox (if present).

See the embedding snippet and explanation in:  
**https://qtj.me/blog-hybrid-corop-farm.html**

---

## Acknowledgements

NetSpectra’s simulator uses:
- **D3.js** for visualization,
- **Turf.js** for point-in-polygon sampling,
- **numeric.js** for eigen-decomposition in the Web Worker,
- **iframe-resizer** for clean iframe embedding.

---

## License

MIT License.
