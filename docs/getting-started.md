---
title: Getting Started
nav_order: 2
---

# Getting Started

## Prerequisites

- Kubernetes 1.26+
- `kubectl` configured against your cluster
- [kind](https://kind.sigs.k8s.io/) (for local development)
- An Anthropic API key

For local development, create a cluster first:

```bash
kind create cluster --name agent-operator-dev
```

## Install

### 1. Install CRDs

```bash
make install
```

This registers `AgentDeployment`, `AgentService`, `AgentConfig`, and `AgentPipeline` with your cluster.

### 2. Deploy Redis

agent-operator uses Redis Streams as the task queue between the operator and agent pods.

```bash
kubectl apply -f config/prereqs/redis.yaml
```

### 3. Create the API key secret

Create one secret per namespace where agents will run:

```bash
kubectl create secret generic agent-operator-api-keys \
  --from-literal=ANTHROPIC_API_KEY=sk-ant-... \
  --from-literal=TASK_QUEUE_URL=redis.agent-infra.svc.cluster.local:6379
```

The operator injects these values into agent pods automatically.

### 4. Run the operator

```bash
make run
```

This runs the operator process locally, connected to your cluster via kubeconfig. For in-cluster deployment, see the manager manifests in `config/manager/`.

### 5. Deploy your first agent

```bash
kubectl apply -f config/samples/agentops_v1alpha1_agentdeployment.yaml
```

## Verify

```bash
# Check the AgentDeployment
kubectl get agdep
# NAME              MODEL                      REPLICAS   READY   AGE
# research-agent    claude-sonnet-4-20250514   2          2       30s

# Check the backing pods
kubectl get pods -l agentops.io/deployment=research-agent
# NAME                              READY   STATUS    RESTARTS   AGE
# research-agent-agent-7d9f-xk2p8   1/1     Running   0          30s
# research-agent-agent-7d9f-mn4q1   1/1     Running   0          30s
```

## Next steps

- [AgentDeployment reference](./crds/agent-deployment) — full spec for the core resource
- [AgentService reference](./crds/agent-service) — routing tasks to agents
- [AgentConfig reference](./crds/agent-config) — reusable configuration
- [AgentPipeline reference](./crds/agent-pipeline) — multi-agent DAGs
- [Semantic Health Checks](./concepts/semantic-health-checks) — how readiness probes work
- [MCP Servers](./concepts/mcp-servers) — connecting tools to agents
- [Scaling](./concepts/scaling) — replicas and limits
