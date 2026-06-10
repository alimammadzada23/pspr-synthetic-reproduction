# PSPR Synthetic Data Reproduction

This repository is a synthetic-data reproduction attempt of the paper:

**Spatially resolved phase reconstruction for atom interferometry**
Seckmeyer et al., *EPJ Quantum Technology*, 2025.

The goal is not to reproduce the full experimental part of the paper. The real experimental atom-gravimeter image datasets are not publicly included in the paper. Instead, this repository reproduces the synthetic-data workflow using the mathematical model described in the paper.


## Installation

```bash
pip install -r requirements.txt
```

Required packages:

```text
numpy
scipy
matplotlib
scikit-learn
scikit-image
```

## How to run

## How to run

Open `pspr_synthetic_reproduction.ipynb` in Google Colab or Jupyter Notebook and run all cells.

The notebook generates synthetic interferometer images, applies PCA-based spatial phase reconstruction, adds atom shot noise, and saves result figures into the `figures/` folder.

## Step 1: Synthetic data generation

In the first step, synthetic atom-interferometer images are generated from the model in the paper.

The synthetic image stack has the form:

```text
I(i, x, y) = A(x, y) + B(x, y) cos(theta(i) + gamma(x, y))
```

Here:

* `i` is the image index,
* `(x, y)` is the pixel position,
* `theta(i)` is the phase shift between images,
* `gamma(x, y)` is the spatial phase pattern,
* `A(x, y)` and `B(x, y)` describe the image background and envelope.

Three spatial phase patterns are tested:

```text
1) gamma pattern based on 2x² + 4y
2) gamma pattern based on 4x + 4y
3) gamma pattern based on 4(0.2x)² + 2y
```

Result: synthetic two-port interferometer images are created. These images act as artificial camera data.


### Step 1 outputs

![Theta scan images] (figures:step1_mean_image_true_phase.png)

![Mean image and true phase] (figures:step1_theta_scan_images.png)

![Three phase patterns] (figures:step1_three_phase_patterns.png)

---

## Step 2: Noise-free PSPR reconstruction

In the second step, the synthetic images are analyzed using a PCA-based spatial phase reconstruction method.

The reconstruction pipeline is:

```text
synthetic images
→ PCA decomposition
→ coefficient ellipse correction
→ arctan2 phase extraction
→ reconstructed spatial phase map
```

Because the input data is noise-free, the reconstructed spatial phase map should match the true phase map almost exactly.

Result: the reconstruction error is close to numerical precision, around `10^-15` to `10^-16` radians depending on the run and machine precision.

### Step 2 output

![Noise-free PSPR reconstruction](step2_noise_free_reconstruction.png)

---

## Step 3: Shot-noise simulation

In the third step, atom shot noise is added to the synthetic images.

The noise-free image intensity is treated as a probability distribution. Then a fixed number of atoms is randomly sampled into pixels. This creates noisy atom-count images.

The noisy data is then analyzed with the same PSPR reconstruction method.

Three scaling tests are performed:

### Figure 5 style result

The number of atoms per image is varied.

Expected behavior:

```text
max(sigma_i) ∝ 1 / sqrt(N_atoms)
```

This means that using more atoms improves the phase reconstruction accuracy.

Example output:

```text
figures/fig5_atom_number_scaling.png
```

### Figure 6 style result

The number of images in the dataset is varied.

Expected behavior: after enough images, the image-phase reconstruction error becomes almost independent of the number of images.

Example output:

```text
figures/fig6_image_number_scaling.png
```

### Figure 7 style result

The pixel-wise spatial phase error is compared with the average number of atoms per pixel.

Expected behavior:

```text
sigma_xy ∝ 1 / sqrt(n_images * Ibar)
```

where `Ibar` is the average number of atoms per pixel per image.

Example output:

```text
figures/fig7_pixel_phase_error.png
```
### Step 3 outputs

![Figure 5: atom number scaling](step3_fig5_atom_number_scaling.png)

![Figure 6: image number scaling](step3_fig6_1_image_number_scaling.png)

![Figure 6: image number scaling](step3_fig6_2_image_number_scaling.png)

![Figure 7: pixel phase error](step3_fig7_pixel_phase_error.png)
---

## Notes

This repository reproduces only the synthetic-data part of the paper.

The experimental atom-gravimeter image datasets are not included because they are not publicly available in the paper. The paper states that those datasets are available from the corresponding authors on reasonable request.

The Monte Carlo results may not look exactly identical to the paper because the paper used a much larger number of simulation runs. This repository uses smaller run counts so the code can run on a normal laptop or Google Colab.

## License

This repository is for educational and research reproduction purposes.
