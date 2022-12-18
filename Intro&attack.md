# Introduce

With the application of computing devices in various fields, more and more personal information and secret data are stored in computing devices For example, mobile and embedded devices not only bind user ID cards and bank cards, but also add mobile payment functions However, these security sensitive information lacks a special protection mechanism and is easy to be stolen by attackers, thus causing extremely serious security risks to users In order to provide a secure execution environment for user applications, the academic community has proposed the concept of Trusted Execution Environment (TEE), which ensures the security of application execution environment by isolating hardware and software.

Trusted Execution Environment is a kind of running environment based on hardware isolation mechanism and independent of other areas of the system. With specific hardware, TEE can ensure that the memory and execution state of the protected application cannot be stolen or tampered with by other components during operation, even if the system components cannot know the relevant content, thus providing secure computing capabilities and supporting privacy protection functions. In privacy computing technology, TEE uses hardware isolation to make the computing process invisible to the outside world. It operates in a protected and secure world with less speed loss, greater advantages in performance, and more applicable security and privacy protection scenarios. For scenarios requiring a large amount of data computing, TEE is often one of the most important technologies under the privacy computing scheme.

RISC-V is a simple, open and free new instruction set architecture. The biggest feature of RISC-V is "openness". Its openness allows it to be freely used for any purpose, and allows anyone to design, manufacture and sell RISC-V based chips or software. This openness is the first time in the processor field. Compared with ARM architecture, RISC-V has obvious advantages. RISC-V is an open architecture. From historical experience, an open ecology will be better than a closed one. RISC-V technology is a latecomer technology, so it can summarize the experience and lessons of predecessors and be relatively simple and clean.

As a new architecture, RISC-V's demand for privacy computing has also been put on the agenda. In this paper, by analyzing a variety of open source TEE implemented based on RISC-V framework, we summarize the paper from the following three aspects: (1) (2) (3), summarize the characteristics and implementation methods of each TEE, and finally propose the possible design direction of RISC-V TEE in the future according to our analysis results.

# Adversary model

### side-channel attack^[1]^

RISC-V TEE performs well against side channel attacks. Side-channel attack is a method to obtain private information by using side channel information such as time consumption, power consumption or electromagnetic radiation of encrypted electronic devices during operation. The effectiveness of such attacks is far higher than that of traditional methods of mathematical analysis or brute force cracking against cryptographic algorithms, which poses a serious threat to cryptographic devices.

1. Cache attack, which obtains some sensitive information in the cache by gaining access to the cache, for example, the attacker gains access to the physical host of the cloud host to gain access to the storage;
2. Timing attack, which infers the operation used by the time of device operation, or infers which storage device the data is located by comparing the time of operation, or uses the time difference of communication to steal data;
3. For the bypass attack based on power consumption monitoring, the operating power consumption of different hardware circuit units of the same device is also different, so the power consumption of a program will change with which hardware circuit unit the program uses, so it can be inferred which hardware unit the data output is located in to steal data;
4. Electromagnetic attack. The equipment will leak electromagnetic radiation during calculation. If properly analyzed, the information contained in the leaked electromagnetic radiation (such as text, sound, image, etc.) can be analyzed. This attack method is used for non cryptographic attacks and other eavesdropping behaviors, such as TEMPEST attacks (such as Van Eyck eavesdropping, radiation monitoring) in addition to cryptographic attacks;
5. Data residue can enable sensitive data that should be deleted to be read out (such as cold-boot attack);

###  physical attack





[1]杨帆,张倩颖,施智平,关永.可信执行环境软件侧信道攻击研究综述[J/OL].软件学报:1-23[2022-12-18].DOI:10.13328/j.cnki.jos.006501.
