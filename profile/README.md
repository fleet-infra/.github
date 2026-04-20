# My homelab

This organization centralizes the configuration and codebase for my homelab. The environment is entirely declarative, managed through IaC and GitOps.

## High-Level architecture

The infrastructure is split into two primary layers: hardware/network provisioning and cluster state management.

```mermaid
flowchart LR
  subgraph GitHub["fleet-infra"]
    RepoTF[/"infra"/]
    RepoK8s[/"gitops"/]
  end

  subgraph BaseInfra["Base infrastructure"]
    TF[Terraform]
    Nodes["Hypervisor / VMs / DNS"]
  end

  subgraph K8s["K8S cluster"]
    Flux[FluxCD]
    Apps["Workloads & Services"]
  end

  RepoTF -->|"Provisions"| TF
  TF -->|"Manages"| Nodes
  Nodes -.->|"Hosts"| K8s

  RepoK8s -->|"Monitors"| Flux
  Flux -->|"Reconciles state"| Apps
```

## Organization Structure

The repositories in this organization are structured by their operational scope:

- [`infra`](https://github.com/fleet-infra/infra): Manages the lifecycle of the base layer. It handles Tailscale ACLs/Auth Keys, Proxmox/VPS resource provisioning, and Cloud DNS records.
- [`gitops`](https://github.com/fleet-infra/gitops): Acts as the Source of Truth for the Kubernetes cluster state. FluxCD reconciles this repository to manage ingress, secrets, and workloads.

_(Note: Detailed architectures, network routings, and deployment structures are documented within their respective repositories.)_

## FAQ

<details>
<summary>Why use a dedicated GitHub Organization for a homelab?</summary>

Creating a separate GitHub organization rather than using a personal user account makes network management significantly easier and more secure.

By tying this organization to [Tailscale](https://tailscale.com), the homelab infrastructure gets its own completely isolated [tailnet](https://tailscale.com/docs/concepts/tailnet). This provides:

- **Strict Network Isolation**: Clearly separates personal devices from infrastructure servers and services.
- **Better Security & ACLs**: Dedicated access control lists, authentication keys, and network tags exclusively for the homelab environment.
- **Clean Access Management**: GitHub Organization teams and members can be mapped directly to Tailscale access roles.

</details>
