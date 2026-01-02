Docmost Application Architecture Overview
1.0 Introduction
Docmost is a sophisticated, real-time collaborative documentation platform engineered for modern teams. This document dissects its technical architecture, providing a comprehensive overview of the application's structure, technology stack, and core functional systems. The purpose of this overview is to furnish software engineers and architects with a clear understanding of the design principles and technological choices that power the Docmost platform.
This document begins with a high-level view of the system's design and project structure. It then delves into detailed examinations of the client (frontend) and server (backend) applications, analyzing their distinct architectural patterns and components. Following this, the overview explores key cross-cutting systems, such as real-time collaboration and authentication, which are integral to the user experience. The document concludes with an analysis of the application's deployment strategy and its licensing model, which cleanly separates open-source functionality from enterprise features.
--------------------------------------------------------------------------------
2.0 High-Level System Design
The strategic foundation of Docmost is built upon a well-defined project structure and a modern, cohesive technology stack. These design choices are paramount, facilitating clear separation of concerns, long-term maintainability, and the scalability required for a feature-rich, real-time application. This design philosophy balances best-in-class specialized libraries (like Hocuspocus for collaboration) with robust, general-purpose frameworks (like NestJS) to optimize both real-time performance and development velocity. This section outlines the monorepo organization and the core technologies that underpin the entire system.
2.1 Project Structure
Docmost is organized as a monorepo, a single repository containing multiple distinct projects and packages. This approach centralizes code management, simplifies dependency handling, and promotes code sharing between the frontend, backend, and specialized feature packages.
The primary directories within the project are structured as follows:
• apps/client: Contains the complete frontend application, a Single Page Application (SPA) built with React.
• apps/server: Houses the backend application, a robust API and real-time service layer built with NestJS.
• packages/ee: A dedicated package for Enterprise Edition (EE) features. This demonstrates a clear architectural separation between the core open-source product and licensed, commercial functionalities.
2.2 Core Technology Stack
The technology stack is a curated selection of powerful, modern tools chosen for performance, developer experience, and scalability. The key components are summarized below:
Component
Technology
Primary Role
Frontend Framework
React
The core library for building the declarative, component-based user interface.
Frontend UI
Mantine UI
A comprehensive component library used for styling and UI elements.
Frontend State
Jotai & TanStack Query
Manages client-side UI state (Jotai) and server state/data fetching (TanStack).
Backend Framework
NestJS with Fastify
A progressive Node.js framework providing a modular structure for the server API.
Database Interaction
Kysely Query Builder
A type-safe SQL query builder for interacting with the database.
Real-time Collaboration
Hocuspocus & Yjs
The core engine for real-time, multi-user document editing and synchronization.
Containerization
Docker
Used to package the application and its dependencies into a portable container.
Package Management
pnpm
A fast, disk space-efficient package manager for the monorepo.
This high-level stack provides a robust foundation, enabling a detailed look into the specific architecture of the client application.
--------------------------------------------------------------------------------
3.0 Frontend Architecture (Client Application)
The Docmost client is a feature-rich Single Page Application (SPA) built with React, serving as the primary user interface for all platform interactions. It is engineered to be responsive, interactive, and highly performant. This section deconstructs its primary components, from the UI and state management systems to the sophisticated collaborative editor at its core.
3.1 Framework, Build Tools, and Styling
The client application is built on a modern frontend toolchain:
• React: Serves as the fundamental library for constructing the user interface through a component-based model.
• Vite: The project utilizes Vite as its build tool and development server. This choice, evidenced in the index.html configuration, provides an extremely fast development experience with features like Hot Module Replacement (HMR).
• Styling: The visual presentation is managed through a combination of the Mantine UI component library, which provides a rich set of pre-built and themeable components, and CSS Modules (.module.css files) for writing locally scoped, conflict-free styles for individual components.
3.2 State Management and Data Fetching
A dual-strategy approach is employed for state management to handle different types of data effectively:
• Jotai: This minimalistic, atom-based library is used for managing global client-side UI state. An example is the sidebar-atom.ts file, which controls the visibility and state of the application's sidebars.
• TanStack Query (React Query): This powerful library is the standard for managing server state, treating the backend as the ultimate source of truth. It handles all aspects of data fetching, caching, invalidation, and optimistic updates, abstracting away the complexities of remote data management. Its use is prevalent throughout the application, with hooks like usePageQuery, useBillingQuery, and useGetApiKeysQuery.
3.3 Routing and Internationalization
• Routing: Client-side navigation is managed by React Router. The application defines its various pages and layouts using declarative <Route> components, enabling seamless transitions without full-page reloads.
• Internationalization (i18n): The application is built to support a global audience with built-in multi-language capabilities. It uses i18next and the useTranslation hook to provide translations for UI text, as evidenced by the extensive list of translation strings for German, Japanese, Korean, Chinese, and other languages.
3.4 The Collaborative Editor
The collaborative rich text editor is the centerpiece of the Docmost experience. Its functionality is enabled by a powerful combination of specialized libraries:
• Tiptap: A headless, extensible framework that provides the foundation for the rich text editor interface and its various features.
• Yjs: A high-performance Conflict-Free Replicated Data Type (CRDT) implementation. Yjs is crucial for enabling real-time collaboration, as it allows document states from multiple users to be merged automatically and without conflict.
• Hocuspocus: This provider acts as the bridge between the client-side Yjs document and the backend WebSocket server, facilitating the real-time communication necessary for seamless multi-user editing.
The editor supports a wide range of features, including:
• Rich text formatting (headings, lists, blockquotes, code blocks)
• Embeddable nodes for images, videos, and file attachments (managed by the backend's abstract file storage system).
• Interactive diagrams via integrations like Mermaid, Draw.io, and Excalidraw
• Inline and block-level math equations
• Real-time commenting and user mentions (enabled by Tiptap's comment mark extension and Yjs synchronization).
• Advanced tables with cell-level controls
The client application's sophisticated features are fully dependent on the robust services provided by the backend.
--------------------------------------------------------------------------------
4.0 Backend Architecture (Server Application)
The Docmost server acts as the authoritative backend, built on the NestJS framework to be modular, scalable, and efficient. It is responsible for all business logic, data persistence, user authentication, and the facilitation of real-time communication services that power the collaborative features of the platform.
4.1 Core Framework and Web Server
• Framework: The server is built with NestJS, a progressive Node.js framework that enforces a structured, modular architecture. This promotes maintainability and scalability by organizing the application into a series of distinct modules.
• Web Server: To achieve high performance and low overhead, the application leverages the Fastify adapter. As configured in apps/server/src/main.ts, Fastify is used in place of the default Express server for handling HTTP requests.
4.2 Database and Data Access Layer
• Query Builder: The application communicates with its database using Kysely, a type-safe SQL query builder. This ensures that all database queries are validated against the database schema at compile time, preventing SQL injection vulnerabilities and eliminating entire classes of runtime errors related to mismatched data types or column names.
• Data Access: A repository pattern is employed to abstract and encapsulate data access logic. Repositories like PageRepo, UserRepo, and SpaceRepo provide a clean API for interacting with the database, separating data concerns from business logic.
• Core Data Model: The database schema is centered around several key entities:
    ◦ Workspaces and Users
    ◦ Spaces (collections of documents)
    ◦ Pages (which support parent-child relationships for hierarchical structures)
    ◦ Comments and Attachments
4.3 Real-time Services
The backend manages two distinct real-time communication channels. This dual-channel approach is a deliberate architectural choice: Hocuspocus provides a specialized, high-performance CRDT synchronization layer essential for the complexity of document editing, while Socket.IO serves as a more general-purpose event bus for broadcasting simpler, non-CRDT state changes across the application.
• Collaboration: A dedicated Hocuspocus server manages WebSocket connections specifically for collaborative document editing. It uses a system of extensions (e.g., Authentication.extension.ts, Database.extension.ts) to integrate authentication and ensure that document changes are persisted to the database.
• General Updates: For other real-time events, such as notifying clients of changes to the page tree structure, the server uses Socket.IO. This is implemented via a NestJS WsGateway, providing a general-purpose channel for broadcasting updates to connected clients.
4.4 Asynchronous Processing and File Storage
• Asynchronous Processing: Background jobs and long-running tasks are managed by BullMQ, a robust queueing system built on Redis. The application defines several queues for different purposes, including:
    ◦ AI_QUEUE: For AI-related processing like search indexing or generative features.
    ◦ SEARCH_QUEUE: For managing search index updates.
    ◦ BILLING_QUEUE: For handling subscription and billing-related tasks.
• File Storage: The file storage system is designed with a flexible driver-based architecture. It supports both a local filesystem driver for simple deployments and an S3-compatible driver for scalable, cloud-based object storage.
These backend systems work in concert to support the key cross-cutting features of the application.
--------------------------------------------------------------------------------
5.0 Key Functional Systems
This section examines major cross-cutting features that rely on tight integration between the client and server. These systems are fundamental to providing a secure, powerful, and intuitive user experience in Docmost.
5.1 Authentication and Authorization
Security is managed through a multi-layered authentication and authorization system.
• Authentication: The primary mechanism for authentication is JSON Web Tokens (JWT). The backend's TokenService is responsible for issuing and validating these tokens, which are then used by the client to authenticate API requests.
• Authorization: The application employs a Role-Based Access Control (RBAC) model with scopes defined at two levels: workspace-level roles (OWNER, ADMIN, MEMBER) govern administrative access, while more granular space-level roles (WRITER, READER) control content permissions.
• Enterprise Authentication: The architecture is designed to be extensible, supporting advanced enterprise security requirements. These features, implemented within the ee packages, include Single Sign-On (SSO) via SAML, OIDC, and LDAP, as well as Multi-Factor Authentication (MFA).
5.2 Search System
Docmost provides a powerful, unified search experience that spans across pages and attachments.
• Pluggable Architecture: The backend is architected with a pluggable search driver, allowing the search implementation to be swapped without altering core business logic. The system can route requests to different providers, with Typesense explicitly supported as a high-performance search engine.
• AI-Powered Search: As an enterprise feature, the application offers an AI-powered search capability. This advanced functionality, evidenced by the useAiSearch hook on the client and the AI_QUEUE on the backend, provides more intelligent and context-aware search results.
These functional systems are packaged for deployment using modern operational practices.
--------------------------------------------------------------------------------
6.0 Deployment and Operations
The Docmost application is designed for modern, container-based deployment environments. This approach ensures consistency, reliability, and scalability across different hosting platforms, from on-premise servers to cloud providers.
6.1 Containerization
The deployment strategy is centered around Docker, as defined in the project's Dockerfile.
• Multi-Stage Docker Build: The Dockerfile implements a multi-stage build process. This is a best practice that separates the build environment (which includes development dependencies and toolchains) from the final runtime environment. The primary benefit is the creation of a smaller, more secure final container image that includes only the artifacts necessary to run the application.
6.2 Dependency Management
• pnpm: The monorepo uses pnpm as its package manager. As specified in the Dockerfile, pnpm is chosen for its efficiency in handling dependencies across multiple projects within the repository, saving disk space and speeding up installation times.
--------------------------------------------------------------------------------
7.0 Licensing and Extensibility
The Docmost architecture employs a clear and deliberate strategy to separate its core open-source functionality from commercial, enterprise-grade features. This model allows for a strong community-driven base product while offering advanced capabilities to enterprise customers.
7.1 Enterprise Edition (ee) Architecture
The separation between the core application and enterprise features is enforced at the code level through dedicated directories and packages. All enterprise-specific code is housed in apps/client/src/ee, apps/server/src/ee, and packages/ee, ensuring a clear boundary. This strategy not only clarifies licensing but also prevents enterprise complexity from impacting the core open-source codebase, ensuring its agility and maintainability.
Key enterprise features identified in the source context include:
• Advanced Authentication: Support for SSO (SAML, OIDC, LDAP) and Multi-Factor Authentication (MFA).
• API Key Management: Functionality for users and workspaces to create, manage, and revoke API keys.
• Subscription Billing and Licensing: A complete system for managing commercial subscriptions, license keys, and billing plans.
• AI-Powered Features: Includes both AI-enhanced search and generative AI capabilities for content creation.
• Advanced Commenting: Additional features for the commenting system, such as the ability to resolve comment threads.

