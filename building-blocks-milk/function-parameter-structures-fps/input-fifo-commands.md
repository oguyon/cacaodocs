# Input fifo commands

## Rationale

The preferred way for high-level interaction with processes is to use milk-fpsCTRL's input fifo. Commands are sent to the fifo for execution.

## Sequencer & Execution Queues

Input fifo commands are processed by the `milk-fpsCTRL` sequencer. The sequencer status is accessible by typing `F3` in the `milk-fpsCTRL` GUI.

<img src="../../.gitbook/assets/file.excalidraw (2).svg" alt="" class="gitbook-drawing">

## List of Commands

### Controlling fps TUI

| Command | Description      | Arguments |
| ------- | ---------------- | --------- |
| rescan  | rescan fps tree  |           |
| exit    | exit fpsCTRL TUI |           |

### Controlling processes

<table><thead><tr><th width="234.33333333333331">Command</th><th width="251">Description</th><th>Arguments</th></tr></thead><tbody><tr><td>confstart</td><td>start conf process</td><td>&#x3C;keyword></td></tr><tr><td>confstop</td><td>stop conf process</td><td>&#x3C;keyword></td></tr><tr><td>confupdate</td><td>update conf process</td><td>&#x3C;keyword></td></tr><tr><td>confwupdate</td><td>update conf process, wait for completion before processing next command</td><td>&#x3C;keyword></td></tr><tr><td>runstart</td><td>start run process</td><td>&#x3C;keyword></td></tr><tr><td>runstop</td><td>stop run process</td><td>&#x3C;keyword></td></tr><tr><td>setval</td><td>set value</td><td>&#x3C;keyword> &#x3C;value></td></tr><tr><td>tmuxstart</td><td>start tmux session</td><td>&#x3C;keyword></td></tr><tr><td>tmuxstop</td><td>stop tmux session</td><td>&#x3C;keyword></td></tr><tr><td>fpsrm</td><td>remove fps</td><td>&#x3C;keyword></td></tr></tbody></table>

Only the `setval` command requires an exact full keyword (for example `mfilt-2.loopON`). Other commands will only use the first part of the keyword (`mfilt-2`) and disregard the rest of the keyword, which is then optional.

### Getting status

| Command  | Description                                 | Arguments           |
| -------- | ------------------------------------------- | ------------------- |
| fpswfile | Write fps content to txt file in datadir    | \<keyword>          |
| getval   | Get value, write to output log              | \<keyword>          |
| fwrval   | Get value, write to file or fifo            | \<keyword> \<fname> |
| cntinc   | counter test to check input fifo connection |                     |

### Execution Queues and Sequencing

| Command       | Description                | Arguments            |
| ------------- | -------------------------- | -------------------- |
| setqindex     | set queue index            | \<queue index>       |
| setqprio      | set current queue priority | \<priority>          |
| queueprio     | set any queue priority     | \<queue> \<priority> |
| waitonrunON   | toggle wait-on-run ON      |                      |
| waitonrunOFF  | toggle wait-on-run OFF     |                      |
| waitonconfON  | toggle wait-on-conf ON     |                      |
| waitonconfOFF | toggle wait-on-conf OFF    |                      |
