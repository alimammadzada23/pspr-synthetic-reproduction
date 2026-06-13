# PSPR Synthetic Data Reproduction

This repository is a synthetic-data reproduction of the method described in:

**Spatially resolved phase reconstruction for atom interferometry**
Seckmeyer et al., *EPJ Quantum Technology*, 2025.

The goal is not to reproduce the full experimental part of the paper. The real experimental atom-gravimeter image datasets are not publicly included with the paper. Instead, this repository reproduces the **synthetic-data workflow** using the mathematical image model described in the paper.

The project generates artificial two-port atom-interferometer images, applies PCA-based spatial phase reconstruction, and tests the effect of atom shot noise on the reconstructed phase.

---

## Project goal

Atom interferometers usually measure a global phase. However, real atom-cloud images can also contain spatially varying phase information across the cloud.

The goal of PSPR, or **spatially resolved phase reconstruction**, is to reconstruct a spatial phase map:

```text
gamma(x, y)
```

from a stack of interferometer images.

In this synthetic reproduction, the true phase map is known in advance. This allows us to test whether the PCA-based reconstruction pipeline can recover the input phase map correctly.

---

## Installation

Install the required Python packages with:

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

---

## How to run

Open the notebook:

```text
pspr_synthetic_reproduction.ipynb
```

in Google Colab or Jupyter Notebook and run all cells.

The notebook generates synthetic interferometer images, applies PCA-based spatial phase reconstruction, adds atom shot noise, and produces the figures shown below.

The figures in this README assume that the PNG files are in the same folder as the README. If you keep them inside a `figures/` folder, change the image paths from

```text
step1_mean_image_true_phase.png
```

to

```text
figures/step1_mean_image_true_phase.png
```

and similarly for the other images.

---

# Method overview

The synthetic image stack is generated from the interferometer image model:

```text
I_i(x, y) = A(x, y) + B(x, y) * cos(theta_i + gamma(x, y))
```

Here:

```text
I_i(x, y)      = intensity at pixel position (x, y) in image i

A(x, y)        = mean image background / density envelope

B(x, y)        = local fringe amplitude

theta_i        = global interferometer phase of image i

gamma(x, y)    = spatial phase pattern to be reconstructed
```

The useful trigonometric identity is:

```text
cos(theta_i + gamma(x, y))
=
cos(theta_i) * cos(gamma(x, y))
-
sin(theta_i) * sin(gamma(x, y))
```

This identity shows why PCA can work.

The changing part of the image can be written as two main components:

```text
cos(theta_i) * cos(gamma(x, y))
```

and

```text
sin(theta_i) * sin(gamma(x, y))
```

So the image stack mainly varies in two dimensions:

```text
1. a cosine-like mode
2. a sine-like mode
```

PCA is used to recover these two dominant modes from the image sequence.

The reconstruction pipeline is:

```text
synthetic image stack
→ subtract mean image
→ PCA decomposition with two components
→ ellipse correction of PCA scores
→ arctan2 phase extraction
→ reconstructed global phases theta_i
→ reconstructed spatial phase map gamma(x, y)
```

---

# Step 1: Synthetic data generation

In Step 1, synthetic atom-interferometer images are generated.

The image model is:

```text
I_i(x, y) = A(x, y) + B(x, y) * cos(theta_i + gamma(x, y))
```

The phase `theta_i` changes from image to image. It represents the global interferometer phase scan.

The spatial phase `gamma(x, y)` changes across the image. This is the phase map that the algorithm should reconstruct.

Three spatial phase patterns are tested:

```text
Pattern 1:

gamma_1(x, y) = 2*x^2 + 4*y
```

```text
Pattern 2:

gamma_2(x, y) = 4*x + 4*y
```

```text
Pattern 3:

gamma_3(x, y) = 4*(0.2*x)^2 + 2*y
```

The synthetic images are two-port interferometer images. The two output ports are modelled as separated atom clouds. A relative pi phase shift is included between the two ports so that they are complementary, as expected for an atom interferometer.

That means when one port is bright, the other port is dark.

---

## Step 1 outputs

### Mean image and true phase

![Mean image and true phase](step1_mean_image_true_phase.png)

### Theta scan images

![Theta scan images](step1_theta_scan_images.png)

### Three tested phase patterns

![Three phase patterns](step1_three_phase_patterns.png)

---

# Step 2: Noise-free PSPR reconstruction

In Step 2, the synthetic images are reconstructed without atom shot noise.

First, each image is flattened into a vector. If an image has shape:

```text
height x width
```

then it becomes one long vector.

A stack of images becomes a data matrix:

```text
number of images x number of pixels
```

After subtracting the mean image, PCA is applied to extract the two dominant components.

The reason this works is the identity:

```text
cos(theta_i + gamma(x, y))
=
cos(theta_i) * cos(gamma(x, y))
-
sin(theta_i) * sin(gamma(x, y))
```

So the mean-subtracted image can be written approximately as:

```text
I_i(x, y) - A(x, y)
=
B(x, y) * cos(theta_i) * cos(gamma(x, y))
-
B(x, y) * sin(theta_i) * sin(gamma(x, y))
```

This shows that the image variation is approximately two-dimensional.

Therefore, PCA can recover two modes corresponding to the sine-like and cosine-like fringe components.

---

## PCA scores and ellipse correction

For each image, PCA gives two coefficients:

```text
w_i1 and w_i2
```

Ideally, these coefficients should lie on a circle as `theta_i` is scanned.

In practice, PCA can rotate and scale the two components, so the points often form an ellipse instead of a perfect circle.

Therefore, an ellipse correction is applied to map the PCA scores back to a circle.

After ellipse correction, the global phase of each image is reconstructed as:

```text
theta_i_rec = atan2(w_i2, w_i1)
```

The spatial phase map is reconstructed from the corrected PCA components `P1(x, y)` and `P2(x, y)` as:

```text
gamma_rec(x, y) = atan2(P2(x, y), P1(x, y))
```

Because the input data is noise-free, the reconstructed phase should match the true phase almost exactly.

In the notebook, the noise-free reconstruction error is close to numerical precision, typically around:

```text
10^-15 to 10^-16 rad
```

---

## Step 2 output

![Noise-free PSPR reconstruction](step2_noise_free_reconstruction.png)

---

# Step 3: Shot-noise simulation

In Step 3, atom shot noise is added to the synthetic images.

The noise-free image intensity is treated as a probability distribution. Then a fixed number of atoms is randomly sampled into pixels.

This means that an ideal smooth image becomes a noisy atom-count image.

The noisy images are then analyzed with the same PSPR reconstruction pipeline:

```text
noisy atom-count images
→ PCA
→ ellipse correction
→ arctan2 phase extraction
→ reconstructed spatial phase map
```

---

## Atom shot noise

If the total number of atoms is `N_atoms`, the relative atom shot noise scales approximately as:

```text
relative shot noise ~ 1 / sqrt(N_atoms)
```

Therefore, using more atoms improves the reconstruction accuracy.

---

## Figure 5 style test: atom number scaling

In this test, the number of atoms per image is varied.

The expected scaling is:

```text
max(sigma_i) ∝ 1 / sqrt(N_atoms)
```

Here:

```text
sigma_i       = uncertainty of the reconstructed image phase

N_atoms       = number of atoms per image
```

The expected result is that more atoms reduce the phase error.

![Figure 5: atom number scaling](step3_fig5_atom_number_scaling.png)

---

## Figure 6 style test: image number scaling

In this test, the number of images in the dataset is varied.

The question is:

```text
How many images are needed for the PCA reconstruction to become stable?
```

After a sufficient number of images, the global image-phase reconstruction error becomes almost independent of the number of images.

This means that, beyond some point, adding more images does not strongly improve the global phase reconstruction if the atom shot noise per image remains fixed.

![Figure 6: image number scaling, part 1](step3_fig6_1_image_number_scaling.png)

![Figure 6: image number scaling, part 2](step3_fig6_2_image_number_scaling.png)

---

## Figure 7 style test: pixel-wise phase error

In this test, the spatial phase error is analyzed pixel by pixel.

The expected scaling is:

```text
sigma_xy ∝ 1 / sqrt(n_images * Ibar(x, y))
```

Here:

```text
sigma_xy        = phase error at pixel (x, y)

n_images        = number of images

Ibar(x, y)      = average atom count at pixel (x, y)
```

This means that pixels with more atoms have lower phase error, while low-density pixels have larger phase error.

It also means that using more images reduces the pixel-wise phase error.

![Figure 7: pixel phase error](step3_fig7_pixel_phase_error.png)

---

# Summary of results

This repository reproduces the synthetic-data part of the PSPR method.

The main results are:

1. Synthetic two-port atom-interferometer images are generated using a known spatial phase map.

2. PCA extracts the two dominant fringe modes from the image stack.

3. Ellipse correction removes the arbitrary PCA scaling and rotation.

4. The global image phases are reconstructed using:

```text
theta_i_rec = atan2(w_i2, w_i1)
```

5. The spatial phase map is reconstructed using:

```text
gamma_rec(x, y) = atan2(P2(x, y), P1(x, y))
```

6. In the noise-free case, the reconstructed phase agrees with the known input phase to numerical precision.

7. With atom shot noise, the reconstruction error follows the expected scaling:

```text
max(sigma_i) ∝ 1 / sqrt(N_atoms)
```

and the pixel-wise phase error follows:

```text
sigma_xy ∝ 1 / sqrt(n_images * Ibar(x, y))
```

---

# Limitations

This repository reproduces only the synthetic-data part of the paper.

The experimental atom-gravimeter image datasets are not included because they are not publicly available in the paper. The paper states that those datasets are available from the corresponding authors on reasonable request.

The Monte Carlo results may not look exactly identical to the paper because the paper used a much larger number of simulation runs. This repository uses smaller run counts so the code can run on a normal laptop or Google Colab.

---

# License

This repository is for educational and research reproduction purposes.
