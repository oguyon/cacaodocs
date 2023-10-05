# Image Streams

## Rationale

Numerical data (images, vectors, matrices) are stored in shared memory for low-latency read/write by real-time processes. They are embedded in a structure containing, in addition to the data itself, the corresponding metadata (for example, image size, data type), as well as semaphores and counters for synchronization. This is implemented with the ImageStreamIO library.

## Data Synchronization

By embedding semaphores and counters within the data stream structure, milk combines modularity with high low-latency data synchronization between processes.



<img src="../.gitbook/assets/file.excalidraw (1).svg" alt="Synchronization between data (streams) and processes" class="gitbook-drawing">

A processing pipeline is a chain of individual processes connected together by data streams. Each process can have multiple input and output streams, but has a single triggering input (usually the semaphore of an input data stream).

This modular architecture allows for deployment of complex data processing pipelines across multiple CPU cores, and even between multiple computers. Each process can be hosted on an individual CPU core, or span multiple cores.
