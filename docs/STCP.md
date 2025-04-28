# **Security Tool Context Protocol (STCP) Specification for Be-Secure Ecosystem**

Version: 0.1 (Draft)  
Date: 2025-04-27  
**Table of Contents**

1. **Introduction**  
   * 1.1. Purpose and Motivation  
   * 1.2. Scope  
   * 1.3. Goals  
   * 1.4. Relationship to Other Standards and Protocols  
   * 1.5. Terminology  
   * 1.6. Document Conventions  
2. **Core Concepts and Architecture**  
   * 2.1. Be-Secure Ecosystem Overview  
   * 2.2. BeSEnvironment (Be-Secure Environment)  
   * 2.3. BeSPlaybook (Be-Secure Playbook)  
     * 2.3.1. Playbook Naming Conventions  
   * 2.4. BeSLab Plugin (Agent)  
   * 2.5. OSAR (Open Source Assessment Report)  
   * 2.6. BeSLab Blueprint  
   * 2.7. Architectural Model and Interactions  
3. **BeSEnvironment and BeSPlaybook Lifecycle Functions**  
   * 3.1. Overview of Lifecycle Functions  
   * 3.2. Initialization Phase  
   * 3.3. Configuration Phase  
   * 3.4. Execution Phase  
   * 3.5. Aggregation/Reporting Phase  
   * 3.6. Termination Phase  
   * 3.7. Playbook Implementation Lifecycle Functions  
   * 3.8. BeSEnvironment Script Lifecycle Functions  
4. **STCP Specification**  
   * 4.1. Protocol Goals and Scope (Reiteration)  
   * 4.2. Communication Model (STCP Actors)  
   * 4.3. Transport Mechanisms  
   * 4.4. Message Format (JSON-RPC 2.0 Adaptation)  
     * 4.4.1. Common Message Headers  
     * 4.4.2. Request Message Structure  
     * 4.4.3. Response Message Structure  
     * 4.4.4. Notification Message Structure  
     * 4.4.5. Error Handling  
   * 4.5. Standardized Commands  
     * 4.5.1. Environment Management Commands  
     * 4.5.2. Plugin Lifecycle Commands  
     * 4.5.3. Execution Commands  
     * 4.5.4. Data Retrieval Commands  
     * 4.5.5. Status and Control Commands  
     * 4.5.6. Capability Discovery Commands  
   * 4.6. Context Passing Mechanisms  
   * 4.7. State Management  
5. **STCP Integration with Lifecycle Functions**  
   * 5.1. Mapping STCP Commands to Lifecycle Phases  
   * 5.2. Interaction Flow: Initialization Phase  
   * 5.3. Interaction Flow: Configuration Phase  
   * 5.4. Interaction Flow: Execution Phase  
   * 5.5. Interaction Flow: Aggregation/Reporting Phase  
   * 5.6. Interaction Flow: Termination Phase  
6. **BeSLab Plugin Onboarding and Execution**  
   * 6.1. Plugin Requirements  
     * 6.1.1. STCP Compliance  
     * 6.1.2. Plugin Manifest Definition  
     * 6.1.3. Security Considerations  
   * 6.2. Capability and Dependency Declaration (within Manifest)  
   * 6.3. BeSLab Blueprint Specification for Plugins  
   * 6.4. Plugin Packaging and Distribution (Considerations)  
   * 6.5. Plugin Instantiation as Agents  
7. **Security and Trust Architecture using DIDs and VCs**  
   * 7.1. Overview of Trust Model  
   * 7.2. Decentralized Identifiers (DIDs) Management  
     * 7.2.1. DID Methods (Recommended/Supported)  
     * 7.2.2. DID Generation and Registration (BeSLabs, Plugins, Users)  
     * 7.2.3. DID Resolution Requirements  
   * 7.3. Verifiable Credentials (VCs) Usage  
     * 7.3.1. VC Data Model and Formats  
     * 7.3.2. VC Types and Schemas  
     * 7.3.3. VC Issuance Processes  
     * 7.3.4. VC Verification Processes  
     * 7.3.5. VC Revocation Mechanisms  
   * 7.4. Secure Communication Channels  
     * 7.4.1. Establishing Secure Sessions  
     * 7.4.2. Message Encryption and Signing  
     * 7.4.3. Mutual Authentication  
8. **OSAR Generation, Exchange, and Sharing**  
   * 8.1. OSAR Definition and Structure  
     * 8.1.1. Leveraging SARIF Standard  
     * 8.1.2. OSAR Metadata  
     * 8.1.3. OSAR Fragments vs. Final Report  
   * 8.2. OSAR Generation Process  
     * 8.2.1. Plugin Outputting OSAR Fragments  
     * 8.2.2. Aggregation by BeSPlaybook/BeSEnvironment  
     * 8.2.3. OSAR VC Attestation  
   * 8.3. Secure OSAR Exchange Protocol  
     * 8.3.1. Requesting OSAR (Summary/Detailed)  
     * 8.3.2. Verifiable Presentations (VPs) for Authorization  
     * 8.3.3. Secure Transfer Mechanism  
   * 8.4. Access Control and Sharing Policies  
9. **Conclusion**  
10. **Appendices**  
    * A. STCP Command Reference Table  
    * B. Verifiable Credential Usage Summary Table  
    * C. Example STCP Message Sequences (JSON)  
    * D. Example Plugin Manifest (JSON)  
    * E. Example OSAR Structure (SARIF-based JSON)  
    * F. Normative References  
    * G. Non-Normative References

## ---

**1\. Introduction**

### **1.1. Purpose and Motivation**

The Be-Secure ecosystem is envisioned as a collaborative platform dedicated to enhancing the security posture of open-source software through standardized assessment methodologies. A critical challenge in realizing this vision is the effective orchestration and interaction of a diverse array of security analysis tools. Currently, integrating and managing these tools often involves bespoke solutions, hindering interoperability and automation.1

The Security Tool Context Protocol (STCP) is introduced to address this challenge. STCP defines a standardized communication protocol specifically designed for the Be-Secure ecosystem. Its primary purpose is to govern the interactions between orchestrating components (BeSPlaybooks), secure execution environments (BeSEnvironments), and the security tools themselves, represented as agents (BeSLab Plugins).

By providing a common language and interaction pattern, STCP aims to enable seamless interoperability between different security tools, regardless of their origin or underlying technology. This standardization facilitates the automated execution of complex security assessments defined in BeSPlaybooks within the controlled confines of BeSEnvironments. Furthermore, STCP incorporates robust security and trust mechanisms from the outset, leveraging Decentralized Identifiers (DIDs) and Verifiable Credentials (VCs) to ensure the authenticity and integrity of components, actions, and results.3 A key outcome facilitated by STCP is the generation of standardized, cryptographically verifiable Open Source Assessment Reports (OSARs), providing trustworthy insights into software security.

### **1.2. Scope**

The scope of the Security Tool Context Protocol (STCP) specification encompasses the definition of the communication protocol governing interactions between the following core components within the Be-Secure ecosystem:

* **BeSPlaybook (Orchestrator):** The entity initiating STCP commands to manage the assessment workflow.  
* **BeSEnvironment (Environment Manager):** The entity receiving commands, managing the execution environment, and relaying commands to plugins.  
* **BeSLab Plugin (Agent):** The entity representing a security tool, receiving commands via the environment, executing tasks, and returning results.

Specifically, this specification defines:

* The architecture and roles of STCP actors.  
* The message formats, transport mechanisms, and standardized commands used for communication.  
* The integration of STCP with the defined lifecycle functions of BeSEnvironments and BeSPlaybooks.  
* The requirements and process for onboarding security tools as STCP-compliant BeSLab Plugins.  
* The security and trust model based on Decentralized Identifiers (DIDs) and Verifiable Credentials (VCs), including their application to identity, attestation, and secure communication.  
* The procedures for generating and securely exchanging Open Source Assessment Reports (OSARs) using STCP and DIDs/VCs.

The following aspects are considered **out of scope** for this specification:

* The internal implementation details or logic of specific security tools encapsulated by BeSLab Plugins.  
* The detailed implementation specifics of BeSEnvironment provisioning beyond the definition provided by the BeSLab Blueprint.  
* The underlying infrastructure (e.g., cloud providers, container runtimes) used to host BeSEnvironments.  
* User interface (UI) or user experience (UX) design for interacting with BeSPlaybooks or viewing OSARs.  
* The specific implementation details of DID methods or VC cryptographic suites beyond referencing conformance to W3C standards.  
* Detailed policy management for OSAR sharing beyond the protocol mechanisms.

This scope definition aligns with the approach taken by similar protocol specifications, such as SARIF, which defines the format but not the APIs or user experiences for interacting with it.5

### **1.3. Goals**

The primary goals for the Security Tool Context Protocol (STCP) are:

1. **Interoperability:** Enable diverse security tools (BeSLab Plugins) from different vendors or developers to work together seamlessly within the Be-Secure ecosystem, eliminating the need for custom integrations.2  
2. **Automation:** Facilitate the fully automated execution of security assessment workflows (BeSPlaybooks) from environment setup to reporting, reducing manual effort and increasing efficiency.6  
3. **Security:** Ensure secure communication channels between components, protecting the integrity and confidentiality of commands, configurations, and results.8  
4. **Trust:** Establish a high degree of trust in the assessment process and its outcomes through cryptographic verification of component identities (Plugins, Environments), execution attestations, and report authenticity using DIDs and VCs.3  
5. **Standardization:** Define clear, unambiguous standards for tool interaction (installation, configuration, execution, data retrieval) and assessment reporting (OSAR format).  
6. **Extensibility:** Design the protocol to be extensible, allowing for the future addition of new commands, capabilities, and plugin types without breaking backward compatibility.11  
7. **Lifecycle Integration:** Tightly integrate STCP operations with the defined lifecycle functions of BeSEnvironments and BeSPlaybooks, providing a coherent operational model.

These goals are informed by the design principles of related protocols like DIDs (Control, Privacy, Security, Proof-based) 3 and OpenC2 (Technology Agnostic, Concise, Abstract, Extensible).7

### **1.4. Relationship to Other Standards and Protocols**

STCP builds upon and relates to several existing standards and protocols:

* **Model Context Protocol (MCP):** STCP draws inspiration from MCP's client-server architecture, the use of JSON-RPC 2.0 for messaging, and patterns for tool discovery and invocation.1 MCP provides a model for connecting applications (like LLM hosts) to external tools/services.14 However, STCP significantly diverges by focusing specifically on the lifecycle of security tools (including installation and configuration, which are typically stateful operations unlike many MCP tool calls), operating within a managed, isolated environment (BeSEnvironment), and mandating a decentralized trust framework based on DIDs and VCs from its inception, rather than relying solely on user consent mechanisms often highlighted for MCP.8  
* **Open Command and Control (OpenC2):** OpenC2 provides a standardized language for commanding cyber defense technologies, focusing on the "Action" part of the security process.6 Its structure (Action, Target, Arguments) 11 offers relevant patterns for defining STCP's execution commands. STCP, however, covers a broader scope, encompassing the entire lifecycle of tool interaction within an assessment playbook, including setup, configuration, and reporting, not just command execution. OpenC2 is designed for operational command and control, while STCP orchestrates assessment workflows.7  
* **FIPA Agent Communication Language (ACL):** FIPA ACL provides a mature framework for general-purpose agent communication based on speech act theory, defining performatives and interaction protocols.17 BeSLab Plugins function as agents within the STCP framework. While conceptually aligned in treating plugins as communicating agents, STCP is purpose-built for the specific domain of security tool orchestration within the Be-Secure context, defining a concrete set of commands and a trust model tailored to this domain, rather than the general-purpose, ontology-dependent communication defined by FIPA ACL.21  
* **Static Analysis Results Interchange Format (SARIF):** STCP mandates that the Open Source Assessment Report (OSAR) generated as the output of a BeSPlaybook execution SHOULD be based on the SARIF standard (specifically version 2.1.0 or later).23 SARIF is an OASIS standard designed for the output of static analysis tools.5 Leveraging SARIF ensures that OSARs are machine-readable and compatible with a wide range of existing security dashboards, IDE integrations, and vulnerability management tools that already consume this format.26 STCP defines how these SARIF-based reports (or fragments) are generated and securely exchanged.  
* **W3C Decentralized Identifiers (DIDs) and Verifiable Credentials (VCs):** STCP fundamentally relies on the W3C standards for DIDs 3 and VCs.4 DIDs provide the basis for decentralized, self-sovereign identifiers for all actors (BeSLabs, Plugins, Users) and artifacts (OSARs). VCs provide the mechanism for making verifiable attestations about these entities and artifacts (e.g., plugin capabilities, execution integrity, report authenticity).10 STCP defines the specific types of VCs used and how they integrate into the protocol flow for establishing trust.  
* **DID Communication (DIDComm):** While STCP defines its own core command structure over HTTPS/STDIO, patterns from DIDComm 9 are influential, particularly for establishing secure, mutually authenticated communication channels using DIDs and for the peer-to-peer exchange of sensitive data like OSARs between BeSLabs or with users.41 DIDComm's focus on secure, private, transport-agnostic messaging aligns well with STCP's security goals.

### **1.5. Terminology**

For the purposes of this specification, the following terms and definitions apply:

* **Agent:** A software component that acts on behalf of an entity, capable of communicating using a defined protocol. In STCP, BeSLab Plugins act as agents. (Ref: FIPA concepts 17)  
* **BeSEnvironment (Be-Secure Environment):** An isolated, ephemeral execution environment, provisioned via a BeSLab Blueprint, hosting BeSLab Plugin execution. Acts as the Environment Manager in STCP.  
* **BeSPlaybook (Be-Secure Playbook):** A script or definition file orchestrating a security assessment workflow, including BeSEnvironment lifecycle management and BeSLab Plugin execution via STCP. Acts as the Orchestrator in STCP.  
* **BeSLab Blueprint:** A template defining the configuration and resources required to instantiate a BeSEnvironment.  
* **BeSLab Plugin:** A software component encapsulating a security tool, acting as an STCP-compliant Agent within a BeSEnvironment.  
* **Decentralized Identifier (DID):** A globally unique persistent identifier that does not require a centralized registration authority, controlled by the DID subject. Conforms to.3  
* **DID Document:** A set of data describing a DID subject, including mechanisms like cryptographic keys and service endpoints used for interaction and verification. Conforms to.3  
* **DID Resolution:** The process of obtaining the DID document associated with a DID. Conforms to.29  
* **Environment Manager:** The STCP actor role fulfilled by the BeSEnvironment, responsible for managing the environment and mediating communication between the Orchestrator and Plugin Agents.  
* **Holder:** An entity that possesses one or more Verifiable Credentials and can create Verifiable Presentations from them. Conforms to.4  
* **Issuer:** An entity that creates and signs Verifiable Credentials. Conforms to.4  
* **Lifecycle Function:** One of the distinct phases (Initialization, Configuration, Execution, Aggregation/Reporting, Termination) in the operation of a BeSEnvironment and BeSPlaybook execution.  
* **Orchestrator:** The STCP actor role fulfilled by the BeSPlaybook runner, responsible for initiating commands and managing the overall assessment workflow.  
* **OSAR (Open Source Assessment Report):** The standardized, verifiable output report generated from a BeSPlaybook execution, typically based on the SARIF format.23  
* **OSAR Fragment:** A partial OSAR containing results from a single BeSLab Plugin execution, intended for aggregation into a final OSAR.  
* **Plugin Agent:** The STCP actor role fulfilled by a BeSLab Plugin instance running within a BeSEnvironment.  
* **Plugin Manifest:** A metadata file associated with a BeSLab Plugin, describing its identity, capabilities, parameters, and dependencies.  
* **Security Tool Context Protocol (STCP):** The protocol defined in this specification for communication between Orchestrators, Environment Managers, and Plugin Agents.  
* **Verifier:** An entity that receives Verifiable Credentials or Verifiable Presentations for verification. Conforms to.4  
* **Verifiable Credential (VC):** A tamper-evident set of claims made by an Issuer about a Subject, cryptographically signed by the Issuer. Conforms to.4  
* **Verifiable Presentation (VP):** A tamper-evident presentation of data from one or more VCs, prepared by a Holder for sharing with a Verifier. Conforms to.4

### **1.6. Document Conventions**

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119\.

Code examples, message formats, and data structures are presented in monospace font. Placeholders within these examples are indicated by \<angle\_brackets\>.

Normative references point to specifications that MUST be adhered to for compliance with STCP. Non-normative references provide context or background information.

## **2\. Core Concepts and Architecture**

### **2.1. Be-Secure Ecosystem Overview**

The Be-Secure ecosystem represents a paradigm shift towards collaborative, transparent, and automated security assessments for open-source software. It moves away from siloed tool execution and proprietary reporting formats towards a standardized framework built on interoperability and trust. The core components – BeSEnvironment, BeSPlaybook, BeSLab Plugin, OSAR, and BeSLab Blueprint – work in concert, orchestrated by the Security Tool Context Protocol (STCP), to achieve this vision. The integration of Decentralized Identifiers (DIDs) and Verifiable Credentials (VCs) provides a foundational layer of trust, ensuring the authenticity and integrity of participants, processes, and results throughout the assessment lifecycle.4

### **2.2. BeSEnvironment (Be-Secure Environment)**

A BeSEnvironment serves as a dedicated, isolated, and ephemeral execution space for security assessment tasks. Its primary purpose is to host one or more BeSLab Plugins (security tools) and execute them against a specified target (e.g., source code repository, container image, deployed application endpoint).

**Characterization:** Each BeSEnvironment is instantiated based on a BeSLab Blueprint, ensuring reproducibility and consistency. It provides a controlled context, isolating the assessment process from the host system and other assessments. This isolation is crucial for security, preventing interference between tools or contamination of results, and for managing dependencies specific to the tools being run.

**Key Attributes:**

* **Isolation:** Provides network and filesystem isolation for running plugins.  
* **Reproducibility:** Instantiated from a defined BeSLab Blueprint.  
* **Ephemeral:** Designed to exist only for the duration of a specific BeSPlaybook execution.  
* **Configurability:** Configured with necessary base operating systems, libraries, environment variables, and potentially secrets required by the plugins, as defined in the blueprint and playbook.  
* **STCP Compliance:** Exposes an STCP-compliant interface to receive commands from the BeSPlaybook (Orchestrator) and relay them to the appropriate BeSLab Plugins (Agents) running within it. It fulfills the **Environment Manager** role in the STCP communication model.

Functionally, a BeSEnvironment acts as a secure sandbox, ensuring that security tools run in a predictable and contained manner.

### **2.3. BeSPlaybook (Be-Secure Playbook)**

A BeSPlaybook defines and orchestrates the entire security assessment process for a specific target. It acts as the central control script, dictating the sequence of actions, the tools to be used, and the flow of data.

**Characterization:** BeSPlaybooks can be defined using either declarative (specifying the desired state or goals) or imperative (specifying step-by-step instructions) formats. They are the primary interface through which a user or automated system initiates and manages an assessment. The playbook runner fulfills the **Orchestrator** role in the STCP communication model.

**Key Attributes:**

* **Workflow Definition:** Specifies the sequence of assessment steps (e.g., checkout code, run SAST, run SCA, build, run DAST).  
* **Target Specification:** Defines the target artifact(s) to be assessed.  
* **Plugin Selection:** Identifies the specific BeSLab Plugins required for the assessment.  
* **Configuration Management:** Provides the necessary configuration parameters for the BeSEnvironment and each BeSLab Plugin.  
* **Context Management:** Manages the flow of information and artifacts between different steps in the playbook.  
* **STCP Initiation:** Sends STCP commands to the BeSEnvironment to manage its lifecycle and trigger plugin actions.  
* **OSAR Aggregation:** Responsible for initiating the collection of OSAR fragments from plugins and compiling the final OSAR.

The BeSPlaybook brings structure and automation to the assessment, transforming a potentially complex set of tool executions into a manageable and repeatable workflow. Its role is analogous to a workflow engine or an orchestration script tailored for security assessments.

#### **2.3.1. Playbook Naming Conventions**

Specific implementations within the Be-Secure ecosystem may adopt naming conventions for playbook files to ensure consistency and clarity. One such convention defines names based on purpose, tool/artifact, and version:

* **Lifecycle File:** besman-\<purpose | intent\>-\< Tool Name | Artifact Name \>-playboook-\<version\>.sh  
  * This file typically contains the main orchestration logic, calling the lifecycle functions.  
* **Steps File:** besman-\<purpose | intent\>-\< Tool Name | Artifact Name \>-playbook-\<version\>.\<extension\>  
  * This file contains the detailed instructions or steps executed within the playbook.  
  * The file extension (.sh, .md, .ipynb) indicates the execution type: automated (.sh), manual (.md), or interactive (.ipynb).

These conventions help organize playbooks and understand their intent and execution mode at a glance.

### **2.4. BeSLab Plugin (Agent)**

A BeSLab Plugin is the fundamental unit of execution within the Be-Secure ecosystem, representing a specific security tool or capability. It acts as an intermediary between the standardized STCP protocol and the proprietary interface or execution logic of the underlying tool.

**Characterization:** Each plugin wraps a security tool (e.g., Semgrep, Trivy, OWASP ZAP, Gitleaks) and exposes its functionality through the STCP interface. It runs within the isolated BeSEnvironment and responds to commands relayed by the Environment Manager. Crucially, each plugin instance acts as an **Agent** in the STCP communication model, possessing its own Decentralized Identifier (DID) and associated cryptographic keys, enabling secure authentication and attestation.3 This agent-based approach aligns with concepts from multi-agent systems where components communicate via standardized languages.18

**Key Attributes:**

* **Tool Encapsulation:** Hides the specific implementation details of the underlying security tool.  
* **STCP Compliance:** Implements the necessary STCP command handlers (e.g., for configuration, execution, reporting).  
* **Capability Declaration:** Describes its capabilities, parameters, and dependencies via a Plugin Manifest. This is analogous to how MCP servers describe their tools.13  
* **Stateful Operation:** Manages its internal state (e.g., installed, configured, running, completed) as dictated by STCP commands.  
* **Standardized Output:** Produces results in the form of OSAR fragments (SARIF-based).  
* **Identity and Attestation:** Possesses a DID and uses VCs to attest to its identity, capabilities, and the integrity of its execution and results.4

BeSLab Plugins are the adaptable building blocks that allow the Be-Secure ecosystem to incorporate a wide variety of security tools while maintaining a consistent orchestration and reporting framework.

### **2.5. OSAR (Open Source Assessment Report)**

The Open Source Assessment Report (OSAR) is the standardized, comprehensive output generated at the conclusion of a BeSPlaybook execution. It serves as the definitive record of the assessment activities and findings.

**Characterization:** OSARs are designed to be machine-readable and cryptographically verifiable, providing a trustworthy account of the security posture of the assessed target at a specific point in time. The format is based on the Static Analysis Results Interchange Format (SARIF) 23, ensuring broad compatibility with existing security tooling and platforms.25

**Key Attributes:**

* **Standardized Format:** Adheres to SARIF 2.1.0 (or later), potentially with defined Be-Secure extensions for additional metadata.  
* **Aggregated Findings:** Consolidates results (OSAR fragments) from all BeSLab Plugins executed within the playbook.  
* **Comprehensive Metadata:** Includes details about the assessment context: target identifier, playbook used, environment details (via reference to its VC), execution timestamps, and list of plugins involved.  
* **Verifiability:** The authenticity and integrity of the final OSAR are attested to by a Verifiable Credential (FinalOSARAuthenticityVC) issued by the orchestrating BeSLab/Playbook runner.4 Individual fragments within the report are also linked to their respective execution attestations via VCs.

The OSAR provides a reliable and actionable summary of security findings, suitable for integration into development workflows, compliance reporting, and vulnerability management systems.

### **2.6. BeSLab Blueprint**

The BeSLab Blueprint serves as the template or recipe for creating BeSEnvironments. It ensures that environments are provisioned consistently and contain the necessary prerequisites for the intended assessment tasks.

**Characterization:** A blueprint is a configuration file (e.g., YAML, JSON) that specifies the desired state of a BeSEnvironment before any playbook-specific configuration occurs. It defines the foundational layer upon which the assessment will run.

**Key Attributes:**

* **Base Environment:** Specifies the base operating system image or virtual machine template.  
* **Dependencies:** Lists required system packages, libraries, or runtimes needed by the anticipated BeSLab Plugins.  
* **Resource Allocation:** May define default resource limits (CPU, RAM, storage).  
* **Network Configuration:** Specifies network settings, such as required connectivity or isolation levels.  
* **Plugin Pre-provisioning:** Can optionally specify BeSLab Plugins to be pre-installed or cached within the environment image for faster setup.  
* **Security Context:** May define baseline security configurations or hardening steps for the environment.

By using blueprints, the Be-Secure ecosystem ensures that the execution environments themselves are reproducible and meet the necessary technical requirements defined by the BeSLab administrators or community.

### **2.7. Architectural Model and Interactions**

The Be-Secure architecture facilitates a structured interaction flow orchestrated via STCP, with DIDs/VCs providing the trust fabric. The key interactions are:

1. **Initiation:** A user or system triggers a BeSPlaybook.  
2. **Environment Setup:** The Playbook Orchestrator parses the playbook, identifies the required BeSLab Blueprint, and sends an initialize\_environment STCP command to the BeSLab infrastructure. The infrastructure provisions the BeSEnvironment based on the blueprint, assigning it an ephemeral DID and potentially issuing a BeSEnvironmentIntegrityVC. The Environment Manager component within the BeSEnvironment starts listening for STCP commands.  
3. **Plugin Configuration:** The Orchestrator sends install\_plugin (if necessary) and configure\_plugin commands via STCP to the Environment Manager. Before configuration, the Orchestrator SHOULD verify the PluginIdentityCapabilityVC associated with the plugin's DID. The Environment Manager routes these commands to the specific Plugin Agent(s) within the environment.  
4. **Execution:** The Orchestrator sends execute\_tool commands to the Environment Manager, which routes them to the target Plugin Agent. The Plugin Agent executes the underlying security tool against the specified target. Execution status MAY be reported asynchronously via STCP notifications.  
5. **Results Aggregation:** Upon completion, the Orchestrator sends get\_osar\_fragment commands. Plugin Agents return their results formatted as SARIF fragments, accompanied by OSARFragmentAuthenticityVCs, potentially linked to ExecutionAttestationVCs issued during execution.  
6. **OSAR Generation:** The Orchestrator verifies the fragment VCs, aggregates the SARIF fragments into a final OSAR, and requests the issuance of a FinalOSARAuthenticityVC for the complete report.  
7. **Termination:** The Orchestrator sends terminate\_plugin commands followed by a destroy\_environment command to the Environment Manager, which cleans up all resources associated with the ephemeral environment.  
8. **OSAR Sharing (Optional):** A user or another BeSLab can request the OSAR. They present a UserAuthorizationVC (within a VP) to the hosting BeSLab. Upon successful verification, the OSAR (with its VC) is securely transferred, potentially using DIDComm patterns.9

This architecture leverages a layered approach, similar in concept to the layered models seen in network protocols 47 or even the OpenC2 architecture 11, separating orchestration (Playbook), environment management (BeSEnvironment), and tool execution (Plugin). The use of STCP as the unifying protocol, combined with the DID/VC trust layer, enables secure and automated operation across these distinct components. The Environment Manager acts as a crucial intermediary, enforcing isolation and managing the state of plugins within the secure execution context, a distinction from simpler direct client-server models like MCP.1

*(Diagram illustrating these interactions would typically be included here in a formal specification document)*

## **3\. BeSEnvironment and BeSPlaybook Lifecycle Functions**

### **3.1. Overview of Lifecycle Functions**

The execution of a BeSPlaybook and the corresponding lifetime of its associated BeSEnvironment follow a structured lifecycle consisting of five distinct phases. This lifecycle provides a conceptual framework for understanding the progression of a security assessment within the Be-Secure ecosystem. The Security Tool Context Protocol (STCP) is intrinsically linked to this lifecycle, with specific protocol interactions occurring during each phase to manage the environment, configure plugins, execute tools, and handle results. This lifecycle management mirrors patterns seen in other orchestrated systems, ensuring a predictable flow from setup to teardown.48

### **3.2. Initialization Phase**

**Purpose:** To establish the necessary preconditions for the security assessment, including setting up the execution context and identifying all required components and targets.

**Activities:** This phase begins with the invocation of a BeSPlaybook. The Orchestrator (playbook runner) parses the playbook definition to understand the assessment scope, target(s), required BeSLab Plugins, and the appropriate BeSLab Blueprint. Based on the blueprint, the Orchestrator initiates the creation of the BeSEnvironment by sending an initialize\_environment STCP command to the BeSLab infrastructure. This triggers the provisioning of the isolated environment. Upon successful provisioning, the Environment Manager within the BeSEnvironment becomes active and establishes a secure STCP communication channel back to the Orchestrator, potentially involving DID/VC-based authentication. Initial context, such as target information, may be passed during this phase. This mirrors the initialization and handshake steps seen in protocols like MCP.2

### **3.3. Configuration Phase**

**Purpose:** To prepare the provisioned BeSEnvironment and the selected BeSLab Plugins for the execution phase, ensuring all tools are correctly installed and configured according to the playbook's requirements.

**Activities:** Once the BeSEnvironment is initialized, the Orchestrator proceeds with configuration steps defined in the BeSPlaybook. This typically involves:

* **Plugin Installation:** If required plugins are not pre-installed via the blueprint, the Orchestrator sends install\_plugin STCP commands to the Environment Manager for each necessary plugin.  
* **Capability Verification (Optional):** The Orchestrator MAY send list\_capabilities commands to verify plugin capabilities against the playbook requirements and SHOULD verify the plugin's identity and declared capabilities using its PluginIdentityCapabilityVC.  
* **Plugin Configuration:** The Orchestrator sends configure\_plugin STCP commands, including tool-specific parameters (e.g., target paths within the environment, analysis rulesets, credentials/tokens required by the tool), to the Environment Manager, which routes them to the respective Plugin Agents. Each plugin processes its configuration and signals readiness. This phase ensures that all tools are present, correctly set up, and ready to operate on the target within the secure environment.

### **3.4. Execution Phase**

**Purpose:** To perform the core security analysis tasks by running the configured BeSLab Plugins against the specified target(s) as dictated by the BeSPlaybook's workflow logic.

**Activities:** The Orchestrator initiates the execution of security tools by sending execute\_tool STCP commands to the Environment Manager, targeting specific Plugin Agents. The command includes any runtime parameters needed for the specific execution step. Since tool execution can be time-consuming, this process is typically asynchronous:

* The execute\_tool command returns an execution\_id for tracking.  
* The Orchestrator monitors progress by polling using report\_status commands or by receiving asynchronous status\_update notifications from the Environment Manager/Plugin Agent.  
* The playbook logic manages dependencies, ensuring steps execute in the correct order (e.g., waiting for a build step to complete before starting a DAST scan).  
* Execution errors are handled, and the Orchestrator MAY issue cancel\_operation commands if needed (e.g., due to timeouts or user intervention). During this phase, ExecutionAttestationVCs MAY be generated by the plugins or environment to provide verifiable proof of specific actions taken.

### **3.5. Aggregation/Reporting Phase**

**Purpose:** To systematically collect the results generated by each executed BeSLab Plugin and compile them into the final, standardized Open Source Assessment Report (OSAR).

**Activities:** After the relevant execution steps are completed, the Orchestrator transitions to aggregation:

* It sends get\_osar\_fragment STCP commands to the Environment Manager, targeting the Plugin Agents involved in the execution phase.  
* Each Plugin Agent returns its findings formatted as a SARIF-compliant OSAR fragment, accompanied by its OSARFragmentAuthenticityVC.  
* The Orchestrator collects all fragments, verifies their associated VCs (checking signatures, validity, and linkage to execution attestations), and performs the aggregation logic. This involves merging the fragments into a single, coherent SARIF document representing the final OSAR.  
* Finally, the Orchestrator (or the BeSLab infrastructure on its behalf) issues the FinalOSARAuthenticityVC, attesting to the integrity and completeness of the aggregated report.

### **3.6. Termination Phase**

**Purpose:** To gracefully conclude the assessment process and release all allocated resources, ensuring a clean shutdown of the ephemeral environment.

**Activities:** Once the OSAR is generated and potentially stored or transmitted, the Orchestrator initiates the termination sequence:

* It MAY send terminate\_plugin STCP commands to Plugin Agents to allow for graceful shutdown and cleanup within the environment.  
* It sends a destroy\_environment STCP command to the Environment Manager.  
* The Environment Manager performs final cleanup actions (e.g., saving final logs, removing temporary files) and then de-provisions all resources associated with the BeSEnvironment (e.g., terminates the container/VM instance).  
* The STCP communication channel is closed. This phase ensures that no unnecessary resources remain allocated and that the ephemeral nature of the BeSEnvironment is maintained.

### **3.7. Playbook Implementation Lifecycle Functions**

While the phases above describe the conceptual lifecycle, specific implementations of BeSPlaybooks within the ecosystem may structure their execution logic using a defined set of functions within a "Lifecycle File" (see Section 2.3.1 for naming conventions). These functions map closely to the lifecycle phases:

* **\_\_besman\_init():** Corresponds to the **Initialization Phase**. This function initializes everything necessary for executing the playbook, including environment setup prerequisites and preparations for publishing reports.  
* **\_\_besman\_execute():** Corresponds to the **Execution Phase**. This function executes the steps defined in the associated "Steps File," which contains the actual instructions for the assessment activity (e.g., running tools via STCP commands).  
* **\_\_besman\_prepare():** Corresponds partially to the **Aggregation/Reporting Phase**. This function filters and prepares the data gathered during execution (e.g., OSAR fragments) for final reporting or publishing.  
* **\_\_besman\_publish():** Corresponds partially to the **Aggregation/Reporting Phase**. This function handles the publishing of the final aggregated reports (e.g., the OSAR) to a designated datastore or location.  
* **\_\_besman\_cleanup():** Corresponds to the **Termination Phase**. This function handles cleanup tasks, such as removing temporary files or triggering environment destruction via STCP.  
* **\_\_besman\_launch():** This function acts as the main entry point for the playbook when invoked by an orchestration utility (like BeSman). It triggers the other lifecycle functions in the correct sequence: \_\_besman\_init, \_\_besman\_execute, \_\_besman\_prepare, \_\_besman\_publish, and \_\_besman\_cleanup.

**Skeletal Code Structure:**

A typical Lifecycle File implementing these functions might follow this structure:

Bash

function \_\_besman\_init {  
    \# This function initializes everything necessary for executing the playbook  
    \# as well as for publishing the reports.  
    \# (e.g., Send 'initialize\_environment' STCP command)  
}

function \_\_besman\_execute {  
    \# This function executes the steps file which contains the instructions  
    \# for the activity. The steps file can be in various formats such as  
    \# 'sh', '.ipynb', or '.md'.  
    \# (e.g., Send 'install\_plugin', 'configure\_plugin', 'execute\_tool' STCP commands)  
}

function \_\_besman\_prepare {  
    \# Filters the data from the report to prepare for publishing.  
    \# (e.g., Send 'get\_osar\_fragment' STCP commands, aggregate results)  
}

function \_\_besman\_publish {  
    \# Publishes the reports to the datastore.  
    \# (e.g., Store final OSAR, issue FinalOSARAuthenticityVC)  
}

function \_\_besman\_cleanup {  
    \# Handles the cleanup tasks.  
    \# (e.g., Send 'terminate\_plugin', 'destroy\_environment' STCP commands)  
}

function \_\_besman\_launch {  
    \# Playbook launch function that gets called by BeSman utility.  
    \# This function triggers the lifecycle methods of a playbook.  
    \_\_besman\_init  
    \_\_besman\_execute  
    \_\_besman\_prepare  
    \_\_besman\_publish  
    \_\_besman\_cleanup  
}

\# Potentially call \_\_besman\_launch here or expect it to be called externally

This structured approach within the playbook implementation complements the abstract lifecycle phases and the STCP interactions occurring within each phase.

### **3.8. BeSEnvironment Script Lifecycle Functions**

Beyond the STCP-driven lifecycle phases managed by the Orchestrator, specific implementations of BeSEnvironments may utilize internal "Environment Scripts" to manage the setup, configuration, and state of tools *within* the environment itself. These scripts provide a way to automate environment setup, ensuring consistency and reproducibility.

**Types of Environments:** BeSEnvironments managed by such scripts can be categorized based on their purpose:

* **Red Teaming Environments (RT):** Pre-installed with tools and utilities required for vulnerability assessment, exploit creation, etc.  
* **Blue Teaming Environments (BT):** Pre-installed with tools required for defensive activities like vulnerability remediation and patching.

**Lifecycle Functions:** An Environment Script SHOULD implement the following lifecycle functions, which can be invoked internally by the Environment Manager (e.g., during initial provisioning or in response to STCP commands like install\_plugin or configure\_plugin):

* **\_\_besman\_install:** Installs the required tools within the environment. This might be called during the initial environment provisioning based on the blueprint or in response to an install\_plugin STCP command.  
* **\_\_besman\_uninstall:** Removes installed tools. This could be part of the cleanup process triggered by terminate\_plugin or destroy\_environment STCP commands.  
* **\_\_besman\_validate:** Checks whether all required tools are installed and necessary configurations are met. This could be used internally to verify environment readiness or potentially exposed via a custom STCP command if needed.  
* **\_\_besman\_update:** Updates configurations of the installed tools. This might be invoked in response to a configure\_plugin STCP command.  
* **\_\_besman\_reset:** Resets the environment (or specific tool configurations) to a default state.

**Implementation:** These functions are typically implemented within a shell script (e.g., Bash) that manages the installation, configuration, and removal of tools directly within the environment's operating system.

**Skeletal Code Structure:**

Bash

\#\!/bin/bash

function \_\_besman\_install {  
    \# Code to install required tools and dependencies  
}

function \_\_besman\_uninstall {  
    \# Code to remove installed tools  
}

function \_\_besman\_update {  
    \# Code to update tool configurations  
}

function \_\_besman\_validate {  
    \# Code to check installation and configuration status  
}

function \_\_besman\_reset {  
    \# Code to reset the environment/tools to a default state  
}

\# Script might be sourced or specific functions called by the Environment Manager

These environment script functions provide a lower-level mechanism for managing the internal state of the BeSEnvironment, complementing the higher-level orchestration provided by STCP and BeSPlaybooks.

## **4\. STCP Specification**

### **4.1. Protocol Goals and Scope (Reiteration)**

The Security Tool Context Protocol (STCP) is designed to facilitate interoperable, automated, and trustworthy execution of security assessment tools within the Be-Secure ecosystem. Its scope is strictly defined to the communication between the Orchestrator (BeSPlaybook runner), the Environment Manager (BeSEnvironment), and the Plugin Agents (BeSLab Plugins). It aims to standardize tool lifecycle management (install, configure, execute, terminate), results retrieval, and status reporting, underpinned by a robust security model using DIDs and VCs.

### **4.2. Communication Model (STCP Actors)**

STCP defines three distinct actor roles involved in the communication flow:

1. **Orchestrator (Client Role):** Typically embodied by the BeSPlaybook execution engine. This actor initiates STCP commands to control the assessment workflow. It manages the overall state of the playbook, requests environment creation/destruction, directs plugin lifecycle actions, triggers tool execution, and aggregates results. It acts as the primary client in the STCP interactions, analogous to the combined role of Host and Client in MCP 14 or the Producer in OpenC2.11  
2. **Environment Manager (Intermediary/Server Role):** Embodied by the BeSEnvironment instance. This actor acts as a server endpoint for the Orchestrator and an intermediary gateway to the Plugin Agents running within its managed environment. It receives STCP commands from the Orchestrator, manages the state of the environment itself (e.g., provisioned resources, installed plugins), routes relevant commands to the appropriate Plugin Agent(s), and relays responses and notifications back to the Orchestrator. It is responsible for enforcing isolation and managing resources within the environment.  
3. **Plugin Agent (Server/Target Role):** Embodied by an instance of a BeSLab Plugin running within the BeSEnvironment. This actor acts as the ultimate target for tool-specific commands relayed by the Environment Manager. It receives commands like configure\_plugin, execute\_tool, get\_osar\_fragment, performs the corresponding actions by interacting with the underlying security tool, manages its own internal state (e.g., configured, running), and provides status updates and results back through the Environment Manager. It is analogous to an MCP Server exposing tool capabilities 1 or an OpenC2 Consumer/Actuator executing commands.11

This three-party model (Orchestrator \-\> Environment Manager \-\> Plugin Agent) is a deliberate design choice for STCP. Unlike simpler two-party models (e.g., direct Host-to-ToolServer in MCP 1), the Environment Manager provides a necessary layer of abstraction and control. This layer is essential for managing the isolated, ephemeral nature of the BeSEnvironment, enforcing security boundaries, handling resource allocation, and potentially multiplexing communication to multiple plugins residing within the same environment. While adding a hop, this architecture enhances security, reproducibility, and manageability, which are paramount in the context of security assessments.

### **4.3. Transport Mechanisms**

STCP defines specific transport mechanisms for communication between its actors:

* **Orchestrator \<-\> Environment Manager:** Communication between the Orchestrator and the Environment Manager MUST occur over a secure channel. The RECOMMENDED transport is **HTTPS (HTTP over TLS)**, utilizing **Server-Sent Events (SSE)** for enabling asynchronous notifications (e.g., status updates) from the Environment Manager to the Orchestrator. This aligns with modern web practices and is supported by MCP for remote communication.1  
  * **Security:** TLS version 1.2 or higher MUST be used. Authentication between the Orchestrator and Environment Manager MUST be mutual and based on their respective DIDs and potentially VCs, as detailed in Section 7\. This ensures both parties verify each other's identity before exchanging commands. OpenC2 over HTTPS similarly mandates TLS for operational use.49  
* **Environment Manager \<-\> Plugin Agent:** Communication between the Environment Manager and the Plugin Agents occurs *within* the isolated BeSEnvironment. For efficiency and simplicity, the RECOMMENDED transport mechanism is **Standard Input/Output (STDIO)**, where the Environment Manager communicates with the plugin process via its stdin and stdout streams. This is suitable for local process communication and is also an option in MCP.1 Alternatively, local communication mechanisms like Unix domain sockets or named pipes MAY be used, provided they maintain the isolation boundaries of the environment.  
  * **Security:** While occurring within the isolated environment, the communication channel SHOULD still be considered mutually authenticated based on the DIDs established during plugin instantiation. Encryption might be optional for local STDIO/socket communication if the environment itself is considered a sufficient trust boundary, but signing messages for integrity is RECOMMENDED.

The choice of transport should ensure reliable message delivery for commands and responses. The use of SSE allows for efficient push-based status updates without constant polling.

### **4.4. Message Format (JSON-RPC 2.0 Adaptation)**

STCP messages MUST adhere to the JSON-RPC 2.0 specification.8 This provides a lightweight, standardized structure for requests, responses, and notifications using JSON.

* **4.4.1. Common Message Headers:** While JSON-RPC 2.0 itself does not define headers, STCP messages MAY include metadata fields alongside the standard jsonrpc, method/result/error, params, and id fields, or within a dedicated metadata object in the params or result. RECOMMENDED common fields include:  
  * stcp\_version: (String) The version of the STCP protocol being used (e.g., "0.1"). REQUIRED.  
  * message\_id: (String) A unique identifier for this specific message instance, generated by the sender. Useful for logging and debugging. REQUIRED.  
  * timestamp: (String) ISO 8601 timestamp indicating when the message was generated. REQUIRED.  
  * correlation\_id: (String) An identifier linking related messages within a larger workflow or transaction. OPTIONAL.  
  * security\_context: (Object) Contains security-related information, such as DID references or embedded VCs/VPs used for message authentication or authorization, as detailed in Section 7\. OPTIONAL (details depend on security mechanism).  
* **4.4.2. Request Message Structure:** A Request message invokes a specific STCP command.  
  JSON  
  {  
    "jsonrpc": "2.0",  
    "method": "\<stcp\_command\_name\>",  
    "params": { /\* Command-specific parameters \*/ },  
    "id": "\<unique\_request\_id\>",  
    "stcp\_version": "0.1",  
    "message\_id": "\<unique\_message\_id\>",  
    "timestamp": "\<iso8601\_timestamp\>"  
    // Optional correlation\_id, security\_context  
  }

  * method: MUST contain the name of the STCP command being invoked (e.g., configure\_plugin).  
  * params: MUST be an Object containing the parameters required by the command. If a command requires no parameters, params MAY be omitted or be an empty object {}.  
  * id: MUST contain a unique value (String or Number) generated by the Orchestrator to correlate the request with the response.  
* **4.4.3. Response Message Structure:** A Response message is sent in reply to a Request message.  
  JSON  
  // Successful Response  
  {  
    "jsonrpc": "2.0",  
    "result": { /\* Command-specific results \*/ },  
    "id": "\<matching\_request\_id\>",  
    "stcp\_version": "0.1",  
    "message\_id": "\<unique\_message\_id\>",  
    "timestamp": "\<iso8601\_timestamp\>"  
    // Optional correlation\_id, security\_context  
  }

  // Error Response  
  {  
    "jsonrpc": "2.0",  
    "error": {  
      "code": \<error\_code\>,  
      "message": "\<error\_message\>",  
      "data": { /\* Optional additional error info \*/ }  
    },  
    "id": "\<matching\_request\_id\>", // or null if error occurred before id could be determined  
    "stcp\_version": "0.1",  
    "message\_id": "\<unique\_message\_id\>",  
    "timestamp": "\<iso8601\_timestamp\>"  
    // Optional correlation\_id  
  }

  * result: MUST be present on success and contain the output of the command. Its structure is command-specific.  
  * error: MUST be present on failure and contain an error object as defined below. result MUST NOT be present if error is present.  
  * id: MUST match the id of the corresponding Request message.  
* **4.4.4. Notification Message Structure:** A Notification message is a one-way message sent from the Environment Manager or Plugin Agent to the Orchestrator (e.g., for status updates).  
  JSON  
  {  
    "jsonrpc": "2.0",  
    "method": "\<notification\_type\>", // e.g., "status\_update"  
    "params": { /\* Notification data \*/ },  
    "stcp\_version": "0.1",  
    "message\_id": "\<unique\_message\_id\>",  
    "timestamp": "\<iso8601\_timestamp\>"  
    // Optional correlation\_id, security\_context  
  }

  * method: Indicates the type of notification.  
  * params: Contains the notification payload.  
  * id: MUST NOT be present for Notifications.  
* **4.4.5. Error Handling:** The error object in Response messages MUST contain:  
  * code: (Integer) A number indicating the error type. Negative values are typically reserved for JSON-RPC predefined errors. STCP defines specific positive integer codes (e.g., 1000-1999 range).  
    * 1001: PluginNotFound \- Specified plugin ID does not exist or is not installed.  
    * 1002: PluginNotReady \- Plugin is not in a state to accept the command (e.g., not configured).  
    * 1003: ConfigurationError \- Invalid configuration parameters provided.  
    * 1004: InstallationFailed \- Plugin installation failed.  
    * 1005: ExecutionFailed \- Tool execution failed or returned non-zero exit code.  
    * 1006: InvalidParameters \- Request parameters are invalid or missing.  
    * 1007: SecurityError \- Authentication, authorization, or VC verification failed.  
    * 1008: Timeout \- Operation timed out.  
    * 1009: OperationCancelled \- Operation was cancelled by request.  
    * 1010: UnsupportedCommand \- The receiving agent does not support the requested command.  
    * 1011: EnvironmentError \- An error occurred within the BeSEnvironment itself.  
    * 1012: ResourceUnavailable \- Required resource (e.g., file, network) not available.  
    * 1013: OutputError \- Error generating or retrieving results/OSAR fragment.  
  * message: (String) A concise, human-readable description of the error.  
  * data: (Optional) An Object or primitive containing additional, application-specific error details (e.g., stack trace snippet, detailed validation errors).

### **4.5. Standardized Commands**

STCP defines a set of standard commands (methods) for managing the lifecycle and execution of security assessments. Each command definition includes its purpose, typical caller/receiver, parameters (params object), and expected result (result object). (See Appendix A for a summary table).

* **4.5.1. Environment Management Commands:**  
  * initialize\_environment  
    * **Purpose:** Requests the provisioning and initialization of a new BeSEnvironment based on a blueprint.  
    * **Caller:** Orchestrator  
    * **Receiver:** BeSLab Infrastructure (leading to Environment Manager activation)  
    * **Params:**  
      * blueprint\_id: (String) Identifier of the BeSLab Blueprint to use.  
      * target\_info: (Object) Information about the primary assessment target (e.g., repository URL, image tag).  
      * session\_context: (Object, Optional) Initial context for the session (e.g., correlation IDs, user info DID).  
    * **Result:**  
      * environment\_id: (String) Unique identifier for the newly created BeSEnvironment instance.  
      * environment\_did: (String) The DID assigned to this ephemeral environment instance.  
      * endpoint\_url: (String) The HTTPS endpoint URL for the Environment Manager.  
  * destroy\_environment  
    * **Purpose:** Requests the termination and cleanup of a specific BeSEnvironment.  
    * **Caller:** Orchestrator  
    * **Receiver:** Environment Manager  
    * **Params:**  
      * environment\_id: (String) Identifier of the environment to destroy.  
    * **Result:**  
      * status: (String) Confirmation of destruction initiation (e.g., "termination\_initiated").  
* **4.5.2. Plugin Lifecycle Commands:**  
  * install\_plugin  
    * **Purpose:** Instructs the Environment Manager to download and install a specific BeSLab Plugin.  
    * **Caller:** Orchestrator  
    * **Receiver:** Environment Manager  
    * **Params:**  
      * plugin\_id: (String) The unique identifier (e.g., DID or name/version) of the plugin to install.  
      * source\_url: (String, Optional) URL to fetch the plugin package from (if not pre-defined or discoverable via plugin\_id).  
      * checksum: (String, Optional) Hash checksum for verifying the downloaded package.  
    * **Result:**  
      * status: (String) Indicates success ("installed") or failure ("installation\_failed").  
      * plugin\_did: (String, Optional) The resolved DID of the installed plugin instance.  
  * configure\_plugin  
    * **Purpose:** Configures an installed Plugin Agent with parameters required for execution.  
    * **Caller:** Orchestrator  
    * **Receiver:** Environment Manager (routes to Plugin Agent)  
    * **Params:**  
      * plugin\_id: (String) Identifier of the plugin to configure.  
      * configuration\_params: (Object) Key-value pairs containing tool-specific configuration (e.g., target\_path, ruleset\_id, api\_key\_ref). Structure defined by plugin manifest.  
    * **Result:**  
      * status: (String) Indicates success ("configured") or failure ("configuration\_failed").  
  * terminate\_plugin  
    * **Purpose:** Requests a graceful shutdown of a specific Plugin Agent.  
    * **Caller:** Orchestrator  
    * **Receiver:** Environment Manager (routes to Plugin Agent)  
    * **Params:**  
      * plugin\_id: (String) Identifier of the plugin to terminate.  
    * **Result:**  
      * status: (String) Indicates success ("terminated") or failure ("termination\_failed").  
* **4.5.3. Execution Commands:**  
  * execute\_tool  
    * **Purpose:** Triggers the execution of the configured tool within the Plugin Agent.  
    * **Caller:** Orchestrator  
    * **Receiver:** Environment Manager (routes to Plugin Agent)  
    * **Params:**  
      * plugin\_id: (String) Identifier of the plugin to execute.  
      * execution\_params: (Object, Optional) Runtime parameters (e.g., specific sub-targets, command-line arguments not part of static config). Structure defined by plugin manifest.  
    * **Result:**  
      * execution\_id: (String) A unique identifier for this specific execution instance, used for tracking asynchronous progress and retrieving results.  
      * status: (String) Initial status, e.g., "execution\_started".  
* **4.5.4. Data Retrieval Commands:**  
  * get\_results  
    * **Purpose:** Retrieves the raw output data generated by a specific tool execution. Use with caution for potentially large outputs.  
    * **Caller:** Orchestrator  
    * **Receiver:** Environment Manager (routes to Plugin Agent)  
    * **Params:**  
      * plugin\_id: (String) Identifier of the plugin.  
      * execution\_id: (String) Identifier of the execution instance.  
      * format: (String, Optional) Hint for desired raw format (e.g., "text", "json", "xml").  
      * pagination\_token: (String, Optional) Token for retrieving subsequent chunks of large results.  
    * **Result:**  
      * results\_data: (String or Object) The raw result data (or first chunk).  
      * next\_pagination\_token: (String, Optional) Token to retrieve the next chunk.  
      * status: (String) "results\_retrieved" or "results\_pending".  
  * get\_osar\_fragment  
    * **Purpose:** Retrieves the results of a specific tool execution formatted as a standardized OSAR (SARIF) fragment.  
    * **Caller:** Orchestrator  
    * **Receiver:** Environment Manager (routes to Plugin Agent)  
    * **Params:**  
      * plugin\_id: (String) Identifier of the plugin.  
      * execution\_id: (String) Identifier of the execution instance.  
    * **Result:**  
      * osar\_fragment: (Object) A valid SARIF log object representing the results of this execution.  
      * fragment\_vc: (Object, Optional) The OSARFragmentAuthenticityVC attesting to this fragment.  
      * status: (String) "fragment\_retrieved" or "fragment\_pending".  
* **4.5.5. Status and Control Commands:**  
  * report\_status  
    * **Purpose:** Queries the current status of a plugin or a specific execution.  
    * **Caller:** Orchestrator  
    * **Receiver:** Environment Manager (routes to Plugin Agent)  
    * **Params:**  
      * plugin\_id: (String) Identifier of the plugin.  
      * execution\_id: (String, Optional) If provided, requests status of a specific execution; otherwise, requests overall plugin status.  
    * **Result:**  
      * status: (String) Current status (e.g., not\_installed, installing, installed, configuring, ready, running, completed, error, terminating, terminated).  
      * details: (Object, Optional) Additional status details (e.g., progress percentage, error message).  
  * cancel\_operation  
    * **Purpose:** Attempts to cancel an ongoing asynchronous operation (typically an execution).  
    * **Caller:** Orchestrator  
    * **Receiver:** Environment Manager (routes to Plugin Agent)  
    * **Params:**  
      * plugin\_id: (String) Identifier of the plugin.  
      * execution\_id: (String) Identifier of the execution to cancel.  
    * **Result:**  
      * status: (String) Indicates if cancellation was successfully initiated ("cancellation\_requested") or failed ("cancellation\_failed"). Actual cancellation depends on tool support.  
* **4.5.6. Capability Discovery Commands:**  
  * list\_capabilities  
    * **Purpose:** Retrieves the capabilities, configurable parameters, and dependencies of a plugin, as defined in its manifest.  
    * **Caller:** Orchestrator  
    * **Receiver:** Environment Manager (routes to Plugin Agent)  
    * **Params:**  
      * plugin\_id: (String) Identifier of the plugin.  
    * **Result:**  
      * capabilities\_info: (Object) A JSON object representing the plugin's manifest content (or relevant parts thereof). Includes supported commands, parameter definitions, etc. Inspired by MCP tools/list.13

### **4.6. Context Passing Mechanisms**

Passing context – information required by a plugin to perform its task correctly – is crucial. STCP primarily uses the params objects within specific commands:

* **Static Configuration:** Tool settings that remain constant for the duration of the playbook execution (e.g., API endpoints, rule configurations, target base paths) are passed via the configuration\_params object in the configure\_plugin command.  
* **Runtime Parameters:** Information specific to a single execution run (e.g., a specific branch to scan, dynamic target URLs, output file names) are passed via the execution\_params object in the execute\_tool command.  
* **Environment Variables:** The BeSLab Blueprint or the initialize\_environment command MAY specify environment variables to be set within the BeSEnvironment, which plugins can then access through standard OS mechanisms.  
* **Secrets Management:** Sensitive information like API keys, passwords, or tokens MUST NOT be passed directly in plaintext within STCP parameters. Instead, mechanisms SHOULD be used such as:  
  * Referencing secret identifiers managed securely by the BeSEnvironment/BeSLab infrastructure (e.g., mounting secrets as files or environment variables accessible only to the specific plugin container/process). The configuration\_params or execution\_params would contain the reference (e.g., api\_key\_ref: "besecure-secret://my-tool-api-key").  
  * Leveraging VC-based authorization where applicable, allowing the plugin to obtain access tokens directly using its own identity.  
* **Inter-Plugin Data:** If one plugin's output is needed as input for another, the Orchestrator is responsible for retrieving the output (using get\_results or get\_osar\_fragment) and passing the relevant data (or a reference to it within the environment's shared storage, if applicable) as a parameter in the subsequent plugin's configure\_plugin or execute\_tool command.

### **4.7. State Management**

A key characteristic distinguishing STCP from stateless protocols is its explicit handling of state, particularly for BeSLab Plugins. Security tools often require distinct phases like installation, configuration, and execution, making state management essential.21

* **Plugin State:** Each Plugin Agent instance within a BeSEnvironment maintains its own state (e.g., not\_installed, installing, installed, configuring, ready, running, completed, error, terminated). STCP commands (install\_plugin, configure\_plugin, execute\_tool, terminate\_plugin) act as triggers for state transitions. The report\_status command allows the Orchestrator to query the current state. Plugin Agents MUST accurately report their state.  
* **Environment State:** The Environment Manager tracks the overall state of the BeSEnvironment (e.g., initializing, ready, running\_playbook, terminating, terminated) and the states of all Plugin Agents it manages.  
* **Orchestrator State:** The Orchestrator tracks the high-level state of the BeSPlaybook execution (e.g., which step is active, overall success/failure) and the state of the BeSEnvironment it is interacting with.  
* **Asynchronicity:** Due to the potentially long-running nature of installation and execution, STCP relies heavily on asynchronous operations. Commands like execute\_tool return immediately with an execution\_id. The Orchestrator uses this ID with report\_status polling or relies on asynchronous status\_update notifications to track completion or failure.  
* **Error Handling and Recovery:** The protocol defines error codes (Section 4.4.5) to signal problems during state transitions or operations. Playbooks SHOULD include logic to handle these errors (e.g., retry, fail fast, run alternative steps). The state information helps determine appropriate recovery actions (e.g., an ExecutionFailed state might allow for get\_results to retrieve partial logs, while an InstallationFailed state would preclude further commands to that plugin).

Robust state tracking and clear reporting via STCP are critical for enabling reliable and resilient automation of multi-step security assessment playbooks.

## **5\. STCP Integration with Lifecycle Functions**

### **5.1. Mapping STCP Commands to Lifecycle Phases**

The STCP commands defined in Section 4.5 are designed to be invoked during specific phases of the BeSEnvironment and BeSPlaybook lifecycle (Section 3). This mapping ensures that protocol interactions align logically with the overall assessment workflow. The following table summarizes the typical mapping:

**Table 5.1: STCP Command to Lifecycle Phase Mapping**

| STCP Command | Typical Lifecycle Phase(s) | Notes |
| :---- | :---- | :---- |
| initialize\_environment | Initialization | Starts the environment setup process. |
| install\_plugin | Configuration | Installs required tools within the environment. |
| list\_capabilities | Configuration | Verifies plugin capabilities before configuration/execution. |
| configure\_plugin | Configuration | Sets up plugins with specific parameters. |
| execute\_tool | Execution | Triggers the actual security tool execution. |
| report\_status | Configuration, Execution, Aggregation | Monitors progress and state changes. |
| cancel\_operation | Execution | Attempts to stop an ongoing execution. |
| get\_results | Aggregation/Reporting | Retrieves raw output (use judiciously). |
| get\_osar\_fragment | Aggregation/Reporting | Retrieves standardized, verifiable results. |
| terminate\_plugin | Termination | Requests graceful shutdown of plugin agents. |
| destroy\_environment | Termination | Triggers cleanup and de-provisioning of the environment. |

This table provides a high-level guide; specific playbooks might involve variations, such as querying status during termination or configuring plugins iteratively during execution phases if the workflow demands it.

### **5.2. Interaction Flow: Initialization Phase**

During the Initialization phase, the primary goal is to set up the execution context.

1. **Playbook Parsing:** The Orchestrator parses the BeSPlaybook, identifying the target, required blueprint (blueprint\_id), and initial session context.  
2. **Environment Request:** The Orchestrator sends an initialize\_environment request to the BeSLab infrastructure, providing the blueprint\_id and target\_info.  
3. **Provisioning:** The BeSLab infrastructure provisions the BeSEnvironment according to the blueprint. This includes setting up the base OS, dependencies, network, and activating the Environment Manager component. An ephemeral DID (environment\_did) is generated for the environment.  
4. **Response & Channel Setup:** The Environment Manager, once active, responds to the Orchestrator (potentially indirectly via the infrastructure) with the environment\_id, its environment\_did, and its endpoint\_url.  
5. **Secure Channel:** The Orchestrator and Environment Manager establish a secure STCP communication channel over HTTPS/SSE using the provided endpoint, performing mutual authentication based on their DIDs (Orchestrator's DID vs. environment\_did).

At the end of this phase, a secure channel exists between the Orchestrator and a ready, but unconfigured, BeSEnvironment.

### **5.3. Interaction Flow: Configuration Phase**

This phase prepares the environment and plugins for execution.

1. **Plugin Iteration:** The Orchestrator iterates through the list of BeSLab Plugins specified in the playbook.  
2. **Installation (if needed):** If a plugin (plugin\_id) is not pre-installed, the Orchestrator sends an install\_plugin request to the Environment Manager. The Environment Manager handles the download/setup and responds with success/failure. The Orchestrator MAY poll report\_status(plugin\_id) until the state is installed or error.  
3. **Capability/Identity Check (Recommended):** The Orchestrator resolves the plugin\_id (if it's a DID or discoverable) to get its DID document and associated PluginIdentityCapabilityVC. It verifies the VC to confirm the plugin's authenticity and stated capabilities. It MAY also call list\_capabilities(plugin\_id) via STCP to get manifest details directly from the running instance.  
4. **Configuration:** The Orchestrator extracts the required configuration\_params from the playbook for the current plugin and sends a configure\_plugin request to the Environment Manager.  
5. **Routing & Execution:** The Environment Manager routes the request to the target Plugin Agent. The Plugin Agent applies the configuration.  
6. **Response:** The Plugin Agent responds with success/failure, which is relayed back to the Orchestrator by the Environment Manager. The Orchestrator MAY poll report\_status(plugin\_id) until the state is ready or error.

This process repeats for all required plugins. At the end of this phase, all necessary plugins are installed, configured, and in the ready state.

### **5.4. Interaction Flow: Execution Phase**

This phase involves running the security tools according to the playbook's logic.

1. **Execution Trigger:** Based on the playbook workflow, the Orchestrator identifies the next plugin (plugin\_id) to execute and sends an execute\_tool request, including any execution\_params, to the Environment Manager.  
2. **Routing & Start:** The Environment Manager routes the command to the Plugin Agent. The Plugin Agent starts the underlying security tool and transitions its state to running. It responds immediately to the Environment Manager with the execution\_id and status "execution\_started", which is relayed to the Orchestrator.  
3. **Asynchronous Monitoring:** The Orchestrator monitors the execution using the execution\_id:  
   * **Polling:** Periodically send report\_status(plugin\_id, execution\_id) requests and check the returned status (running, completed, error).  
   * **Notifications:** Listen for asynchronous status\_update notifications pushed from the Environment Manager via SSE, containing the plugin\_id, execution\_id, current status, and potentially progress details.  
4. **Dependency Management:** The Orchestrator manages the workflow, potentially waiting for one execution to reach the completed state before sending the execute\_tool command for a dependent step.  
5. **Error Handling/Cancellation:** If an error occurs (error state reported), the Orchestrator handles it according to playbook logic (e.g., fail, log, continue). If cancellation is required, the Orchestrator sends a cancel\_operation(plugin\_id, execution\_id) request.  
6. **Completion:** Once a Plugin Agent finishes execution, it transitions its state to completed or error and reports this via status updates/polling.

This phase continues until all execution steps in the playbook are completed or an unrecoverable error occurs.

### **5.5. Interaction Flow: Aggregation/Reporting Phase**

This phase focuses on collecting and finalizing the assessment results.

1. **Fragment Retrieval Trigger:** After successful completion of relevant execution steps, the Orchestrator identifies the executions (execution\_ids) from which results are needed.  
2. **Request Fragments:** For each required result, the Orchestrator sends a get\_osar\_fragment(plugin\_id, execution\_id) request to the Environment Manager.  
3. **Routing & Generation:** The Environment Manager routes the request to the appropriate Plugin Agent. The Plugin Agent formats its results as a SARIF OSAR fragment and generates/retrieves its associated OSARFragmentAuthenticityVC.  
4. **Response:** The Plugin Agent responds with the osar\_fragment (JSON object) and the fragment\_vc, relayed back by the Environment Manager.  
5. **Verification & Aggregation:** The Orchestrator receives the fragments and VCs. It MUST verify each fragment\_vc (checking signature, validity, issuer trust, linkage to the execution). Verified fragments are then merged into a single final OSAR (SARIF) document according to defined aggregation rules. This involves merging the JSON structures according to a defined strategy (e.g., combining run objects, merging results under appropriate tool definitions). The aggregator MUST ensure the final document is a valid SARIF file.  
6. **Final Attestation:** The Orchestrator requests the issuance of the FinalOSARAuthenticityVC for the complete OSAR file from the BeSLab infrastructure or its designated issuer service.

At the end of this phase, a complete, verified OSAR is available.

### **5.6. Interaction Flow: Termination Phase**

This final phase ensures clean shutdown and resource release.

1. **Plugin Termination (Optional):** The Orchestrator MAY send terminate\_plugin requests to running Plugin Agents via the Environment Manager to allow for graceful shutdown.  
2. **Environment Destruction Request:** The Orchestrator sends a destroy\_environment request to the Environment Manager, providing the environment\_id.  
3. **Cleanup:** The Environment Manager performs any final cleanup tasks within the environment (e.g., saving logs not captured elsewhere).  
4. **De-provisioning:** The Environment Manager signals the underlying infrastructure to de-provision all resources associated with the BeSEnvironment (terminate VM/container, release storage/network resources).  
5. **Channel Closure:** The STCP communication channel between the Orchestrator and the (now destroyed) Environment Manager is closed.

This concludes the BeSPlaybook execution and the lifecycle of the associated BeSEnvironment.

## **6\. BeSLab Plugin Onboarding and Execution**

### **6.1. Plugin Requirements**

For a security tool to function as a BeSLab Plugin within the Be-Secure ecosystem, it must meet several requirements ensuring interoperability, security, and manageability.

* **6.1.1. STCP Compliance:** The core requirement is adherence to this STCP specification. The plugin wrapper or the tool itself must implement the server-side logic for handling relevant STCP commands received via the Environment Manager (typically over STDIO or a local socket). This includes correctly parsing command parameters, performing the requested actions (install, configure, execute, report status, provide results), managing its internal state transitions, and formatting responses and notifications according to the JSON-RPC 2.0 structure defined herein.  
* **6.1.2. Plugin Manifest Definition:** Each BeSLab Plugin MUST be accompanied by a machine-readable manifest file, typically named plugin-manifest.json. This file serves as the plugin's self-description and is crucial for discovery, configuration, and environment provisioning. The manifest MUST be a JSON object containing at least the following fields:  
  * manifestVersion: (String) Version of the manifest schema itself.  
  * pluginName: (String) Human-readable name of the plugin.  
  * pluginVersion: (String) Version of the plugin.  
  * pluginDid: (String) The DID of the plugin publisher or the plugin code itself, used for identity verification.  
  * description: (String) Brief description of the plugin's function.  
  * stcpCommands: (Array of Strings) List of STCP commands supported by this plugin (e.g., \["configure\_plugin", "execute\_tool", "get\_osar\_fragment", "report\_status"\]).  
  * configurationSchema: (Object) A JSON Schema definition describing the structure and types of the configuration\_params expected by the configure\_plugin command.  
  * executionSchema: (Object) A JSON Schema definition describing the structure and types of the execution\_params expected by the execute\_tool command.  
  * outputSchema: (Object) Describes the expected output format, specifically referencing the SARIF version for get\_osar\_fragment.  
  * dependencies: (Object) Lists dependencies required for the plugin to run, categorized (e.g., os\_packages, libraries, runtime\_versions). Used by the BeSLab Blueprint.  
  * resourceHints: (Object, Optional) Provides hints for typical resource usage (e.g., min\_ram\_mb, min\_cpu\_cores). (See Appendix D for an example manifest). This manifest provides structured metadata similar to how MCP servers describe their available tools.13  
* **6.1.3. Security Considerations:** Plugins execute code, potentially from various sources, and operate on potentially sensitive targets within the BeSEnvironment. Therefore, plugins SHOULD be designed with security in mind:  
  * **Least Privilege:** Operate with the minimum necessary permissions within the environment.  
  * **Input Handling:** If processing external input beyond STCP parameters, perform appropriate validation and sanitization.  
  * **Secret Handling:** Adhere to secure secret management practices dictated by the Environment Manager (e.g., reading secrets from mounted volumes or environment variables rather than expecting them in plaintext STCP parameters).  
  * **Code Integrity:** The plugin package integrity SHOULD be verifiable (e.g., via checksums provided during installation or VCs attesting to the code).

### **6.2. Capability and Dependency Declaration (within Manifest)**

The Plugin Manifest serves as the primary mechanism for declaring capabilities and dependencies.

* **Capabilities:** The stcpCommands array explicitly lists which STCP commands the plugin implements. This allows the Orchestrator and Environment Manager to know which interactions are valid. For example, a plugin baked into a blueprint's base image might not need to support the install\_plugin command. The configurationSchema and executionSchema define the specific parameters the plugin accepts for configuration and execution, effectively declaring its functional interface via STCP.  
* **Dependencies:** The dependencies object provides structured information needed by the BeSLab Blueprint processor to ensure the BeSEnvironment is correctly provisioned. It might specify required Linux packages (e.g., {"os\_packages": \["git", "python3-pip"\]}), specific library versions, or required base runtimes (e.g., {"runtime\_versions": {"java": "11+"}}). This declarative approach enables automated environment setup tailored to the plugin's needs.

### **6.3. BeSLab Blueprint Specification for Plugins**

BeSLab Blueprints utilize the information from Plugin Manifests to define appropriate BeSEnvironments. A blueprint specification SHOULD include sections for:

* **Plugin References:** Identifying the plugins intended to run within environments created from this blueprint. Plugins can be referenced by their DID (pluginDid from the manifest) or potentially by a source URL or registry identifier.  
* **Dependency Resolution:** The blueprint processor reads the dependencies section of the manifests for all referenced plugins and includes the necessary steps in the environment build process (e.g., apt-get install, pip install).  
* **Resource Allocation:** Blueprint defaults for CPU, RAM, and storage can be informed or overridden based on the resourceHints provided in the manifests.  
* **Pre-installation:** Blueprints MAY specify that certain plugins should be pre-installed into the base environment image defined by the blueprint, using the source\_url or other location information potentially derived from the plugin reference. This can significantly speed up the Configuration phase during playbook execution.

### **6.4. Plugin Packaging and Distribution (Considerations)**

While the exact packaging format (e.g., container image, zip archive, executable binary) and distribution mechanism (e.g., OCI registry, dedicated BeSLab registry, simple HTTPS download) are outside the strict scope of the STCP protocol itself, the ecosystem design should consider them.

* **Packaging:** The chosen format should facilitate installation within the BeSEnvironment. Container images are a strong candidate due to their encapsulation of dependencies.  
* **Distribution:** Mechanisms should be secure. Referencing plugins by DID (pluginDid) allows verification of the publisher's identity. Distribution channels SHOULD allow fetching the Plugin Manifest easily. Verifiable Credentials could potentially be used to attest to the provenance and security scans of the plugin package itself, enhancing supply chain security.10

### **6.5. Plugin Instantiation as Agents**

When a BeSPlaybook requires a plugin, the Environment Manager is responsible for instantiating it within the BeSEnvironment. This typically involves:

1. **Locating/Installing:** Finding the plugin package (if not pre-installed) based on the install\_plugin command or blueprint definition.  
2. **Starting:** Launching the plugin as a process or container within the isolated environment.  
3. **Establishing Communication:** Setting up the internal STCP communication channel, typically by connecting the Environment Manager to the plugin process's standard input and standard output streams (STDIO).  
4. **Identity Association:** Ensuring the running instance is associated with the correct Plugin DID for authentication and attestation purposes.

Once instantiated and the communication channel is established, the plugin process acts as the STCP Plugin Agent, ready to receive and process commands relayed by the Environment Manager.

## **7\. Security and Trust Architecture using DIDs and VCs**

### **7.1. Overview of Trust Model**

STCP employs a decentralized trust model fundamentally based on W3C standards for Decentralized Identifiers (DIDs) 3 and Verifiable Credentials (VCs).4 This approach shifts trust away from centralized authorities or platform-specific mechanisms towards cryptographic proof and selective disclosure, managed by the entities themselves.36

The core principles are:

* **Decentralized Identity:** Every key actor (BeSLabs, Plugin Publishers/Instances, Users) and significant artifact (OSARs, potentially Environments) possesses a unique DID they control.3  
* **Verifiable Attestations:** Claims about actors or artifacts (e.g., plugin capabilities, execution completion, report integrity, user authorization) are made using VCs, which are digitally signed by a trusted issuer and can be independently verified by any party.10  
* **Cryptographic Verification:** Trust is established through cryptographic verification of signatures on VCs and messages, linked back to the DIDs of the involved parties via DID resolution.4  
* **User Control & Privacy:** Holders control their DIDs and VCs, deciding when and what information to share via Verifiable Presentations (VPs), enabling privacy-preserving interactions.3

This model aims to provide strong guarantees about the authenticity of components, the integrity of actions, and the reliability of results within the Be-Secure ecosystem.

### **7.2. Decentralized Identifiers (DIDs) Management**

DIDs form the bedrock of identity within the STCP ecosystem.

* **7.2.1. DID Methods (Recommended/Supported):** To ensure interoperability and address different entity types, STCP recommends specific DID methods compliant with:  
  * **did:web:** RECOMMENDED for stable entities with domain control, such as BeSLab infrastructure providers or established plugin publishers. did:web allows DID documents to be hosted at a well-known location on a web domain, making resolution straightforward using standard HTTPS.53  
  * **did:peer:** RECOMMENDED for ephemeral entities or peer-to-peer interactions where publishing to a public ledger or web server is unnecessary or undesirable. This is suitable for ephemeral BeSEnvironment instances or potentially for direct, temporary connections between agents.39 did:peer DIDs encode necessary information for interaction directly within the identifier itself or require peer-to-peer exchange of DID documents.  
  * **did:key:** MAY be used for simple use cases where the DID is derived directly from a public key, primarily for signing/verification purposes. Implementations MUST be capable of resolving DIDs using the methods employed within their operational context. Other conformant DID methods MAY be supported if agreed upon by interacting parties.  
* **7.2.2. DID Generation and Registration:**  
  * **BeSLabs/Organizations:** Should generate persistent DIDs (likely did:web) representing their infrastructure or administrative domain. The corresponding DID documents must be published according to the chosen method's specification (e.g., at /.well-known/did.json for did:web).  
  * **BeSLab Plugins:** Publishers should generate persistent DIDs (e.g., did:web) representing the canonical plugin identity. This DID is included in the Plugin Manifest (pluginDid). Ephemeral DIDs (e.g., did:peer) MAY be generated for specific running instances of a plugin within a BeSEnvironment, potentially linked to the publisher's DID via VCs.  
  * **BeSEnvironments:** Ephemeral DIDs (likely did:peer) MUST be generated for each BeSEnvironment instance upon initialization. This DID identifies the specific execution context.  
  * **Users/Community Members:** Individuals interacting with the system (e.g., requesting OSARs) should use their own DIDs, managed via compatible wallets/agents.53 Method choice depends on user preference and wallet support.  
* **7.2.3. DID Resolution Requirements:** All STCP components that need to verify signatures or establish secure communication (Orchestrators, Environment Managers, potentially Plugin Agents, OSAR Verifiers) MUST implement or have access to a DID Resolver conforming to the W3C DID Resolution specification.29 The resolver must support the DID methods used within the ecosystem (at least did:web and did:peer) and be able to retrieve DID documents to obtain public keys and service endpoints.

### **7.3. Verifiable Credentials (VCs) Usage**

VCs provide the mechanism for making trustworthy attestations within the ecosystem. Their use is critical for establishing confidence in plugin capabilities, execution integrity, and report authenticity.10

* **7.3.1. VC Data Model and Formats:** All VCs used within STCP MUST conform to the W3C Verifiable Credentials Data Model v1.1 or later.32 The RECOMMENDED format is **JSON-LD**, allowing for semantic context and extensibility.44 For specific use cases involving compact representation or integration with OAuth/OIDC flows, **JWT-VCs** MAY be used.54 Implementations MUST be prepared to parse and verify VCs in the supported formats.  
* **7.3.2. VC Types and Schemas:** The following specific VC types are defined for use within STCP. Each VC type SHOULD have a formally defined JSON Schema and JSON-LD context published at a stable URL.  
  * **PluginIdentityCapabilityVC:**  
    * **Issuer:** Trusted BeSLab administrator or Plugin Publisher authority.  
    * **Holder/Subject:** The Plugin's persistent DID (pluginDid).  
    * **Claims:** Plugin Name, Version, Publisher DID, Hash of Manifest (sha256), Attested STCP Capabilities (list of supported commands), Link to Manifest URL, Validity Period.  
    * **Purpose:** Authenticates the plugin's origin and cryptographically binds its identity to its declared capabilities as per the manifest. Verified by Orchestrator during Configuration phase.  
  * **BeSEnvironmentIntegrityVC:**  
    * **Issuer:** BeSLab Infrastructure/Provisioning Service.  
    * **Holder/Subject:** The ephemeral DID of the BeSEnvironment instance.  
    * **Claims:** Blueprint ID Used, Hash of Environment Configuration (sha256), Provisioning Timestamp, Issuing BeSLab Operator DID, Validity Period (typically session duration).  
    * **Purpose:** Attests to the integrity and configuration basis of the specific execution environment instance. May be included in subsequent attestations.  
  * **ExecutionAttestationVC:**  
    * **Issuer:** The Plugin Agent instance (self-issued or issued by Environment Manager acting as proxy).  
    * **Holder:** The Orchestrator.  
    * **Subject:** The specific execution instance (execution\_id).  
    * **Claims:** Executing Plugin Instance DID, Executed STCP Command (e.g., execute\_tool), Hash of Command Parameters (sha256), Execution Start/End Timestamps, Final Status (completed/error), Hash of Output/OSAR Fragment (sha256), Reference to BeSEnvironmentIntegrityVC, Validity Period (typically short).  
    * **Purpose:** Provides verifiable proof that a specific command was executed by a specific plugin within a specific environment, linking the execution to its output hash. Generated upon completion of relevant commands (e.g., execute\_tool).  
  * **OSARFragmentAuthenticityVC:**  
    * **Issuer:** The Plugin Agent instance (or Environment Manager proxying).  
    * **Holder:** The Orchestrator.  
    * **Subject:** The OSAR fragment itself (identified by hash).  
    * **Claims:** OSAR Fragment Hash (sha256), Generating Plugin Instance DID, Reference to corresponding ExecutionAttestationVC, Generation Timestamp, Validity Period.  
    * **Purpose:** Ensures the integrity of the OSAR fragment and links it directly to the verified execution context that produced it. Accompanies the fragment returned by get\_osar\_fragment.  
  * **FinalOSARAuthenticityVC:**  
    * **Issuer:** The Orchestrator or designated BeSLab authority.  
    * **Holder:** Intended recipients of the OSAR.  
    * **Subject:** The final aggregated OSAR file (identified by hash).  
    * **Claims:** Final OSAR Hash (sha256), BeSPlaybook ID/Version/Hash, List of included OSAR Fragment Hashes or VC references, Target Information, Assessment Timestamp, Issuing Orchestrator/BeSLab DID, Validity Period.  
    * **Purpose:** Ensures the integrity and authenticity of the complete, aggregated assessment report, providing a chain of trust back to individual plugin executions.  
  * **UserAuthorizationVC:**  
    * **Issuer:** BeSLab administrator or Community/Project Manager.  
    * **Holder/Subject:** User's DID.  
    * **Claims:** Authorized Actions (e.g., read:osar:summary, read:osar:detailed), Target Scope (e.g., project\_id, public), Validity Period.  
    * **Purpose:** Grants specific permissions to users for accessing OSARs or potentially other Be-Secure resources. Used within VPs for authorization checks during OSAR exchange.53  
* **7.3.3. VC Issuance Processes:** Issuance follows the standard Issuer-Holder model.32 For example:  
  * Plugin Publisher generates a key pair and DID, publishes DID doc, requests PluginIdentityCapabilityVC from BeSLab admin, providing manifest hash and proof of control over DID. Admin verifies, issues VC to Plugin DID.  
  * Environment Manager generates ephemeral DID upon initialize\_environment, issues BeSEnvironmentIntegrityVC to itself.  
  * Plugin Agent, upon completing execute\_tool, generates ExecutionAttestationVC (signing with its instance key) and OSARFragmentAuthenticityVC (signing similarly) before returning results.  
  * Orchestrator, after aggregation, generates FinalOSARAuthenticityVC (signing with its key).  
  * User requests UserAuthorizationVC from admin, proving identity/membership. Admin issues VC to User DID. VC API specifications \[VC-API\] MAY guide the implementation of issuance endpoints.56  
* **7.3.4. VC Verification Processes:** Verification is performed by entities needing to trust a claim.4 The process typically involves:  
  1. Receiving the VC (or VP containing the VC).  
  2. Resolving the Issuer's DID from the issuer field in the VC to get their DID document and public key(s).  
  3. Verifying the cryptographic proof/signature on the VC using the issuer's public key.  
  4. Checking the VC's validity period (issuanceDate, expirationDate).  
  5. Checking the VC's revocation status (see below).  
  6. (For VPs) Verifying the VP proof/signature using the Holder's public key (obtained via DID resolution).  
  7. Validating claims against expected schemas and application logic.  
* **7.3.5. VC Revocation Mechanisms:** Since VCs can become invalid before their expiration date (e.g., plugin vulnerability discovered, user access revoked), a revocation mechanism MUST be supported. The RECOMMENDED approach is to use status lists (e.g., CredentialStatusList2021 or newer standards). The VC MUST include a credentialStatus property pointing to a service where its current status can be checked.34 Verifiers MUST check the revocation status as part of the verification process. Issuers are responsible for maintaining and updating the status information.

The integration of these specific VCs provides granular, verifiable trust points throughout the STCP workflow, addressing the need for secure and trustworthy execution and reporting.

### **7.4. Secure Communication Channels**

Beyond authenticating components and attesting to actions/data, STCP requires secure transport-level communication. DIDs and associated cryptography enable the establishment of secure, mutually authenticated channels.

* **7.4.1. Establishing Secure Sessions:** When initiating communication (e.g., Orchestrator to Environment Manager, User to BeSLab for OSAR exchange), parties SHOULD establish a secure session leveraging their DIDs. This involves:  
  1. Discovering the counterparty's DID and resolving it to find their service endpoint(s) and public key(s).29  
  2. Performing a mutually authenticated key exchange protocol (e.g., Diffie-Hellman using keys from DID documents) to establish shared session keys. DIDComm protocols offer standardized ways to achieve this secure channel establishment between DID-enabled agents.9 While STCP primarily uses HTTPS, the authentication and key agreement SHOULD leverage the DID infrastructure rather than relying solely on traditional PKI or simple API keys.  
* **7.4.2. Message Encryption and Signing:** All STCP messages exchanged over potentially insecure networks (like the Orchestrator-Environment Manager link) MUST be protected.  
  * **Encryption:** Message payloads (especially params and result containing sensitive configuration or findings) SHOULD be encrypted using symmetric session keys established during session setup, or using asymmetric encryption with the recipient's public key if sessions are not used.  
  * **Signing:** All messages MUST be digitally signed by the sender using the private key associated with their DID. The recipient MUST verify this signature using the sender's public key obtained via DID resolution. This ensures message authenticity (origin) and integrity (tamper-evidence).9 DIDComm message formats inherently incorporate encryption and signing.9 For STCP over HTTPS, these protections might be implemented using JOSE (JSON Object Signing and Encryption) standards applied to the JSON-RPC payload, or rely on robust mutual TLS authentication tied to DIDs.  
* **7.4.3. Mutual Authentication:** Secure communication requires that both parties authenticate each other before exchanging significant data or commands. This is achieved through the DID-based verification of signatures and potentially challenge-response mechanisms during session establishment.41 This prevents impersonation and ensures commands are only accepted from authorized Orchestrators and responses/results only come from the intended Environment Manager or Plugin Agent.

## **8\. OSAR Generation, Exchange, and Sharing**

### **8.1. OSAR Definition and Structure**

The Open Source Assessment Report (OSAR) is the primary artifact produced by a BeSPlaybook execution, providing a standardized and verifiable record of the assessment findings.

* **8.1.1. Leveraging SARIF Standard:** The OSAR format MUST be based on the OASIS Static Analysis Results Interchange Format (SARIF) Version 2.1.0.23 SARIF provides a rich, standardized schema for representing static analysis results, including information about rules, results, locations in code, and execution context.5 Using SARIF ensures maximum compatibility with existing security tools, platforms, and developer workflows.26 Be-Secure MAY define specific profiles or extensions to SARIF (using its standard extension mechanisms) to accommodate metadata unique to the Be-Secure process, such as:  
  * Custom properties within run.tool.driver or run.properties to store the pluginDid of the generating plugin.  
  * Custom properties within result.properties to link findings back to specific playbook steps or associated VCs.  
  * Standardized taxonomies for rules or findings relevant to open-source assessment.  
* **8.1.2. OSAR Metadata:** Beyond the findings themselves, the OSAR (within the SARIF run object(s)) MUST include comprehensive metadata about the assessment context:  
  * run.tool: Information about the BeSPlaybook orchestrator and the BeSLab plugins executed (mapping to SARIF toolComponent objects). Includes pluginDid.  
  * run.invocations: Details about the execution, including start/end times.  
  * run.artifacts: Information about the target(s) assessed (e.g., repository URI, commit hash, container image digest).  
  * run.properties (or custom extension):  
    * besecure:playbookId: Identifier of the BeSPlaybook used.  
    * besecure:playbookVersion: Version of the playbook.  
    * besecure:environmentDid: DID of the BeSEnvironment instance.  
    * besecure:environmentVcRef: Reference to the BeSEnvironmentIntegrityVC.  
    * besecure:finalOsarVcRef: Reference to the FinalOSARAuthenticityVC associated with this report.  
* **8.1.3. OSAR Fragments vs. Final Report:** It is important to distinguish between:  
  * **OSAR Fragment:** A valid SARIF log file containing the results (run.results) and context (run.tool, run.artifacts, etc.) from a *single* BeSLab Plugin execution (execution\_id). Generated by the Plugin Agent via get\_osar\_fragment.  
  * **Final OSAR:** A single, valid SARIF log file that aggregates the findings and context from *multiple* OSAR fragments generated during a BeSPlaybook execution. This aggregation is performed by the Orchestrator. The final OSAR might contain multiple run objects (one per fragment) or a single run object with multiple toolComponent entries and merged results. The aggregation strategy SHOULD be consistent within a BeSLab deployment.

### **8.2. OSAR Generation Process**

The generation of a verifiable OSAR involves steps performed by plugins and the orchestrator.

* **8.2.1. Plugin Outputting OSAR Fragments:** Upon successful completion of an execute\_tool command relevant for reporting, the corresponding Plugin Agent MUST be prepared to respond to a get\_osar\_fragment STCP command. The response payload MUST include:  
  * The osar\_fragment as a SARIF JSON object.  
  * The corresponding OSARFragmentAuthenticityVC (as defined in 7.3.2.4), issued and signed by the Plugin Agent (or Environment Manager proxy), attesting to the fragment's integrity and linking it to the specific execution context (via reference to the ExecutionAttestationVC).  
* **8.2.2. Aggregation by BeSPlaybook/Orchestrator:** During the Aggregation/Reporting lifecycle phase, the Orchestrator performs the following:  
  1. Invokes get\_osar\_fragment for all relevant completed executions.  
  2. Receives the SARIF fragments and their associated OSARFragmentAuthenticityVCs.  
  3. **Verifies** each OSARFragmentAuthenticityVC to ensure the fragment is authentic, unmodified, and originated from the expected plugin execution within the correct environment. This involves checking the signature against the Plugin Agent's DID/key and verifying the link to a valid ExecutionAttestationVC.  
  4. **Aggregates** the verified SARIF fragments into a single final OSAR document. This involves merging the JSON structures according to a defined strategy (e.g., combining run objects, merging results under appropriate tool definitions). The aggregator MUST ensure the final document is a valid SARIF file.  
* **8.2.3. OSAR VC Attestation:** Once the final OSAR is assembled, the Orchestrator (or a designated BeSLab issuing service) generates and signs the FinalOSARAuthenticityVC (as defined in 7.3.2.5). This VC attests to the integrity of the complete aggregated report and provides metadata linking it back to the playbook and constituent fragments. The OSAR file and its corresponding VC are stored together.

### **8.3. Secure OSAR Exchange Protocol**

Exchanging OSARs between BeSLabs or providing them to authorized users requires a secure protocol leveraging STCP concepts and the DID/VC trust framework.

* **8.3.1. Requesting OSAR (Summary/Detailed):** A requesting entity (User Agent, another BeSLab Orchestrator) initiates the exchange. While STCP commands could be defined for this, a dedicated protocol built on DIDComm 9 is well-suited for this peer-to-peer interaction. The request message SHOULD specify:  
  * Target identifier (e.g., project name, repository URL).  
  * Optional Playbook identifier (to request results from a specific assessment type).  
  * Desired format (e.g., summary \- perhaps only metadata and counts, detailed \- the full OSAR SARIF file).  
  * The requester's DID.  
* **8.3.2. Verifiable Presentations (VPs) for Authorization:** The requesting entity MUST prove their authorization to access the requested OSAR. They construct a Verifiable Presentation (VP) containing a valid UserAuthorizationVC (or equivalent organizational VC) that grants the necessary permissions for the requested scope.53 The VP is signed by the requester using the private key associated with their DID (the holder/subject of the authorization VC). The VP is sent to the BeSLab hosting the OSAR. The hosting BeSLab acts as the Verifier: it verifies the VP signature, verifies the enclosed UserAuthorizationVC (checking signature, validity, issuer trust, revocation status), and checks if the claims grant access to the requested OSAR.4  
* **8.3.3. Secure Transfer Mechanism:** If authorization is verified, the hosting BeSLab transmits the requested OSAR (summary or full SARIF file) along with its corresponding FinalOSARAuthenticityVC. This transfer MUST occur over a secure, mutually authenticated channel established using the DIDs of the requester and the hosting BeSLab, likely leveraging DIDComm encryption and transport protocols.

#### **Works cited**

1. What is Model Context Protocol (MCP)? \- TechTalks, accessed April 26, 2025, [https://bdtechtalks.com/2025/03/31/model-context-protocol-mcp/?utm\_source=rss\&utm\_medium=rss\&utm\_campaign=model-context-protocol-mcp](https://bdtechtalks.com/2025/03/31/model-context-protocol-mcp/?utm_source=rss&utm_medium=rss&utm_campaign=model-context-protocol-mcp)  
2. Model Context Protocol (MCP) an overview \- Philschmid, accessed April 26, 2025, [https://www.philschmid.de/mcp-introduction](https://www.philschmid.de/mcp-introduction)  
3. Decentralized Identifiers (DIDs) v1.0 \- W3C, accessed April 26, 2025, [https://www.w3.org/TR/did-1.0/](https://www.w3.org/TR/did-1.0/)  
4. A Survey on Decentralized Identifiers and Verifiable Credentials \- arXiv, accessed April 26, 2025, [https://arxiv.org/html/2402.02455v1](https://arxiv.org/html/2402.02455v1)  
5. OASIS Static Analysis Results Interchange Format (SARIF) Technical Committee | Charter, accessed April 26, 2025, [https://www.oasis-open.org/committees/sarif/charter.php](https://www.oasis-open.org/committees/sarif/charter.php)  
6. OpenC2, accessed April 26, 2025, [https://openc2.org/](https://openc2.org/)  
7. The Role of OASIS OpenC2 in Cybersecurity Automation and Orchestration \- ResearchGate, accessed April 26, 2025, [https://www.researchgate.net/publication/370226708\_The\_Role\_of\_OASIS\_OpenC2\_in\_Cybersecurity\_Automation\_and\_Orchestration](https://www.researchgate.net/publication/370226708_The_Role_of_OASIS_OpenC2_in_Cybersecurity_Automation_and_Orchestration)  
8. Specification \- Model Context Protocol, accessed April 26, 2025, [https://modelcontextprotocol.io/specification/2025-03-26](https://modelcontextprotocol.io/specification/2025-03-26)  
9. DIDComm Messaging Specification v2 Editor's Draft \- Decentralized Identity Foundation, accessed April 26, 2025, [https://identity.foundation/didcomm-messaging/spec/](https://identity.foundation/didcomm-messaging/spec/)  
10. Verifiable Credentials and Decentralised Identifiers: Technical Landscape \- GS1 Reference, accessed April 26, 2025, [https://ref.gs1.org/docs/2025/VCs-and-DIDs-tech-landscape](https://ref.gs1.org/docs/2025/VCs-and-DIDs-tech-landscape)  
11. html \- Open Command and Control (OpenC2) Language Specification Version 1.0, accessed April 26, 2025, [https://docs.oasis-open.org/openc2/oc2ls/v1.0/cs01/oc2ls-v1.0-cs01.html](https://docs.oasis-open.org/openc2/oc2ls/v1.0/cs01/oc2ls-v1.0-cs01.html)  
12. The Role of OASIS OpenC2 in Cybersecurity Automation and Orchestration \- ResearchGate, accessed April 26, 2025, [https://www.researchgate.net/publication/366668777\_The\_Role\_of\_OASIS\_OpenC2\_in\_Cybersecurity\_Automation\_and\_Orchestration](https://www.researchgate.net/publication/366668777_The_Role_of_OASIS_OpenC2_in_Cybersecurity_Automation_and_Orchestration)  
13. Model Context Protocol (MCP): A comprehensive introduction for developers \- Stytch, accessed April 26, 2025, [https://stytch.com/blog/model-context-protocol-introduction/](https://stytch.com/blog/model-context-protocol-introduction/)  
14. The Model Context Protocol (MCP): A Guide for AI Integration | Generative-AI \- Wandb, accessed April 26, 2025, [https://wandb.ai/byyoung3/Generative-AI/reports/The-Model-Context-Protocol-MCP-A-Guide-for-AI-Integration--VmlldzoxMTgzNDgxOQ](https://wandb.ai/byyoung3/Generative-AI/reports/The-Model-Context-Protocol-MCP-A-Guide-for-AI-Integration--VmlldzoxMTgzNDgxOQ)  
15. Model Context Protocol: Introduction, accessed April 26, 2025, [https://modelcontextprotocol.io/introduction](https://modelcontextprotocol.io/introduction)  
16. Open Command and Control (OpenC2) Architecture Specification Version 1.0, accessed April 26, 2025, [https://docs.oasis-open.org/openc2/oc2arch/v1.0/oc2arch-v1.0.html](https://docs.oasis-open.org/openc2/oc2arch/v1.0/oc2arch-v1.0.html)  
17. An Introduction to FIPA Agent Communication Language: Standards for Interoperable Multi-Agent Systems \- SmythOS, accessed April 26, 2025, [https://smythos.com/ai-agents/ai-agent-development/fipa-agent-communication-language/](https://smythos.com/ai-agents/ai-agent-development/fipa-agent-communication-language/)  
18. Types of Agent Communication Languages \- SmythOS, accessed April 26, 2025, [https://smythos.com/ai-agents/ai-agent-development/types-of-agent-communication-languages/](https://smythos.com/ai-agents/ai-agent-development/types-of-agent-communication-languages/)  
19. Agent Communications Language \- Wikipedia, accessed April 26, 2025, [https://en.wikipedia.org/wiki/Agent\_Communications\_Language](https://en.wikipedia.org/wiki/Agent_Communications_Language)  
20. Standards and Interoperability – IEEE Power & Energy Society Multi-Agent Systems Working Group, accessed April 26, 2025, [https://site.ieee.org/pes-mas/agent-technology/standards-and-interoperability/](https://site.ieee.org/pes-mas/agent-technology/standards-and-interoperability/)  
21. sarl/sarl-acl: FIPA Agent Communication Language for SARL \- GitHub, accessed April 26, 2025, [https://github.com/sarl/sarl-acl](https://github.com/sarl/sarl-acl)  
22. A Review of FIPA Standardized Agent Communication Language and Interaction Protocols, accessed April 26, 2025, [https://www.jncet.org/Manuscripts/Volume-5/Special%20Issue-2/Vol-5-special-issue-2-M-32.pdf](https://www.jncet.org/Manuscripts/Volume-5/Special%20Issue-2/Vol-5-special-issue-2-M-32.pdf)  
23. What is Static Analysis Results Interchange Format (SARIF)? | Aptori App & API Security, accessed April 26, 2025, [https://www.aptori.com/glossary/static-analysis-results-interchange-format-sarif](https://www.aptori.com/glossary/static-analysis-results-interchange-format-sarif)  
24. Static Analysis Results Interchange Format (SARIF) Version 2.1.0 Plus Errata 01 \- Index of /, accessed April 26, 2025, [https://docs.oasis-open.org/sarif/sarif/v2.1.0/sarif-v2.1.0.html](https://docs.oasis-open.org/sarif/sarif/v2.1.0/sarif-v2.1.0.html)  
25. SARIF support for code scanning \- GitHub Docs, accessed April 26, 2025, [https://docs.github.com/en/code-security/code-scanning/integrating-with-code-scanning/sarif-support-for-code-scanning](https://docs.github.com/en/code-security/code-scanning/integrating-with-code-scanning/sarif-support-for-code-scanning)  
26. Bring Your Own Results (BYOR) \- Checkmarx Documentation, accessed April 26, 2025, [https://docs.checkmarx.com/en/34965-230340-bring-your-own-results--byor-.html](https://docs.checkmarx.com/en/34965-230340-bring-your-own-results--byor-.html)  
27. SARIF Output for Checkmarx One (Example for GitHub Action), accessed April 26, 2025, [https://docs.checkmarx.com/en/34965-112039-sarif-output-for-checkmarx-one--example-for-github-action-.html](https://docs.checkmarx.com/en/34965-112039-sarif-output-for-checkmarx-one--example-for-github-action-.html)  
28. Explore OASIS Static Analysis Results Interchange Format (SARIF) (\#118496) \- GitLab, accessed April 26, 2025, [https://gitlab.com/gitlab-org/gitlab/-/issues/118496](https://gitlab.com/gitlab-org/gitlab/-/issues/118496)  
29. Decentralized Identifier Resolution (DID Resolution) v0.3 \- W3C, accessed April 26, 2025, [https://www.w3.org/TR/did-resolution/](https://www.w3.org/TR/did-resolution/)  
30. Decentralized identifier \- Wikipedia, accessed April 26, 2025, [https://en.wikipedia.org/wiki/Decentralized\_identifier](https://en.wikipedia.org/wiki/Decentralized_identifier)  
31. Decentralized Identifiers (DIDs) v1.0 becomes a W3C Recommendation | 2022, accessed April 26, 2025, [https://www.w3.org/press-releases/2022/did-rec/](https://www.w3.org/press-releases/2022/did-rec/)  
32. Verifiable Credentials Data Model v2.0 \- W3C, accessed April 26, 2025, [https://www.w3.org/TR/vc-data-model-2.0/](https://www.w3.org/TR/vc-data-model-2.0/)  
33. Verifiable credentials \- Wikipedia, accessed April 26, 2025, [https://en.wikipedia.org/wiki/Verifiable\_credentials](https://en.wikipedia.org/wiki/Verifiable_credentials)  
34. Verifiable Credentials Data Model v1.1 \- W3C, accessed April 26, 2025, [https://www.w3.org/TR/vc-data-model/](https://www.w3.org/TR/vc-data-model/)  
35. Verifiable Credentials Use Cases \- W3C on GitHub, accessed April 26, 2025, [https://w3c.github.io/vc-use-cases/](https://w3c.github.io/vc-use-cases/)  
36. DID and VC: Untangling Decentralized Identifiers and Verifiable Credentials for the Web of Trust | Request PDF \- ResearchGate, accessed April 26, 2025, [https://www.researchgate.net/publication/349476251\_DID\_and\_VC\_Untangling\_Decentralized\_Identifiers\_and\_Verifiable\_Credentials\_for\_the\_Web\_of\_Trust](https://www.researchgate.net/publication/349476251_DID_and_VC_Untangling_Decentralized_Identifiers_and_Verifiable_Credentials_for_the_Web_of_Trust)  
37. Introduction to Verifiable Credentials \- SpruceID, accessed April 26, 2025, [https://spruceid.com/learn/verifiable-credentials](https://spruceid.com/learn/verifiable-credentials)  
38. Data Compliance: How Verifiable Credentials Helps With Compliance \- Dock.io, accessed April 26, 2025, [https://www.dock.io/post/data-compliance](https://www.dock.io/post/data-compliance)  
39. An overview of DIDComm messaging \- ACA-Py, accessed April 26, 2025, [https://aca-py.org/latest/gettingStarted/DIDCommMessaging/](https://aca-py.org/latest/gettingStarted/DIDCommMessaging/)  
40. DIDComm, accessed April 26, 2025, [https://didcomm.org/](https://didcomm.org/)  
41. What is DIDComm? (With Pictures\!) \- Indicio.tech, accessed April 26, 2025, [https://indicio.tech/blog/what-is-didcomm-with-pictures/](https://indicio.tech/blog/what-is-didcomm-with-pictures/)  
42. Guest Blog: DIDComm Demo, accessed April 26, 2025, [https://blog.identity.foundation/dif-guest-blog-didcomm-demo/](https://blog.identity.foundation/dif-guest-blog-didcomm-demo/)  
43. Decentralized Identifier Resolution (DID Resolution) v0.3 \- W3C on GitHub, accessed April 26, 2025, [https://w3c.github.io/did-resolution/](https://w3c.github.io/did-resolution/)  
44. Verifiable Credentials Overview \- W3C, accessed April 26, 2025, [https://www.w3.org/TR/vc-overview/](https://www.w3.org/TR/vc-overview/)  
45. Verifiable Credentials: The Ultimate Guide 2025 \- Dock Labs, accessed April 26, 2025, [https://www.dock.io/post/verifiable-credentials](https://www.dock.io/post/verifiable-credentials)  
46. The Complete Guide to Verifiable Credentials \- Wipro DICE ID, accessed April 26, 2025, [https://www.diceid.com/understanding-verifiable-credentials](https://www.diceid.com/understanding-verifiable-credentials)  
47. Standardised Communications Protocols | ARDC \- Australian Research Data Commons, accessed April 26, 2025, [https://ardc.edu.au/resource/standardised-communications-protocols/](https://ardc.edu.au/resource/standardised-communications-protocols/)  
48. What is Model Context Protocol (MCP): Explained \- Composio, accessed April 26, 2025, [https://composio.dev/blog/what-is-model-context-protocol-mcp-explained/](https://composio.dev/blog/what-is-model-context-protocol-mcp-explained/)  
49. openc2-impl-https/open-impl-https-v1.1-cs01.md at published \- GitHub, accessed April 26, 2025, [https://github.com/oasis-tcs/openc2-impl-https/blob/published/open-impl-https-v1.1-cs01.md](https://github.com/oasis-tcs/openc2-impl-https/blob/published/open-impl-https-v1.1-cs01.md)  
50. OpenC2 in the News, accessed April 26, 2025, [https://openc2.org/news2.html](https://openc2.org/news2.html)  
51. Issue & Manage Verifiable Credentials | Ping Identity, accessed April 26, 2025, [https://www.pingidentity.com/en/platform/capabilities/verifiable-credentials.html](https://www.pingidentity.com/en/platform/capabilities/verifiable-credentials.html)  
52. What are Verifiable Credentials and how do they work? \- One Identity, accessed April 26, 2025, [https://www.oneidentity.com/learn/what-are-verifiable-credentials-in-cybersecurity.aspx](https://www.oneidentity.com/learn/what-are-verifiable-credentials-in-cybersecurity.aspx)  
53. Introduction to Microsoft Entra Verified ID, accessed April 26, 2025, [https://learn.microsoft.com/en-us/entra/verified-id/decentralized-identifier-overview](https://learn.microsoft.com/en-us/entra/verified-id/decentralized-identifier-overview)  
54. Securing Verifiable Credentials using JOSE and COSE \- W3C, accessed April 26, 2025, [https://www.w3.org/TR/vc-jose-cose/](https://www.w3.org/TR/vc-jose-cose/)  
55. Verifiable Credentials Pattern \- DE4A Service Interoperability Solutions Toolbox, accessed April 26, 2025, [https://wiki.de4a.eu/index.php/Verifiable\_Credentials\_Pattern](https://wiki.de4a.eu/index.php/Verifiable_Credentials_Pattern)  
56. Verifiable Credential Issuer API Architecture Model \- GitHub, accessed April 26, 2025, [https://github.com/w3c-ccg/vc-api/blob/main/architecture.md](https://github.com/w3c-ccg/vc-api/blob/main/architecture.md)  
57. Verifiable Credentials API v0.7, accessed April 26, 2025, [https://w3c-ccg.github.io/vc-api/](https://w3c-ccg.github.io/vc-api/)
