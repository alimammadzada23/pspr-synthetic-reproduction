# PSPR Synthetic Data Reproduction

This repository is a synthetic-data reproduction of the method described in:

**Spatially resolved phase reconstruction for atom interferometry**
Seckmeyer et al., *EPJ Quantum Technology*, 2025.

The goal is not to reproduce the full experimental part of the paper. The real experimental atom-gravimeter image datasets are not publicly included with the paper. Instead, this repository reproduces the **synthetic-data workflow** using the mathematical image model described in the paper.

The project generates artificial two-port atom-interferometer images, applies PCA-based spatial phase reconstruction, and tests the effect of atom shot noise on the reconstructed phase.

---

## Project goal

Atom interferometers usually measure a global phase. However, real atom-cloud images can also contain spatially varying phase information across the cloud. The goal of PSPR, or **spatially resolved phase reconstruction**, is to reconstruct a phase map

$$
\gamma(x,y)
$$

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

Open the notebook

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
step1_theta_scan_images.png
```

and similarly for the other images.

---

# Method overview

The synthetic image stack is generated from the interferometer image model

$$
I_i(x,y)=A(x,y)+B(x,y)\cos\left[\theta_i+\gamma(x,y)\right].
$$

Here:

* (I_i(x,y)) is the intensity at pixel position ((x,y)) in image (i),
* (A(x,y)) is the mean image background / density envelope,
* (B(x,y)) is the local fringe amplitude,
* (\theta_i) is the global interferometer phase of image (i),
* (\gamma(x,y)) is the spatial phase pattern to be reconstructed.

The useful identity is

$$
\cos\left[\theta_i+\gamma(x,y)\right]
=====================================

## \cos\theta_i\cos\gamma(x,y)

\sin\theta_i\sin\gamma(x,y).
$$

This shows that the image stack mainly varies in two dimensions:

1. a cosine-like mode,
2. a sine-like mode.

Therefore, PCA can recover the two dominant fringe modes from the image sequence.

The reconstruction pipeline is:

```text
synthetic image stack
→ subtract mean image
→ PCA decomposition with two components
→ ellipse correction of PCA scores
→ arctan2 phase extraction
→ reconstructed global phases θ_i
→ reconstructed spatial phase map γ(x,y)
```

---

# Step 1: Synthetic data generation

In Step 1, synthetic atom-interferometer images are generated.

The model is

$$
I_i(x,y)=A(x,y)+B(x,y)\cos\left[\theta_i+\gamma(x,y)\right].
$$

The phase (\theta_i) changes from image to image. It represents the global interferometer phase scan.

The spatial phase (\gamma(x,y)) changes across the image. This is the phase map that the algorithm should reconstruct.

Three spatial phase patterns are tested:

$$
\gamma_1(x,y)=2x^2+4y,
$$

$$
\gamma_2(x,y)=4x+4y,
$$

$$
\gamma_3(x,y)=4(0.2x)^2+2y.
$$

The synthetic images are two-port interferometer images. The two output ports are modelled as separated atom clouds. A relative (\pi) phase shift is included between the two ports so that they are complementary, as expected for an atom interferometer.

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

First, each image is flattened into a vector. If an image has shape

```text
height × width
```

then it becomes one long vector.

A stack of images becomes a data matrix:

```text
number of images × number of pixels
```

After subtracting the mean image, PCA is applied to extract the two dominant components.

Because the image model can be written as

$$
I_i(x,y)-A(x,y)
===============

## B(x,y)\cos\theta_i\cos\gamma(x,y)

B(x,y)\sin\theta_i\sin\gamma(x,y),
$$

the image variation is approximately two-dimensional. PCA therefore recovers two modes corresponding to the sine-like and cosine-like components.

---

## PCA scores and ellipse correction

For each image, PCA gives two coefficients:

$$
w_{i,1}, \quad w_{i,2}.
$$

Ideally, these coefficients should lie on a circle as (\theta_i) is scanned. In practice, PCA can rotate and scale the components, so the points often form an ellipse.

Therefore, an ellipse correction is applied to map the PCA scores back to a circle.

After ellipse correction, the global phase of each image is reconstructed as

$$
\theta_i^{\mathrm{rec}}
=======================

\operatorname{atan2}(w_{i,2},w_{i,1}).
$$

The spatial phase map is reconstructed from the corrected PCA components (P_1(x,y)) and (P_2(x,y)) as

$$
\gamma^{\mathrm{rec}}(x,y)
==========================

\operatorname{atan2}\left(P_2(x,y),P_1(x,y)\right).
$$

Because the input data is noise-free, the reconstructed phase should match the true phase almost exactly.

In the notebook, the noise-free reconstruction error is close to numerical precision, typically around

$$
10^{-15} \text{ to } 10^{-16} \ \mathrm{rad}.
$$

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

If the total number of atoms is (N_{\mathrm{atoms}}), the relative atom shot noise scales approximately as

$$
\frac{1}{\sqrt{N_{\mathrm{atoms}}}}.
$$

Therefore, using more atoms improves the reconstruction accuracy.

---

## Figure 5 style test: atom number scaling

In this test, the number of atoms per image is varied.

The expected scaling is

$$
\max(\sigma_i)
\propto
\frac{1}{\sqrt{N_{\mathrm{atoms}}}}.
$$

Here:

* (\sigma_i) is the uncertainty of the reconstructed image phase,
* (N_{\mathrm{atoms}}) is the number of atoms per image.

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

The expected scaling is

$$
\sigma_{xy}
\propto
\frac{1}{\sqrt{n_{\mathrm{images}}\bar{I}(x,y)}}.
$$

Here:

* (\sigma_{xy}) is the phase error at pixel ((x,y)),
* (n_{\mathrm{images}}) is the number of images,
* (\bar{I}(x,y)) is the average atom count at pixel ((x,y)).

This means that pixels with more atoms have lower phase error, while low-density pixels have larger phase error.

It also means that using more images reduces the pixel-wise phase error.

![Figure 7: pixel phase error](step3_fig7_pixel_phase_error.png)

---

# Summary of results

This repository reproduces the synthetic-data part of the PSPR method.

The main results are:

1. Synthetic two-port atom-interferometer images are
