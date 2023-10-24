# Computing Control Modes

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

{% content-ref url="../tech-notes/computing-control-modes-basics.md" %}
[computing-control-modes-basics.md](../tech-notes/computing-control-modes-basics.md)
{% endcontent-ref %}

{% content-ref url="../tech-notes/control-modes-advanced.md" %}
[control-modes-advanced.md](../tech-notes/control-modes-advanced.md)
{% endcontent-ref %}

***

##
