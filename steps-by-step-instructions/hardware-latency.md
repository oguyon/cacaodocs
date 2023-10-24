# Hardware Latency

The time delay between issuing a command to the DM and having the corresponding signal in the WFS stream is the hardware latency. It must be known to [measure the system response](hardware-latency.md#5.2.-running-linear-response-measurement), as well as for some advanced control features such as pseudo-open loop reconstruction and predictive control.

```bash
# -w option is for "wait until done"
cacao-aorun-020-mlat -w
```
