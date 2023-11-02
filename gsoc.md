---
layout: splash
title: "Google Summer of Code 2023: Enhancing OpenPiton+Ariane With a High-Performance Data Cache"
permalink: /projects/gsoc/
---
# Google Summer of Code 2023: Enhancing OpenPiton+Ariane With a High-Performance Data Cache
*Project advisors: Jonathan Balkind and Cesar Fuguet*

*Organization Name: FOSSi Foundation*

## Synopsis

The growing demand for High-Performance Computing (HPC) requires open-source solutions to meet modern application performance needs. Different open-source architecture frameworks have been developed by the community, such as OpenPiton. However, these frameworks often have performance limitations that hinder their ability to execute compute-intensive tasks.

[OpenPiton](https://doi.org/10.1145/2872362.2872414) is an open-source framework for designing many-core processors, which is compatible with different core architectures (RISC-V 32-bit, RISC-V 64-bit, x86, and SPARCv9). In this work, we focused on the integration of CVA6/Ariane into OpenPiton [(OpenPiton+Ariane)](https://carrv.github.io/2019/papers/carrv2019_paper_12.pdf).

The architecture of OpenPiton comprises one chipset and one or more tiles. The chipset houses modules used to communicate the tiles with the peripherals, such as the UART. The tile is used to build the mesh for many-core designs. In these designs, the tiles are interconnected by three NoC routers to generate the mesh. Each tile comprises the three NoC routers, the Ariane core, and the cache hierarchy, which consists of private L1 data and instruction caches, a private L1.5 cache, and a shared distributed L2 cache. 

Despite its advantages, OpenPiton faces performance limitations that restrict its adoption in the HPC domain. In this project, we centered on improving the performance of the integration of CVA6/Ariane into OpenPiton (OpenPiton+Ariane) by replacing the L1 data cache of Ariane with the High-Performance, Multi-Requester, Out-of-Order, [L1 data cache (HPDcache)](https://github.com/openhwgroup/cv-hpdcache).

## Project Idea

The main goal of the project can be summarized in Figures 1 and 2. Figure 1 shows the default OpenPiton design with Ariane's L1 data cache, while Figure 2 includes the HPDcache and the necessary support to connect it with the L1.5 cache. Moreover, the design is configurable, enabling users to choose between using Ariane's L1 data cache or the HPDcache.

|![Image of default design](/assets/images/GSoC_draw_Ariane.drawio.png)|
|:--:| 
|*Figure 1: OpenPiton+Ariane design with the L1 data cache of Ariane.*|
|:--:|

|![Image of HPDC](/assets/images/GSoC_draw_HPDC.drawio.png)|
|:--:| 
|*Figure 2: OpenPiton+Ariane design with the L1 HPDcache.*|
|:--:|

## Final report

The main goal of the project was developed in two stages. In the first stage, we focused on the design of the wrapper that allows the integration of the HPDcache into OpenPiton. Subsequently, in the second stage, we worked on the integration of the HPDcache into the OpenPiton MESI coherence protocol and on the verification of the whole system. In addition, as the main goal was completed ahead of schedule, we developed two extra features. First, we added support to enable write coalescing on OpenPiton, followed by the integration of OpenPiton+Ariane’s L1D and L1.5 parametric cache line size feature (PCL). 

### First stage

As mentioned before, the first stage involved integrating the HPDcache into OpenPiton through a wrapper. We began this process by reviewing the documentation and the RTL code of both systems. Once familiar, we started the design phase, planning how to incorporate HPDcache into OpenPiton.
After we completed the design, we started the implementation phase with the connection between HPDcache and the CVA6/Ariane core in OpenPiton. Then, we implemented the wrapper that allows performing the integration of the HPDcache with the P-Mesh Transaction-Response Interface (TRI). At this stage, we were focused on the cache accesses of a single core, avoiding the complexities introduced by the coherence protocol. To this end, we implemented the load, store, and instruction refill operations. 

The first stage concluded with an initial validation of the implementation, which included simple tests involving load and store instructions from a single core with no virtual memory. 

### Second stage

The second stage focused on integrating the HPDcache into the OpenPiton MESI coherence protocol and verifying the correctness of the entire system. Specifically, in this stage, we added support for atomic operations and for invalidations sent by the L1.5 cache to the HPDcache. 

Additionally, we integrated two extra features. In the first place, we added support for enabling write coalescing on OpenPiton. The usage of this feature helps reduce energy consumption and improve performance by aggregating multiple smaller write transactions, between L1D and L1.5, into a single one. Write coalescing can be enabled using the macro *WRITE_BYTE_MASK*. Secondarily, we integrated the PCL feature, which allows changing the cache line size of L1D and L1.5 caches between 16, 32 and 64 bytes. The desired value has to be selected during the model’s build through the *-config_l15_l1d_cacheline_size parameter*. 

As we wanted to maintain the possibility of using the default design (Ariane’s L1 data cache), we made the design configurable through the macro *PITON_ARIANE_HPDC*. In this way, if the macro is set, the selected data cache is the HPDcache, whereas if it is not used, Ariane's L1 data cache will be the one used.

During the development of this stage, extensive validation was necessary to ensure the system functions correctly. To this end, we followed a validation and debugging process with increasingly complex tests. First, we began checking that the RISC-V ISA tests work correctly, starting with the physical RISC-V ISA tests and then proceeding with the virtual ones. Next, we tested the system with single-core and multi-core benchmarks that stress the cache hierarchy, such as matmul, vvadd, spmv or stream. With the purpose of accelerating the simulations of up to 64 cores, we used the tool [Metro-Mpi](https://jbalkind.github.io/docs/date_2023_camera_ready.pdf). This allowed us to further validate the coherence protocol. Finally, we simulated the Linux boot process using the platform Veloce. This test allowed us to stress the system with non-traditional benchmark behaviors, ensuring that the system is functional. 

## Actual status of the project

The summarized status of the project is displayed in the table below.

| Version                           | RISCV P/V tests  | Multicore tests | Linux boot |
|-----------------------------------|:----------:|:---------:|:----------:|
| **CVA6’s WT D$**                  |<b><span style="color:green">PASS</span></b>|<b><span style="color:green">PASS</span></b>| <b><span style="color:green">PASS</span></b>|
| **CVA6’s WT D$ + PCL**            |<b><span style="color:green">PASS</span></b>|<b><span style="color:green">PASS</span></b>| <b><span style="color:orange">WIP</span></b>|
| **HPD$**                          |<b><span style="color:green">PASS</span></b>|<b><span style="color:green">PASS</span></b>| <b><span style="color:green">PASS</span></b>|
| **HPD$ + write coalescing**       |<b><span style="color:green">PASS</span></b>|<b><span style="color:green">PASS</span></b>| <b><span style="color:green">PASS</span></b>|
| **HPD$ + write coalescing + PCL** |<b><span style="color:green">PASS</span></b>|<b><span style="color:orange">WIP</span></b>| <b><span style="color:orange">WIP</span></b>|


Currently, we are working on the integration of the project into the official repositories. This progress can be tracked through the Pull Requests (PRs) made to [OpenPiton](https://github.com/PrincetonUniversity/openpiton/pull/136), [CVA6/Ariane](https://github.com/openhwgroup/cva6/pull/1575), and [HPDCache](https://github.com/openhwgroup/cv-hpdcache/pull/4) repositories.  
On the other hand, as the PCL extra feature is still in progress its updates can be followed on the [*OP_ariane_hpdc_pcl*](https://github.com/AileonN/Openpiton_noliete/tree/OP_ariane_hpdc_pcl) branch of the forked repository. 
