.. meta::
  :description: TransferBench documentation 
  :keywords: TransferBench, API, ROCm, documentation, HIP


Using TransferBench
---------------------
  
Users have control over the SRC and DST memory locations by indicating memory type followed by the device index. TransferBench supports the following:

* coarse-grained pinned host memory
* unpinned host memory
* fine-grained host memory
* coarse-grained global device memory
* fine-grained global device memory
* null memory (for an empty transfer).

In addition, users can determine the size of the transfer (number of bytes to copy) for their tests.

Users can also specify executors of the transfer. The options are CPU, kernel-based GPU, and SDMA-based GPU (DMA) executors. TransferBench also provides the option to choose the number of sub-executors. In case of a CPU executor this argument specifies the number of CPU threads, while for a GPU executor it defines the number of compute units (CU). If DMA is specified as the executor, the sub-executor argument determines the number of streams to be used.

Refer to the following example to use TransferBench.

--------------------
ConfigFile format
--------------------

A Transfer is defined as a single operation where an executor reads and adds together values from Source (SRC) memory locations, then writes the sum to destination (DST) memory locations.
This simplifies to a simple copy operation when dealing with single SRC/DST.

.. code-block:: bash

   SRC 0                DST 0
   SRC 1 -> Executor -> DST 1
   SRC X                DST Y

Three Executors are supported by TransferBench::

   Executor:        SubExecutor:
   1) CPU           CPU thread
   2) GPU           GPU threadblock/Compute Unit (CU)
   3) DMA           N/A.                                 (May only be used for copies (single SRC/DST)

Each single line in the configuration file defines a set of Transfers (a Test) to run in parallel

There are two ways to specify a test:

1) Basic

   The basic specification assumes the same number of SubExecutors (SE) used per Transfer
   A positive number of Transfers is specified followed by that number of triplets describing each Transfer

   Transfers SEs (srcMem1->Executor1->dstMem1) ... (srcMemL->ExecutorL->dstMemL)

2) Advanced

   A negative number of Transfers is specified, followed by quintuplets describing each Transfer
   A non-zero number of bytes specified will override any provided value

   -Transfers (srcMem1->Executor1->dstMem1 SEs1 Bytes1) ... (srcMemL->ExecutorL->dstMemL SEsL BytesL)

Argument Details:::

   Transfers:   Number of Transfers to be run in parallel
   SEs      :   Number of SubExectors to use (CPU threads/ GPU threadblocks)
   srcMemL   :   Source memory locations (Where the data is to be read from)
   Executor  :   Executor is specified by a character indicating type, followed by device index (0-indexed)
                  - C: CPU-executed  (Indexed from 0 to NUMA nodes - 1)
                  - G: GPU-executed  (Indexed from 0 to GPUs - 1)
                  - D: DMA-executor  (Indexed from 0 to GPUs - 1)
   dstMemL   :   Destination memory locations (Where the data is to be written to)
   bytesL    :   Number of bytes to copy (0 means use command-line specified size)
                  Must be a multiple of 4 and may be suffixed with ('K','M', or 'G')

                  Memory locations are specified by one or more (device character / device index) pairs
                  Character indicating memory type followed by device index (0-indexed)
                  Supported memory locations are:
                  - C:    Pinned host memory       (on NUMA node, indexed from 0 to [NUMA nodes-1])
                  - U:    Unpinned host memory     (on NUMA node, indexed from 0 to [NUMA nodes-1])
                  - B:    Fine-grain host memory   (on NUMA node, indexed from 0 to [NUMA nodes-1])
                  - G:    Global device memory     (on GPU device indexed from 0 to [GPUs - 1])
                  - F:    Fine-grain device memory (on GPU device indexed from 0 to [GPUs - 1])
                  - N:    Null memory              (index ignored)

Examples:::

   1 4 (G0->G0->G1)                   Uses 4 CUs on GPU0 to copy from GPU0 to GPU1
   1 4 (C1->G2->G0)                   Uses 4 CUs on GPU2 to copy from CPU1 to GPU0
   2 4 G0->G0->G1 G1->G1->G0          Copes from GPU0 to GPU1, and GPU1 to GPU0, each with 4 SEs
   -2 (G0 G0 G1 4 1M) (G1 G1 G0 2 2M) Copies 1Mb from GPU0 to GPU1 with 4 SEs, and 2Mb from GPU1 to GPU0 with 2 SEs

Round brackets and arrows' ->' may be included for human clarity, but will be ignored and are unnecessary
Lines starting with will be ignored. Lines starting with will be echoed to output

Single GPU-executed Transfer between GPUs 0 and 1 using 4 CUs::

   1 4 (G0->G0->G1)

Single DMA executed Transfer between GPUs 0 and 1::

   1 1 (G0->D0->G1)

Copy 1Mb from GPU0 to GPU1 with 4 CUs, and 2Mb from GPU1 to GPU0 with 8 CUs::

   -2 (G0->G0->G1 4 1M) (G1->G1->G0 8 2M)

"Memset" by GPU 0 to GPU 0 memory::

   1 32 (N0->G0->G0)

"Read-only" by CPU 0::

   1 4 (C0->C0->N0)

Broadcast from GPU 0 to GPU 0 and GPU 1::

   1 16 (G0->G0->G0G1)



