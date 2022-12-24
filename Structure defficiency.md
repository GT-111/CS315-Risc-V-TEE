## TEE deficiency

​	Existing TEEs commonly offer only a single enclave type which is a one-size-fits-all approach, however, different services need varied configured enclaves that can adjust to their demands. Based on the structure deficiency, applications need to be modified when being ported to TEE which produces additional latency, and resource costs. 

​	 Keystone provides kernel-space enclaves on RISC-V. A Keystone enclave comprises a runtime (RT) which executes in S-mode and an enclave application (eapp) executing in U-mode. Both RT and eapp are part of the enclave address space which is isolated from the untrusted OS and other user applications.Most importantly, the RT is specified by the user and can be configured according to the needs of the target eapp it needs to support. The RT manages the life-cycle of the user code executing in the enclave, manages memory, services syscalls, etc. Specifically, the RT uses a limited set of API functions exposed by M-mode via the RISC-V supervisor binary interface (SBI) to exit or pause the enclave. Further, the RT can request the SM to perform operations on behalf of the eapp (e.g., attestation, get random values) via the SBI. Keystone follows the design principle of maintaining strict compatibility with existing user-code programming notions. The gap between Keystone restrictions and traditional programs is bridged via the RT. Specifically, an eapp may continue to assume a fully functional OS, and the RT can provide the necessary safe functionality,which means thee developer can choose to execute existing non-enclave applications inside an enclave without any modification. However, Keystone design is still limited to a single enclave type.

​	CURE's design implements "Dynamic enclave boundaries" which means the trust boundaries of an enclave can be freely configured in order that enclaves at different privilege levels can be supported. Sub-space enclaves provide vertical isolation at all execution privilege levels; User space enclaves provide isolated execution to unprivileged applications. Kernel-space enclaves can comprise only the kernel space, or the kernel and user space. A runtime (RT) component similar to Keystone's RT is implemented in inside. Self-contained enclaves allow isolated execution environments that span multiple privilege levels.

​	Penglai utilizes shadow enclave to boost startup by forking a new instance and server enclave to achieve the enclave chain which is common in serverless scenarios..  Shadow enclave which is a clean template is the only entity that can be forked and only contains code and data segments. During the fork, the SM will share the read-execute code and read-only segments, copy other writable parts, and initialize the stack of a new instance based on Shadow Enclave. Despite the enclave and shadow enclave, Penglai also implements another type of enclave called server enclave which does not have running context (e.g., time slice, ocall handler) but inherits it from other enclaves. Hence, the server enclave cannot run alone. When creating a server enclave, it needs to be assigned a unique name as its identification. Other enclaves can acquire the handle of this server enclave with its unique server name. Besides, the server enclave can also perform partial functionalities of OS. 

​	Second, they cannot efficiently support emerging applications such as *Function* *as* a Service, Machine Learning as a Service, etc. The requirement for extra secure channels to peripherals (e.g., accelerators), or the demand for the computational power of multiple cores varies among applications. 	

​	CURE enables the exclusive assignment of system resources, e.g., peripherals, CPU cores, or cache resources to single enclaves. CURE adds two registers per peripheral to the arbiter of the peripheral bus to achieve enclave-to-peripheral binding.