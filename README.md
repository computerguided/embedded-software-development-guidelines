# **Software Architecture Guidelines**

## **1\. Introduction & Core Philosophy**

The objective of this document is to define the core principles governing our embedded software development process. These guidelines are designed to reduce complexity, mitigate risk, and ensure that our software is robust, secure, portable, and maintainable over long product lifecycles.

**The Golden Rule:** We prioritise **system maintainability and clarity** over micro-optimisations, unless a specific performance bottleneck is proven via profiling.

## **2\. Architectural Structure**

*The structural foundation of our code.*

### **2.1. Layering and Abstraction (HAL & OSAL)**

Software must be strictly layered to decouple the application logic from the underlying hardware and Real-Time Operating System (RTOS).

* **Hardware Abstraction Layer (HAL):** Application code must never access hardware registers or vendor-specific driver functions (e.g., SDKs) directly. It must go through a defined HAL.  
* **Operating System Abstraction (OSAL):** Application code must not directly call specific RTOS functions (e.g., xTaskCreate or osDelay). Instead, use a generic abstraction layer or framework wrapper.  
* **Goal:** This allows the application to be portable. It enables the software to run on different microcontrollers, different operating systems, or inside a PC simulator (SIL) with zero changes to the business logic.

### **2.2. Components and Modularity**

Functionality must be divided into independent components with high cohesion (related logic stays together) and low coupling (minimal dependencies between modules).

* **Encapsulation:** Modules must hide their internal state. Interaction with a module happens *only* through its public interface (API).  
* **Interfaces:** Design the interface first. This allows high-level conceptual discussions without getting bogged down in implementation details.  
* **Failure Contracts:** Interfaces must explicitly define potential error cases and failure modes, not just successful operations. A developer using a component must know exactly what failures to expect to ensure robust integration.  
* **Testability:** High modularity ensures that components can be tested in isolation (Unit Testing). This allows developers to verify complex logic thoroughly using mocks or stubs for dependencies, without requiring the full system context or target hardware.  
* **Component Deactivation:** It must be possible to switch off any separate functional part (at build-time or runtime) and the system should still operate, simply lacking that specific functionality. This ensures strict decoupling and prevents dependencies where disabling one non-critical module causes the entire system to fail.

### **2.3. Frameworks**

We utilise frameworks to standardise the "plumbing" of the embedded system (message passing, state machines, task scheduling).

* **Inversion of Control:** Frameworks call our code; we do not write the main loops ourselves.  
* **Rationale:** While framework internals may be complex (written by senior engineers), they must provide simple, logical interfaces for application developers. This speeds up feature development and ensures consistent behaviour across the system.

### **2.4. Maintainability Over Premature Optimisation**

Code readability is paramount. Embedded systems have constraints, but "clever" code that saves a few bytes at the cost of readability is a technical debt.

* **Guideline:** Write clear, logical code first. Only optimize for speed or memory size if verifying performance metrics indicate a failure to meet requirements.  
* **Clean Code Practices:**  
  * **Small Functions:** Break down complex logic into small, focused functions with descriptive names. This improves readability and testability compared to monolithic "spaghetti" functions.  
  * **OOP Techniques:** Apply Object-Oriented Programming concepts (Encapsulation, Polymorphism) to structure data and behaviour logically. This applies even when using C (e.g., using structs with function pointers or opaque pointers) to manage complexity and state effectively.

## **3\. The Development Ecosystem**

*The tools and processes that enable speed and stability.*

### **3.1. Simulation-First Approach (Software-in-the-Loop)**

We do not rely solely on target hardware for testing.

* **Simulation:** Developers must be able to compile and run the application logic on a host computer (Windows/Linux).  
* **Time Abstraction & Control:** To enable deterministic testing and effective debugging, the simulation environment must allow full control over system time. This includes the ability to pause execution, step through time, and schedule events at precise timestamps, independent of the host computer's wall-clock speed.  
* **Sync/Async Separation:** Design the system to separate synchronous business logic from asynchronous events (e.g., interrupts, timers). Use interfaces (e.g., Asynchronous Interface Handlers \- AIH) to bridge the two. This decoupling ensures that the core logic remains deterministic and testable, while the simulation environment can explicitly trigger "asynchronous" events in a controlled manner.  
* **Benefit:** This drastically reduces debugging time, removes hardware bottlenecks, and allows logic verification before hardware prototypes exist.

### **3.2. Continuous Quality Verification**

Quality is not a phase at the end of the project; it is a continuous activity.

* **Static Analysis:** All code must pass static analysis tools (linting/MISRA rules) to catch errors before compilation.  
* **Automated Testing:** We utilise Continuous Integration (CI) pipelines to run unit tests and regression tests on every commit.

### **3.3. Iterative Development**

We break projects into small, manageable iterations rather than attempting a "Big Bang" release.

* **Risk Mitigation:** High-risk elements (e.g., complex algorithms, new communication buses) are prioritised in early iterations to fail fast and recover early.

### **3.4. Manage Testable Requirements (Traceability)**

Every feature in the code must originate from a documented requirement.

* **The V-Model:** We maintain traceability. Requirements $\\rightarrow$ Design $\\rightarrow$ Code $\\rightarrow$ Test.  
* **Testability:** If a requirement cannot be tested (either automatically or manually), it is not a valid requirement.

## **4\. Team Dynamics & Knowledge Management**

*Ensuring the team operates as a single unit.*

### **4.1. Consistency (The Principle of Least Astonishment)**

We solve similar problems in similar ways.

* **Patterns:** Use standard design patterns. If a developer moves to a new module, the structure, naming conventions, and error handling should look familiar.  
* **Benefit:** This reduces cognitive load for new hires and simplifies cross-team maintenance.

### **4.2. Knowledge Dissemination**

No part of the software should be understood by only one person.

* **Peer-Programming:** Complex or unique functionality requires two developers (e.g., one writes the interface, one writes the implementation).  
* **Code Reviews:** No code enters the main branch without review. This is not just for catching bugs, but for spreading knowledge of how the system works.  
* **Visualisation:** Extensively use diagrams (UML, flowcharts) to document behaviour. A diagram is often worth 1000 lines of code comments and facilitates deliberating about the operation of the code.  
* **Documentation Strategy:** Documentation must reside within the code repository to ensure it is updated whenever the code is modified. While appropriate naming and consistency should make the code largely self-explanatory, explicit documentation is required to capture **design decisions** and architectural rationale that cannot be inferred from the code alone.

## **5\. Robustness, Safety & Security**

*Ensuring the system is resilient.*

### **5.1. Unified Error Handling Strategy**

The system must have a consistent strategy for failure.

* **Guideline:** Define system-wide rules for error reporting. Do we use return codes? Global error flags? Exceptions?  
* **Recovery:** The system must have a defined safe state and a recovery mechanism (e.g., Watchdog timers) for when critical errors occur.  
* **Documentation of Failure Modes (DFMEA):** Developers must explicitly describe potential error cases and residual risks within the software design documentation. This is necessary for Design Failure Mode and Effects Analysis (DFMEA) and ensures transparency regarding the software's known limitations.

### **5.2. Security by Design**

Security is not an add-on.

* **Input Validation:** Never trust external inputs (sensors, communication buses). All inputs must be validated against ranges and types before processing.  
* **Principle of Least Privilege:** Tasks and modules should only have access to the memory and peripherals they absolutely need.  
* **Credential Management:** Never hardcode passwords, URLs, API keys, or encryption keys directly in the source code. Use secure storage mechanisms (e.g., secure elements, encrypted flash sectors) or inject secrets during the manufacturing/provisioning process.

### **5.3. Configuration Management**

The software architecture must support hardware variants without code duplication.

* **Build-Time Configuration:** Use configuration files or pre-processor definitions to enable or disable functionality, keeping the core codebase unified.  
* **Runtime Configuration:** Use non-volatile memory (e.g., EEPROM, Flash) to store configuration parameters (e.g., calibration data, feature flags, network settings). This allows the software to adapt to its environment or specific hardware revision at startup without requiring recompilation.
