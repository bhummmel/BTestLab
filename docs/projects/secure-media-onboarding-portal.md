
# Secure Media Onboarding Portal

## Overview

The Secure Media Onboarding Portal project focused on evaluating and designing a controlled user onboarding workflow for a self-hosted Jellyfin media server.

The objective was to simplify the process of granting friends and family access to a privately managed media library while reducing the administrative overhead associated with manually configuring accounts, distributing connection instructions and troubleshooting remote access.

The project considered a web-based onboarding portal that could guide users through account activation, client installation, server configuration and initial authentication.

Security was a central design requirement. The portal would need to prevent unauthorized account creation, protect administrative credentials and ensure that new users received only the permissions necessary to access the media service.

The evaluation also examined how the onboarding workflow could integrate with existing Unraid, Docker, Jellyfin and reverse proxy infrastructure.

The project was approached as an application architecture and security design exercise. A complete production portal, including automated provisioning and external access controls, was not established as a verified deployment.

---

## Objectives

The primary objective was to develop an architecture for securely onboarding users to a self-hosted media service.

The project focused on:

- Simplifying Jellyfin account creation and activation.
- Reducing manual administrative work.
- Providing clear connection and client setup instructions.
- Restricting account creation to authorized users.
- Protecting Jellyfin administrative credentials.
- Evaluating invitation-based registration.
- Supporting mobile, desktop and television clients.
- Integrating the portal with containerized infrastructure.
- Establishing appropriate logging and auditing.
- Designing reliable account provisioning and recovery workflows.

An additional objective was to provide a user experience suitable for people without technical knowledge of self-hosted services.

---

## Infrastructure Environment

The proposed portal was designed around the existing homelab media infrastructure.

| Component | Role |
|---|---|
| Unraid | Primary application and storage host |
| Docker | Containerized application deployment |
| Jellyfin | Self-hosted media server |
| Reverse proxy | Proposed HTTPS entry point |
| DNS | Service discovery and hostname resolution |
| OPNsense | Firewall and network access control |
| Invitation service | Proposed account authorization mechanism |
| Onboarding web application | Proposed user-facing registration interface |
| Jellyfin API | Account provisioning and management |
| Media automation services | Library acquisition and organization |

The existing Jellyfin environment provides the media service that newly onboarded users would access.

The onboarding portal would operate as a separate application rather than replacing Jellyfin's authentication system.

This separation was important for reducing the privileges exposed through the public registration interface.

---

## System Architecture

The proposed architecture places the onboarding application between the user and Jellyfin's administrative API.

Users would access the portal through a secure web connection and complete an invitation-based registration process.

The portal backend would validate the invitation and perform authorized account provisioning.

```text
                Invited User
                     |
                  HTTPS
                     |
                     v
             +----------------+
             | Reverse Proxy  |
             | TLS Termination|
             +----------------+
                     |
                     v
             +----------------+
             | Onboarding     |
             | Web Portal     |
             +----------------+
                     |
                     v
             +----------------+
             | Registration   |
             | Backend        |
             +----------------+
                |          |
                v          v
          Invitation     Jellyfin
          Database       API
                             |
                             v
                       Jellyfin Server
                             |
                             v
                       Media Libraries
```

The architecture separates the public-facing registration interface from privileged media server operations.

The backend would be responsible for enforcing invitation validity, account policies and API authorization.

---

## Existing Jellyfin Infrastructure

Jellyfin operates as part of the broader self-hosted media environment.

The server provides access to locally stored movies and television content through compatible client applications.

Supporting infrastructure includes media library management and automation services.

Jellyfin provides centralized user accounts, library permissions and playback settings.

The onboarding portal would extend these capabilities by automating selected administrative workflows.

It would not replace Jellyfin's existing account database or authorization mechanisms.

Instead, it would create accounts through supported management interfaces and rely on Jellyfin to enforce media access permissions.

This design minimizes unnecessary duplication of authentication functionality.

---

## User Onboarding Workflow

The proposed onboarding process was designed to minimize the number of technical decisions required from the user.

A typical workflow would involve:

1. An administrator creates an invitation.
2. The invitation is delivered to the intended recipient.
3. The recipient opens the onboarding portal.
4. The portal validates the invitation.
5. The recipient selects a username and password.
6. The backend validates the account information.
7. A Jellyfin account is provisioned with restricted permissions.
8. The invitation is marked as consumed.
9. The user receives connection instructions.
10. The user signs in through a supported Jellyfin client.

The onboarding portal would display a clear success or failure state after registration.

It would also provide guidance for installing Jellyfin applications and connecting to the server.

The workflow was designed to reduce support requests caused by incorrect server addresses or unfamiliar application interfaces.

---

## Invitation-Based Registration

Open public registration was not considered appropriate for a privately hosted media service.

Instead, the proposed architecture uses invitations to authorize account creation.

Each invitation would contain a cryptographically secure, unpredictable token.

The token would identify a specific authorization to register an account.

Important invitation properties include:

- Unique token values.
- Defined expiration times.
- Single-use redemption.
- Revocation capability.
- Server-side validation.
- Association with an invitation record.
- Restrictions on the permissions assigned to new accounts.

Invitation tokens should be stored as hashes rather than plaintext wherever practical.

This would reduce the consequences of a database disclosure.

The system would also need to ensure that concurrent registration requests cannot redeem the same invitation multiple times.

An atomic database operation or equivalent transactional mechanism would be required.

---

## Account Provisioning

The backend would use Jellyfin's supported administrative API to create new user accounts.

Provisioning would occur only after successful invitation validation.

The application would need to enforce account naming rules, password requirements and authorization policies.

The proposed process would include:

- Checking whether the requested username is available.
- Creating the account.
- Applying the intended library permissions.
- Disabling administrative privileges.
- Configuring appropriate user policies.
- Recording the provisioning result.
- Returning a clear status to the onboarding interface.

New users should receive a standard non-administrative account.

Administrative permissions should never be derived from user-supplied registration fields.

The backend should apply predefined account policies rather than accepting arbitrary permission settings from the browser.

---

## Jellyfin API Integration

The Jellyfin API provides a potential integration point for automated user administration.

A provisioning service would need authenticated access to the appropriate administrative endpoints.

The integration should be implemented as a server-side operation.

The browser should never receive the Jellyfin administrative API key or other privileged credentials.

A typical interaction would follow this sequence:

```text
User Registration Request
          |
          v
Validate Invitation
          |
          v
Validate User Input
          |
          v
Backend Authorization
          |
          v
Jellyfin API Request
          |
          v
Create User Account
          |
          v
Apply User Permissions
          |
          v
Confirm Provisioning
          |
          v
Display Setup Instructions
```

The application would also need to handle partial failures.

For example, an account could be created successfully while a subsequent permission update fails.

The backend must not report the registration as fully successful until the required account policy has been applied and verified.

---

## Account Permission Management

Least-privilege account provisioning was a primary security requirement.

The onboarding service should apply a predefined permission profile to each new account.

Potential policy settings include:

- Access to approved media libraries.
- No server administration privileges.
- Restrictions on account management.
- Appropriate remote access permissions.
- Playback and transcoding permissions.
- Device access controls where supported.

The portal should not grant unrestricted access to all available libraries by default.

Different invitation types could eventually support different account profiles.

However, the initial design favors a simple standard-user policy to reduce implementation complexity.

The project reinforced the importance of separating account creation from authorization policy management.

---

## Authentication & Credential Handling

The proposed portal would handle sensitive registration information.

Password handling therefore requires careful design.

User passwords should be transmitted only through HTTPS.

The application should not log plaintext passwords or include them in error reports.

Passwords should be submitted directly to the authorized backend process and forwarded to Jellyfin through its supported API.

The portal should not maintain an unnecessary duplicate password database.

Jellyfin would remain responsible for authenticating users after their accounts are created.

Administrative credentials used by the provisioning backend must be stored securely and excluded from source control.

This design reduces the amount of sensitive information maintained by the onboarding application.

---

## HTTPS & Reverse Proxy Architecture

A secure onboarding portal requires encrypted communications.

The proposed deployment would use a reverse proxy to terminate HTTPS connections.

The reverse proxy would forward authorized traffic to the onboarding application over the internal container network.

TLS certificates would protect credentials and invitation tokens during transmission.

The reverse proxy would also provide a centralized location for configuring security headers and request restrictions.

The portal's public exposure should be limited to the endpoints required for onboarding.

Administrative functions should be separated from public registration routes.

The architecture was designed to avoid exposing internal management interfaces unnecessarily.

---

## Docker Deployment

The portal was considered suitable for deployment as a containerized application on Unraid.

A proposed deployment would separate the web application, backend service and persistent data storage where appropriate.

Containerization offers several operational benefits:

- Consistent application dependencies.
- Simplified deployment and updates.
- Isolation from unrelated services.
- Centralized log collection.
- Persistent configuration management.
- Easier rollback to known application versions.

The application should communicate with Jellyfin through an internal network path whenever possible.

Only the reverse proxy should require direct public exposure.

Administrative API credentials would be injected through protected runtime configuration rather than embedded in the application image.

The project emphasized that containerization provides deployment isolation but does not replace application-level authentication and authorization.

---

## Database & Invitation State

The proposed portal requires persistent storage for invitation records and provisioning status.

A lightweight relational database such as SQLite could be sufficient for a small deployment.

A larger multi-instance architecture might justify PostgreSQL.

The initial design would favor the simplest database capable of enforcing the required consistency guarantees.

An invitation record could contain:

| Field | Purpose |
|---|---|
| Invitation ID | Internal unique identifier |
| Token hash | Secure invitation lookup |
| Creation timestamp | Audit information |
| Expiration timestamp | Validity enforcement |
| Redemption timestamp | Single-use tracking |
| Status | Active, redeemed, expired or revoked |
| Account identifier | Link to provisioned Jellyfin user |
| Policy profile | Authorized account permission template |

The database should not store unnecessary personal information.

Invitation records should be retained only as long as required for administration, troubleshooting and audit purposes.

---

## Transactional Provisioning & Failure Recovery

Account creation spans two systems: the onboarding database and Jellyfin.

Because these systems do not share a single database transaction, failures can produce inconsistent state.

For example, Jellyfin might create an account successfully while the portal fails to record the result.

A second registration attempt could then encounter a username conflict.

The proposed design would use explicit provisioning states to track progress.

Possible states include:

```text
INVITED
   |
   v
VALIDATING
   |
   v
PROVISIONING
   |
   +------> FAILED
   |
   v
VERIFYING
   |
   +------> REQUIRES_RECONCILIATION
   |
   v
COMPLETED
```

Retry behavior should be idempotent wherever possible.

The backend should verify whether an account already exists before repeating an operation.

If a provisioning operation partially succeeds, the system should reconcile the account state before allowing the invitation to be reused or marked complete.

This approach improves reliability and reduces the risk of duplicate or improperly configured accounts.

---

## Client Setup & User Experience

A major objective was to simplify the first-time Jellyfin experience.

New users may be unfamiliar with server addresses, self-hosted applications or the distinction between a media server and its client software.

The onboarding portal would provide platform-specific instructions.

Supported client categories could include:

- Web browsers.
- Android phones and tablets.
- iPhone and iPad.
- Android TV and Google TV.
- Apple TV.
- Desktop applications.
- Other compatible streaming devices.

The portal would display the correct server address and explain how to authenticate.

Where supported, it could provide direct links to official client installation pages.

The user interface should prioritize clarity over exposing unnecessary technical settings.

A successful onboarding experience would allow a new user to access the media library without requiring direct assistance from the server administrator.

---

## Remote Access Considerations

The onboarding workflow depends on users being able to reach the Jellyfin service after registration.

The existing media environment has included work on remote access and external user connectivity.

A registration portal does not automatically solve network access problems.

The architecture must account for:

- Public DNS resolution.
- HTTPS certificate validity.
- Reverse proxy routing.
- Firewall configuration.
- Jellyfin external accessibility.
- Client compatibility.
- Authentication and session management.

The portal should provide only connection instructions that match the actual deployed access method.

If the media server is restricted to a VPN, the onboarding process would also need to explain how to establish that connection.

The project reinforced the importance of treating user provisioning and network connectivity as separate infrastructure requirements.

---

## Abuse Prevention & Rate Limiting

A publicly accessible registration interface creates opportunities for automated requests and unwanted account creation attempts.

Invitation validation reduces exposure but does not eliminate abuse risks.

The proposed portal should implement request rate limiting and appropriate input validation.

Additional controls include:

- Limiting repeated invalid invitation attempts.
- Restricting username length and character sets.
- Rejecting malformed registration requests.
- Avoiding detailed error messages that disclose sensitive account information.
- Expiring unused invitations.
- Allowing administrators to revoke invitations.
- Recording security-relevant events.

The portal should not reveal whether arbitrary usernames or email addresses belong to existing users unless necessary for the authorized registration workflow.

These controls would help protect the application from automated misuse.

---

## Administrative Interface

A future administrative interface could simplify invitation management.

Potential features include creating invitations, viewing redemption status and revoking unused tokens.

Administrative functions should require separate authentication and authorization.

They should not be accessible simply because a visitor possesses a registration invitation.

The initial architecture favors keeping administration on a trusted internal network.

This would reduce the public-facing attack surface.

A command-line administration workflow could also provide a simpler initial implementation.

The evaluation emphasized that administrative convenience should not come at the expense of access control.

---

## Logging & Auditability

Operational logging would be necessary to diagnose failed registrations and unexpected account activity.

Useful events include:

- Invitation creation.
- Invitation redemption.
- Invitation expiration.
- Registration validation failures.
- Jellyfin API failures.
- Successful account provisioning.
- Permission assignment failures.
- Administrative revocation.

Logs should include timestamps and appropriate correlation identifiers.

Sensitive values such as passwords, complete invitation tokens and administrative API credentials should not be recorded.

The system should distinguish between expected validation failures and infrastructure errors.

This would help identify whether registration problems originate from user input, expired invitations, database availability or Jellyfin connectivity.

---

## Monitoring & Telemetry

A production portal would require monitoring of both the application and its dependencies.

Important indicators include:

- Portal availability.
- Reverse proxy connectivity.
- Database availability.
- Jellyfin API reachability.
- Registration success and failure counts.
- Provisioning latency.
- Container restart frequency.
- Unexpected authentication failures.

A basic health check should confirm that the application process is operational.

A more comprehensive readiness check could validate required dependencies without exposing sensitive information.

Monitoring could eventually integrate with the broader homelab observability environment.

The project highlighted the distinction between a web page being accessible and the complete registration workflow functioning correctly.

---

## Security Architecture

Security was a defining requirement of the proposed portal.

The application would have the ability to create accounts on an internal service, making it a privileged integration.

The architecture therefore follows several principles.

### Least Privilege

The backend should have only the permissions required for account provisioning, to the extent supported by Jellyfin's administrative API.

### Secret Isolation

Administrative API credentials must remain on the server and never be exposed to the browser.

### Explicit Authorization

Every registration must be associated with a valid invitation.

### Secure Transport

All external registration traffic must use HTTPS.

### Input Validation

User-controlled values must be validated before being passed to internal services.

### Auditability

Account creation and invitation redemption should generate appropriate audit events.

### Network Segmentation

The public-facing application should have limited access to internal infrastructure.

The evaluation reinforced the importance of treating onboarding services as security-sensitive applications rather than simple registration forms.

---

## Troubleshooting Methodology

The proposed troubleshooting approach separates user interface problems from backend provisioning failures.

### Stage 1: Verify Portal Accessibility

Confirm DNS resolution, HTTPS connectivity and reverse proxy routing.

### Stage 2: Validate Application Health

Confirm that the application container is running and accepting requests.

### Stage 3: Inspect Invitation State

Verify that the invitation exists, has not expired and has not already been redeemed.

### Stage 4: Validate Backend Connectivity

Confirm that the backend can communicate with the database and Jellyfin API.

### Stage 5: Inspect Provisioning Results

Determine whether account creation and permission assignment completed successfully.

### Stage 6: Verify Jellyfin Authentication

Test whether the newly created account can authenticate through Jellyfin.

### Stage 7: Validate Media Access

Confirm that the account has access only to its intended libraries and features.

This methodology would help identify failures without requiring administrators to repeatedly recreate accounts or modify production configuration.

---

## Technical Challenges Identified

### Secure Invitation Management

Invitation tokens must be unpredictable, time-limited and resistant to multiple redemption.

### Administrative API Access

The portal requires privileged access to Jellyfin while preventing exposure of administrative credentials.

### Partial Provisioning Failures

Account creation and permission assignment may fail independently.

### Network Exposure

The registration interface must be reachable by invited users without unnecessarily exposing internal services.

### Client Diversity

Different Jellyfin clients require different installation and connection instructions.

### Configuration Management

The portal must remain synchronized with the actual Jellyfin server address and access policies.

### Recovery & Reconciliation

Application failures must not leave invitations or user accounts in inconsistent states.

These challenges informed the proposed application architecture.

---

## Reliability & Recovery

A reliable onboarding service must preserve invitation state and recover gracefully from temporary infrastructure failures.

Persistent database storage would prevent invitation records from being lost when the application container restarts.

Application configuration and database backups would support recovery following host or storage failures.

Jellyfin configuration backups would remain a separate responsibility.

The portal should not be considered a backup of the media server's user database.

After restoration, the application would need to reconcile invitation records with existing Jellyfin accounts.

A documented recovery procedure would include validating database integrity, API connectivity and account provisioning behavior.

The project reinforced the importance of designing recovery procedures before exposing an application to external users.

---

## Evaluation Outcome

The evaluation established an architectural direction for a secure, invitation-based Jellyfin onboarding service.

The proposed design separates public registration, invitation authorization and privileged account provisioning.

It also defines important controls for credential handling, permission assignment, rate limiting and operational logging.

The existing Unraid and Jellyfin infrastructure provides a suitable foundation for hosting the application.

However, the available project information does not establish that a complete onboarding portal was implemented, externally deployed or validated through production user registrations.

The project should therefore be classified as a security-focused application design and integration evaluation.

A future proof of concept would be necessary to validate the selected framework, database implementation and Jellyfin API workflow.

---

## Lessons Learned

User onboarding is an important operational component of self-hosted services, particularly when access is provided to nontechnical users.

Automated account creation introduces security requirements that are not present in a purely manual administrative workflow.

Invitation-based authorization provides a more controlled alternative to unrestricted public registration.

Privileged API credentials must remain isolated from public-facing application code.

Provisioning reliability requires explicit handling of partial failures and inconsistent state.

A successful registration does not guarantee that the user can connect to the media server or access the intended libraries.

Monitoring should validate the complete onboarding workflow rather than application availability alone.

The project strengthened understanding of secure web application architecture, identity provisioning, API integration and service administration.

---

## Future Improvements

Future work would focus on implementing and validating the proposed architecture.

Potential improvements include:

- Developing a functional invitation-based registration prototype.
- Implementing secure token generation and storage.
- Integrating account creation with the Jellyfin API.
- Enforcing predefined user permission profiles.
- Adding a persistent invitation database.
- Implementing atomic invitation redemption.
- Building a simple administrative invitation interface.
- Providing platform-specific Jellyfin setup instructions.
- Adding HTTPS and reverse proxy integration.
- Implementing rate limiting and structured audit logging.
- Testing provisioning failure recovery.
- Evaluating compatibility with Jellyfin software updates.
- Conducting controlled end-to-end onboarding tests.

These improvements would allow the architecture to progress from design evaluation to a tested application deployment.

---

## Summary

The Secure Media Onboarding Portal project investigated how to simplify access to a self-hosted Jellyfin media server while maintaining appropriate security controls.

The proposed solution uses invitation-based registration, server-side account provisioning and a dedicated onboarding interface to reduce manual administrative work.

The evaluation addressed authentication, authorization, API integration, database consistency, reverse proxy configuration, containerization and operational monitoring.

A central architectural principle was separating the public-facing registration workflow from privileged Jellyfin administration.

Although the complete application was not established as a verified production deployment, the project defined a practical foundation for implementing secure user onboarding.

The work demonstrates skills in application architecture, Linux and Docker administration, API integration, access control, identity provisioning, infrastructure security and technical documentation.
