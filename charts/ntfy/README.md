# ntfy

> A Helm chart for ntfy - a simple HTTP-based pub-sub notification service

* <https://github.com/sarab97/helm-charts>
* <https://github.com/binwiederhier/ntfy>

## Usage

Helm must be installed and setup to your kubernetes cluster to use the charts. Refer to Helm's [documentation](https://helm.sh/docs) to get started.

### Basic Installation

```sh
helm install ntfy ./charts/ntfy -n ntfy --create-namespace
```

To uninstall:

```sh
helm delete ntfy -n ntfy
```

## Tailscale Funnel (Recommended)

This chart includes built-in support for [Tailscale Funnel](https://tailscale.com/kb/1223/funnel), allowing you to expose ntfy to the public internet securely without port forwarding or traditional ingress controllers.

### Prerequisites

1. A [Tailscale account](https://tailscale.com/)
2. Funnel enabled for your node tag in Tailscale ACLs
3. A reusable auth key with appropriate tags

### Step 1: Configure Tailscale ACLs

In your [Tailscale ACL editor](https://login.tailscale.com/admin/acls), add:

```json
{
  "tagOwners": {
    "tag:ntfy": ["autogroup:admin"]
  },
  "nodeAttrs": [
    {
      "target": ["tag:ntfy"],
      "attr": ["funnel"]
    }
  ]
}
```

### Step 2: Create Auth Key

1. Go to [Tailscale Settings > Keys](https://login.tailscale.com/admin/settings/keys)
2. Generate a new auth key:
   - **Reusable**: Yes
   - **Tags**: `tag:ntfy`

### Step 3: Create the Kubernetes Secret

```bash
# Using the provided script
./tools/apps/create-ntfy-tailscale-secret.sh "tskey-auth-xxxxx-yyyyy"

# Or manually
kubectl create namespace ntfy
kubectl create secret generic tailscale-auth-ntfy \
  --namespace ntfy \
  --from-literal=TS_AUTHKEY="tskey-auth-xxxxx-yyyyy"
```

### Step 4: Configure values.yaml

The default values are pre-configured for Tailscale. Update these settings for your environment:

```yaml
tailscale:
  enabled: true
  hostname: "ntfy"                      # Machine name on Tailscale
  domain: "your-tailnet.ts.net"         # Your Tailscale domain
  tags: "tag:ntfy"                      # Must match ACL tags
  authSecretName: "tailscale-auth-ntfy"

config:
  enabled: true
  data:
    base-url: "https://ntfy.your-tailnet.ts.net"
```

### Step 5: Deploy

```bash
helm install ntfy ./charts/ntfy -n ntfy
```

### Step 6: Verify

```bash
# Check pods
kubectl get pods -n ntfy

# Check Tailscale logs
kubectl logs -n ntfy -l app.kubernetes.io/name=ntfy -c tailscale

# Check Tailscale status
kubectl exec -n ntfy deployment/ntfy -c tailscale -- tailscale status
```

Your ntfy instance will be accessible at: `https://ntfy.your-tailnet.ts.net`

## Configuration

### Tailscale Settings

| Parameter | Description | Default |
|-----------|-------------|---------|
| `tailscale.enabled` | Enable Tailscale sidecar | `true` |
| `tailscale.image` | Tailscale container image | `tailscale/tailscale:v1.76.1` |
| `tailscale.hostname` | Machine hostname on Tailscale | `ntfy` |
| `tailscale.domain` | Your Tailscale domain | `""` |
| `tailscale.tags` | Tailscale ACL tags | `tag:ntfy` |
| `tailscale.authSecretName` | Secret containing TS_AUTHKEY | `tailscale-auth-ntfy` |
| `tailscale.proxyPort` | Port to proxy traffic to | `80` |
| `tailscale.stateSize` | PVC size for Tailscale state | `100Mi` |

### ntfy Settings

| Parameter | Description | Default |
|-----------|-------------|---------|
| `config.enabled` | Enable ntfy configuration | `true` |
| `config.data.base-url` | Public URL for ntfy | `""` |
| `config.data.auth-default-access` | Default auth access | `deny-all` |
| `persistence.enabled` | Enable persistent storage | `true` |
| `persistence.size` | Storage size | `1Gi` |

## Traditional Ingress (Alternative)

If you prefer traditional ingress instead of Tailscale:

```yaml
tailscale:
  enabled: false

ingress:
  enabled: true
  className: "nginx"
  hosts:
    - host: ntfy.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: ntfy-tls
      hosts:
        - ntfy.example.com
```
