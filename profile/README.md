# Homelab Infrastructure

This org contains all configurations for my homelab as it is provisioning the base infrastructure via [Terraform](https://developer.hashicorp.com/terraform) and deploying workloads via [FluxCD](https://fluxcd.io/).

## GitOps Architecture

The cluster state is entirely declarative and managed by FluxCD, utilizing Kustomize for base and environment-specific overlays.

```mermaid
flowchart LR
  GitRepo[/"GitHub Repository"/]

  subgraph Clusters["Clusters (FluxCD)"]
    C_K3D["k3d-lab"]
    C_PROD["production"]
  end

  subgraph Infra["Infrastructure (Controllers)"]
    I_BASE["base/"]
    I_K3D["k3d-lab/"]
    I_PROD["production/"]
  end

  subgraph Apps["Applications (Workloads)"]
    A_BASE["base/"]
    A_K3D["k3d-lab/"]
    A_PROD["production/"]
  end

  GitRepo -->|"Syncs"| Clusters

  C_K3D ==>|"1. Applies"| I_K3D
  C_K3D ==>|"2. Applies"| A_K3D

  C_PROD ==>|"1. Applies"| I_PROD
  C_PROD ==>|"2. Applies"| A_PROD

  I_K3D -.->|"Kustomize"| I_BASE
  I_PROD -.->|"Kustomize"| I_BASE

  A_K3D -.->|"Kustomize"| A_BASE
  A_PROD -.->|"Kustomize"| A_BASE
```

## Hybrid Access Strategy

We use a dual-layer security model to balance accessibility and high-level protection.

```mermaid
flowchart TD
  subgraph Users["User Access"]
    U_Public["Public / Local Network"]
    U_Admin["Admin / Private Devices (Tailnet)"]
  end

  subgraph Cluster["Kubernetes Cluster"]
    direction TB

    subgraph Controllers["Ingress Controllers"]
      TR["Traefik Proxy"]
      TSO["Tailscale Operator"]
    end

    subgraph Apps["Applications"]
      A_PUB["Public Services"]
      A_PRIV["Private Services"]
    end
  end

  U_Public -->|Port 80/443| TR
  TR -->|Standard Routing| A_PUB

  U_Admin -->|Secure Tunnel| TSO
  TSO -->|Direct Tunneling| A_PRIV

  TSO -.->|Admin UI Access| TR
```

## FAQ

**Why use a dedicated GitHub Organization for a homelab?**

Creating a separate GitHub organization rather than using a personal user account makes network management significantly easier. By tying the organization to [Tailscale](https://tailscale.com), the homelab infrastructure gets its own completely isolated [tailnet](https://tailscale.com/docs/concepts/tailnet).
