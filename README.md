# SecureSOHONetwork

The implementation and future development of a highly secure and private server infrastructure.

## Highly Secure and Private Home Network Setup

### 

#### Statement:

This application represents a highly secure and private home network infrastructure, which includes a production-style self-hosting environment using Docker Swarm, private PKI, and a zero-trust overlay network, and a secure and private remote access solution of Tailscale.

#### Goal:

The goal is to securely operate stateful applications(Nextcloud and Vaultwarden) without exposing any public network surface, while maintaining automated certificate lifecycle management and strong service isolation.

#### Key Components:

- Docker Swarm for container orchestration
- Traefik as an internal reverse proxy
- Step CA as a private Certificate Authority(ACME)
- Bind9 for internal DNS
- Nextcloud and Vaultwarden as sample production services
- Encrypted networking over a private overlay(Tailscale)

All application traffic is encrypted using short-lived certificates issued by an internal CA. No services are exposed directly to the public internet.

#### Architecture Summary:

- All access occurs over a private overlay network
- Identity is established via TLS certificates and not network location(ex: IP address)
- Certificates are automatically issued and rotated using ACME
- Applications are isolated using Swarm overlay networks and secrets
- Persistent data is stored outside containers and backed up separately

#### High Level Architecture

.png

#### Network & Trust Model:

- No implicit trust based on IP or subnet
- All services require TLS issued by the internal CA
- DNS resolution is handled internally
- Overlay network enforces encrypted transport between nodes

###### Trust Boundaries

- Client devices
- Overlay network
- Swarm internal networks
- Application containers

Laptop → Tailscale \[Device Identity Authentication\]

Services → Step CA \[Certificate Issuance\]

Applications → Step CA \[Trust anchor\]

Clients → Applications \[TLS trust\]

#### Certificate Lifecycle(Private PKI):

Certificates are managed using **Step CA** with an ACME provisioner

1. Services request certificates via ACME
2. Step CA validates requests and issues short-lived certs
3. Traefik terminates TLS using internally trust certificates
4. Certificates are automatically renewed before expiration

###### Certificate Flow

Traefik/Service → Step CA \[ACME certificate request\]

Step CA → Traefik/Service \[Short-lived TLS certificate issued *72hrs*\]

Step CA → Applications \[CA Root Certificate\]

Service → Step CA \[Automated renewal before expiration\]

#### Docker Swarm Architecture:

- Swarm managers coordinate scheduling and secrets
- Worker nodes run application workloads
- Overlay networks isolate traffic between services
- Persistent volumes store application state

###### Swarm Architecture

.png

#### Application Stack:

Traefik

- Internal reverse proxy
- Automatic certificate renewal via ACME
- Routes traffic to internal services only

Nextcloud

- Stateful file storage and collaboration platform
- Backend database isolated on a private network
- TLS terminated at Traefik

Vaultwarden

- Self-hosted password manager
- Encrypted storage
- Restricted access to authorized clients only

#### Security Considerations:

- No public ingress
- Short-lived certificates reduce blast radius
- Secrets injected using Swarm secrets
- Applications run with least privilege
- Internal DNS prevents dependency on external resolvers

###### Client → Application Zero-Trust Flow

Client → Tailscale \[Client connects over overlay\]

Client → Bind9 \[Internal DNS resolution\]

Client → Traefik \[TLS connection establishment\]

Traefik → Nextcloud/Vaultwarden \[Encrypted internal traffic with Docker Swarm\]

##### Trust Boundaries

Client Device Zone

Overlay Network Zone

Swarm Internal Network Zone

Application Containers

#### Repository Structure:

```
docker secret create adminSecret adminSecretFil
```

#### How This Architecture Implies Zero-Trust:

- Network location does not imply trust
- Server identity is cryptographically verified
- Access is authenticated at every hop
