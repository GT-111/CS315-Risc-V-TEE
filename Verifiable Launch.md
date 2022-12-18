# Verifiable Launch

​	The typical verifiable boot procedure in TEEs is enclave measurement and attestation. Intuitively, a measurement of an enclave, and more generally

any software is a fingerprint of its initial state, typically constructed by a series of one or more cryptographic hashes
over the initial memory of the enclave. 
	 A trusted bootloader is typically the first component in this process and is considered the root of trust for the boot process. It is the first code to run upon system initialization. It is responsible for checking the measurements and/or signatures of the following components locally or utilizing a piece of trusted hardware such as a TPM.
	In one common setting, processor hardware or firmware in a boot ROM cryptographically measures the bootloader software and all software loaded by the bootloader, then uses the processor’s secret key to sign the measurement, producing a certificate. The manufacturer signs the public key associated with the processor’s secret key. The local or remote client verifies the signature of the processor’s public key using the manufacturer’s public key, verifies the measurement certificate utilizing the processor’s public key, and checks the measurement itself against a client-side stored hash of the expected system state. 

​	This manuscript focuses specifically on  RISC-V TEEs. The following section discusses the types of RTMs, measurement processes, and attestation schemes of the TEEs.

### Root of Trust (RTM)

 	The measurement process itself must be trustworthy; it begins at the Root of-Trust for Measurement (RTM) and is implemented as a chain-of-trust through the enclave’s TCB, which finally measures the enclave itself. Central to a verifiable launch process is an RTM, which serves as the trust anchor for the measurement process.

​	Currently, TEEs use one of three types of RTMs, namely, static (SRTM), dynamic (DRTM), and hardware-based (HW).**(ref1)**. The TEEs mentioned in this manuscript all support SRTM, keystone supports both SRTM and HW and Timber-V doesn't provide the specification of RTM.

​	SRTM is created by an unbroken chain of trust from the code module that first executes at system reset to the code that runs in the enclave

​	In Cure, similar t the common setting, the first bootloader which is a program that loads a “payload” binary segment from untrusted storage into trusted system memory stored in CPU Ready-Only Memory (ROM), verifies the firmware through a chain of trust. After verification, the firmware starts execution from a predefined address in the firmware code and loads the current firmware state from non-volatile memory (NVM) where it is stored encrypted, integrity- and rollback-protected. The cryptographic keys to decrypt and verify the firmware state are passed by the bootloader which loads the firmware into Random-access Memory (RAM). 

​	 Keystone root-of-trust can be either tamper-proof software (e.g., a  bootloader) or hardware (e.g., crypto engine**(ref2**)).  At each CPU reset the root-of-trust measures the SM image, generates a fresh attestation key from a cryptographically secure source of randomness, and stores it in the SM's private memory. the root-of-trust then signs the measurement and the public key with a hardware-visible secret. these are standard operations, that can be implemented in numerous ways.

​	For Sanctum, at reset, the processor must execute instructions beginning at an address corresponding to a trusted “boot ROM”.This first-stage bootloader derives the payload-specific keys (SK_P, PK_P ), and uses the device key to sign PK_P and the trusted manufacturer’s endorsement of the processor. And Penglai follows Sanctum to implement the secure boot using the tamper-proof software approach.

​	

## measurement

​	The root of trust signs trusted system software (Payload) and its derived key pair (PKP, SKP ) at boot, differing in their construction of SKDEV and their use of the trusted boot ROM memory.

​	The root of trust for measurement roots a chain of trust. Every component in the chain is typically measured (and optionally verified) before it starts executing. The measurement process for all these components in the chain of trust and the enclave itself is similar. It entails mapping the binary executable of the component being measured to memory and computing one or more cryptographic hashes on that memory. 

​	Sanctum's measurement root (mroot) is stored in a ROM at the top of the physical address space and covers the reset vector. Its main responsibility is to compute a cryptographic hash of the security monitor and generate a monitor attestation key pair and certificate based on the monitor’s hash and hand down the key to the monitor. 

​	Keystone uses the initial virtual memory layout for measurement as the physical layout can legitimately vary (within limits) across different executions 

​	In Penglai, the MMT meta-zone will also be initialized in this phase. After all these early configurations, the bootloader will load and calculate the measurement of the secure monitor performing the hash measurement.

## 	Attestation

 	It is the third and final step of verifiable launch, where a verifier checks that the enclave has been launched correctly and that its initial state is as expected. More specifically, the verifier ensures that the enclave’s measurement and its underlying TCB match their expected reference values. 

​	In general, the basic attestation process is well-understood. First, the enclave’s underlying TCB constructs an attestation report containing the enclave’s measurement and its TCB. Then, the attestation report is integrity protected either through a cryptographic signature or a Message Authentication Code (MAC)

​	There are two flavors of attestation: local attestation and remote attestation.

​	Among the surveyed RISC-V TEEs, Sanctum and Timber-V support both Remote and local Attestation, whereas the rest apply a remote attestation scheme.

#### local Attestation

​	Local attestation is applicable when a verifier is co-located with the enclave on the same platform. 

​	Sanctum is capable of handling local attestations without the presence of a trusted remote party. In this use case requests for attestation are authenticated by the signing enclave in order to prevent arbitrary enclaves from obtaining another enclave’s attestation.

​	Local attestation is implicitly achieved using shared memory without the involvement of cryptographic secrets. By offering and accepting shared memory, both involved enclaves identify their communication partner via its EID, thus mutually attesting each other.

#### remote Attestation

​	 In contrast, remote attestation is meant for use by a remote verifier that is not on the same platform as the enclave being attested.

​	Attestation by hardware components of a platform allows remote clients to verify details of the system state. In particular, clients need to verify both the authenticity of the system performing the attestation and the integrity of their own remote application. 

​	A device vendor creates a unique asymmetric key pair **SK_d** and **PK_d** for each device, which is provisioned to the device during production, and a public key certificate **Cert_d** signed by the device vendor which can later be used to prove the legitimacy of the device in a remote attestation scheme.

​	To receive an integrity report which the SM creates by signing **Sig_encl** with **SK_d** and by attaching  **Cert_d** . The integrity report is then sent to the service provider by the enclave. 

##### Remote Attestation with Generic Payload

​	Keystone, Cure, Timber-V, Penglai uses a standard scheme(**ref3) to bind the **Attestation** with a secure channel construction, The SM allows the enclave to include limited arbitrary data (e.g., Diffe-Hellman key parameters) to be included in the signed Attestation report. 

##### Remote Attestation with Sanctum Payload

​	The primary distinction is that the payload in the case of Sanctum is its security monitor which is responsible for enforcing some of the isolation properties for client enclaves as well as spawning a special “signing enclave”.

​	The security monitor hashes a client enclave as it is initialized, and delegates its endorsement to the signing enclave, which is exclusively able to access **SK_p**. We rely on the isolation properties of Sanctum enclaves to guarantee the privacy of the signature. 

​	To avoid performing cryptographic operations in the security monitor, Sanctum instead implements a message passing primitive, whereby an enclave can receive a private message directly from another enclave, along with the sender’s measurement.

##### Secure Memory initialization with Shadow Fork in Penglai as a special case

​	calculating the measurement of memory takes up the majority of time in attestation (e.g., >90%) To mitigate this overhead, the monitor will calculate the measurement for a shadow enclave in advance (creation phase). A user can leverage enclave_fork with a manifest containing the sealed enclave measurement and the user’s public key. Later, the monitor will unseal the enclave measurement (using the user’s public key) and check it with the shadow enclave’s measurement. If the measurement is matched, the monitor will fork a new instance based on the shadow enclave. Otherwise, the monitor will deny the request. Therefore, we can mitigate the attestation costs during the boot critical path.

## **Secure Monitor**

​	Enclaves are managed by the software TCB, called Security Monitor (SM). ALL the surveyed RISC-V TEE utilize similar software varying in implementation and functionality details. 

​	All CURE enclaves are managed by the software TCB, called Security Monitor (SM), as in other TEE architectures. The SM is implemented as a sub-space enclave separated from the firmware in memory, which is enforced by the hardware security primitives. The system  performs a secure boot on reset, verifies the firmware (including the SM), and then jumps to the entry point of the SM

​	Keystone uses the isolation and programmable mode provided by RISC-V to build critical TEE guarantees.  Security monitor (SM) which enforces the TEE guarantees across the entire platform executes in M-mode and is programmed in C/assembly. The SM allows customization so that the platform operators are able to specify the SM configuration and trust that the SM will enforce all relevant security boundaries between enclaves and the host/OS. Such a design allows Keystone to transparently extend the guarantees given by the hardware.

​	Sanctum uses RISC-V’s “machine mode” (highest privilege software) to host the SM which is authenticated at boot, and maintains that the SM has exclusive access to a portion of DRAM (thus maintaining its integrity and protecting its keys). The SM routes interrupt and configure hardware invariants to ensure enclaves do not involuntarily share resources with any other software.

​	During system boot, Penglai's secure monitor is loaded and verified by the boot ROM (aka. secure boot). It then takes control of the system and protects itself with hardware-supported memory isolation (e.g., RISC-V PMP). It also leverages encryption and Mountable Merkle Tree to protect itself from physical memory attacks 

​	During system boot, the CPU bootrom will configure the MMT meta-zone range in physical memory, initialize all SubTree root entries, construct RootTree nodes and allocate the first subtree to protect the memory of the secure monitor. After this, the secure monitor takes control of subsequent booting stages.secure monitor is the only privileged software that can manage secure memory and record memory status (secure, normal and SubTree node) in the bitmap. As the secure monitor cannot directly allocate memory, the host kernel will allocate free memory (used as secure memory and SubTree node later) and transfer it to the secure monitor. The secure monitor configures secure memory and its corresponding SubTree in this memory.

​	Timber-V develops a small trust manager for TIMBER-V, called TagRoot whose functions are similar to the SM in other  TEEs.  Loading of TagRoot itself is protected using secure boot . Once loaded, TagRoot can protect itself in an isolated execution container similar to enclaves by using tag isolation via TS-tag and secure entry points protected via TC-tag together with TS-mode MPU slots.It runs in trusted supervisor mode (TS-mode) and offers privileged trusted services to the untrusted operating system as well as unprivileged trusted services to enclaves.

## references

## Refs

1. Schneider, Moritz et al. “SoK: Hardware-supported Trusted Execution Environments.” *ArXiv* abs/2205.12742 (2022): n. pag.

​	2. crypto engine:Secure Boot and Remote Attestation in the Sanctum Processor

​	3.2:Ilia Lebedev, Kyle Hogan, and Srinivas Devadas. 2018. Secure Boot and Remote Aestation in the Sanctum Processor. In CSF.