# Steps-by-step instructions

{% hint style="success" %}
**Before proceeding with the instructions in this page**, read the [Getting Situated ](getting-situated.md)page, and keep the [Building Blocks](building-blocks-milk/) page handy. Some familiarity with the programs <mark style="color:green;">`milk-fpsCTRL`</mark>, <mark style="color:green;">`milk-streamCTRL`</mark> and <mark style="color:green;">`milk-procCTRL`</mark> is helpful to follow what is happening "under the hood". A [stream viewer](installation/stream-viewers.md) is also highly recommended to visualize data along the way.
{% endhint %}

{% content-ref url="getting-situated.md" %}
[getting-situated.md](getting-situated.md)
{% endcontent-ref %}

{% content-ref url="building-blocks-milk/" %}
[building-blocks-milk](building-blocks-milk/)
{% endcontent-ref %}

## 1. Deployment from an Example: Setting up Processes

### 1.1. Download & modify existing example

Download from source to current directory one of the cacao example loops. Run command <mark style="color:green;">`cacao-loop-deploy`</mark> with -h option to see list of examples.

```bash
cacao-loop-deploy -c <examplename>
```

{% hint style="info" %}
The loop number. loop name, DM index and DM simulation index can be changed from their default values by setting the corresponding environment variables. For example:

```bash
CACAO_LOOPNUMBER=7 cacao-loop-deploy -c <examplename>
CACAO_LOOPNUMBER=7 CACAO_DMINDEX="03" cacao-loop-deploy -c <examplename>
```
{% endhint %}

OPTIONAL: Edit file `<examplename>-conf/cacaovars.bash` as needed

### 1.2. Deploy processes and tmux sessions

Run deployment (starts the conf processes):

```bash
cacao-loop-deploy -r <examplename>
```

{% hint style="info" %}
The copy and run steps can be done at once with:

```bash
cacao-loop-deploy <examplename>
```
{% endhint %}

Go to rootdir, from which user controls the loop:

```bash
cd <loopname>-rootdir
```

{% hint style="warning" %}
From this point on, all commands should be run from the root directory. To check we are in the correct directory, run command <mark style="color:green;">`cacao-check-cacaovars`</mark>
{% endhint %}

### 1.3. Start logging (optional)

Message logging is optional.

{% content-ref url="tech-notes/message-logging.md" %}
[message-logging.md](tech-notes/message-logging.md)
{% endcontent-ref %}

### 1.4. Select simulation or hardware mode

```bash
# Simulation mode
./scripts/aorun-setmode-sim
# Hardare mode, will connect to hardware
./scripts/aorun-setmode-hardw
```

***

## 2. Start DM and WFS

**If in hardware mode**, run hardware DM:

```bash
cacao-aorun-000-dm start
```

**If in simulator mode**, run simulation DM and WFS:

```bash
cacao-aorun-001-dmsim start
cacao-aorun-002-simwfs -w start
```

## 3. WFS Acquisition

### 3.1. Taking the WFS dark

If the WFS input has a bias and/or dark, acquire the "dark", which is the WFS input when no signal is present:

```bash
# -n option: number of frames averaged
cacao-aorun-005-takedark -n 2000
```

{% hint style="warning" %}
Make sure to turn off or block light source before running the <mark style="color:green;">`cacao-aorun-005-takedark`</mark> command, and remember to turn on / unblock after done.
{% endhint %}

### 3.2. Starting WFS acquisition

```bash
cacao-aorun-025-acqWFS -w start
```

### 3.3. Acquiring the WFS reference

This is the WFS response to a flat (unaberrated) input wavefront, and defines the convergence point for the AO loop. This step can be skipped is the WFS input is already referenced to zero for a flat wavefront.

```bash
cacao-aorun-026-takeref -n 2000
```

***

## 4. Measuring Hardware Latency

The time delay between issuing a command to the DM and having the corresponding signal in the WFS stream is the hardware latency. It must be known to [measure the system response](steps-by-step-instructions.md#5.2.-running-linear-response-measurement), as well as for some advanced control features such as pseudo-open loop reconstruction and predictive control.

```bash
# -w option is for "wait until done"
cacao-aorun-020-mlat -w
```

***

## 5. Acquiring System Linear Calibration

The system linear calibration consists of the following files:

<table><thead><tr><th width="382">File</th><th>Description</th></tr></thead><tbody><tr><td><code>./conf/RMmodesDM/RMmodesDM.fits</code></td><td>Calibration DM modes</td></tr><tr><td><code>./conf/RMmodesWFS/RMmodesWFS.fits</code></td><td>WFS linear response to DM modes</td></tr><tr><td><code>./conf/dmmask.fits</code></td><td>DM mask (active actuators = 1)</td></tr><tr><td><code>./conf/wfsmask.fits</code></td><td>WFS mask (active pixels = 1)</td></tr></tbody></table>

These are the input to the [computation of control modes](steps-by-step-instructions.md#6.-computing-control-modes).

### 5.1. Preparing DM Poke Modes

The <mark style="color:green;">`cacao-mkDMpokemodes`</mark> command computes a few different sets of poke modes, from which we can choose the DM poke modes used for [acquiring the WFS linear response](steps-by-step-instructions.md#5.2.-acquiring-wfs-linear-response-to-dm-poke-modes).

```bash
# -z is for number of Zernike modes
# -c is for max spatial frequency [CPA]
cacao-mkDMpokemodes -z 5 -c 25
```

The following files are written to `./conf/RMmodesDM/`:

<table><thead><tr><th width="264">File</th><th>Description</th></tr></thead><tbody><tr><td><strong>FpokeC.&#x3C;CPA>.fits</strong></td><td>Fourier modes up to spatial frquency CPA (integer)</td></tr><tr><td><strong>ZpokeC.&#x3C;NUM>.fits</strong></td><td>First NUM Zernike modes.</td></tr><tr><td><strong>SmodesC.fits</strong></td><td>Simple (single actuator) pokes</td></tr><tr><td><strong>HpokeC.fits</strong></td><td>Hadamard modes</td></tr><tr><td>Hmat.fits</td><td>Hadamard matrix</td></tr><tr><td>Hpixindex.fits</td><td>Hadamard pixel index</td></tr></tbody></table>

DM poke modes can also be prepared independently of the <mark style="color:green;">`cacao-mkDMpokemodes`</mark> command, following the file name convention `./conf/RMmodesDM/<name>.fits`.

### 5.2. Acquiring WFS Linear Response to DM poke modes

The <mark style="color:green;">`cacao-aorun-030-acqlinResp`</mark> command acquires the WFS response corresponding to a set of [DM poke modes](steps-by-step-instructions.md#5.1.-preparing-dm-poke-modes). For example, to acquire the calibration with Hadamard poke modes:&#x20;

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

The <mark style="color:green;">`cacao-aorun-030-acqlinResp`</mark> command also creates/updates the sym link `./conf/RMmodesWFS/RMmodesWFS.fits` to point to the new `<name>.WFSresp.fits` file, so that the [compute control matrix step](steps-by-step-instructions.md#6.-computing-control-matrix) could be run.

### 5.3. Representing WFS response in zonal space (optional)

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

### 5.4. DM and WFS masks (optional)

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

***

## 6. Computing Control Modes

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

For more information, including fine tuning:

{% content-ref url="tech-notes/computing-control-modes-basics.md" %}
[computing-control-modes-basics.md](tech-notes/computing-control-modes-basics.md)
{% endcontent-ref %}

{% content-ref url="tech-notes/control-modes-advanced.md" %}
[control-modes-advanced.md](tech-notes/control-modes-advanced.md)
{% endcontent-ref %}

***

## 7. Running the Control Loop

Select GPUs for the modal decomposition (WFS->modes) and expansion (modes->DM) MVMs

```bash
# MVMs on CPU (GPUindex = 99)
# if on GPU, set to GPUindex=0 or device index
#
cacao-fpsctrl setval wfs2cmodeval GPUindex 99
cacao-fpsctrl setval mvalC2dm GPUindex 99
```

### 7.1. Start WFS -> mode coefficient values

```bash
ccacao-aorun-050-wfs2cmval start
```

### 7.2. Start modal filtering

```bash
cacao-aorun-060-mfilt start
```

### 7.3. Start mode coeff values -> DM

```bash
cacao-aorun-070-cmval2dm start
```

### 7.4. Closing the loop and setting loop parameters with mfilt:

Set loop gain

```bash
cacao-fpsctrl setval mfilt loopgain 0.1
```

Set loop mult

```bash
cacao-fpsctrl setval mfilt loopmult 0.98
```

Close loop

```bash
cacao-fpsctrl setval mfilt loopON ON
```
