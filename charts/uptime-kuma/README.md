# uptime-kuma

> A Helm chart for Uptime Kuma - A self-hosted monitoring tool

* <https://github.com/sarab97/helm-charts>
* <https://github.com/louislam/uptime-kuma>

## Usage

Helm must be installed and setup to your kubernetes cluster to use the charts. Refer to Helm's [documentation](https://helm.sh/docs) to get started.

### Quick Start

```sh
helm install uptime-kuma ./charts/uptime-kuma -n uptime-kuma --create-namespace
```

To uninstall:

```sh
helm delete uptime-kuma -n uptime-kuma
```

## Tailscale Funnel

This chart supports exposing Uptime Kuma via Tailscale Funnel for secure public access without traditional ingress.

### Prerequisites

1. **Tailscale Account** with Funnel enabled
2. **Auth Key** from [Tailscale Admin Console](https://login.tailscale.com/admin/settings/keys)
   - Make it reusable
   - Add tag: `tag:uptime-kuma`

3. **Tailscale ACLs** - Add to your [ACL policy](https://login.tailscale.com/admin/acls):

```json
{
  "tagOwners": {
    "tag:uptime-kuma": ["autogroup:admin"]
  },
  "nodeAttrs": [
    {
      "target": ["tag:uptime-kuma"],
      "attr": ["funnel"]
    }
  ]
}
```

### Create the Tailscale Secret

```bash
cd pve-1/tools/apps
./create-uptime-tailscale-secret.sh tskey-auth-xxxxx
```

### Deploy

```bash
helm install uptime-kuma ./charts/uptime-kuma -n uptime-kuma
```

### Verify

```bash
# Check Tailscale sidecar logs
kubectl logs -n uptime-kuma deployment/uptime-kuma -c tailscale

# Access publicly at
https://uptime.tail108d23.ts.net
```

## Integrating with ntfy for Notifications

Uptime Kuma has built-in support for ntfy push notifications.

### Setup in Uptime Kuma UI

1. Access Uptime Kuma at `https://uptime.tail108d23.ts.net`
2. Go to **Settings** → **Notifications**
3. Click **Setup Notification**
4. Select **ntfy** as the notification type
5. Configure:
   - **Server URL**: `https://ntfy.tail108d23.ts.net`
   - **Topic**: Your topic name (e.g., `alerts`, `uptime`)
   - **Username**: `admin`
   - **Password**: Your ntfy admin password

### Test the Integration

1. Click **Test** in the notification settings
2. You should receive a push notification on your devices subscribed to the topic

## Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `tailscale.enabled` | Enable Tailscale sidecar | `true` |
| `tailscale.image` | Tailscale container image | `tailscale/tailscale:v1.76.1` |
| `tailscale.hostname` | Tailscale machine hostname | `uptime` |
| `tailscale.domain` | Tailscale domain | `tail108d23.ts.net` |
| `tailscale.tags` | Tailscale ACL tags | `tag:uptime-kuma` |
| `tailscale.authSecretName` | Secret name for auth key | `tailscale-auth-uptime` |
| `tailscale.stateSize` | PVC size for Tailscale state | `100Mi` |
| `persistence.enabled` | Enable persistent storage | `true` |
| `persistence.size` | PVC size for Uptime Kuma data | `2Gi` |
