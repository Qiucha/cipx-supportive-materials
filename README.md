# PAI1 supporting materials

## Week 3: Maximum likelihood estimation

[Cellphone localization demo](cellphone_localization_3d.html) — explore how reception measurements from three cell towers constrain a phone's position, and how gradient ascent searches for a maximum-likelihood estimate.

### Open the demo

Download `cellphone_localization_3d.html` and open it in a modern browser with WebGL enabled. On GitHub, use **Download raw file**; clicking the link above displays the source. You can also download the repository as a ZIP, extract it, and open the HTML file.

No installation, build step, or local server is required. An internet connection is needed to load Three.js from its CDN; styles are included in the HTML.

### Try it

- Click **Climb** to animate ascent, or **+1 Step** to advance once. **Reset** returns to the chosen starting point.
- Adjust the starting coordinates and step size γ (0.25–3, default 0.5). Changing either resets the climb and updates the predicted path. Small γ crawls and may stop at the 50-step limit; around 0.5–1.25 the climb converges smoothly; from about 1.5 it overshoots, by 2–2.5 it zig-zags, and near 3 it fails to converge.
- Move the actual phone with its **X** and **Y** sliders; reception counts and surfaces update, and the climb resets.
- Click **Resample** to draw new random reception counts and watch the maximum-likelihood estimate scatter around the true phone; **MLE error** shows the distance between them. **Expected** restores the deterministic counts.
- Switch between **Likelihood** and **Log-likelihood** to change surface height. This preserves the climb position and reprojects its path; ascent uses the log-likelihood gradient in both views.
- Toggle **Combine towers** beside the display controls to replace individual tower surfaces with their combined model: summed log-likelihoods or multiplied likelihoods. Toggle it off to restore the separate surfaces and combined wireframe. This preserves the climb and uses only enabled towers.
- Move or disable towers; compare **Tri** and **Collinear** layouts. With collinear towers, try starting on opposite sides of the tower line to explore ambiguous solutions.
- Drag to orbit, right-drag to pan, and scroll to zoom. **3D**, **Top**, and **Side** select camera views.

### What the model shows

The true phone position starts at `(4, 2)` and can be moved with the phone sliders. Each tower is 8 units above the ground and receives a transmission with probability

```text
p(d) = 1 / (1 + exp((d − 32) / 4.5)),
```

where `d` is the 3D distance from the ground-level phone to the tower. For each tower, the demo uses `k = round(50 × p(d_true))` successful receptions out of 50 transmissions by default. These are deterministic, rounded expected counts. **Resample** switches to random counts `k ~ Binomial(50, p(d_true))`, drawn independently per tower; while resampling is active, moving the phone or towers draws new counts.

Assuming independent receptions, the position-dependent log-likelihood is

```text
ℓ(x, y) = Σ [kᵢ log pᵢ(x, y) + (50 − kᵢ) log(1 − pᵢ(x, y))],
```

summed over enabled towers. Terms constant in position are omitted. The climber follows `(x, y) ← (x, y) + γ ∇ℓ(x, y)`.

- **Cyan, amber, and purple surfaces:** the individual models for towers A, B, and C. Disabled towers disappear. Each surface is normalized to its own grid maximum to make the tower constraints visible; their displayed heights are not additive.
- **Pale wireframe (filled surface with Combine towers enabled):** the combined model, formed by summing log-likelihoods (or multiplying likelihoods). The climber and both paths follow this surface.
- **Likelihood height:** `20 × exp(ℓ − ℓ_grid max)`. **Log-likelihood height:** `20 × max(0, 1 + (ℓ − ℓ_grid max) / 18)`, a shifted, scaled log-likelihood with a floor at 18 log units below the maximum. The same transforms apply to each individual model. These relative displays avoid tiny likelihood values and negative log heights; they are not physical altitudes. Near-zero triangles are omitted for visibility.
- **Lime phone and ground ring:** actual phone position, drawn through the surfaces to keep it visible. Its model is symbolic; reception distances use a ground-level point. **Blue diamond:** combined maximum on a 55 × 55 grid spanning `[-35, 35]` on each ground axis; it is an approximation, and rounded counts can shift the estimate away from the true position.
- **Colored arrows:** individual tower gradients and their sum. These are log-likelihood gradients, with lengths scaled and capped for display.
- **Amber path:** predicted ascent from the selected start. **Rose path:** steps already taken. Both use at most 50 steps and stop when the gradient norm falls below 0.04; stopping at the limit does not imply convergence.

One tower generally leaves a ring of possible positions; collinear towers can leave mirror-image solutions. With all towers disabled, the surfaces and maximum marker are hidden because there is no position information. Gradient ascent depends on the starting point and step size and does not guarantee a global maximum.

### Implementation

The demo stays in one HTML file with embedded CSS and JavaScript and a pinned Three.js r128 CDN dependency. It renders only when the scene changes, reuses trajectory buffers and dot meshes, and recomputes grid likelihoods only when phone or tower settings change. Switching display modes reuses the cached grid values; trajectory and surface buffers are reused.
