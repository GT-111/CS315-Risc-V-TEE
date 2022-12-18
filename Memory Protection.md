# Memory Protection

### Hardware Primitives

首先是几乎所有的TEE都用到的一个hardware primitive，也就是PMP，它是由RISC-V指令集架构所提供的一种用于各种模式下保护内存的方式。一些简单的TEE如Keystone选择直接使用PMP来保护enclave的内存，而一些更加fine grained的TEE则仅使用PMP去保护security monitor，Tagroot这样用于提供安全管理的可信软件的内存或是ownership bitmap，Host page table这些关键的区域的内存。

除此之外，还有一些各个TEE自己提出的硬件原语，比如Penglai用于实现memory isolation的Guarded Page Table和用于实现memory integrity protection 的 MMT.



### Implementation

实现内存安全要考虑的最重要的问题无非是两个，一个是内存隔离，一个是内存的完整性保护。

##### Memory Isolation

在实现内存隔离的方式上，Keystone是直接依赖于RISCV架构所提供的物理内存保护这一硬件原语。每一个enclave都需要一个PMP来保护其所对应的内存区域，被PMP保护的内存区域理论上很安全，但是这样的管理方式并不是很高效，因为PMP的数量和具体硬件有关，这使得部署Keystone的机器所能支持的enclave数量上限为PMP的数量。

Timber-V将memory在User mode和Supervisor mode这一划分的基础上进一步分为U和TU，S和TS。其中U mode用于支持user app的运行，TU支持Enclave的运行，S mode支持untrusted OS的运行，TS则用于支持Tag Root的运行，tag root相当于secure monitor的作用。除此之外，他还为每32位的word加了一个2 bit的tag，分别是用于表示untrusted memory的N tag，trusted user memory的TU tag， trusted supervisor memory的TS tag以及用于进入enclave的secure entry points的TC tag。tag engine会在运行时的每一次memory access保证trusted memory不会被untrusted code访问，最核心tagroot memory不会被TU tag所表示的enclave所访问。 理论上使用tag也可以做到隔离各个进程，但是这样开销太大，所以就改为了使用MPU来隔离各个进程。

Penglai使用ownership bitmap来标记每一个物理页的状态。其中monitor和enclave的secure的，untrusted OS和apps是不安全的。理论上在不受信任的应用访问内存的时候要依靠security monitor这个安全软件来check这个ownership bitmap，以判断能否取访问。而为了保护这个bitmap的安全性，则本身使用了PMP这个硬件原语来保护。但是仅仅使用ownership bitmap效率会比较低，因为他访问同一个地址需要访问两次内存。 penglai的聪明之处在于它将这个check给移到了address mapping 的阶段。在内存中专门使用PMP物理保护了一个Host page table area，这个区域专门用于放untrusted OS和user app这样unsecure的进程的page table。通过修改page table walker的方式，在这个区域中的所有page table都不会包含对于secure memory page的mapping。这样一来在物理地址mapping的时候就避免了不受信任的应用访问到安全的内存。

Sanctum 引⼊了⼀种简单的软件/硬件协同设计，它具有与SGX相同的抵御软件攻击的能⼒，并增加了针对内存访问模式泄漏的保护，例如页⾯错误监视攻击和缓存定时攻击。Sanctum使⽤概念上简单的缓存分区⽅案，该⽅案将计算机的DRAM分为相等⼤⼩的连续DRAM区域，并且每个DRAM区域在共享的末级缓存（LLC）中使⽤不同的集合，每个DRAM区域恰好分配给⼀个容器，因此容器在DRAM和LLC中都是隔离的。通过在上下⽂开关上进⾏刷新，可以将容器隔离在其他缓存中。像XOM，Aegis和Bastion⼀样，Sanctum还认为虚拟机管理程序，OS和应⽤程序软件在概念上属于⼀个单独的容器，通过使容器彼此隔离的相同措施，可以保护容器免受不受信任的外部软件的侵害。Sanctum依赖于受信任的安全监视器，该监视器是处理器执⾏的第⼀部分固件，并且具有与Aegis安全内核相同的安全属性。该监视器通过处理器ROM中的引导程序代码进⾏度量，并且其加密哈希包含在软件证明度量中。监视器验证操作系统的资源分配决策，例如，它确保两个不同的容器都⽆法访问DRAM区域。每个Sanctum容器管理映射其DRAM区域的⾃⼰的页表，并处理⾃⼰的页错误，因此，恶意操作系统⽆法学习会导致容器中出现页⾯错误的虚拟地址。Sanctum的硬件修改与安全监视器配合使⽤，以确保容器的页表仅引⽤容器DRAM区域内的内存。Sanctum的设计完全专注于软件攻击，不能提供任何物理攻击的保护。作者希望Sanctum的硬件修改能够与Aegis或Ascend中的物理攻击保护结合在⼀起。



### Hardware Changes

为了TEE的可移植性和便于部署，TEE的设计应该秉持minimal hardware change这样一个设计原则。

蓬莱引入了自己设计的register和page table walker这样的硬件修改，这个对于硬件的修改还是比较大的。

Keystone则是一个纯软件的TEE，对于硬件完全没有修改，这使得它可以部署在绝大多数市面上商用产品上。

Timber-V主要是给每32bit的word增加了2bit的tag，这对硬件的修改也不小。

### Overhead 

Timber-v 并不只依赖于tag isolation，如果只依赖于tag，那么tag的大小将会非常大。这样overhead会比较大。

MPU的使用可以减小tagging overhead。

