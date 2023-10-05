---
description: Fine-tuning control modes
---

# Control Modes - Advanced

## Rationale

Approaches to fine-tuning control modes.

## 1. Useful Tools and Scripts

### 1.1. Splitting cubes

Use [FITSIO extended file syntax](https://heasarc.gsfc.nasa.gov/docs/software/fitsio/filters.html) to split subset of modes from a larger set.

<details>

<summary>Extracting a subset of modes</summary>

```bash
#!/usr/bin/env bash

# extract first 10 modes from modes.fits
# save result in modes10.fits

milk << EOF
loadfits "modes.fits[*, *, 1:10]" modes10
saveFITS modes10 "modes10.fits"
exitCLI
EOF
```

</details>

### 1.2. Merging modes (concatenation)

Use <mark style="color:green;">`milk-images-merge`</mark> to merge (concatenate) modes.

```bash
# Create list of FITS files to merge
echo "modesA.fits" > mergelist.txt
echo "modesB.fits" >> mergelist.txt
echo "modesC.fits" >> mergelist.txt

# merge files along axis 2 (z axis)
milk-images-merge mergelist.txt 2 modesABC.fits
```

The corresponding milk script allows for more fine tuning (for example multiplying one of the input files by a scaling coefficients).

<details>

<summary>Concatenating modes</summary>

```bash
#!/usr/bin/env bash

milk << EOF
loadfits "modesA.fits" imA
loadfits "modesB.fits" imB
immerge imA imB imAB 0
saveFITS imAB "modesAB.fits"
exitCLI
EOF
```

</details>

### 1.3. Marginalizing one mode set against another one

Use <mark style="color:green;">`milk-modes-project`</mark> to ensure orthogonality between sets of modes.

{% code fullWidth="false" %}
```bash
# Make modesA orthogonal to modesTTF.fits
# modesTTF.fits are tip, tilt and focus modes
# modesA_noTTF.fits (residual) has tip, tilt and focus removed
#
milk-modes-project modesA.fits modesTTF.fits 0.001 modesA_TTFpart.fits modesA_noTTF.fit
```
{% endcode %}

<details>

<summary>Projection and marginalization between mode bases</summary>

```bash
#!/usr/bin/env bash

# Decompose modesA against modesB

modesA="$1"
modesB="$2"
SVDlim="0.001"

# project modesA onto space defined by modesB
# projection, residual

# Main steps:
# construct orthonormal basis from modesB using PCA
# decompose modesA using the PCA modes
# reconstruct projection using coefficients and PCA modes
# compute residual by subtraction reconstruction from original
# 
# OUTPUT:
# imrec.fits : reconstruction (projection)
# imres.fits : residual
#
milk-all << EOF
loadfits "$2" modesB
linalg.compSVD modesB svdU svdS svdV ${SVDlim}

loadfits "$1" modesA
linalg.sgemm .transpA 1
linalg.sgemm modesA svdU mcoeff

linalg.sgemm .transpA 0
linalg.sgemm .transpB 1
linalg.sgemm svdU mcoeff imrec
saveFITS imrec "imrec.fits"

imres=modesA-imrec
saveFITS imres "imres.fits"

exitCLI
EOF
```

</details>

### 1.4. Computing Control modes

<details>

<summary>Computing Control Modes</summary>

```bash
#!/usr/bin/env bash

# input
RMmDM="conf/EMmodesDM/RMmodesDM_custom.fits"
RMmWFS="conf/RMmodesWFS/RMmodesWFS_custom.fits"
DMm="conf/dmmask_custom.fits"
WFSm="conf/wfsmask_custom.fits"
SVDlim="0.01"

# output
CMmDM="conf/CMmodesDM/CMmodesDM_custom.fits"
CMmWFS="conf/CMmodesDM/CMmodesWFS_custom.fits"

cacao << EOF
cacaocc.compsCM $RMmDM $RMmWFS $DMm $WFSm $CMmDM $CMmWFS $SVDlim
exitCLI
EOF

```

</details>

## 2. Forcing DM-space Control Modes to be Ordered by Spatial Frequency

### 2.1. Synthetic RM

{% hint style="warning" %}
This feature requires a zonal response matrix and a geometric representation of the DM, where DM actuator pixel coordinates correspond to actual geometrical position.
{% endhint %}

A synthetic set of modes, ordered by spatial frequency, is created in DM space. The first few modes (5 by default) are Zernike modes. The following modes are Fourier modes of increasing spatial frequency. The WFS response to these modes is computed from the zonal response matrix.

The synthetic RM modes are weighted by spatial frequency prior to computing the CM. Doing so will ensure that the resulting CM is approximately ordered by spatial frequency.

```bash
# -c option for max spatial frequency [CPA]
# -z option for number of Zernike modes
# -a option for power law coefficient
#
cacao-aorun-033-RM-mksynthetic -c 7
```

### 2.2. Custom Assembly of CM

Using [tools and scripts](control-modes-advanced.md#1.-useful-tools-and-scripts) for custom assembly of CM.

### 2.3. Force control modes to match user-specified set of modes

The output of the SVD-based CM computation may need to be transformed such that the first control modes are tip, tilt and focus, or to enforce spatial frequency ordering. Here we show how to do this by rotation in <mark style="color:yellow;">`NCMmodes`</mark> dimensions, where <mark style="color:yellow;">`NCMmodes`</mark> is the number of modes in the control mode basis. Rotations preserve otrhonormality, ensuring the control mode basis stays orthonormal.

```bash
#!/usr/bin/env bash

# Create set of Fourier modes, store as modesF.fits
#
milk-all << EOF
lintools.mkFouriermodes modesF 256 256 8.0 0.8 100 1.0 0
saveFITS modesF "modesF.fits"
exitCLI
EOF

# Perform PCA of modesF.fits by SVD
# Store the result as mF.fits
#
# The output is ordered by singular value
# This is not a friendly ordering, spatial frequency is not preserved
# The first modes are NOT low-order mores :(
#
milk-all << EOF
loadfits "modesF.fits" modesF
linalg.compSVD modesF mFsvdU mFsvdS mFsvdV 0.01
saveFITS mFsvdU "mF.fits"
exitCLI
EOF

# The basis mF.fits is orthonormal (this is good), but not
# ordered by spatial frequency (this is bad)
#
# We started with modesF.fits, which is not orthogonal (this is bad),
# but is ordered by spatial frequency (this is good)
#
# We will now rotate mF.fits to order it by spatial frequency, by
# asking the rotated version of mF.fits to match modesF.fits
#

# First, we compute decompose modesF.fits against mF.fits
# The result is stored in modesFmFxp.fits
#
milk-all << EOF
loadfits "mF.fits" mF
loadfits "modesF.fits" modesF
linalg.sgemm .transpA 1
linalg.sgemm modesF mF out
listim
saveFITS out "modesFmFxp.fits"
exitCLI
EOF

# Next, we compute the rotation that will turn 
# the cross-product into a lower triangular matrix, ensuring
# a good match between the two sets of modes.
# The rotation is stored as matArot.fits
# We then apply the rotation to mF.fits
#
milk-all << EOF
loadfits "modesFmFxp.fits" matAB
linalg.basisrotmatch matAB matArot
# matABr.fits should now be lower triangular
saveFITS matAB "matABr.fits"
saveFITS matArot "matArot.fits"
loadfits "mF.fits" mF
# Apply rotation to mF
linalg.sgemm mF matArot mF1
saveFITS mF1 "mF1.fits"
exitCLI
EOF

# The new rotated basis mF1.fits now satisfies our constraints
# Let's check it is orthonormal

milk-all << EOF
loadfits "mF1.fits" mF1
linalg.sgemm .transpA 1
linalg.sgemm mF1 mF1 out
saveFITS out "mF1xp.fits"
exitCLI
EOF

```

