# Computing Control Modes

{% hint style="warning" %}
Before proceeding to this step, make sure the required input files are ready:

* conf/RMmodesDM/RMmodesDM.fits (response matrix, DM space)
* conf/RMmodesWFS/RMmodesWFS.fits (response matrix, WFS space)
* conf/dmmask.fits (DM pixel mask, indicating which actuators are linearly coupled to WFS)
* conf/wfsmask.fits (WFS pixel mask, indicating which WFS pixels are linearly coupled to DM)
{% endhint %}

## 1. Computing CMs by SVD

Control modes are computed and their representation in both WFS and DM spaces. First, set the SVD limit:

```bash
cacao-fpsctrl setval compstrCM svdlim 0.001
```

Then run the <mark style="color:green;">`cacao-aorun-039-compstrCM`</mark> script to compute CM and load it to shared memory:

```bash
cacao-aorun-039-compstrCM
```

Inspect result:

```
conf/CMmodesDM/CMmodesDM.fits
conf/CMmodesWFS/CMmodesWFS.fits
```

Check especially the number of modes controlled.

{% hint style="info" %}
By convention:

* Control modes are orthogonal in WFS space (dot product between two modes = 0) over the WFS pixel mask (wfsmask)
* Control modes are of unity norm in DM space over the DM pixel mask (dmmask)

WFS-space orthogonality ensures that the modal coefficients can be obtained by matrix-vector multiplication (MVM) between CMmodesWFS and the input WFS signal.

DM-space normalization ensures that the modal coefficients obtained by the above MVM operation have a physical meaning (unit = micron RMS over dmmask).
{% endhint %}



## 2. Ordering CMs by Spatial Frequency

The SVD-based process orders modes by singular value, such that DM patterns creating the strongest linear WFS response are first.

Ordering CMs by spatial frequency (low-order modes first) is often preferable when implementing and tuning modal control.

```bash
# Input: 
# - conf/CMmodesDM.fits
# - conf/CMmodesWFS.fits
# - conf/dmmask.fits
# - conf/wfsmask.fits
#
# Output:
# - conf/CMmodesDM/CMmodesDM_sf.fits
# - conf/CMmodesWFS/CMmodesWFS_sf.fits
#
cacao-aorun-040-compfCM -g 0 -c 25
```



***

## Additional Information, Fine Tuning:

{% content-ref url="../tech-notes/computing-control-modes-basics.md" %}
[computing-control-modes-basics.md](../tech-notes/computing-control-modes-basics.md)
{% endcontent-ref %}

{% content-ref url="../tech-notes/control-modes-advanced.md" %}
[control-modes-advanced.md](../tech-notes/control-modes-advanced.md)
{% endcontent-ref %}

***

##
