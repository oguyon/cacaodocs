# WFS acquisition

### 1. Taking the WFS dark

If the WFS input has a bias and/or dark, acquire the "dark", which is the WFS input when no signal is present:

```bash
# -n option: number of frames averaged
cacao-aorun-005-takedark -n 2000
```

{% hint style="warning" %}
Make sure to turn off or block light source before running the <mark style="color:green;">`cacao-aorun-005-takedark`</mark> command, and remember to turn on / unblock after done.
{% endhint %}

### 2. Starting WFS acquisition

```bash
cacao-aorun-025-acqWFS -w start
```

### 3. Acquiring the WFS reference

This is the WFS response to a flat (unaberrated) input wavefront, and defines the convergence point for the AO loop. This step can be skipped is the WFS input is already referenced to zero for a flat wavefront.

```bash
cacao-aorun-026-takeref -n 2000
```
