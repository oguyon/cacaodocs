# Acquiring Linear Response Calibration

The system linear calibration consists of the following files:

<table><thead><tr><th width="382">File</th><th>Description</th></tr></thead><tbody><tr><td><code>./conf/RMmodesDM/RMmodesDM.fits</code></td><td>Calibration DM modes</td></tr><tr><td><code>./conf/RMmodesWFS/RMmodesWFS.fits</code></td><td>WFS linear response to DM modes</td></tr><tr><td><code>./conf/dmmask.fits</code></td><td>DM mask (active actuators = 1)</td></tr><tr><td><code>./conf/wfsmask.fits</code></td><td>WFS mask (active pixels = 1)</td></tr></tbody></table>

These are the input to the [computation of control modes.](../tech-notes/computing-control-modes-basics.md)

### 1. Preparing DM Poke Modes

The <mark style="color:green;">`cacao-mkDMpokemodes`</mark> command computes a few different sets of poke modes, from which we can choose the DM poke modes used for [acquiring the WFS linear response](acquiring-linear-response-calibration.md#5.2.-acquiring-wfs-linear-response-to-dm-poke-modes).

```bash
# -z is for number of Zernike modes
# -c is for max spatial frequency [CPA]
cacao-mkDMpokemodes -z 5 -c 25
```

The following files are written to `./conf/RMmodesDM/`:

<table><thead><tr><th width="264">File</th><th>Description</th></tr></thead><tbody><tr><td><strong>FpokeC.&#x3C;CPA>.fits</strong></td><td>Fourier modes up to spatial frquency CPA (integer)</td></tr><tr><td><strong>ZpokeC.&#x3C;NUM>.fits</strong></td><td>First NUM Zernike modes.</td></tr><tr><td><strong>SmodesC.fits</strong></td><td>Simple (single actuator) pokes</td></tr><tr><td><strong>HpokeC.fits</strong></td><td>Hadamard modes</td></tr><tr><td>Hmat.fits</td><td>Hadamard matrix</td></tr><tr><td>Hpixindex.fits</td><td>Hadamard pixel index</td></tr></tbody></table>

DM poke modes can also be prepared independently of the <mark style="color:green;">`cacao-mkDMpokemodes`</mark> command, following the file name convention `./conf/RMmodesDM/<name>.fits`.

### 2. Acquiring WFS Linear Response to DM poke modes

The <mark style="color:green;">`cacao-aorun-030-acqlinResp`</mark> command acquires the WFS response corresponding to a set of [DM poke modes](acquiring-linear-response-calibration.md#5.1.-preparing-dm-poke-modes). For example, to acquire the calibration with Hadamard poke modes:&#x20;

```bash
# -n : number of cycles (increase for higher SNR)
# HpokeC is the DM poke modes
cacao-aorun-030-acqlinResp -n 4 HpokeC
```

The output is a FITS cube, written to `./conf/RMmodesWFS/<name>.WFSresp.fits`. For example, the response to Hadamard pokes will be `./conf/RMmodesWFS/HpokeC.WFSresp.fits`.&#x20;

{% hint style="info" %}
The WFS linear response to multiple sets of DM poke modes can be acquired. For example with:

<pre class="language-bash"><code class="lang-bash"><strong># Creates files: 
</strong><strong># ./conf/HpokeC.WFSresp.fits
</strong><strong># ./conf/ZpokeC.5.WFSresp.fits
</strong><strong># ./conf/TipTiltFoc.WFSresp.fits
</strong><strong>cacao-aorun-030-acqlinResp -n 4 HpokeC
</strong>cacao-aorun-030-acqlinResp -n 10 ZpokeC.5
cacao-aorun-030-acqlinResp -n 30 TipTiltFoc
</code></pre>
{% endhint %}

The <mark style="color:green;">`cacao-aorun-030-acqlinResp`</mark> command also creates/updates the sym link `./conf/RMmodesWFS/RMmodesWFS.fits` to point to the new `<name>.WFSresp.fits` file, so that the [compute control matrix step](acquiring-linear-response-calibration.md#6.-computing-control-matrix) could be run.

{% hint style="info" %}
Use the -s option to **save all intermediate files** for custom assembly/averaging of the response matrix. Files will appear in directory ./LOOPNAME\_rundir/measlinrespm/DATE. By default (no -s option), only timing info and poke sequence info will be written. With the -s option, FITS files corresponding to each RM (time step and iteration) will be written. Example directory content with -s option:

```
mode_linresp_raw.fits      RMpokeTiming.iter0002.txt       wfsresp.tstep001.iter0002.fits  wfsresp.tstep003.iter0002.fits
RMacqulog.txt              RMpokeTiming.iter0003.txt       wfsresp.tstep001.iter0003.fits  wfsresp.tstep003.iter0003.fits
RMpokelog.iter0000.txt     wfsresp.tstep000.iter0000.fits  wfsresp.tstep002.iter0000.fits  wfsresp.tstep004.iter0000.fits
RMpokelog.iter0001.txt     wfsresp.tstep000.iter0001.fits  wfsresp.tstep002.iter0001.fits  wfsresp.tstep004.iter0001.fits
RMpokelog.iter0002.txt     wfsresp.tstep000.iter0002.fits  wfsresp.tstep002.iter0002.fits  wfsresp.tstep004.iter0002.fits
RMpokelog.iter0003.txt     wfsresp.tstep000.iter0003.fits  wfsresp.tstep002.iter0003.fits  wfsresp.tstep004.iter0003.fits
RMpokeTiming.iter0000.txt  wfsresp.tstep001.iter0000.fits  wfsresp.tstep003.iter0000.fits
RMpokeTiming.iter0001.txt  wfsresp.tstep001.iter0001.fits  wfsresp.tstep003.iter0001.fits

```
{% endhint %}

{% hint style="info" %}
**Stopping the RM acquisition**. &#x20;
{% endhint %}

### 3. Representing WFS response in zonal space (optional)

{% hint style="info" %}
Zonal representation is useful for visualization, showing the WFS response to individual DM actuators. Zonal representation also makes DM mask computation relatively straightforward.
{% endhint %}

If a Hadamard response matrix was acquired, the zonal space WFS response is decoded with:

```bash
cacao-aorun-031-RMHdecode
```

Any WFS response can also be projected to the zonal basis with the following script:

{% code lineNumbers="true" %}
```bash
#!/usr/bin/env bash

cacao
# load custom RM files
loadfits "customRMmodesDM.fits" RMmodesDM
loadfits "customRMmodesWFS.fits" RMmodesWFS
# convert to zonal representation
# last argument is SVD limit - may require tuning
cacaocc.RM2zonal RMmodesDM RMmodesWFS RMmodesDMz RMmodesWFSz 0.01
# save results
saveFITS RMmodesDMz "conf/RMmodesDM/RMmodesDMz.fits"
saveFITS RMmodesDMz "conf/RMmodesWFS/RMmodesWFSz.fits"
exitCLI
```
{% endcode %}

### 4. DM and WFS masks (optional)

If a zonal response matrix exists, then DM and WFS maps and masks can be computed:

```bash
cacao-aorun-032-RMmkmask
```

Check results:

```
conf/dmmask.fits
conf/wfsmask.fits
```

If needed, rerun command with non-default parameters (see -h for options). Note: we are not going to apply the masks in this example, so OK if not net properly. The masks are informative here, allowing us to view which DM actuators and WFS pixels have the best response.
