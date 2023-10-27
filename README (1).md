# cacao User Guide

## Overview

_**cacao**_ (Compute and Control for Adaptive Optics) deploys and manages processes for real-time control of adaptive optics systems, and provides user interfaces to interact with them.

## Installation and System Configuration

Check installation guide

{% content-ref url="installation/" %}
[installation](installation/)
{% endcontent-ref %}

## Building Blocks

{% hint style="info" %}
cacao is a plugin of milk. Commands inherited from milk start with "milk-" while cacao-specific commands start with "cacao-".
{% endhint %}

cacao is built around 3 types of data structures, provided by milk, and hosted on the system's shared memory :

* **Streams** contain numerical data (images, matrices and vectors) for real-time use
* **Function Parameter Structures** (FPS) provide interface to variables and parameters
* **Process Info** (procinfo) provide control and status of real-time processes

To view and interact with stream, FPS and procinfo structures, run:

```
milk-streamCTRL
milk-fpsCTRL
milk-procCTRL
```

{% hint style="info" %}
To learn more about a milk or cacao command, run it with the -h argument.&#x20;
{% endhint %}

{% content-ref url="building-blocks-milk/" %}
[building-blocks-milk](building-blocks-milk/)
{% endcontent-ref %}



