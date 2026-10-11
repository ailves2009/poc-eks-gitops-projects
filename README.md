# poc-eks-gitops-projects

**Who may deploy what, where.** One Argo CD `AppProject` per file under `projects/`, synced
automatically by the `projects` Application of
[poc-eks-gitops-root](https://github.com/ailves2009/poc-eks-gitops-root) (sync wave −1, before
any ApplicationSet).

| Project | Used by | Guardrails |
|---|---|---|
| `infra-poc-plt-eks` | infra ApplicationSet (metrics-server) | values repo + metrics-server Helm repo only; `kube-system` only; cluster-scoped: ClusterRole(Binding), APIService |
| `demo` | workloads ApplicationSet (`workloads/*/demo/*`) | charts + values repos only; namespaces `demo-*`; cluster-scoped: Namespace only; namespaced kinds limited to what `poc-lib` renders; nightly sync freeze |

## Rules

- **The guardrail is the lists, not the namespace.** `sourceRepos`, `clusterResourceWhitelist`
  and `namespaceResourceWhitelist` name exactly what a project may render; a chart that starts
  emitting a new kind fails its sync until the platform adds the kind here.
- **Project name = application name** for workloads: `workloads/<env>/<app>/` in the values repo
  maps to project `<app>`. A new application starts with a new file here; CI of the root repo
  fails while the project is missing.
- **Roles** (`operator`, `developer`) grant `get` + `sync` inside the project. The global
  default is `role:readonly` (set in the Argo CD install). With SSO they would be bound to groups.
- **`syncWindows`**: a nightly deny window with `manualSync: true` — automation pauses, people
  can still sync.
- The `resources-finalizer` keeps a project that is still in use from being deleted.

## CI

`kubeconform -strict` against the real AppProject CRD schema; each file holds one AppProject
named after the file.
