---
description: From system calibration (response matrix) to control modes
---

# Computing Control Modes - Basics

## 1. Notations and Conventions

<table><thead><tr><th width="196">Symbol</th><th width="552.3333333333333">Description</th></tr></thead><tbody><tr><td><mark style="color:yellow;">DMsize</mark></td><td>Number of DM actuators</td></tr><tr><td><mark style="color:yellow;">DMxsize</mark></td><td>X-size of 2D DM actuator grid (=<code>DMsize</code> if DM is not on 2D grid)</td></tr><tr><td><mark style="color:yellow;">DMysize</mark></td><td>Y-size of 2D DM actuator grid (=1 if DM is not on a 2D grid)</td></tr><tr><td><mark style="color:yellow;">WFSsize</mark></td><td>Number of WFS pixels</td></tr><tr><td><mark style="color:yellow;">WFSxsize</mark></td><td>X-size of WFS image array (=<code>WFSsize</code> if not on a 2D array)</td></tr><tr><td><mark style="color:yellow;">WFSysize</mark></td><td>Y-size of WFS image array (=1 if not on a 2D array)</td></tr><tr><td><mark style="color:yellow;">NRMmode</mark></td><td>Number of calibration (response matrix) modes</td></tr><tr><td><mark style="color:yellow;">NCMmode</mark></td><td>Number of control modes</td></tr><tr><td><em><mark style="color:yellow;"><strong>RMmodesDM</strong></mark></em></td><td>Response Matrix (RM) modes, expressed in DM space</td></tr><tr><td><em><mark style="color:yellow;"><strong>RMmodesWFS</strong></mark></em></td><td>Response Matrix (RM) modes, expressed in WFS space</td></tr><tr><td><em><mark style="color:yellow;"><strong>CMmodesDM</strong></mark></em></td><td>Control modes (CM), expressed in DM space</td></tr><tr><td><em><mark style="color:yellow;"><strong>CMmodesWFS</strong></mark></em></td><td>Control modes (CM), expressed in WFS space</td></tr></tbody></table>

## 2. System Calibration (Response Matrix)

The AO system response (referred to as response matrix, or RM) is stored as a pair of 3D arrays:

* _<mark style="color:yellow;">**RMmodesDM**</mark>_: Calibration modes in DM space. Stored as a 3D cube: first 2 dimensions encode actuator index, 3rd dimension is the calibration mode index. If DM actuators are not on a regular square grid, the datcube size is <mark style="color:yellow;">DMsize</mark> x 1 x <mark style="color:yellow;">NRMmode</mark>.
* _<mark style="color:yellow;">**RMmodesWFS**</mark>_:  WFS linear response to the calibration modes.

For example, with a zonal response matrix, _<mark style="color:yellow;">**RMmodesDM**</mark>_ is a zonal poke data cube, where <mark style="color:yellow;">NRMmode</mark>=<mark style="color:yellow;">DMsize</mark>, and  each slice of _<mark style="color:yellow;">**RMmodesDM**</mark>_ has a single non-zero valued actuator.&#x20;

## 3. Control Modes

CACAO uses modal control by default: the WFS signal is first expressed (decomposed) as a linear sum of the WFS control modes. The coefficients of this decomposition are then processed (modal filtering) and expanded to a deformable mirror (DM) control command.

The control modes are expressed in both WFS space (_<mark style="color:yellow;">**CMmodesWFS**</mark>_) and DM space (_<mark style="color:yellow;">**CMmodesDM**</mark>_), each stored as a 3D data cube.

For example, if the first control mode is tilt, the first frame of _<mark style="color:yellow;">**CMmodesDM**</mark>_ is the tilt mode on the DM, and the first frame of _<mark style="color:yellow;">**CMmodesWFS**</mark>_ is the corresponding WFS response. Each subsequent mode adds a frame to both the _<mark style="color:yellow;">**CMmodesDM**</mark>_ and _<mark style="color:yellow;">**CMmodesWFS**</mark>_ datacubes.&#x20;

## 4. Computing Control Modes by SVD of the Response Matrix

The WFS-space control modes _<mark style="color:yellow;">**CMmodesWFS**</mark>_ are used to perform a linear decomposition of the input WFS signal, so **they must be mutually orthogonal**. No such constraint exists on the measured calibration modes _<mark style="color:yellow;">**RMmodesWFS**</mark>_, or even _<mark style="color:yellow;">**RMmodesDM**</mark>_. This is the fundamental reason why the calibration modes cannot be used as the control modes.

The purpose of the response matrix is to explore all possible DM commands that are to be controlled, and measure their WFS response. Not knowing in advance what the WFS response is, this process will unavoidably lead to some redundancy in _<mark style="color:yellow;">**RMmodesWFS**</mark>_, even if the input DM calibration modes _<mark style="color:yellow;">**RMmodesDM**</mark>_ are orthogonal in DM space. There are usually DM modes to which the WFS in blind (this is the measurement null space). For example, a DM actuator behind a pupil obstruction (such as the one due to the secondary mirror of a telescope) will have no response.

The most convenient path to computing the control modes is to perform a **principal components analysis (PCA)** of the _<mark style="color:yellow;">**RMmodesWFS**</mark>_ data cube to construct _<mark style="color:yellow;">**CMmodesWFS**</mark>_, ensuring orthogonality of the control modes in WFS space. Each resulting WFS-space control mode is a linear combination of the input WFS-space calibration modes (the coefficients of this transformation are computed by the PCA), and the same linear combination can be applied to the DM-space calibration modes to compute the DM-space control modes _<mark style="color:yellow;">**CMmodesDM**</mark>_. This process ensures that all control modes are orthogonal in WFS space, and provides the control mode representation in both WFS and DM spaces.

This SVD-based computation of control modes is performed by cacao script <mark style="color:green;">`cacao-aorun-039-compstrCM`</mark>.

