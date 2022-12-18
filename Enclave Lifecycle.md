# Enclave Lifecycle?

## Enclave Creation&Execution

​	keystone: During enclave creation, Keystone measures the enclave memory to ensure that the OS has loaded the enclave binaries correctly to the physical memory. Keystone uses the initial virtual memory layout for measurement as the physical layout can legitimately vary (within limits) across different executions. Specifically, the SM expects the OS to initialize the enclave page tables and allocate physical memory for the enclave. During the enclave creation, the SM uses the initial page table provided by the OS, and SM checks if there are invalid mappings, ensuring unique virtual to physical mapping. Then the SM hashes page content along with virtual addresses and configuration data. For execution, the SM sets the PMP entries and transfers control to the enclave entry point.

​	

sanctum:	The OS creates an enclave by issuing a create enclave call that creates the enclave meta-data structure, which is Sanctum’s equivalent of the SECS. The enclave meta-data structure contains an array of mailboxes whose size is established at enclave creation time, so the number of pages required by the structure varies from enclave to enclave. 

## Enclave Destruction

​	keystone: The OS or the enclave itself may initiate enclave tear-down. During this phase, the SM clears the enclave memory region before it returns memory to the OS. All enclave resources, PMP entries, and enclave metadata are cleaned and then freed.