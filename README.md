# Cisco Certification Labs Repository

Welcome to the Cisco Certification Labs repository! This project is a centralized collection of lab exercises designed to help you prepare for various Cisco certification exams. The goal is to provide hands-on experience with the concepts and technologies covered in the exams.

## Repository Structure

This repository is organized by exam. Each exam has its own directory, which contains subdirectories for each specific lab. This structure makes it easy to find the labs relevant to the exam you are studying for.


.
├── CCNA-200-301
│   ├── Lab-01-Basic-Device-Configuration
│   │   ├── README.md
│   │   └── configuration_files
│   ├── Lab-02-VLANs-and-Trunking
│   │   ├── README.md
│   │   └── configuration_files
│   └── README.md  <-- Lists all labs for the CCNA exam
├── ENCOR-350-401
│   ├── Lab-01-Enterprise-Network-Design
│   │   ├── README.md
│   │   └── configuration_files
│   └── README.md  <-- Lists all labs for the ENCOR exam
└── README.md      <-- This file


## Available Exams

Below is a list of the Cisco certification exams for which labs are available in this repository. Click on an exam to see the full list of labs for that certification.

* [CCNA (200-301) - Implementing and Administering Cisco Solutions](CCNA-200-301/README.md)
* [ENCOR (350-401) - Implementing and Operating Cisco Enterprise Network Core Technologies](ENCOR-350-401/README.md)
* *... more exams to be added*

---

# Exam-Specific README Template

Below is a template you can use for the `README.md` file inside each exam folder (e.g., `CCNA-200-301/README.md`).

---

# Cisco Certified Network Associate (CCNA) 200-301 Labs

This section contains all the labs for the CCNA 200-301 exam. Each lab is in its own folder and includes a `README.md` with detailed instructions and any necessary configuration files.

## Labs

### Network Fundamentals
* **[Lab 01: Basic Device Configuration](./Lab-01-Basic-Device-Configuration/)**
    * *Topics covered: Hostnames, banners, enable secret, console and VTY line security.*
* **[Lab 02: IPv4 and IPv6 Addressing](./Lab-02-IPv4-and-IPv6-Addressing/)**
    * *Topics covered: Subnetting, IPv4 and IPv6 address assignment on interfaces.*

### Network Access
* **[Lab 03: VLANs and Trunking](./Lab-03-VLANs-and-Trunking/)**
    * *Topics covered: Creating VLANs, assigning ports to VLANs, configuring trunk links (802.1Q).*
* **[Lab 04: EtherChannel](./Lab-04-EtherChannel/)**
    * *Topics covered: Configuring LACP and PAgP.*
* **[Lab 05: Spanning Tree Protocol (STP)](./Lab-05-Spanning-Tree-Protocol/)**
    * *Topics covered: Observing STP convergence, configuring PortFast and BPDU Guard.*

### IP Connectivity
* **[Lab 06: Static Routing](./Lab-06-Static-Routing/)**
    * *Topics covered: Configuring static and default routes.*
* **[Lab 07: OSPFv2 for IPv4](./Lab-07-OSPFv2-for-IPv4/)**
    * *Topics covered: Single-area OSPFv2 configuration, verifying adjacencies.*
* **[Lab 08: First Hop Redundancy Protocols (FHRP)](./Lab-08-First-Hop-Redundancy-Protocols/)**
    * *Topics covered: HSRP configuration and verification.*

### IP Services
* **[Lab 09: Network Address Translation (NAT)](./Lab-09-Network-Address-Translation/)**
    * *Topics covered: Static NAT, Dynamic NAT, and PAT configuration.*
* **[Lab 10: Network Time Protocol (NTP)](./Lab-10-Network-Time-Protocol/)**
    * *Topics covered: Configuring NTP clients and servers.*
* **[Lab 11: DHCP](./Lab-11-DHCP/)**
    * *Topics covered: Configuring a router as a DHCP server.*

### Security Fundamentals
* **[Lab 12: Access Control Lists (ACLs)](./Lab-12-Access-Control-Lists/)**
    * *Topics covered: Standard and extended ACLs for traffic filtering.*
* **[Lab 13: Port Security](./Lab-13-Port-Security/)**
    * *Topics covered: Configuring port security to restrict MAC addresses.*
* **[Lab 14: SSH Configuration](./Lab-14-SSH-Configuration/)**
    * *Topics covered: Securing device access with SSH.*

### Automation and Programmability
* **[Lab 15: Introduction to REST APIs and JSON](./Lab-15-Introduction-to-REST-APIs-and-JSON/)**
    * *Topics covered: Understanding the structure of JSON, using a REST client to interact with a device.*

---

## How to Contribute

Contributions are welcome! If you have a lab you'd like to add or an improvement to an existing one, please follow these steps:

1.  **Fork** the repository.
2.  Create a **new branch** for your feature or bug fix.
3.  Add your lab or make your changes. Ensure you follow the existing directory structure.
4.  Submit a **pull request** with a clear description of your changes.
