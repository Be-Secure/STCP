# **Security Tool Context Protocol (STCP) Specification for Be-Secure Ecosystem**

## **1\. Introduction to Security Tool Context Protocol (STCP)**

### **1.1. Purpose and Vision**

The modern cybersecurity landscape involves a diverse array of specialized tools for assessment, analysis, and reporting. Integrating these tools into cohesive workflows, particularly within collaborative environments like Be-Secure, presents significant challenges. Often, integration requires custom scripting and ad-hoc methods, leading to fragmented systems, increased complexity, potential vendor lock-in, and inconsistent security postures. This complexity hinders the efficient use and combination of best-of-breed security tools.

The Security Tool Context Protocol (STCP) is proposed for the Be-Secure ecosystem to address these challenges. STCP aims to establish an open protocol standardizing the integration, invocation, management, and secure data exchange between security tools (acting as BeSLab plugins or 'Agents') and the Be-Secure orchestration components, namely BeSEnvironments and BeSPlaybooks. It acts as a standardized interface layer, akin to a universal adapter or "USB port," enabling security tools to connect seamlessly within the Be-Secure framework, thereby reducing complexity and fostering interoperability.

The vision for STCP is to cultivate a "plug-and-play" ecosystem for security tools within Be-Secure labs (BeSLabs). This protocol will facilitate the seamless onboarding of new tools, enable their reliable execution orchestrated by BeSPlaybooks within defined BeSEnvironments, and support the trustworthy generation and secure sharing of assessment results, specifically Open Source Assessment Reports (OSARs). By providing a standardized communication and interaction framework, STCP aims to enhance interoperability, improve scalability, and significantly reduce the friction associated with integrating and managing diverse security tools. This standardization is foundational; by defining a common protocol, the effort required to incorporate a new security tool shifts from building bespoke integrations to implementing a standard interface. This reduction in per-tool integration complexity directly fosters a richer, more diverse, and adaptable security tooling ecosystem within Be-Secure, preventing lock-in to pre-integrated solutions.

### **1.2. Core Concepts**

STCP provides a universal interface for security tools within the Be-Secure framework.

The core concepts of STCP are:

* **STCP Orchestrator:** This entity represents the control logic that initiates, manages, and coordinates the security assessment workflow. Typically, this logic resides within or is invoked by a BeSPlaybook executing within a BeSEnvironment. It functions as the client component in the protocol, managing connections and translating workflow steps into protocol messages.  
* **STCP Agent (BeSLab Plugin):** This represents a security tool that has been packaged or wrapped to comply with the STCP specification. It acts as a server component within the protocol, running within or accessible to a BeSEnvironment. The Agent exposes the underlying tool's functionalities (e.g., vulnerability scanning, static analysis, configuration checks, report generation) through a standardized STCP interface.  
* **STCP Capabilities:** These are the specific actions, functions, or data retrieval operations exposed by an STCP Agent via its interface. Examples include 'run\_static\_analysis', 'get\_dependency\_vulnerabilities', 'configure\_target\_repository', or 'generate\_partial\_osar'. These represent the discrete functionalities offered by the underlying tool.  
* **Standardized Communication:** Interaction between the STCP Orchestrator and STCP Agents occurs exclusively through messages defined by the STCP specification, using a designated communication protocol (e.g., JSON-RPC 2.0) over specified transport mechanisms.

### **1.3. Key Terminology**

* **BeSEnvironment (Be-Secure Environment):** A configured environment containing a specific set of security tools and utilities, tailored for tasks like Red Teaming (RT) or Blue Teaming (BT). Managed via lifecycle functions (\_\_besman\_install, \_\_besman\_uninstall, etc.).  
* **BeSPlaybook (Be-Secure Playbook):** A scripted workflow defining the steps for a security assessment or task. Managed via lifecycle functions (\_\_besman\_init, \_\_besman\_execute, etc.) and executed by the besman utility \[User Query\].  
* **BeSLab (Be-Secure Lab):** A conceptual or physical instance of a security assessment environment, typically comprising one or more BeSEnvironments, associated BeSPlaybooks, and potentially leveraging STCP Agents. Can be defined by a BeSLab blueprint.  
* **BeSLab Plugin:** A software component, often wrapping a security tool, designed to integrate into the BeSLab ecosystem. In this specification, STCP-compliant plugins are referred to as STCP Agents.  
* **STCP Agent:** A BeSLab Plugin that implements the server-side STCP interface, exposing security tool capabilities.  
* **STCP Orchestrator:** The client-side STCP logic, typically within a BeSPlaybook's execution context, responsible for invoking STCP Agents.  
* **Open Source Assessment Report (OSAR):** A standardized report format for documenting the findings of security assessments conducted within the Be-Secure ecosystem, defined by BeS-Schema.1  
* **Decentralized Identifier (DID):** A new type of globally unique identifier that enables verifiable, decentralized digital identity, controlled by the DID subject. DIDs resolve to DID Documents containing verification methods.  
* **Verifiable Credential (VC):** A tamper-resistant digital credential containing claims made by an issuer about a subject, secured using cryptographic proofs. Used for authentication, authorization, and attestation.3

## **2\. STCP Architecture**

### **2.1. Architectural Model**

STCP adopts a distributed, client-server architectural model tailored for the operational context of BeSLabs. This model facilitates decoupling between the orchestration logic and the specific security tool implementations.

* **BeSPlaybook/BeSEnvironment (Orchestrator Role):** These components act as the primary control points within the STCP architecture. The execution environment of a BeSPlaybook, operating within a specific BeSEnvironment, houses or interacts with the STCP Orchestrator (client) logic. It is responsible for initiating STCP requests based on playbook steps and managing the overall assessment workflow.  
* **BeSLab Plugin (Agent/Server Role):** Each STCP-compliant security tool runs as an Agent (server). This Agent process executes within the BeSEnvironment or is network-accessible to it. It listens for incoming STCP requests on a predefined communication channel, executes the corresponding security tool functions, and returns results via STCP responses. The Agent exposes the tool's capabilities through the standardized STCP interface.  
* **Communication Channel:** STCP defines standardized transport mechanisms for communication between Orchestrators and Agents. STCP supports:  
  * **Standard Input/Output (stdio):** Suitable for Agents running as local child processes of the Orchestrator within the same BeSEnvironment container or machine.18  
  * **HTTP/HTTPS with Server-Sent Events (SSE) or WebSockets:** For scenarios where Agents might run as separate services, potentially in different containers or machines within a distributed BeSLab, allowing for network-based communication. Security considerations like TLS and authentication are paramount for network transports.18  
  * Other transports like gRPC could be considered in future extensions.

### **2.2. Core Components**

The STCP architecture comprises two primary logical components:

* **STCP Orchestrator (Client-side Logic):** This logic typically resides within the besman utility or the execution environment spawned when a BeSPlaybook is run. Its responsibilities include:  
  * **Discovery:** Identifying available STCP Agents within the target BeSEnvironment and querying their capabilities (metadata, functions, parameters).20  
  * **Invocation:** Translating high-level commands or steps defined in a BeSPlaybook into specific STCP request messages (e.g., agent/execute).20  
  * **Session Management:** Establishing and managing communication sessions with one or more STCP Agents over the chosen transport.18  
  * **Response Handling:** Receiving and processing responses (both successful results and errors) from Agents.18  
  * **Integration:** Feeding results back into the BeSPlaybook workflow, potentially storing intermediate data or making it available for subsequent steps like OSAR generation.  
  * **Security Context Management:** Handling the presentation of VCs for authorization if required by Agents, and verifying VCs presented by Agents.2  
* **STCP Agent (Server-side Logic / BeSLab Plugin):** This component acts as a wrapper around a specific security tool, making it STCP-compliant. Its responsibilities include:  
  * **Interface Implementation:** Implementing the server-side STCP protocol, listening for requests on the configured transport.18  
  * **Capability Advertisement:** Providing metadata about the Agent itself and the capabilities (functions) of the underlying tool it exposes via STCP (e.g., in response to an agent/discover request).21  
  * **Request Handling:** Receiving, parsing, and validating incoming STCP requests.18  
  * **Tool Execution:** Invoking the appropriate functions of the underlying security tool based on the capability requested and parameters provided in the STCP message.20  
  * **Result Formatting:** Collecting the output from the security tool and formatting it into the standardized STCP response structure. Where applicable, this structure should align with or facilitate the creation of BeS-Schema defined formats, such as partial OSAR data.1  
  * **State Management:** Handling initialization, configuration (e.g., setting target parameters), and cleanup operations specific to the wrapped tool, triggered via corresponding STCP messages.18  
  * **Security Operations:** Optionally implementing DID/VC verification logic for incoming requests and potentially generating or presenting its own VCs for attestation or authentication.

### **2.3. Conceptual Architecture Diagram Description**


![STCP Component Architecture](https://github.com/Be-Secure/STCP/blob/main/images/STCPComponentArchitecture.png)

A diagram illustrating the STCP architecture would show the following components and interactions:

* **Central Orchestration:** A box representing the **BeSPlaybook Execution Context** containing the **STCP Orchestrator** logic.  
* **Runtime Environment:** A surrounding container representing the **BeSEnvironment**, indicating it provides the runtime for Agents.  
* **Agents/Plugins:** Inside the BeSEnvironment, multiple boxes representing **STCP Agents (BeSLab Plugins)**, each wrapping an underlying **Security Tool (A, B,...)**.  
* **Communication:** Arrows labeled "STCP (JSON-RPC 2.0 over stdio/HTTP)" connecting the **STCP Orchestrator** to each **STCP Agent**. These arrows represent the standardized protocol communication for discovery, configuration, execution, and cleanup.  
* **Tool Interaction:** Internal arrows within each Agent box showing the Agent interacting with its underlying **Security Tool**.  
* **DID/VC Infrastructure:** A separate cloud or network symbol representing the **DID/VC Infrastructure** (including DID Registries, VC Issuers/Verifiers). Arrows connect both the **STCP Orchestrator** and **STCP Agents** to this infrastructure, labeled "DID Resolution, VC Issuance/Verification". This shows how identity and trust are managed.  
* **Reporting Standard:** A document symbol labeled **BeS-Schema** representing the OSAR definition standard. An arrow points from this to the **BeSPlaybook Execution Context**, indicating its use during report preparation.  
* **Output Storage:** A database symbol labeled **Datastore**. An arrow connects the **BeSPlaybook Execution Context** to the Datastore, labeled "Publish OSAR \+ Attestation VC", showing the final output destination.  
* **Environment Management:** An arrow from the **BeSEnvironment** pointing towards the **STCP Agents**, labeled "Lifecycle Management (Install, Validate, etc.)", indicating the environment's role in managing the agents themselves.

This diagram visually emphasizes the decoupling achieved by STCP, the standardized communication layer, the integration with DID/VC trust mechanisms, and the flow towards generating standardized, attested reports.

### **2.4. Communication Protocol**

STCP mandates the use of **JSON-RPC 2.0** as the underlying communication protocol for all interactions between Orchestrators and Agents.20 JSON-RPC 2.0 provides a lightweight, well-defined, and transport-agnostic structure for requests, responses, and notifications, making it suitable for both local inter-process communication (IPC) via stdio and network communication via HTTP or other protocols.18

Key aspects of using JSON-RPC 2.0 in STCP:

* **Request Object:** Contains jsonrpc (version "2.0"), method (STCP method name, e.g., "agent/execute"), params (object or array containing parameters for the method), and id (unique request identifier).18  
* **Response Object (Success):** Contains jsonrpc ("2.0"), result (payload containing the outcome of the method execution), and id (matching the request ID).18  
* **Response Object (Error):** Contains jsonrpc ("2.0"), error (an object with code, message, and optional data), and id (matching the request ID, or null if the error context prevents determining it).18  
* **Notification Object:** Contains jsonrpc ("2.0"), method (notification name, e.g., "agent/progressUpdate"), and params. Notifications do not have an id and do not receive responses.18

STCP defines a set of standard error codes within the JSON-RPC error object's code field, extending the standard JSON-RPC codes:

* \-32600: Invalid Request (Standard JSON-RPC)  
* \-32601: Method not found (Standard JSON-RPC)  
* \-32602: Invalid params (Standard JSON-RPC)  
* \-32603: Internal error (Standard JSON-RPC)  
* \-32000: Agent Error (Generic STCP Agent error)  
* \-32001: Tool Not Found (Underlying security tool missing or not executable)  
* \-32002: Configuration Error (Invalid or missing configuration for the agent/tool)  
* \-32003: Execution Failed (Tool executed but failed or returned an error)  
* \-32004: Invalid Parameters (STCP parameters valid JSON-RPC, but semantically incorrect for the capability)  
* \-32005: Timeout Error (Tool execution exceeded allowed time)  
* \-32006: Permission Denied (Orchestrator lacks authorization for the requested capability)  
* \-32007: Resource Unavailable (Required resource, e.g., target file, is inaccessible)

### **2.5. Message Types (Requests, Responses, Notifications)**

STCP defines specific methods for interaction, following the JSON-RPC 2.0 structure.18

* **Requests (Orchestrator \-\> Agent):** Messages sent by the Orchestrator to invoke actions or retrieve information from an Agent. They require a response.18  
* **Responses (Agent \-\> Orchestrator):** Messages sent by the Agent in reply to a Request. They contain either a result payload on success or an error object on failure.18 Result payloads should be structured JSON, potentially adhering to schemas derived from BeS-Schema 1 for assessment data.  
* **Notifications (Agent \-\> Orchestrator):** Messages sent by the Agent to the Orchestrator without expecting a response. Used for asynchronous updates like progress or logging.18

**Table 2.5.1: Core STCP Message Methods and Payloads**

| Method Name | Direction | Description | Request Parameters (Example JSON) | Success Response Payload (Example JSON) | Relevant Error Codes |
| :---- | :---- | :---- | :---- | :---- | :---- |
| agent/discover | O \-\> A | Queries the agent for its metadata and capabilities. | {} | { "agent\_id": "did:example:agent123", "agent\_name": "Trivy Agent", "agent\_version": "1.2.0", "tool\_name": "Trivy", "tool\_version": "0.45.1", "stcp\_version": "1.0", "capabilities": \[...\] } (See Section 5.2 for capability structure) | \-32601, \-32603 |
| agent/configure | O \-\> A | Configures the agent or underlying tool for a specific task. | { "capability\_id": "scan\_container\_image", "config": { "target\_image": "nginx:latest", "severity\_threshold": "HIGH" } } | { "status": "configured", "details": "Agent configured for image scan." } | \-32601, \-32602, \-32001, \-32002, \-32006 |
| agent/execute | O \-\> A | Executes a specific capability (security tool function). | { "capability\_id": "scan\_container\_image", "arguments": { "output\_format": "json" }, "execution\_context": { "playbook\_run\_id": "run-abc" } } | { "status": "completed", "results": {... } } (Structure depends on capability, ideally aligns with BeS-Schema partial output) | \-32601, \-32602, \-32001, \-32002, \-32003, \-32004, \-32005, \-32006, \-32007 |
| agent/getStatus | O \-\> A | Queries the current status of the agent and underlying tool. | {} | { "status": "ready", "tool\_status": "idle", "version": "1.2.0", "config\_hash": "abc..." } | \-32601, \-32603, \-32001 |
| agent/cleanup | O \-\> A | Instructs the agent to perform cleanup (remove temp files, reset state). | { "execution\_context": { "playbook\_run\_id": "run-abc" } } | { "status": "cleaned", "details": "Temporary files removed." } | \-32601, \-32603, \-32001, \-32003 |
| agent/progressUpdate | A \-\> O (Notify) | Provides progress updates for long-running tasks. | { "capability\_id": "scan\_container\_image", "progress": 75, "message": "Scanning layers..." } | N/A (Notification) | N/A (Notification) |
| agent/logMessage | A \-\> O (Notify) | Sends logging information from the agent or tool. | { "level": "info", "message": "Tool initialization complete." } | N/A (Notification) | N/A (Notification) |
| agent/requestVC | A \-\> O (Request) | (Optional) Agent requests a VC from the Orchestrator for authorization. | { "vc\_requirements": { "type": "PlaybookExecutionAuthorization", "issuer": "did:example:bes\_authority" } } | { "vc\_presentation": {... } } (Presented VC) | \-32601, \-32603 |
| agent/presentVC | O \-\> A (Request) | (Optional) Orchestrator presents a VC to the Agent (e.g., in response to requestVC). | { "vc\_presentation": {... } } | { "verification\_status": "success" } | \-32601, \-32602, \-32006 (Verification Failed) |

*(Note: O \-\> A \= Orchestrator to Agent, A \-\> O \= Agent to Orchestrator)*

## **3\. STCP Integration with BeSEnvironment Lifecycle**

### **3.1. Overview of BeSEnvironment Lifecycle Functions**

The BeSEnvironment provides the runtime context where security tools operate. Its state and configuration are managed through a set of lifecycle functions, typically implemented as shell script functions invoked by the besman utility. The functions are:

* \_\_besman\_install: Installs the required tools.  
* \_\_besman\_uninstall: Removes the installed tools.  
* \_\_besman\_validate: Checks whether all the tools are installed and required configurations are met.  
* \_\_besman\_update: Update configurations of the tools.  
* \_\_besman\_reset: Reset the environment to the default state.

STCP must integrate with these lifecycle functions to ensure that STCP Agents (representing the tools) are properly managed, discoverable, and reflect the intended state of the BeSEnvironment.

### **3.2. STCP Interactions for Tool Management (\_\_besman\_install, \_\_besman\_uninstall)**

The \_\_besman\_install and \_\_besman\_uninstall functions are primarily responsible for file system operations (installing/removing tool binaries and configuration files) . STCP can augment these processes for better agent lifecycle management at the protocol layer.

* **Post-\_\_besman\_install:** After the shell script successfully installs the tool files, an additional step can manage the STCP Agent aspect. If the installed tool includes an STCP Agent wrapper, this wrapper could be started. Upon startup, the Agent should ideally register itself with a local discovery service within the BeSEnvironment (see Section 5.3). Alternatively, the \_\_besman\_install script itself could conclude by sending an STCP notification or calling a local STCP utility to signal the successful installation and availability of the new Agent. This ensures the Orchestrator can discover and interact with the newly installed tool via STCP.  
* **Pre-\_\_besman\_uninstall:** Before the \_\_besman\_uninstall script removes the tool's files, it's beneficial to gracefully shut down the corresponding STCP Agent. The script could send an STCP agent/cleanup message or a specific agent/shutdown message to the target Agent. This allows the Agent to release resources, save any final state, and deregister from the discovery service before its files are removed, ensuring a cleaner teardown.

### **3.3. STCP Interactions for Environment Validation and State (\_\_besman\_validate, \_\_besman\_update, \_\_besman\_reset)**

These lifecycle functions directly relate to the operational state and configuration of the tools within the environment, making STCP integration highly valuable.

* **\_\_besman\_validate:** This function's core purpose is to verify the environment's integrity . STCP enhances this by enabling validation at the protocol level. The \_\_besman\_validate script can invoke STCP Orchestrator logic. This logic would iterate through the list of STCP Agents expected to be present in the environment (based on the environment blueprint). For each expected Agent, it sends an agent/getStatus request via STCP. The Agent responds with its status, version information, and potentially a hash of its current configuration. The Orchestrator aggregates these responses. A failure to respond or an unhealthy status from any critical Agent would cause validation to fail. Success implies all required tools are installed, running, and reachable via STCP. This integration transforms \_\_besman\_validate significantly. Instead of just checking file existence, it verifies the operational readiness of the tools through the STCP interface. The collected data from agent/getStatus calls (agent IDs, versions, status, configuration hashes) constitutes a detailed snapshot of the environment's state. This structured data can be packaged into the credentialSubject of a Verifiable Credential (VC).3 The BeSEnvironment itself, identified by a DID , can issue this VC, cryptographically attesting to its validated configuration at a specific point in time.2 This 'BeSEnvironment State Credential' provides a trustworthy, shareable proof of the environment's readiness, crucial for ensuring reproducible security assessments and maintaining audit trails, directly addressing the need for trustworthy information exchange.  
* **\_\_besman\_update:** When tool configurations need updating , this script can leverage STCP. After updating configuration files on disk, the script can invoke STCP Orchestrator logic to send agent/configure messages to the relevant Agents. This prompts the Agents to reload their configuration or apply the changes dynamically, ensuring the running tool reflects the updated state.  
* **\_\_besman\_reset:** To reset tools to a default state , this script can similarly use STCP. It would trigger agent/configure messages with default parameters or potentially a dedicated agent/reset message if implemented by the Agent, instructing it to revert to its initial configuration.

**Table 3.3.1: Mapping STCP Messages to BeSEnvironment Functions**

| BeSEnvironment Function | Trigger Point | Corresponding STCP Message(s) | Purpose of Interaction |
| :---- | :---- | :---- | :---- |
| \_\_besman\_install | Environment Creation/Modification | agent/register (Implicit/Notify) | Agent registers with discovery service upon startup post-installation. |
| \_\_besman\_uninstall | Environment Deletion/Modification | agent/cleanup / agent/shutdown | Gracefully stop the agent, deregister before file removal. |
| \_\_besman\_validate | Validation Command (besman validate) | agent/getStatus | Query each expected agent to verify it's running, configured, and reachable via STCP. |
| \_\_besman\_update | Update Command (besman update) | agent/configure | Instruct agents to reload or apply updated configuration settings. |
| \_\_besman\_reset | Reset Command (besman reset) | agent/configure / agent/reset | Instruct agents to revert to default configuration settings. |

## **4\. STCP Integration with BeSPlaybook Lifecycle**

### **4.1. Overview of BeSPlaybook Lifecycle Functions and Naming**

BeSPlaybooks define automated, interactive, or manual workflows for security tasks. Their execution is managed by a lifecycle comprising several functions, invoked sequentially by the besman utility. The standard lifecycle functions are:

* \_\_besman\_init(): Initializes everything necessary for executing the playbook as well as for publishing the reports.  
* \_\_besman\_execute(): Executes the steps file which contains the instructions for the activity. The steps file can be in various formats such as 'sh', '.ipynb', or '.md'.  
* \_\_besman\_prepare(): Filters the data from the report to prepare for publishing.  
* \_\_besman\_publish(): Publishes the reports to the datastore.  
* \_\_besman\_cleanup(): Handles the cleanup tasks.  
* \_\_besman\_launch(): Playbook launch function that gets called by BeSman utility. This function triggers the lifecycle methods of a playbook.

Naming conventions dictate the format for the lifecycle file (besman-\<purpose | intent\>-\< Tool Name | Artifact Name \>-playboook-\<version\>.sh) and the corresponding steps file (besman-\<purpose | intent\>-\< Tool Name | Artifact Name \>-playbook-\<version\>.sh/md/ipynb) \[User Query\]. STCP integrates primarily within the \_\_besman\_init, \_\_besman\_execute, \_\_besman\_prepare, \_\_besman\_publish, and \_\_besman\_cleanup functions to orchestrate the security tools (Agents).

### **4.2. STCP Orchestration via Playbook (\_\_besman\_launch, \_\_besman\_init, \_\_besman\_execute)**

The execution of a BeSPlaybook serves as the primary driver for STCP interactions.

* **\_\_besman\_launch:** This function acts as the trigger, initiating the playbook's lifecycle sequence. It doesn't typically interact directly with STCP but calls the functions that do.  
* **\_\_besman\_init:** During initialization, STCP plays a crucial role. This function may:  
  * Use STCP agent/discover messages to verify that all STCP Agents required by the playbook are present and available in the target BeSEnvironment.  
  * Send STCP agent/configure messages to Agents to set up initial parameters specific to this playbook run (e.g., target repository URL, scan scope, credentials). This ensures tools are correctly primed before execution begins.  
* **\_\_besman\_execute:** This is the heart of STCP orchestration within the playbook . It reads the steps file associated with the playbook. For automated steps (e.g., in a .sh or .ipynb file), this function translates the commands intended for specific security tools into STCP agent/execute requests.  
  * The STCP Orchestrator logic within or called by \_\_besman\_execute identifies the target Agent based on the step's requirements.  
  * It constructs the JSON-RPC request, including the capability\_id corresponding to the desired tool action and the necessary arguments derived from the playbook step or context.  
  * The request is sent to the appropriate Agent via the configured STCP transport.  
  * The Orchestrator waits for the response, collecting the result payload upon success or handling error responses (e.g., logging the error, potentially halting the playbook).  
  * Intermediate results received from Agents are stored or made available for subsequent steps within the \_\_besman\_execute phase or for later processing in \_\_besman\_prepare.

### **4.3. STCP for Result Handling and Reporting (\_\_besman\_prepare, \_\_besman\_publish)**

After the core security tasks are completed in \_\_besman\_execute, the focus shifts to processing and disseminating the results, where STCP continues to play a supporting role.

* **Result Collection:** Raw results from tool executions are accumulated during the \_\_besman\_execute phase, primarily from the result payloads of successful STCP agent/execute responses. Agents might also generate output files, the paths to which could be returned in STCP responses.  
* **\_\_besman\_prepare:** This function takes the collected raw data (whether directly from STCP responses or read from files generated by Agents) and transforms it into the standardized Open Source Assessment Report (OSAR) format. This mapping process adheres strictly to the schema defined in the BeS-Schema repository.1 While the primary role of STCP is execution, an Agent *could* optionally expose an STCP capability like agent/getFormattedResults that provides its output pre-formatted according to a specific part of the BeS-Schema, potentially simplifying the \_\_besman\_prepare logic.  
* **\_\_besman\_publish:** This function takes the final, structured OSAR data prepared in the previous step. Critically, this is where the trustworthy exchange mechanism using DIDs and VCs is implemented. The \_\_besman\_publish function orchestrates the creation of an OSARAttestationCredential (see Section 6.3). This VC, signed by the DID of the BeSLab or the specific Playbook execution context , cryptographically binds metadata (like the OSAR hash, generating playbook DID, environment DID, timestamp) to the report.3 The OSAR (as a JSON object) and its corresponding VC are then published to the designated datastore or transmitted to relevant parties.

The clear separation within the playbook lifecycle between executing tools via STCP (\_\_besman\_execute) and preparing/publishing the attested report (\_\_besman\_prepare, \_\_besman\_publish) provides a robust workflow. STCP facilitates the reliable gathering of raw assessment data from diverse tools, while the subsequent stages focus on standardizing this data into the OSAR format 1 and creating a verifiable, trustworthy artifact using VCs.2 This structure directly supports the user's requirement for generating OSARs and ensuring their secure, trustworthy exchange.

### **4.4. STCP for Cleanup (\_\_besman\_cleanup)**

The final phase of the playbook lifecycle involves cleaning up resources used during execution.

* **\_\_besman\_cleanup:** This function can invoke STCP Orchestrator logic to send agent/cleanup messages to all Agents that were involved in the playbook execution. This instructs the Agents to perform necessary cleanup actions, such as deleting temporary files created during their operation, resetting internal state, releasing locks, or terminating specific processes spawned for the task. This ensures the BeSEnvironment is left in a clean state for subsequent playbook runs.

**Table 4.4.1: Mapping STCP Messages to BeSPlaybook Functions**

| BeSPlaybook Function | Phase Description | Corresponding STCP Message(s) | Purpose of Interaction |
| :---- | :---- | :---- | :---- |
| \_\_besman\_init | Initialization & Prerequisite Check | agent/discover, agent/configure | Verify required agents are present; set initial configuration (e.g., target scope). |
| \_\_besman\_execute | Core Task Execution | agent/execute | Invoke specific security tool capabilities (scans, analysis) via agents; collect raw results. |
| \_\_besman\_prepare | Result Processing & Formatting | agent/getFormattedResults (Optional) | Transform raw results (from STCP responses or files) into standardized OSAR format (BeS-Schema). |
| \_\_besman\_publish | Reporting & Attestation | (Interacts with VC Issuance Service) | Create OSAR Attestation VC; publish OSAR and VC bundle to datastore/recipients. |
| \_\_besman\_cleanup | Post-Execution Cleanup | agent/cleanup | Instruct agents to remove temporary data, reset state, release resources. |

## **5\. BeSLab Plugin (Agent) Onboarding and Execution**

### **5.1. Requirements for STCP-Compliant BeSLab Plugins**

For a security tool to function as an STCP Agent within the Be-Secure ecosystem, its corresponding BeSLab Plugin must adhere to specific requirements to ensure interoperability and participation in the "plug-and-play" model STCP enables. These requirements are:

* **STCP Interface Implementation:** The plugin MUST implement the server-side STCP protocol, correctly handling incoming JSON-RPC 2.0 requests and generating valid responses or notifications according to this specification.18  
* **Transport Listener:** The Agent MUST listen for incoming STCP requests on one of the designated transport mechanisms configured for the BeSEnvironment (e.g., stdio, HTTP).18  
* **Metadata Provision:** The Agent MUST provide mandatory metadata about itself and the underlying tool upon request (typically via the agent/discover method), as detailed in Section 5.2.  
* **Capability Exposure:** The core functionalities of the security tool MUST be exposed as discrete STCP capabilities, each clearly defined with its parameters and expected output structure.21  
* **Standard Message Handling:** The Agent MUST correctly implement handlers for standard STCP messages, including at minimum: agent/discover, agent/configure, agent/execute, agent/getStatus, and agent/cleanup.  
* **Optional Security Integration:** Agents MAY need to support DID/VC-based interactions (e.g., verifying incoming authorization VCs, presenting capability VCs) if the specific BeSLab deployment or playbook requires enhanced security for certain operations.3

### **5.2. Metadata and Interface Definition**

Effective discovery and orchestration require Agents to advertise their identity and capabilities in a standardized format.20 STCP mandates that Agents provide the following metadata, typically returned in the result payload of an agent/discover request or potentially available via a static manifest file associated with the plugin:

* agent\_id (String, Required): A unique identifier for this specific agent instance. This SHOULD ideally be a Decentralized Identifier (DID) to enable verifiable identity.  
* agent\_name (String, Required): A human-readable name for the agent (e.g., "Semgrep SAST Agent").  
* agent\_version (String, Required): The version number of the STCP agent wrapper/plugin software itself.  
* tool\_name (String, Required): The canonical name of the underlying security tool being wrapped (e.g., "Semgrep").  
* tool\_version (String, Required): The version number of the underlying security tool being used by the agent.  
* stcp\_version (String, Required): The version of the STCP specification that this agent implements (e.g., "1.0").  
* description (String, Optional): A brief description of the agent's purpose.  
* capabilities (Array\[Object\], Required): An array describing each function the agent exposes via STCP. Each capability object MUST contain:  
  * capability\_id (String, Required): A unique, machine-readable identifier for the capability within the agent's scope (e.g., run\_sast\_scan, get\_dependencies). Should follow a consistent naming convention (e.g., verb\_noun).  
  * description (String, Required): A human-readable explanation of what the capability does.  
  * parameters (Array\[Object\], Optional): An array defining the input parameters accepted by the capability. Each parameter object should include:  
    * name (String, Required): Parameter name.  
    * type (String, Required): Parameter data type (e.g., "string", "integer", "boolean", "object", "array").  
    * required (Boolean, Required): Whether the parameter is mandatory.  
    * description (String, Optional): Explanation of the parameter.  
  * output\_schema (Object | String, Optional): A description of the structure of the result payload returned on successful execution of this capability. This could be an inline JSON Schema definition, a reference URI to an external schema (e.g., a specific section within BeS-Schema 1), or a textual description.

### **5.3. Registration and Discovery Process**

For an STCP Orchestrator (e.g., running a BeSPlaybook) to interact with Agents in a BeSEnvironment, it must first discover which Agents are available and what capabilities they offer.20 STCP supports the following discovery mechanisms:

* **Option A: Local Registry Service (Recommended Baseline):**  
  * A lightweight registry service runs within the BeSEnvironment.  
  * When an STCP Agent starts (typically triggered after \_\_besman\_install), it registers itself with this local service, providing its endpoint information (e.g., path for stdio, port for HTTP) and potentially its basic metadata.  
  * The STCP Orchestrator queries this known local registry service to get a list of active Agents and their endpoints.  
  * The Orchestrator then sends agent/discover messages directly to each discovered Agent to retrieve detailed capability information.  
* **Option B: Decentralized Discovery (Advanced):**  
  * Leverages DIDs and VCs for a more decentralized approach.  
  * Each Agent, identified by a DID, could publish a "Capability Credential" (a VC) attesting to its existence, endpoint, and capabilities.  
  * These VCs could be stored in a DID-associated registry (like a Verifiable Data Registry linked to the BeSLab's DID) or announced via a DIDComm-based protocol.5  
  * The Orchestrator queries this registry or listens for announcements to discover available Agents and their verifiable capabilities. This approach enhances trust as the capabilities themselves are attested cryptographically.  
* **Direct agent/discover Broadcast/Multicast (Less Recommended):** In simpler network environments, the Orchestrator could potentially broadcast an agent/discover request, and all listening Agents would respond. This is generally less scalable and efficient than using a registry.

The chosen discovery mechanism should be configured consistently within a BeSLab deployment.

### **5.4. STCP Execution Flow: Playbook \-\> Environment \-\> Plugin (Agent)**

The end-to-end flow of executing a security tool via STCP within the Be-Secure framework typically proceeds as follows:

1. **Playbook Invocation:** A user or automated process invokes besman to launch a specific BeSPlaybook against a target BeSEnvironment. The \_\_besman\_launch function starts the lifecycle.  
2. **Initialization:** The \_\_besman\_init function executes \[User Query\]. It uses the configured discovery mechanism (e.g., querying the local registry) to find required STCP Agents within the BeSEnvironment. It may send agent/configure messages to prime these Agents with playbook-specific settings (e.g., target URL, analysis scope).  
3. **Step Execution:** The \_\_besman\_execute function begins processing the playbook's steps file \[User Query\]. It encounters a step requiring a security tool action (e.g., "Run Trivy scan on image X").  
4. **STCP Request Generation:** The STCP Orchestrator logic within \_\_besman\_execute identifies the appropriate Agent (e.g., the "Trivy Agent") and the corresponding capability (e.g., scan\_container\_image). It constructs a JSON-RPC 2.0 agent/execute request, populating the params field with arguments derived from the playbook step (e.g., {"arguments": {"target\_image": "X", "output\_format": "json"}}). A unique request id is generated.18  
5. **Request Transmission:** The Orchestrator sends the JSON-RPC request message to the target STCP Agent over the configured transport (e.g., writes to the Agent's stdin, sends an HTTP POST request).18  
6. **Agent Processing:** The STCP Agent receives the request via its transport listener. It parses the JSON-RPC message, validates the method (agent/execute) and parameters against its advertised capability definition.  
7. **Tool Invocation:** The Agent invokes the underlying security tool's functionality (e.g., executes the trivy image X \--format json command) using the provided parameters.  
8. **Result Collection & Formatting:** The Agent captures the output from the security tool (e.g., the JSON output from Trivy).  
9. **STCP Response Generation:** The Agent constructs a JSON-RPC 2.0 response message. On success, it includes the captured output within the result field (e.g., {"status": "completed", "results": \<Trivy JSON output\>}) and the original request id. On failure, it creates an error object with an appropriate code and message (e.g., {"code": \-32003, "message": "Trivy scan failed", "data": \<error details\>}) and the request id.18  
10. **Response Transmission:** The Agent sends the JSON-RPC response back to the Orchestrator over the transport (e.g., writes to stdout, sends an HTTP response).18  
11. **Response Handling:** The STCP Orchestrator receives the response, matches it to the original request using the id, and processes the outcome. Success results are logged and stored for later use (e.g., by \_\_besman\_prepare). Errors are logged, and may trigger playbook failure logic.  
12. **Iteration:** Steps 3-11 repeat for all subsequent steps in the playbook that involve STCP Agent interactions.  
13. **Finalization:** After \_\_besman\_execute completes, the \_\_besman\_prepare, \_\_besman\_publish, and \_\_besman\_cleanup functions run \[User Query\], potentially involving further STCP messages like agent/getFormattedResults (optional, during prepare) or agent/cleanup.

### **5.5. Interaction Flow Diagram Description (Simplified Playbook Execution)**


![STCP Interaction Flow Diagram](https://github.com/Be-Secure/STCP/blob/main/images/STCPInteractionFlow.png)
A diagram illustrating the typical STCP interaction flow during a playbook execution would show vertical lifelines for the key actors and sequential message exchanges:

* **Actors/Lifelines:** User, besman Utility, BeSPlaybook (showing internal calls to \_\_besman\_init, \_\_besman\_execute, \_\_besman\_prepare, \_\_besman\_publish, \_\_besman\_cleanup), STCP Orchestrator, STCP Agent (representing one tool), DID/VC Service, Datastore.  
* **Sequence:**  
  1. User sends "Launch Playbook" message to besman.  
  2. besman sends \_\_besman\_launch() message to BeSPlaybook.  
  3. BeSPlaybook internally calls \_\_besman\_init().  
  4. BeSPlaybook sends "Discover/Configure Agents" message to STCP Orchestrator.  
  5. STCP Orchestrator sends agent/discover and/or agent/configure messages to STCP Agent.  
  6. STCP Agent sends Response messages back to STCP Orchestrator.  
  7. BeSPlaybook internally calls \_\_besman\_execute().  
  8. BeSPlaybook sends "Execute Tool Action" message to STCP Orchestrator.  
  9. STCP Orchestrator sends agent/execute message (with capability, args) to STCP Agent.  
  10. STCP Agent (internally invokes underlying tool).  
  11. STCP Agent sends Response message (with results) back to STCP Orchestrator.  
  12. BeSPlaybook internally calls \_\_besman\_prepare() (processing results using BeS-Schema).  
  13. BeSPlaybook internally calls \_\_besman\_publish().  
  14. BeSPlaybook sends "Request OSAR Attestation VC" message to DID/VC Service.  
  15. DID/VC Service sends "OSAR Attestation VC" message back to BeSPlaybook.  
  16. BeSPlaybook sends "Store OSAR \+ VC" message to Datastore.  
  17. BeSPlaybook internally calls \_\_besman\_cleanup().  
  18. BeSPlaybook sends "Cleanup Agents" message to STCP Orchestrator.  
  19. STCP Orchestrator sends agent/cleanup message to STCP Agent.  
  20. STCP Agent sends Response message back to STCP Orchestrator.

This sequence diagram visually clarifies the step-by-step communication flow, highlighting how STCP messages facilitate tool execution within the playbook lifecycle and how DIDs/VCs are integrated for report attestation.

## **6\. Secure and Trustworthy Information Exchange via DIDs and VCs**

A core requirement for STCP is to facilitate the secure and trustworthy exchange of information, including configurations, commands, results, and OSARs, between BeSLabs, Agents, and community members. This is achieved by integrating Decentralized Identifiers (DIDs) and Verifiable Credentials (VCs) into the protocol's framework.

### **6.1. Role of DIDs for BeSLabs and Community Members**

DIDs provide the foundational layer of identity within the decentralized STCP ecosystem. Unlike traditional identifiers leased from authorities , DIDs are generated and controlled by the entity they represent (e.g., a person, organization, device, or even a software component).26 Each DID resolves to a DID Document, a standardized data structure containing cryptographic public keys and service endpoints associated with the DID subject, enabling authentication and secure interactions. Different DID methods define how DIDs are created, resolved, updated, and deactivated on various underlying systems (like blockchains or other distributed ledgers).8

Within the STCP context, DIDs are applied to key entities:

* **BeSLabs:** Each distinct BeSLab instance (whether operated by an organization or an individual researcher) SHOULD possess its own DID. This DID serves as the verifiable identifier for the lab, used as the issuer ID for environment attestations and OSAR reports generated within that lab.  
* **BeSLab Plugins (STCP Agents):** Individual STCP Agent implementations or even specific running instances MAY have their own DIDs. This allows for fine-grained, verifiable identification of the exact software component performing a security action, enhancing traceability. The tool vendor or the Be-Secure authority could issue these DIDs or associated VCs.  
* **BeSPlaybooks:** Authoritative versions of BeSPlaybook definitions can be associated with DIDs. This allows users to verify they are running an official, untampered version of a specific assessment workflow. The DID could resolve to metadata about the playbook, including its hash.28  
* **Community Members:** Individuals or organizations participating in the Be-Secure community (e.g., contributing playbooks, consuming OSARs, running BeSLabs) SHOULD use DIDs to represent themselves. These DIDs are used for authentication, access control based on verifiable attributes (via VCs), and signing contributions or reviews.

Employing DIDs for these core entities transforms the ecosystem. It enables the creation of a verifiable graph of interactions. For instance, an OSAR, when packaged as a VC, can cryptographically link back to the DID of the specific BeSLab where it was generated, the DID of the BeSPlaybook definition that was executed, and potentially even the DIDs of the individual STCP Agents (tools) whose results contributed to the report. This provides an unprecedented level of verifiable provenance and auditability for security assessments, significantly enhancing the trustworthiness of shared information compared to traditional methods that rely solely on implicit trust in the source or channel.

### **6.2. Role of VCs for Authentication, Authorization, and Data Attestation**

While DIDs provide the stable identifiers , Verifiable Credentials (VCs) provide the mechanism for making trustworthy, tamper-evident statements (claims) about these identifiers.3 VCs are digitally signed data structures containing claims issued by one entity (Issuer DID) about another (Subject DID or entity).4 They operate within a trust model involving an Issuer, a Holder (who possesses and presents the VC), and a Verifier (who checks the VC's validity).2 Cryptographic proofs embedded within the VC ensure its integrity and authenticity, allowing the Verifier to confirm who issued the claims and that they haven't been altered.2

STCP leverages VCs for several critical functions:

* **Agent Authentication & Capability Attestation:** An STCP Agent could be required to present a VC upon connection or before executing certain actions. This VC, likely issued by a trusted entity (e.g., Be-Secure governing body, tool vendor), would attest to the Agent's identity (its DID), its version, and potentially the specific STCP capabilities it is authorized to perform. The Orchestrator verifies this VC before trusting the Agent.  
* **Orchestrator Authorization:** Conversely, an Agent handling sensitive operations or data might require the requesting Orchestrator (representing a specific playbook run or user) to present a VC. This VC would prove the Orchestrator's authorization to invoke that capability, perhaps attesting to the user's role or the playbook's approved status.  
* **BeSEnvironment State Attestation:** As discussed in Section 3.3, the result of \_\_besman\_validate can be packaged into a VC. This BeSEnvironmentStateCredential provides verifiable proof of the environment's configuration and the status of its constituent Agents at a specific time.  
* **OSAR Attestation:** This is a primary use case. Generated OSARs are bundled with an OSARAttestationCredential. This VC contains metadata about the report (e.g., its hash, generator DIDs) and is signed by the issuing BeSLab's DID. It provides verifiable proof of the report's origin, integrity, and context (see Section 7).  
* **Community Access Control:** Access to shared resources like OSAR repositories or specific BeSLab functionalities can be governed using VCs. A user might need to present a VC proving their membership in a specific group or possession of certain permissions to gain access.

### **6.3. Key Verifiable Credential Structures**

To ensure interoperability, STCP defines recommended structures for key VCs based on the W3C VC Data Model.3 Each VC includes standard properties like @context, id (unique VC URI), type, issuer (Issuer's DID), issuanceDate, and proof.2 The differentiating part is the credentialSubject field, which contains the specific claims.4

**Table 6.3.1: Key STCP Verifiable Credential Schemas**

| VC Type Name | Description | Example credentialSubject Fields (Illustrative JSON) | Typical Issuer (DID Role) | Typical Holder (DID Role) |
| :---- | :---- | :---- | :---- | :---- |
| STCPAgentCapabilityCredential | Attests to an agent's identity, version, and the STCP capabilities it is authorized to expose. | { "id": "did:example:agent123", "agentName": "Trivy Agent", "agentVersion": "1.2.0", "toolName": "Trivy", "toolVersion": "0.45.1", "stcpVersion": "1.0", "authorizedCapabilities": \["scan\_container\_image", "scan\_filesystem"\] } | Tool Vendor / Be-Secure Authority | STCP Agent |
| BeSEnvironmentStateCredential | Attests to the validated configuration and status of a BeSEnvironment at a specific time. | { "id": "did:example:env\_xyz", "validationTimestamp": "2024-01-15T10:00:00Z", "blueprintId": "bes\_rt\_v1", "validatedAgents": \[ { "agentId": "did:example:agent123", "version": "1.2.0", "status": "ready", "configHash": "abc..." },... \] } | BeSEnvironment Instance / BeSLab Operator | BeSEnvironment / Orchestrator |
| OSARAttestationCredential | Attests to the authenticity, integrity, and generation context of an Open Source Assessment Report (OSAR). | { "id": "urn:uuid:...", "osarUri": "ipfs://...", "osarHash": "sha256-...", "reportFormat": "BeS-Schema-OSAR-v1.1", "assessmentTarget": { "type": "OSSPoI", "id": "pkg:github/org/repo@v1.2.3" }, "playbookDid": "did:example:playbook\_sast\_v2", "environmentDid": "did:example:env\_xyz", "generationTimestamp": "2024-01-15T12:30:00Z", "summary": { "critical": 1, "high": 5,... } } | BeSLab Operator / Playbook Runner | OSAR Recipient / Datastore |
| PlaybookExecutionCredential | Authorizes a specific playbook execution context (Orchestrator) to perform actions. | { "id": "did:example:orchestrator\_run987", "playbookDid": "did:example:playbook\_sast\_v2", "userDid": "did:example:user456", "allowedCapabilities": \["agent:did:example:agent123/scan\_container\_image"\], "executionId": "run-abc" } | Be-Secure Authority / BeSLab Operator | STCP Orchestrator |
| CommunityMembershipCredential | Attests that a DID holder is a member of a specific group within the Be-Secure community. | { "id": "did:example:user456", "communityGroup": "BeSecure-Contributors", "role": "Researcher", "memberSince": "2023-06-01" } | Be-Secure Community Admin | Community Member |

### **6.4. Protocol Mechanisms for Secure Exchange**

STCP needs mechanisms to handle the exchange and verification of these VCs during interactions. Two primary approaches are considered:

* **Approach 1: Embedded within STCP Messages:**  
  * Optional fields like vc\_presentation can be added to relevant STCP JSON-RPC request or response params/result objects (e.g., agent/configure, agent/execute).  
  * Specific STCP methods like agent/requestVC and agent/presentVC (see Table 2.5.1) can be defined for explicit VC exchange negotiation.  
  * **Pros:** Keeps interactions within the STCP protocol.  
  * **Cons:** Can bloat standard messages; may duplicate functionality available in dedicated VC exchange protocols.  
* **Approach 2: Leveraging External VC Exchange Protocols (Recommended for Complex Interactions):**  
  * Use established protocols like DIDComm 5 or OIDC for SIOP (Self-Issued OpenID Provider) with Verifiable Presentations for initial authentication, capability negotiation, or authorization *before* initiating core STCP agent/execute calls.  
  * STCP messages then proceed based on the established secure context.  
  * **Pros:** Leverages specialized, robust protocols for VC exchange; keeps STCP focused on tool orchestration.  
  * **Cons:** Requires integration with additional protocols and libraries.

Regardless of the exchange method, the **Verification Process** by the receiving party (Orchestrator or Agent) is critical and typically involves these steps:

1. **Parse the VC/VP:** Extract the claims and the proof.4  
2. **Resolve Issuer DID:** Use DID Resolution 28 to obtain the issuer's DID Document.  
3. **Retrieve Verification Method:** Extract the appropriate public key or verification method from the DID Document specified in the VC's proof.  
4. **Verify Proof:** Cryptographically verify the VC's signature or proof using the issuer's public key and the specified algorithm.2 This ensures data integrity and authenticity.  
5. **Check Status (if applicable):** If the VC includes a credentialStatus property, check the referenced status list (e.g., a revocation registry) to ensure the VC has not been revoked.2  
6. **Validate Claims:** Check timestamps (issuance, expiration) and evaluate the claims within the credentialSubject against the verifier's policy requirements (e.g., Is the capability authorized? Is the environment state acceptable?).4

## **7\. Open Source Assessment Report (OSAR) Specification and Sharing**

### **7.1. OSAR Structure and Content (Referencing BeS-Schema)**

The Open Source Assessment Report (OSAR) is the standardized artifact for communicating security assessment findings within the Be-Secure ecosystem. STCP facilitates the generation of data *for* the OSAR and the secure sharing *of* the OSAR, but the definitive structure and content requirements for the OSAR itself are governed by the **BeS-Schema** specification.

* **Mandate:** All OSARs generated as the output of BeSPlaybooks orchestrated via STCP MUST conform to the JSON schema defined in the official BeS-Schema repository ((([https://github.com/Be-Secure/bes-schema](https://github.com/Be-Secure/bes-schema)))).1 This ensures consistency and interoperability, allowing tools and platforms across the Be-Secure ecosystem to easily consume and process assessment results.1  
* **Key Sections:** Based on the BeS-Schema goals 1, an OSAR is expected to include structured information covering:  
  * **Assessment Metadata:** Information about the assessment itself (e.g., unique ID, start/end times, type of assessment).  
  * **Target Information:** Details of the open source software project or component assessed (e.g., name, version, repository URL, potentially referencing OSSPoI schema).  
  * **Execution Context:** Information about how the assessment was performed (e.g., reference to the BeSPlaybook DID, BeSEnvironment DID, tools/agents used).  
  * **Findings:** Detailed list of vulnerabilities, weaknesses, or other findings, potentially referencing OSSVoI schema. Each finding should include severity, description, location, evidence, and remediation advice.  
  * **Tool Outputs:** Raw or summarized outputs from the specific security tools (STCP Agents) used during the assessment.  
  * **Summary:** An executive summary of the key findings and overall security posture.

Adherence to the BeS-Schema is crucial for automated processing, data aggregation, and cross-lab information sharing.1

### **7.2. OSAR Generation within BeSPlaybook Lifecycle**

The BeSPlaybook lifecycle provides a structured process for generating the OSAR, integrating STCP outputs seamlessly:

1. **Data Collection (\_\_besman\_execute):** As the playbook executes steps via STCP agent/execute calls \[User Query\], the STCP Orchestrator receives results from various Agents. These results contain the raw findings and data generated by the security tools.  
2. **Preparation & Formatting (\_\_besman\_prepare):** This function takes the aggregated raw results collected during execution \[User Query\]. Its primary responsibility is to parse this data and map it precisely into the fields and structure defined by the OSAR schema in BeS-Schema.1 This might involve transforming different tool output formats into the common OSAR structure, correlating findings, and calculating summary statistics.  
3. **Finalization & Attestation (\_\_besman\_publish):** Once the OSAR JSON object is fully populated according to the schema, the \_\_besman\_publish function takes over . It coordinates the creation of the OSARAttestationCredential (see Table 6.3.1), which includes hashing the final OSAR JSON to ensure integrity. The OSAR and its VC are then bundled and stored or transmitted.

### **7.3. Secure Sharing of OSAR via STCP (DID/VC based)**

Sharing OSARs securely and trustworthily between different BeSLabs, organizations, or individual community members is a key goal facilitated by STCP's integration with DIDs and VCs.

* **Attestation:** The cornerstone of secure sharing is the OSARAttestationCredential generated during \_\_besman\_publish. This VC acts as a digital, verifiable "cover letter" for the OSAR. Signed by the generating BeSLab's DID , it makes verifiable claims about the OSAR, such as:  
  * The cryptographic hash of the OSAR content, guaranteeing integrity.  
  * The DID of the BeSPlaybook used, providing context on the methodology.  
  * The DID of the BeSEnvironment used, providing context on the execution environment.  
  * The timestamp of generation.  
  * Optionally, a summary of findings.  
* **Packaging:** The OSAR (the JSON data conforming to BeS-Schema 1) is packaged together with its corresponding OSARAttestationCredential (the VC). This bundle represents the complete, verifiable assessment artifact.  
* **Verification upon Receipt:** When a recipient receives this OSAR bundle, they can first verify the OSARAttestationCredential using the standard VC verification process (Section 6.4). This allows them to:  
  * Confirm the identity of the issuing BeSLab (via DID resolution 28).  
  * Verify the integrity of the OSAR data by recalculating the hash of the received OSAR JSON and comparing it to the osarHash claim in the VC.  
  * Trust the context provided (which playbook/environment was used). Only after successful verification of the VC should the recipient proceed to parse and utilize the OSAR content itself.  
* **Access Control:** Sharing can be further secured using VCs for access control. The sender might encrypt the OSAR bundle for specific recipient DIDs, or the repository storing OSARs might require requesters to present a VC (e.g., CommunityMembershipCredential) proving their authorization to access certain reports.

This DID/VC-based approach elevates OSAR sharing beyond simple file transfer. It aligns with the broader movement towards verifiable data ecosystems 5, where data artifacts carry intrinsic, cryptographic proof of their origin, integrity, and context. This is essential for building trust and enabling reliable collaboration in distributed security assessment efforts, directly fulfilling the requirement for secure and trustworthy exchange between BeSLabs and community members.

## **8\. STCP Summary and Framework Overview**

### **8.1. Synthesized View of STCP Framework**

The Security Tool Context Protocol (STCP) provides a vital standardization layer for the Be-Secure ecosystem. It addresses the challenges of integrating and orchestrating a diverse set of security tools by defining a common interface and interaction pattern, acting as a universal adapter for security tools within BeSLabs.20

At its core, STCP employs a client-server architecture where STCP Orchestrators (typically embedded within BeSPlaybook execution logic) use JSON-RPC 2.0 20 over defined transports (like stdio or HTTP) 18 to discover, configure, execute, and manage STCP Agents.18 These Agents are BeSLab Plugins that wrap individual security tools, exposing their functionalities as standardized STCP capabilities.20

STCP is deeply integrated with the existing Be-Secure lifecycle management functions. It enhances BeSEnvironment management (\_\_besman\_install, \_\_besman\_validate, etc.) by providing protocol-level mechanisms for tool registration, validation, and configuration . Crucially, it enables the \_\_besman\_validate function to produce a verifiable attestation (VC) of the environment's state. Similarly, STCP is the engine driving tool execution within the BeSPlaybook lifecycle (\_\_besman\_init, \_\_besman\_execute, etc.), facilitating the collection of raw data needed for generating standardized Open Source Assessment Reports (OSARs).

A key differentiator and strength of STCP is its mandated use of Decentralized Identifiers (DIDs) and Verifiable Credentials (VCs).3 DIDs provide stable, verifiable identities for all key actors and artifacts (BeSLabs, Agents, Playbooks, Users, Environments, Reports). VCs leverage these DIDs to enable secure authentication, fine-grained authorization, and cryptographic attestation of data, most notably for ensuring the integrity and provenance of OSARs. The structure of OSARs themselves is strictly governed by the BeS-Schema specification 1, ensuring consistency across the ecosystem.

In essence, STCP provides the standardized communication pathways, while DIDs/VCs provide the trust infrastructure, and BeS-Schema provides the data format standard for assessment results, creating a cohesive and robust framework for collaborative open source security assessment.

### **8.2. Benefits and Future Considerations**

The adoption of STCP within the Be-Secure ecosystem offers numerous advantages:

* **Standardization:** Replaces ad-hoc integrations with a common protocol, simplifying development and maintenance.18  
* **Interoperability:** Enables any STCP-compliant tool (Agent) to work with any STCP-compliant orchestrator (Playbook).20  
* **Reduced Integration Effort:** Significantly lowers the barrier to onboarding new security tools into BeSLabs.18  
* **Scalability:** Facilitates the management of larger, more complex BeSLab deployments with diverse tooling.20  
* **Enhanced Trust and Auditability:** The mandatory use of DIDs and VCs provides verifiable identity and cryptographic proof for actions and data artifacts (like environment state and OSARs).2  
* **Verifiable Reporting:** Ensures OSARs are generated in a standard format (BeS-Schema 1) and can be shared with verifiable proof of origin and integrity.

While STCP version 1.0 provides a strong foundation, several areas warrant future consideration and development:

* **Advanced Agent Capabilities:** Defining support for asynchronous operations, long-polling, or more complex workflow patterns within agents.  
* **Sophisticated Discovery:** Exploring and standardizing more advanced, potentially decentralized discovery mechanisms beyond the baseline local registry.  
* **Transport Binding Specifications:** Formally specifying bindings for additional transports like gRPC or WebSockets.  
* **STCP SDKs:** Developing Software Development Kits (SDKs) in key languages (e.g., Python, Go, Java, TypeScript) to accelerate the development of both Agents and Orchestrator logic.21  
* **VC Profile Governance:** Establishing clear governance and detailed profiles for the VCs used within the Be-Secure STCP ecosystem, including credential revocation mechanisms.2  
* **External Integrations:** Defining standard STCP capabilities or patterns for integrating BeSLabs with external systems like threat intelligence platforms, ticketing systems, or code repositories.  
* **Security Hardening:** Continuous review and enhancement of the security considerations for STCP, particularly around transport security, authentication, and authorization patterns using VCs.18

#### **Works cited**

1. Be-Secure/bes-schema: This repository defines the data ... \- GitHub, accessed April 27, 2025, [https://github.com/Be-Secure/bes-schema](https://github.com/Be-Secure/bes-schema)  
2. Verifiable credentials \- Wikipedia, accessed April 27, 2025, [https://en.wikipedia.org/wiki/Verifiable\_credentials](https://en.wikipedia.org/wiki/Verifiable_credentials)  
3. Decentralized Identifiers (DIDs) v1.0 \- W3C, accessed April 27, 2025, [https://www.w3.org/TR/2021/WD-did-core-20210118/](https://www.w3.org/TR/2021/WD-did-core-20210118/)  
4. Verifiable Credentials Data Model v2.0 \- W3C, accessed April 27, 2025, [https://www.w3.org/TR/vc-data-model-2.0/](https://www.w3.org/TR/vc-data-model-2.0/)  
5. Verifiable Credentials Data Model v1.1 \- W3C, accessed April 27, 2025, [https://www.w3.org/TR/vc-data-model/](https://www.w3.org/TR/vc-data-model/)  
6. Verifiable Credentials: The Ultimate Guide 2025 \- Dock Labs, accessed April 27, 2025, [https://www.dock.io/post/verifiable-credentials](https://www.dock.io/post/verifiable-credentials)  
7. DIF & ToIP on Decentralized Identifiers (DIDs) v1.0 as W3C Standard, accessed April 27, 2025, [https://blog.identity.foundation/w3cdidspec/](https://blog.identity.foundation/w3cdidspec/)  
8. A Primer for Decentralized Identifiers \- W3C Credentials Community Group, accessed April 27, 2025, [https://w3c-ccg.github.io/did-primer/](https://w3c-ccg.github.io/did-primer/)  
9. Verifiable Credentials Explained | Curity Identity Server, accessed April 27, 2025, [https://curity.io/resources/learn/verifiable-credentials/](https://curity.io/resources/learn/verifiable-credentials/)  
10. EBSI W3C VCs and VPs, accessed April 27, 2025, [https://hub.ebsi.eu/vc-framework/ebsi-w3c-vc-vp](https://hub.ebsi.eu/vc-framework/ebsi-w3c-vc-vp)  
11. Model Context Protocol \- GitHub, accessed April 27, 2025, [https://github.com/modelcontextprotocol](https://github.com/modelcontextprotocol)  
12. w3c/did: W3C Decentralized Identifier Specification \- GitHub, accessed April 27, 2025, [https://github.com/w3c/did](https://github.com/w3c/did)  
13. Decentralized Identifiers (DIDs) \- W3C | Verifiable Credentials and Self Sovereign Identity Web Directory, accessed April 27, 2025, [https://decentralized-id.com/web-standards/w3c/decentralized-identifier/](https://decentralized-id.com/web-standards/w3c/decentralized-identifier/)  
14. w3c/vc-data-model-2.0-test-suite: W3C Verifiable Credentials v2.0 test suite \- GitHub, accessed April 27, 2025, [https://github.com/w3c/vc-data-model-2.0-test-suite](https://github.com/w3c/vc-data-model-2.0-test-suite)  
15. w3c/vc-data-model: W3C Verifiable Credentials v2.0 Specification \- GitHub, accessed April 27, 2025, [https://github.com/w3c/vc-data-model](https://github.com/w3c/vc-data-model)  
16. Verifiable Credentials Overview \- W3C, accessed April 27, 2025, [https://www.w3.org/TR/vc-overview/](https://www.w3.org/TR/vc-overview/)  
17. Verifiable Credentials Data Model \- 1Kosmos Product Documentation, accessed April 27, 2025, [https://docs.1kosmos.com/productdocs/docs/verifiable-credentials/data-model/](https://docs.1kosmos.com/productdocs/docs/verifiable-credentials/data-model/)  
18. Model Context Protocol: The USB-C for AI: Simplifying LLM Integration, accessed April 27, 2025, [https://www.infracloud.io/blogs/model-context-protocol-simplifying-llm-integration/](https://www.infracloud.io/blogs/model-context-protocol-simplifying-llm-integration/)  
19. Core architecture \- Model Context Protocol, accessed April 27, 2025, [https://modelcontextprotocol.io/docs/concepts/architecture](https://modelcontextprotocol.io/docs/concepts/architecture)  
20. Model Context Protocol (MCP): A comprehensive introduction for developers \- Stytch, accessed April 27, 2025, [https://stytch.com/blog/model-context-protocol-introduction/](https://stytch.com/blog/model-context-protocol-introduction/)  
21. Introducing the Model Context Protocol \- Anthropic, accessed April 27, 2025, [https://www.anthropic.com/news/model-context-protocol](https://www.anthropic.com/news/model-context-protocol)  
22. Model Context Protocol (MCP) Explained \- Humanloop, accessed April 27, 2025, [https://humanloop.com/blog/mcp](https://humanloop.com/blog/mcp)  
23. Model Context Protocol (MCP) \- Anthropic API, accessed April 27, 2025, [https://docs.anthropic.com/en/docs/agents-and-tools/mcp](https://docs.anthropic.com/en/docs/agents-and-tools/mcp)  
24. Model Context Protocol: Introduction, accessed April 27, 2025, [https://modelcontextprotocol.io/introduction](https://modelcontextprotocol.io/introduction)  
25. The Model Context Protocol (MCP): A Guide for AI Integration | Generative-AI \- Wandb, accessed April 27, 2025, [https://wandb.ai/byyoung3/Generative-AI/reports/The-Model-Context-Protocol-MCP-A-Guide-for-AI-Integration--VmlldzoxMTgzNDgxOQ](https://wandb.ai/byyoung3/Generative-AI/reports/The-Model-Context-Protocol-MCP-A-Guide-for-AI-Integration--VmlldzoxMTgzNDgxOQ)  
26. Decentralized Identifiers (DIDs) v1.0 \- W3C, accessed April 27, 2025, [https://www.w3.org/TR/did-1.0/](https://www.w3.org/TR/did-1.0/)  
27. Decentralized Identifiers (DIDs) v1.1 \- W3C, accessed April 27, 2025, [https://www.w3.org/TR/did-1.1/](https://www.w3.org/TR/did-1.1/)  
28. Decentralized Identifier Resolution (DID Resolution) v0.3 \- W3C on GitHub, accessed April 27, 2025, [https://w3c.github.io/did-resolution/](https://w3c.github.io/did-resolution/)  
29. Decentralized Identifier Working Group Charter \- W3C, accessed April 27, 2025, [https://www.w3.org/2020/12/did-wg-charter.html](https://www.w3.org/2020/12/did-wg-charter.html)  
30. Decentralized Identifier Extensions \- W3C, accessed April 27, 2025, [https://www.w3.org/TR/did-extensions/](https://www.w3.org/TR/did-extensions/)
