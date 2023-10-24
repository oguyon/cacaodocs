# Running the Control Loop

Select GPUs for the modal decomposition (WFS->modes) and expansion (modes->DM) MVMs

```bash
# MVMs on CPU (GPUindex = 99)
# if on GPU, set to GPUindex=0 or device index
#
cacao-fpsctrl setval wfs2cmodeval GPUindex 99
cacao-fpsctrl setval mvalC2dm GPUindex 99
```

### 1. Start WFS -> mode coefficient values

```bash
ccacao-aorun-050-wfs2cmval start
```

### 2. Start modal filtering

```bash
cacao-aorun-060-mfilt start
```

### 3. Start mode coeff values -> DM

```bash
cacao-aorun-070-cmval2dm start
```

### 4. Closing the loop and setting loop parameters with mfilt

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
