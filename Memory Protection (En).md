

# Memory Protection (En)

### Hardware Primitives

The hardware primitives are the basic building blocks of a hardware supported trusted execution environment. One of the most frequently used hardware primitives is Physical Memory Protection (PMP), which is provided by the RISC-V instruction set architecture to protect physical memory under all three modes. Some simple TEEs such as Keystone, chose to directly use PMP to protect the memory of enclaves while at the same time other more fine-grained TEEs only use physical memory protection to protect the memory of trusted softwares that are used to provide security management. For example the security monitor in CURE, the TagRoot in Timber-v are protected by PMP. Also, key memory regions such as ownership bitmap and Host Page Table in Penglai also relies on the protection of PMP.

Besides this commonly used hardware primitive, some TEEs also proposed proprietary hardware primitives to implement more advanced security features.  For example, Timber-V introduced security primitives to augment the register files of each CPU core, the system bus and the shared cache. Penglai proposed Guarded page table to implement memory isolation and Mountable Merkle Tree to implement scalable memory integrity protection.



### Implementation

The two biggest security concerns of memory protection are memory isolation and memory integrity protection. Memory isolation ensures that the secure memory of enclaves cannot be referenced by untrusted host OS or other malicious enclaves. Memory integrity protection ensures that the secure memory's content is not contaminated.

The implementation of memory protection varies, some TEE relies purely on the hardware primitives provided by RISC-V architecture while some other TEEs modified hardware to gain more advantages.

##### Memory Isolation

For the well-known Keystone TEE, it relies directly on PMP to implement memory isolation. Every enclave needs a PMP to protect its corresponding memory region. The memory region protected by PMP is theoretically very secure. However this way of managing memory is not very efficient as the number of PMPs is decided by the specific design of the hardware. The number of enclaves supported by Keystone is upper bounded by the number of PMPs of the hardware, which makes it not scalable enough for grand scale deployment.

Timber-V  divides four security domains which are U, TU, S and TS domains respectively. The user mode is used to support the running of user apps. The trusted user mode is used to support the execution of Enclaves. The supervisor mode is used to support the execution of untrusted OS. The trusted supervisor mode is used to support the running of TagRoot, which acts like the role of Supervisor in other TEEs. Besides all these, Timber-V also adds a 2 bit tag for every 32 bit word, which are the N tag for untrusted memory, TU tag for trusted user memory, TS tag for trusted supervisor memory and TC tag designated to mark the secure entry points to enter enclave. The tag engine will ensure that during execution, for every memory access, the trusted memory will not be accessed by untrusted code, the core TagRoot memory will not be accessed by malicious enclave marked by TU tag. Theoretically speaking, simply using tagging mechanism can fully work to isolate all individual processes. However this could introduce huge costs, so the designers of Timber-V TEE cleverly use one single MPU to isolate the processes.

Penglai uses ownership bitmap to mark the state of each physical page. The pages of security monitor and enclaves are marked as secure while the pages untrusted OS and apps are marked as insecure. To make sure the ownership bitmap is secure, a PMP is designated to protect it. The memory security protection of Penglai deeply relies on the secure software, security monitor, which is an important part of trusted computation base of Penglai. One security design intuition may suggest that for every memory access initiated by the untrusted OS or user apps, the security monitor should check the ownership bitmap to see whether it should grant the privilege to access the specified memory page or not. However this could cause an efficiency problem as accessing one single memory address would cause two memory accesses. Therefore Penglai moved the checking procedure to the address mapping stage. Penglai designated a host page table area protected by PMP in memory, it is used to store the page tables of the insecure processes such as untrusted OS and user apps. By modifying the page table walker, all the page tables in this area will not include mapping to secure memory pages. In this way, the unwanted memory accesses from untrusted software to secure memory is prevented at the stage of physical memory address mapping. 



### Hardware Changes

Minimal hardware change is a fundamental design principle that TEE designers should hold to ensure the portability and deployment of TEE.

Keystone, as a pure software TEE, makes no modifications to the hardware, making it able to be deployed to most of the commercial RISC-V products without special design to the hardware.

Penglai introduced self designed registers and page table walkers, which brings pretty big modification to hardware.

Timber-V only adds 2 bit tag for every 32 bit word, which is moderate.



