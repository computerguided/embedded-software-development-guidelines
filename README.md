# **Embedded Software Architecture Guidelines**

*A unified set of principles for building robust, maintainable, and long-lived embedded software systems.*

---

## **1. Introduction & Core Philosophy**

**The Objective**\
The objective of this document is to define the core architectural principles governing our embedded software development. These guidelines aim to reduce complexity, mitigate technical and organisational risk, and ensure that our software remains **robust, secure, portable, and maintainable** throughout long product lifecycles.

These principles apply to all embedded software developed within the organisation, independent of target platform, operating system, or project size.

**The Golden Rule**\
We prioritise **system maintainability and clarity** over micro-optimisations, unless a concrete performance bottleneck is demonstrated through measurement and profiling.

---

## **2. Architectural Structure**

*The structural foundation of the software system.*

### **2.1. Layering and Abstraction (HAL & OSAL)**

**Principle**\
Application logic shall be independent of hardware details and the underlying Real-Time Operating System (RTOS).

**Rationale**\
Decoupling business logic from platform-specific concerns reduces risk, improves testability, and enables long-term portability across hardware platforms, operating systems, and simulation environments.

**Implementation Guidance**

- **Hardware Abstraction Layer (HAL):** Application code shall never access hardware registers or vendor-specific driver functions (e.g., SDKs) directly. All hardware interaction must occur through a defined HAL.
- **Operating System Abstraction Layer (OSAL):** Application code shall not directly call RTOS-specific APIs (e.g., `xTaskCreate`, `osDelay`). Instead, use a generic abstraction layer or approved framework wrapper.
- **Portability Target:** With a compliant HAL and OSAL, application logic shall run unchanged on different microcontrollers, different operating systems, or within a PC-based Software-in-the-Loop (SIL) environment.

---

### **2.2. Components and Modularity**

**Principle**\
The system shall be composed of independent components with high cohesion and low coupling.

**Rationale**\
Strong modularity limits the impact of changes, enables parallel development, improves testability, and reduces the likelihood of cascading failures.

**Implementation Guidance**

- **Encapsulation:** Components must hide internal state. Interaction with a component shall occur *only* via its public interface (API).
- **Interfaces First:** Interfaces shall be designed and reviewed before implementation to support architectural discussion without implementation bias.
- **Failure Contracts:** Interfaces must explicitly document expected error conditions and failure modes, not only successful behaviour.
- **Testability:** Each component shall be testable in isolation using mocks or stubs for its dependencies.
- **Component Deactivation:** Any non-critical functional component must be disableable (at build-time or runtime) without preventing the remainder of the system from operating.

---

### **2.3. Frameworks**

**Principle**\
Application code shall focus on domain-specific logic and avoid direct responsibility for infrastructure concerns.

**Rationale**\
Centralising infrastructure logic improves consistency, reduces duplication, and ensures that complex control mechanisms are implemented and maintained by experienced engineers.

**Implementation Guidance**

- **Framework Usage:** Use approved frameworks to standardise infrastructure concerns such as message passing, state machines, and task scheduling.
- **Inversion of Control:** Frameworks invoke application code; application code shall not implement its own main control loops.
- **Interface Simplicity:** Framework internals may be complex, but exposed interfaces must remain simple, explicit, and predictable.

---

### **2.4. Maintainability Over Premature Optimisation**

**Principle**\
Code clarity and long-term maintainability shall take precedence over premature optimisation.

**Rationale**\
While embedded systems operate under resource constraints, overly clever or opaque code increases defect risk and creates long-term technical debt.

**Implementation Guidance**

- **Optimisation Policy:** Write clear, readable code first. Optimise for speed or memory only when measured data demonstrates that requirements are not met.
- **Clean Code Practices:**
  - **Small Functions:** Decompose complex behaviour into small, focused functions with descriptive names.
  - **Object-Oriented Techniques:** Apply encapsulation and polymorphism to structure behaviour and state. In C, this may be achieved using opaque structs, function pointers, or similar patterns.

---

## **3. The Development Ecosystem**

*The tools and processes that enable speed, quality, and stability.*

### **3.1. Simulation-First Approach (Software-in-the-Loop)**

**Principle**\
Application logic shall not depend on physical target hardware for correct behaviour or verification.

**Rationale**\
Deterministic, early verification dramatically reduces debugging time, removes hardware bottlenecks, and enables development before hardware availability.

**Implementation Guidance**

- **Host Execution:** Application logic must be buildable and executable on a host computer (Windows/Linux).
- **Time Abstraction and Control:** The simulation environment shall provide full control over system time, including pause, single-step execution, fast-forwarding, and precise event scheduling independent of wall-clock time.
- **Synchronous / Asynchronous Separation:** Separate synchronous business logic from asynchronous events (e.g., interrupts, timers). Bridge the two using explicit interfaces such as Asynchronous Interface Handlers (AIH).
- **Determinism Target:** Core logic shall behave deterministically under simulation, with all asynchronous behaviour explicitly triggered by tests or the simulation environment.

---

### **3.2. Continuous Quality Verification**

**Principle**\
Software quality shall be continuously verified throughout development.

**Rationale**\
Early detection of defects reduces rework cost and prevents the accumulation of latent faults.

**Implementation Guidance**

- **Static Analysis:** All code must pass configured static analysis tools (e.g., linting, MISRA compliance checks) prior to integration.
- **Automated Testing:** Continuous Integration (CI) pipelines shall execute unit tests and regression tests on every commit.

---

### **3.3. Iterative Development**

**Principle**\
Development shall proceed in small, incremental iterations rather than large, monolithic releases.

**Rationale**\
Iterative development limits risk exposure and enables early validation of assumptions and design decisions.

**Implementation Guidance**

- **Risk-Driven Planning:** High-risk elements (e.g., complex algorithms, new communication technologies) shall be addressed early to enable fast failure and recovery.
- **Vertical Slices:** Prefer end-to-end increments that deliver a thin, usable feature path (even if limited), rather than completing one subsystem in isolation.
- **Timeboxed Spikes:** Use short, explicitly timeboxed experiments to de-risk unknowns. Capture outcomes and decisions in the repository.
- **Definition of Done:** Each iteration shall produce an increment that is integrated, buildable, and testable, with updated documentation where relevant.
- **Integration Cadence:** Integrate to the mainline frequently (e.g. daily) to avoid long-lived branches and difficult merges.
- **Feature Flags and Staging:** Use feature flags (compile-time or runtime) to ship incomplete or experimental functionality safely without destabilising releases.
- **Incremental Interfaces:** Stabilise interfaces early; allow implementations to evolve behind the interface across iterations.
- **Demonstrate and Review:** Conclude iterations with a demonstrable increment and a brief review of lessons learned (including follow-up actions).

---

### **3.4. Testable Requirements and Traceability**

**Principle**\
Every implemented feature shall be traceable to a documented and testable requirement.

**Rationale**\
Traceability ensures completeness, supports certification activities, and prevents undocumented behaviour from entering the system.

**Implementation Guidance**

- **V-Model Alignment:** Maintain traceability from Requirements → Design → Code → Test.
- **Testability Rule:** Requirements that cannot be verified, either automatically or manually, shall not be implemented.

---

## **4. Team Dynamics & Knowledge Management**

*Ensuring the team operates as a cohesive and resilient unit.*

### **4.1. Consistency**

**Principle**\
Similar problems shall be solved using consistent architectural and coding approaches.

**Rationale**\
Consistency reduces cognitive load, accelerates onboarding, and simplifies cross-team maintenance.

**Implementation Guidance**

- **Design Patterns:** Apply established and agreed-upon design patterns.
- **Uniform Structure:** Module structure, naming conventions, and error-handling approaches shall be consistent across the codebase.

---

### **4.2. Knowledge Dissemination**

**Principle**\
No critical part of the system shall depend on the knowledge of a single individual.

**Rationale**\
Shared understanding increases organisational resilience and reduces operational risk.

**Implementation Guidance**

- **Peer Programming:** Complex or unique functionality shall involve at least two developers.
- **Code Reviews:** All changes must be reviewed before merging, with reviews focusing on correctness, clarity, and knowledge sharing.
- **Visualisation:** Use diagrams (e.g., UML, flowcharts) to document behaviour and support architectural discussions.
- **Documentation Strategy:** Documentation shall reside within the code repository and capture design decisions and architectural rationale that cannot be inferred from code alone.

---

## **5. Robustness, Safety & Security**

*Ensuring the system is resilient under normal and fault conditions.*

### **5.1. Unified Error Handling Strategy**

**Principle**\
The system shall handle errors in a consistent, predictable, and recoverable manner.

**Rationale**\
A unified error-handling strategy prevents undefined behaviour and simplifies fault diagnosis and recovery.

**Implementation Guidance**

- **Error Reporting Rules:** Define and apply system-wide conventions for error reporting (e.g., return codes, status objects, or exceptions).
- **Recovery Strategy:** Define safe states and recovery mechanisms (e.g., watchdog timers) for critical failures.
- **DFMEA Documentation:** Explicitly document failure modes and residual risks to support Design Failure Mode and Effects Analysis (DFMEA).

---

### **5.2. Security by Design**

**Principle**\
Security considerations shall be integrated into the system architecture from the outset.

**Rationale**\
Retrofitting security is costly and error-prone; early integration reduces attack surface and systemic risk.

**Implementation Guidance**

- **Input Validation:** Validate all external inputs (e.g., sensors, communication interfaces) against expected ranges and types.
- **Principle of Least Privilege:** Tasks and modules shall access only the memory regions and peripherals strictly required.
- **Credential Management:** Secrets such as passwords, cryptographic keys, and URLs shall not be hardcoded. Use secure storage or controlled provisioning mechanisms.

---

### **5.3. Configuration Management**

**Principle**\
The software shall support multiple hardware variants and deployments without code duplication.

**Rationale**\
Centralised configuration reduces maintenance overhead and prevents the proliferation of variant-specific code.

**Implementation Guidance**

- **Build-Time Configuration:** Use configuration files or preprocessor definitions to enable or disable features while maintaining a unified codebase.
- **Runtime Configuration:** Store configurable parameters (e.g., calibration data, feature flags, network settings) in non-volatile memory to allow adaptation without recompilation.

