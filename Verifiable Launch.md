# Verifiable Launch

 While the ISA abstraction describes the abstraction over which software can use hardware resources, the more general interface that describes all interactions between software and hardware is the Execution Environment (EE), which includes not just the ISA, but other abstractions, such as concurrency and level of confinement.

We call an EE that implements trustworthy execution a TEE.

##  Firmware Trusted Execution Environments

### ROT(bootloader, hardware )



1.keystone(bootloader, hardware )



A Keystone root-of-trust can be either tamperproof software (e.g., a zeroth-order bootloader) or hardware (e.g., crypto engine). At each CPU reset the root-of-trust measures the SM image, generates a fresh attestation key from a cryptographically secure source of randomness, and stores it to the SM private memory. e root-of-trust then signs the measurement and the public key with a hardware-visible secret. ese are standard operations, can be implemented in numerous ways.



2Cure(bootloader)

We assume that the system performs secure boot on reset, whereas the first bootloader stored in CPU Ready-Only Memory (ROM), verifies the firmware through a chain of trust [53]. After verification, the firmware starts execution from a predefined address in the firmware code and loads the current firmware state from non-volatile memory (NVM) where it is stored encrypted, integrity- and rollback-protected. The cryptographic keys to decrypt and verify the firmware state are passed by the bootloader which loads the firmware into Random-access Memory (RAM). Rollback protection can be achieved, e.g., by making use of non-volatile memory with Replay Protected Memory Block (RPMB) partitions or by using eFuses as secure monotonic counters [56]. When a system shutdown is performed, the firmware stores its state in the NVM, encrypted and integrity- and rollback-protected.

3.Sanctum(hardware )

At reset, the processor must execute instructions beginning at an address corresponding to a trusted “boot ROM”.This first-stage bootloader, which we denote the “root of trust” is a program that loads a “payload” binary segment from untrusted storage into trusted system memory, derives the payload-specific keys (SKP , PKP ), and uses the device key to sign PKP and the trusted manufacturer’s endorsement of the processor.

Several implementation variations on this common theme are explored in this section, differing in their construction of SKDEV and their use of the trusted boot ROM memory.

 **ROT**--crypto engine:Secure Boot and Remote Attestation in the Sanctum Processor





**Firmware** is software that implements functionality that is logically part of the hardware. firmware is part of the trusted computing base (TCB) of all software, as it is responsible for critical system operations.

## measurement

1.Sanctum

​	The measurement root (mroot) is stored in a ROM at the top of the physical address space, and covers the reset vector. Its main responsibility is to compute a cryptographic hash of the security monitor and generate a monitor attestation key pair and certificate based on the monitor’s hash

​	The measurement root (mroot) is stored in a ROM at the top of the physical address space, and covers the reset vector. Its main responsibility is to compute a cryptographic hash of the security monitor and generate a monitor attestation key pair and certificate based on the monitor’s hash hand down the key to the monitor. 

​	The security monitor contains a header that includes the location of an attestation key existence flag. If the flag is not set, the measurement root generates a monitor attestation key pair, and produces a monitor attestation certificate by signing the monitor’s public attestation key with the processor’s private attestation key. The monitor attestation certificate includes the monitor’s hash.

### **SM**

: enclaves are managed by the software TCB, called Security Monitor (SM)

The SM state contains SK_d, OK_d , Certf_d , Chain_p(Certificate chain)  and a structure D_encl for each enclave installed on the device.

1.CURE

All CURE enclaves are managed by the software TCB, called Security Monitor (SM).a system that performs a secure boot on reset, verifies(Attestation) the firmware (including the SM) and then jumps to the entry point of the SM.

2.KeyStone（implement our SM on top of the Berkeley Boot Loader）

​	Keystone uses the isolation and programmable mode provided by RISC-V to build critical TEE guarantees. Specifically, we design a security monitor (SM) which enforces the TEE guarantees across the entire platform. The SM executes in M-mode and is programmed in C/assembly.

​	For customization, the platform operator specifies the SM configuration and trusts that the SM will enforce all relevant security boundaries between enclaves and the host/OS. Such a design allows Keystone to transparently extend the guarantees given by the hardware. For example, if the device has additional features (e.g., cache partitioning,**ref1**), the SM can enforce additional protection or improve the performance without changing the rest of the layers (e.g., OS, user applications). More importantly, it ensures that untrusted layers (e.g., OS) executing above the SM cannot circumvent enforcement.

3.Sanctum:

Sanctum’s hardware modifications enable strong isolation (integrity and confidentiality) of software containers (enclaves) with an insidious threat model of a remote software adversary able to subvert system software and actively tamper with network traffic. Sanctum also guarantees the integrity of an honest “Security Monitor” (SM), which is authenticated at boot, and maintains meaningful isolation guarantees, as was formally verified [37]. Sanctum uses RISC-V’s “machine mode” (highest privilege software) to host the SM, and maintains that the SM has exclusive access to a portion of DRAM (thus maintaining its integrity and protecting its keys). The SM routes interrupts and configures hardware invariants to ensure enclaves do not involuntarily share resources with any other software.



### **Attestation**

 is the third and final step of verifiable launch, where a verifier checks that the enclave has been launched correctly and that its initial state is as expected. More specifically, the verifier ensures that the enclave’s measurement and its underlying TCB match their expected reference values. In general, the basic attestation process is well-understood. First, the enclave’s underlying TCB constructs an attestation report containing the enclave’s measurement and its TCB. Then, the attestation report is integrity protected either through a cryptographic signature or a Message Authentication Code (MAC)

#### local

Local attestation is applicable when a verifier is co-located with the enclave on the same platform.

1.Scanctum

​	Sanctum is capable of handling local attestations without the presence of a trusted remote party. In this use case requests for attestation are authenticated by the signing enclave in order to prevent arbitrary enclaves from obtaining another enclave’s attestation.

#### remote

​	Attestation by hardware components of a platform allows remote clients to verify details of the system state. In particular, clients need to verify both the authenticity of the system performing the attestation and the integrity of their own remote application. 

​	A device vendor creates a unique asymmetric key pair **SK_d** and **PK_d** for each device, which is provisioned to the device during production, and a public key certificate **Cert_d** signed by the device vendor which can later be used to prove the legitimacy of the device in a remote attestation scheme.

​	To receive an integrity report which the SM creates by signing **Sig_encl** with **SK_d** and by attaching  **Cert_d** . The integrity report is then sent to the service provider by the enclave. 

1.Sanctum

​	The security monitor hashes a client enclave as it is initialized, and delegates its endorsement to the signing enclave, which is exclusively able to access **SK_p**. We rely on the isolation properties of Sanctum enclaves to guarantee the privacy of the signature. To avoid performing cryptographic operations in the security monitor, Sanctum instead implements a message passing primitive, whereby an enclave can receive a private message directly from another enclave, along with the sender’s measurement.

2.Cure



## references

1:Cryptographic Accelerators for Trusted Execution Environment in RISC-V processors