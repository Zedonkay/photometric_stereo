# Photometric Stereo Shape Reconstruction

This project implements both calibrated and uncalibrated photometric stereo techniques to reconstruct 3D surface shapes from multiple images captured under different lighting conditions.

## Overview

Photometric stereo is a computer vision technique that uses multiple images of the same object taken under different lighting directions to estimate surface normals and reconstruct 3D shape. This implementation covers:

- **Calibrated Photometric Stereo**: When lighting directions are known
- **Uncalibrated Photometric Stereo**: When lighting directions must be estimated
- **Surface Integration**: Converting surface normals to depth maps
- **Integrability Enforcement**: Ensuring physically consistent normal fields

## Features

### Core Algorithms
- **Lambertian Sphere Rendering**: Generate synthetic training data
- **Pseudonormal Estimation**: Extract surface normals from image intensities
- **SVD-based Uncalibrated Method**: Estimate lighting and normals simultaneously
- **Frankot-Chellappa Integration**: Convert normals to depth maps
- **Bas-Relief Ambiguity Analysis**: Explore inherent ambiguities in photometric stereo

### Visualization Tools
- 3D surface plotting with customizable views
- Albedo and normal map visualization
- Interactive parameter exploration for bas-relief transformations

## Requirements

```python
numpy
matplotlib
scikit-image
scipy
warnings
os
shutil
```

## Data Structure

The project expects input data in the following format:
- `input_1.tif` through `input_7.tif`: Seven images under different lighting
- `sources.npy`: 3×7 matrix of lighting directions (for calibrated method)

## Key Functions

### Question 1: Calibrated Photometric Stereo

#### `renderNDotLSphere(center, rad, light, pxSize, res)`
Renders a hemispherical bowl under given lighting conditions for validation.

#### `estimatePseudonormalsCalibrated(I, L)`
Estimates surface normals when lighting directions are known.
- **Input**: Image matrix I (7×P), Lighting matrix L (3×7)
- **Output**: Pseudonormal matrix B (3×P)

#### `estimateAlbedosNormals(B)`
Separates albedo and surface normals from pseudonormals.
- **Input**: Pseudonormal matrix B (3×P)
- **Output**: Albedos vector, Normals matrix (3×P)

#### `estimateShape(normals, s)`
Integrates surface normals to reconstruct depth map using Frankot-Chellappa algorithm.

### Question 2: Uncalibrated Photometric Stereo

#### `estimatePseudonormalsUncalibrated(I)`
Estimates both surface normals and lighting directions using SVD factorization.
- **Input**: Image matrix I (7×P)
- **Output**: Pseudonormal matrix B (3×P), Estimated lighting L (3×7)

#### `enforceIntegrability(N, s, sig=3)`
Finds optimal transformation to make normal field integrable, addressing the fundamental ambiguity in uncalibrated photometric stereo.

## Usage Example

```python
# Load data
I, L, s = loadData(data_dir)

# Calibrated reconstruction
B_cal = estimatePseudonormalsCalibrated(I, L)
albedos_cal, normals_cal = estimateAlbedosNormals(B_cal)
surface_cal = estimateShape(normals_cal, s)

# Uncalibrated reconstruction
B_uncal, L_est = estimatePseudonormalsUncalibrated(I)
B_integrable = enforceIntegrability(B_uncal, s)
albedos_uncal, normals_uncal = estimateAlbedosNormals(B_integrable)
surface_uncal = estimateShape(normals_uncal, s)

# Visualization
plotSurface(surface_cal)
displayAlbedosNormals(albedos_cal, normals_cal, s)
```

## Key Insights

### Calibrated vs Uncalibrated
- **Calibrated**: More accurate when lighting is precisely known
- **Uncalibrated**: More flexible but suffers from bas-relief ambiguity

### Integrability Challenge
Raw normals from uncalibrated methods often violate integrability constraints, leading to unrealistic reconstructions. The `enforceIntegrability` function solves this by finding the optimal linear transformation.

### Bas-Relief Ambiguity
Uncalibrated photometric stereo has an inherent ambiguity - the same images can be explained by a family of surfaces related by bas-relief transformations parameterized by (μ, ν, λ).

## Technical Details

### Frankot-Chellappa Algorithm
- Converts surface normals (nx, ny, nz) to depth gradients (∂z/∂x, ∂z/∂y)
- Uses Fourier domain integration to enforce integrability
- Handles boundary conditions through frequency domain projection

### SVD Decomposition
For uncalibrated stereo: I = L^T × B
- Factorize I using SVD: I = UΣV^T  
- Take rank-3 approximation: I₃ = U₃Σ₃V₃^T
- Set L^T = U₃Σ₃^(1/2) and B = Σ₃^(1/2)V₃^T

## Output Files

The notebook generates several visualization files:
- `1b-*.png`: Rendered sphere examples
- `1f-*.png`: Calibrated albedo and normal maps  
- `2b-*.png`: Uncalibrated albedo and normal maps
- `faceCalibrated*.png`: 3D surface reconstructions

## Applications

This implementation is suitable for:
- 3D face reconstruction
- Surface defect inspection
- Archaeological artifact documentation
- Material property estimation
- Computer graphics and animation

## Limitations

- Assumes Lambertian (matte) surfaces
- Requires controlled lighting conditions
- Sensitive to shadows and specular reflections
- Uncalibrated method has inherent bas-relief ambiguity

## References

- Woodham, R.J. "Photometric method for determining surface orientation from multiple images"
- Hayakawa, H. "Photometric stereo under a light source with arbitrary motion"
- Frankot, R.T. & Chellappa, R. "A method for enforcing integrability in shape from shading algorithms"
