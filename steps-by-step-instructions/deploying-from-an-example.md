# Deploying from an example

### 1. Download & modify existing example

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

### 2. Deploy processes and tmux sessions

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

### 3. Start logging (optional)

Message logging is optional.

{% content-ref url="../tech-notes/message-logging.md" %}
[message-logging.md](../tech-notes/message-logging.md)
{% endcontent-ref %}

### 4. Select simulation or hardware mode

```bash
# Simulation mode
./scripts/aorun-setmode-sim
# Hardare mode, will connect to hardware
./scripts/aorun-setmode-hardw
```
