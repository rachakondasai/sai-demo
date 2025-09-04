---

## `projects/nginx-sample-architecture.md`

```markdown
# 🧭 NGINX Sample — Architecture & End-to-End Flow

This doc explains **what reads what**, **who applies what**, and **how the files link up** from Git → ArgoCD → Helm → Kubernetes.

---

## 1) Who’s the “master”?

- **Single source of truth = your Git repository.**
- The **ArgoCD Application** (CRD) only *points* to the repo path/branch and defines where to deploy + how to sync.
- **Helm** renders `templates/*.yaml` using `values.yaml` into pure Kubernetes manifests.
- ArgoCD compares desired (from Git) vs live (cluster) and **reconciles** (self-heal + prune).

---

## 2) High-Level Architecture

```mermaid
flowchart LR
    Dev[Developer commit / PR] --> Repo[GitHub Repo]
    subgraph ArgoCD
      RS[Repo Server\n(fetch repo + Helm render)]
      AC[Application Controller\n(diff, apply, self-heal, prune)]
    end
    Repo --> RS
    RS -->|helm template (values.yaml + templates/*)| AC
    AC -->|Server-side apply| API[(Kubernetes API)]

    subgraph NS[Namespace: nginx-sample]
      DEP[Deployment] --> RSSet[ReplicaSet] --> PODS[Pods]
      SVC[Service] --> PODS
    end

    API --> DEP
    API --> SVC

    User[Client/Tester] -->|HTTP 80| SVC




flowchart LR
    App[ArgoCD Application.yaml] --> SRC[spec.source\n(repoURL/path/revision)]
    SRC --> RS[ArgoCD Repo-Server]
    RS --> Helm[Helm Engine]

    subgraph Chart
      ChartY[Chart.yaml\n(chart metadata)]
      Values[values.yaml\n(defaults: replicas, image, service)]
      Tpls[templates/*.yaml\n(Go templates use .Values/.Release)]
    end

    ChartY --> Helm
    Values --> Helm
    Tpls --> Helm

    Helm --> Manifests[Rendered K8s YAML]
    Manifests --> AC[ArgoCD App-Controller] --> API[(Kubernetes API)]
