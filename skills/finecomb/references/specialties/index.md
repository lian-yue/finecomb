# Part IV: Specialties by target type

The sections below add checkpoints for specific kinds of targets. Choose them based on the [factual baseline](../baseline.md). General criteria are reused through links, and the check status is recorded only once, in the [report](../report.md#part-v-evidence-levels-and-report-format).

Choose by **what kind of thing** the target contains, using the baseline's [mapping known attack mechanisms](../baseline.md#mapping-known-attack-mechanisms) table, and read only the files you choose. A specialty whose subject the target does not have is not read, and inside a chosen specialty, rows whose subject is absent are skipped. Record each specialty excluded by a [review option](../scope.md#review-options) as "Not checked (excluded by the caller)".

## Communication

- [4.1 Networking and connections](4.1-networking-and-connections.md)
- [4.2 Protocols and frame parsing](4.2-protocols-and-frame-parsing.md)
- [4.3 Decoding untrusted external data](4.3-decoding-untrusted-external-data.md)
- [4.21 Outbound requests and server-side request forgery](4.21-outbound-requests-and-server-side-request-forgery.md)
- [4.28 Tunnels, proxies and the network data plane](4.28-tunnels-proxies-and-the-network-data-plane.md)
- [4.62 DNS servers and resolvers](4.62-dns-servers-and-resolvers.md)
- [4.63 Telecom signaling and SMS](4.63-telecom-signaling-and-sms.md)
- [4.77 Mail servers and mail handling](4.77-mail-servers-and-mail-handling.md)

## Identity

- [4.4 Cryptography and credentials](4.4-cryptography-and-credentials.md)
- [4.5 Authentication, sessions and tokens](4.5-authentication-sessions-and-tokens.md)
- [4.46 Federated identity and single sign-on](4.46-federated-identity-and-single-sign-on.md)
- [4.61 Certificate authorities and PKI operations](4.61-certificate-authorities-and-pki-operations.md)
- [4.64 End-to-end encrypted messaging](4.64-end-to-end-encrypted-messaging.md)
- [4.75 Biometrics and identity verification](4.75-biometrics-and-identity-verification.md)

## Data

- [4.6 Caching and storage](4.6-caching-and-storage.md)
- [4.7 Databases and queries](4.7-databases-and-queries.md)
- [4.8 File systems and paths](4.8-file-systems-and-paths.md)
- [4.37 Data processing, batch jobs and machine learning](4.37-data-processing-batch-jobs-and-machine-learning.md)
- [4.53 Search, indexing and retrieval](4.53-search-indexing-and-retrieval.md)
- [4.54 Sync, collaboration and offline clients](4.54-sync-collaboration-and-offline-clients.md)
- [4.67 Location, presence and visibility of personal data](4.67-location-presence-and-visibility-of-personal-data.md)
- [4.73 Backup, archive and disaster recovery](4.73-backup-archive-and-disaster-recovery.md)
- [4.76 Health data and medical interoperability](4.76-health-data-and-medical-interoperability.md)

## Asynchrony

- [4.9 Events, publish-subscribe and observers](4.9-events-publish-subscribe-and-observers.md)
- [4.10 Message queues and async jobs](4.10-message-queues-and-async-jobs.md)
- [4.11 Timers and scheduling](4.11-timers-and-scheduling.md)
- [4.24 Webhooks and external events](4.24-webhooks-and-external-events.md)

## Processes

- [4.12 Dependency wiring and service lifecycle](4.12-dependency-wiring-and-service-lifecycle.md)
- [4.13 Logs, metrics and tracing](4.13-logs-metrics-and-tracing.md)
- [4.14 Server request handling and middleware](4.14-server-request-handling-and-middleware.md)
- [4.15 Command line and process entry points](4.15-command-line-and-process-entry-points.md)
- [4.22 Subprocesses, dynamic execution and decoder side effects](4.22-subprocesses-dynamic-execution-and-decoder-side-effects.md)
- [4.55 Serverless and edge functions](4.55-serverless-and-edge-functions.md)
- [4.57 Workflow automation and low-code platforms](4.57-workflow-automation-and-low-code-platforms.md)
- [4.74 Notebooks and interactive computing](4.74-notebooks-and-interactive-computing.md)

## Representation

- [4.16 Data models and generated contracts](4.16-data-models-and-generated-contracts.md)
- [4.17 Templates and text output](4.17-templates-and-text-output.md)
- [4.18 User interfaces and accessibility](4.18-user-interfaces-and-accessibility.md)
- [4.19 Internationalization and localization](4.19-internationalization-and-localization.md)

## Building blocks

- [4.20 Concurrency primitives and general-purpose containers](4.20-concurrency-primitives-and-general-purpose-containers.md)
- [4.25 Distributed coordination and leases](4.25-distributed-coordination-and-leases.md)

## Security records

- [4.23 Security audit logs](4.23-security-audit-logs.md)

## Model applications

- [4.26 LLMs and tool calling](4.26-llms-and-tool-calling.md)

## Languages and rules

- [4.27 Interpreters, compilers and virtual machines](4.27-interpreters-compilers-and-virtual-machines.md)
- [4.29 Rule and policy matching](4.29-rule-and-policy-matching.md)
- [4.31 Code generators and build-time tools](4.31-code-generators-and-build-time-tools.md)

## Systems and platforms

- [4.30 OS interfaces, system calls and descriptors](4.30-os-interfaces-system-calls-and-descriptors.md)
- [4.32 Cross-language boundaries and native extensions](4.32-cross-language-boundaries-and-native-extensions.md)
- [4.35 Client apps, extensions and auto-update](4.35-client-apps-extensions-and-auto-update.md)
- [4.36 Embedded, firmware and real-time constraints](4.36-embedded-firmware-and-real-time-constraints.md)
- [4.51 Trusted execution environments and enclaves](4.51-trusted-execution-environments-and-enclaves.md)
- [4.52 Industrial control and cyber-physical safety](4.52-industrial-control-and-cyber-physical-safety.md)
- [4.66 Hardware designs and hardware security](4.66-hardware-designs-and-hardware-security.md)
- [4.68 Container runtimes and isolation](4.68-container-runtimes-and-isolation.md)
- [4.78 Sandboxes and broker processes](4.78-sandboxes-and-broker-processes.md)

## Privilege and mobile

- [4.47 Local privileged components and local privilege escalation](4.47-local-privileged-components-and-local-privilege-escalation.md)
- [4.48 Kernel drivers, device emulation and virtualization](4.48-kernel-drivers-device-emulation-and-virtualization.md)
- [4.49 Mobile app components and inter-process communication](4.49-mobile-app-components-and-inter-process-communication.md)
- [4.59 Remote access and device management](4.59-remote-access-and-device-management.md)

## Delivery

- [4.33 Build scripts, CI and infrastructure as code](4.33-build-scripts-ci-and-infrastructure-as-code.md)
- [4.34 Publishable libraries, SDKs and packages](4.34-publishable-libraries-sdks-and-packages.md)
- [4.69 Package registries and artifact repositories](4.69-package-registries-and-artifact-repositories.md)
- [4.70 Code hosting and version control](4.70-code-hosting-and-version-control.md)

## Platforms and security tools

- [4.56 Plugin, extension and app platforms](4.56-plugin-extension-and-app-platforms.md)
- [4.58 Multi-tenant platforms and hosting of user content](4.58-multi-tenant-platforms-and-hosting-of-user-content.md)
- [4.60 Security tools that process untrusted content](4.60-security-tools-that-process-untrusted-content.md)

## Business and outbound

- [4.39 Payments, accounting and billing](4.39-payments-accounting-and-billing.md)
- [4.40 Notifications and outbound messages](4.40-notifications-and-outbound-messages.md)
- [4.65 Trading, exchanges and order matching](4.65-trading-exchanges-and-order-matching.md)
- [4.71 Online games and multiplayer servers](4.71-online-games-and-multiplayer-servers.md)
- [4.72 Media streaming, DRM and content delivery](4.72-media-streaming-drm-and-content-delivery.md)

## On-chain

- [4.38 Smart contracts and on-chain interaction](4.38-smart-contracts-and-on-chain-interaction.md)
- [4.41 DeFi economics and oracles](4.41-defi-economics-and-oracles.md)
- [4.42 Cross-chain bridges and messaging](4.42-cross-chain-bridges-and-messaging.md)
- [4.43 Wallets, signing and off-chain components](4.43-wallets-signing-and-off-chain-components.md)
- [4.44 Zero-knowledge proofs and circuits](4.44-zero-knowledge-proofs-and-circuits.md)
- [4.45 Blockchain nodes, consensus and protocol implementations](4.45-blockchain-nodes-consensus-and-protocol-implementations.md)
- [4.50 Account abstraction and smart contract wallets](4.50-account-abstraction-and-smart-contract-wallets.md)
