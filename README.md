# **Software Architecture Guidelines**

## **1. Introduction & Core Philosophy**

The objective of this document is to define the core principles governing our embedded software development process. These guidelines are designed to reduce complexity, mitigate risk, and ensure that our software is robust, secure, portable, and maintainable over long product lifecycles.

**The Golden Rule:** We prioritise **system maintainability and clarity** over micro-optimisations, unless a specific performance bottleneck is proven via profiling.

## **2. Architectural Structure**

*The structural foundation of our code.*

### **2.1. Layering and Abstraction (HAL & OSAL)**

**Principle:** Application logic must be independent of hardware details and the underlying Real-Time Operating System (RTOS).

**Rationale:** Decoupling business logic from platform specifics reduces risk, improves testability, and enables long-term portability.

**Implementation Guidance:**

* **Hardware Abstraction Layer (HAL):** Application code must never access hardware registers or vendor-specific driver functions (e.g., SDKs) directly. All hardware interaction must go through a defined HAL.
* **Operating System Abstraction (OSAL):** Application code must not directly call specific RTOS functions (e.g., `xTaskCreate`, `osDelay`). Instead, use a generic abstraction layer or framework wrapper.
* **Portability Target:** With a compliant HAL/OSAL, the application shall be able to run on different microcontrollers, different operating systems, or inside a PC simulator (SIL) without changes to business logic.

### **2.2. Components and Modularity**

**Principle:** The system shall be composed of independent components with high cohesion and low coupling.

**Rationale:** Strong modularity limits the blast radius of changes, improves testability, and enables parallel development.

**Implementation Guidance:**

* **Encapsulation:** Modules must hide their internal state. Interaction with a module happens *only* through its public interface (API).
* **Interfaces First:** Design and review interfaces before implementation. This enables high-level design discussions without implementation bias.
* **Failure Contracts:** Interfaces must explicitly define potential error cases and failure modes, not just successful operations.
* **Testability:** Each component shall be testable in isolation using mocks or stubs for its dependencies.
* **Component Deactivation:** It must be possible to disable any non-critical functional component (at build-time or runtime) while the rest of the system remains operational, simply without that functionality.

### **2.3. Frameworks**

**Principle:** Application code shall focus on domain logic, not on infrastructure concerns such as scheduling, message routing, or state orchestration.

**Rationale:** Centralising infrastructure logic reduces duplication, enforces consistency, and allows complexity to be handled by well-reviewed core mechanisms.

**Implementation Guidance:**

* **Framework Usage:** Use frameworks to standardise common infrastructure concerns (e.g., message passing, state machines, task scheduling).
* **Inversion of Control:** Frameworks call application code; application code shall not implement its own main control loops.
* **Interface Simplicity:** Framework internals may be complex, but exposed interfaces must remain simple, explicit, and predictable for application developers.

### **2.4. Maintainability Over Premature Optimisation**

**Principle:** Code clarity and maintainability take precedence over micro-optimisations.

**Rationale:** Embedded systems have constraints, but unreadable or overly clever code creates long-term technical debt and increases defect risk.

**Implementation Guidance:**

* **Optimisation Policy:** Write clear, logical code first. Optimise for speed or memory only when measured performance data shows requirements are not met.
* **Clean Code Practices:**

  * **Small Functions:** Decompose complex logic into small, focused functions with descriptive names.
  * **OOP Techniques:** Apply Object-Oriented Programming concepts (e.g., encapsulation, polymorphism) to structure data and behaviour. In C, this may be achieved using opaque structs, function pointers, or similar patterns to manage complexity and state.

## **3. The Development Ecosystem**

*The tools and processes that enable speed and stability.*

### **3.1. Simulation-First Approach (Software-in-the-Loop)**

**Principle:** Application logic shall not depend on physical target hardware for correct behaviour or verification.

**Rationale:** Early and deterministic verification reduces debugging time, removes hardware bottlenecks, and enables development before hardware availability.

**Implementation Guidance:**

* **Host Execution:** Developers must be able to compile and run application logic on a host computer (Windows/Linux).
* **Time Abstraction & Control:** The simulation environment shall provide full control over system time, including pause, single-step, fast-forward, and precise event scheduling independent of wall-clock time.
* **Sync/Async Separation:** Separate synchronous business logic from asynchronous events (e.g., interrupts, timers). Use defined interfaces (e.g., Asynchronous Interface Handlers – AIH) to bridge the two.
* **Determinism Target:** Core logic shall be deterministic under simulation, with all asynchronous behaviour explicitly triggered by the test or simulation environment.

### **3.2. Continuous Quality Verification**

**Principle:** Software quality shall be continuously verified throughout development, not assessed only at the end.

**Rationale:** Early detection of defects reduces rework cost and prevents defect accumulation.

**Implementation Guidance:**

* **Static Analysis:** All code must pass static analysis tools (e.g., linting, MISRA checks) before integration.
* **Automated Testing:** Continuous Integration (CI) pipelines shall execute unit tests and regression tests on every commit.

### **3.3. Iterative Development**

**Principle:** Development shall proceed in small, incremental iterations rather than large, monolithic releases.

**Rationale:** Iterative development limits risk exposure and enables early validation of assumptions.

**Implementation Guidance:**

* **Risk-Driven Planning:** Prioritise high-risk elements (e.g., complex algorithms, new communication buses) in early iterations to enable fast failure and early recovery.

### **3.4. Manage Testable Requirements (Traceability)**

**Principle:** Every implemented feature shall be traceable to a documented and testable requirement.

**Rationale:** Traceability ensures completeness, supports certification activities, and prevents undocumented behaviour from entering the system.

**Implementation Guidance:**

* **V-Model Alignment:** Maintain traceability from Requirements → Design → Code → Test.

* **Testability Rule:** Requirements that cannot be verified (automatically or manually) shall not be implemented.

* **The V-Model:** We maintain traceability. Requirements → Design → Code → Test.

* **Testability:** If a requirement cannot be tested (either automatically or manually), it is not a valid requirement.

## **4. Team Dynamics & Knowledge Management**

*Ensuring the team operates as a single unit.*

### **4.1. Consistency**

**Principle:** Similar problems shall be solved using consistent architectural and coding approaches.

**Rationale:** Consistency reduces cognitive load, accelerates onboarding, and simplifies long-term maintenance.

**Implementation Guidance:**

* **Design Patterns:** Apply established and agreed-upon design patterns.
* **Uniform Structure:** Module structure, naming conventions, and error handling shall follow common conventions across the codebase.

### **4.2. Knowledge Dissemination**

**Principle:** No critical part of the system shall depend on the knowledge of a single individual.

**Rationale:** Shared understanding increases resilience against personnel changes and reduces operational risk.

**Implementation Guidance:**

* **Peer Programming:** Complex or unique functionality shall involve at least two developers.
* **Code Reviews:** All changes must be reviewed before merging into the main branch, with reviews focusing on correctness, clarity, and knowledge sharing.
* **Visualisation:** Use diagrams (e.g., UML, flowcharts) to document behaviour and support architectural discussions.
* **Documentation Strategy:** Documentation shall reside within the code repository and capture design decisions and architectural rationale that cannot be inferred from code alone.

## **5. Robustness, Safety & Security**

*Ensuring the system is resilient.*

### **5.1. Unified Error Handling Strategy**

**Principle:** The system shall handle errors in a consistent, predictable, and recoverable manner.

**Rationale:** A unified error strategy prevents undefined behaviour and simplifies fault analysis and recovery.

**Implementation Guidance:**

* **Error Reporting Rules:** Define and apply system-wide conventions for error reporting (e.g., return codes, status objects, exceptions).
* **Recovery Strategy:** Define a safe state and recovery mechanisms (e.g., watchdog timers) for critical failures.
* **DFMEA Documentation:** Explicitly document failure modes and residual risks to support Design Failure Mode and Effects Analysis (DFMEA).

### **5.2. Security by Design**

**Principle:** Security considerations shall be integrated into the system architecture from the outset.

**Rationale:** Retrofitting security is costly and error-prone; early integration reduces attack surface and systemic risk.

**Implementation Guidance:**

* **Input Validation:** Validate all external inputs (e.g., sensors, communication interfaces) against expected ranges and types.
* **Least Privilege:** Tasks and modules shall only access the memory regions and peripherals they require.
* **Credential Management:** Secrets (passwords, keys, URLs) shall not be hardcoded; use secure storage or provisioning mechanisms.

### **5.3. Configuration Management**

**Principle:** The software shall support multiple hardware variants and deployments without code duplication.

**Rationale:** Centralised configuration reduces maintenance overhead and prevents variant-specific code forks.

**Implementation Guidance:**

* **Build-Time Configuration:** Use configuration files or preprocessor definitions to enable or disable features while maintaining a unified codebase.
* **Runtime Configuration:** Store configurable parameters (e.g., calibration data, feature flags, network settings) in non-volatile memory to allow adaptation without recompilation.
