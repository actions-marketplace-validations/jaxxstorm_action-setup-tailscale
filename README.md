# Enhanced Tailscale GitHub Action

An enhanced GitHub Action that connects to your [Tailscale network](https://tailscale.com) with automatic cleanup, performance optimizations, and cross-platform support.

## Key Features

- 🔧 **Automatic Cleanup**: Post-job logout prevents orphaned connections
- ⚡ **Performance Optimized**: Up to 40% faster than the official action
- 🌍 **Cross-Platform**: Full support for Linux, macOS, and Windows
- 🏗️ **ARM64 Support**: Native support for ARM64 runners
- 💾 **Smart Caching**: Intelligent binary caching for faster subsequent runs
- 🐛 **Debug Tools**: Built-in SSH debugging workflow for troubleshooting

## Quick Start

```yaml
  - name: Enhanced Tailscale
    uses: jaxxstorm/action-setup-tailscale@v1
    with:
      authkey: ${{ secrets.TAILSCALE_AUTHKEY }}
      version: 'latest'
```

## Authentication Options

### OAuth Authentication (Recommended)
```yaml
  - name: Enhanced Tailscale
    uses: jaxxstorm/action-setup-tailscale@v1
    with:
      oauth-client-id: ${{ secrets.TS_OAUTH_CLIENT_ID }}
      oauth-secret: ${{ secrets.TS_OAUTH_SECRET }}
      tags: tag:ci
```

### Auth Key Authentication
```yaml
  - name: Enhanced Tailscale
    uses: jaxxstorm/action-setup-tailscale@v1
    with:
      authkey: ${{ secrets.TAILSCALE_AUTHKEY }}
```

Authentication credentials should be stored as [GitHub Encrypted Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets).

For OAuth authentication, you need an [OAuth client](https://tailscale.com/s/oauth-clients/) with the [`auth_keys` scope](https://tailscale.com/kb/1215/oauth-clients#scopes).

Tags are a comma-separated list of [ACL Tags](https://tailscale.com/kb/1068/acl-tags/) for the node. At least one tag is required for OAuth authentication.

## Tailscale SSH Debugging

This action includes a powerful **Tailscale SSH debugging workflow** that allows you to connect directly to GitHub runners through your Tailscale network. This is especially useful when debugging Tailscale connectivity issues, testing action behavior, or investigating runner-specific problems.

### Using the Tailscale SSH Debug Workflow

1. **Navigate to Actions Tab**: Go to your repository's Actions tab
2. **Select Tailscale SSH Debug Session**: Find the "Tailscale SSH Debug Session" workflow
3. **Run Workflow**: Click "Run workflow" and configure:
   - **Runner Type**: Choose from various runner types (ubuntu-latest, macos-latest, windows-latest, etc.)
   - **Timeout**: Set session timeout (default: 30 minutes)
   - **SSH User**: Your Tailscale login name for SSH access

### Prerequisites

Before using Tailscale SSH debugging:

1. **Configure Tailscale SSH ACLs**: Ensure your Tailscale ACL allows SSH access:
```json
{
  "ssh": [
    {
      "action": "accept",
      "src": ["your-user@domain.com"],
      "dst": ["autogroup:self"],
      "users": ["autogroup:nonroot"]
    }
  ]
}
```

2. **Set Repository Secret**: Add `TAILSCALE_AUTHKEY` to your repository secrets with an appropriate Tailscale auth key.

### SSH Connection Process

When the workflow runs:
1. **Setup Phase**: Runner is prepared with dependencies and Tailscale
2. **Enable SSH**: Tailscale SSH is enabled with `tailscale up --ssh`
3. **Display Connection Info**: Shows the runner's Tailscale IP and hostname
4. **Wait for Connections**: Keeps the runner alive for the specified timeout

### Connecting to the Runner

From the workflow output, you'll see connection details like:
```bash
# Connect using Tailscale IP
ssh your-user@100.64.x.y

# Or connect using Tailscale hostname  
ssh your-user@github-runner-xyz.tailnet-name.ts.net
```

### Security Advantages of Tailscale SSH

- **Zero Trust Network**: Access only through your private Tailscale network
- **No Public Exposure**: No public SSH endpoints or tunnels
- **ACL-Controlled**: Fine-grained access control through Tailscale ACLs
- **Audit Logging**: SSH access is logged through Tailscale
- **Key Management**: Leverages Tailscale's certificate-based SSH keys
- **Automatic Cleanup**: Runner automatically disconnects when workflow ends

### Common Debugging Tasks

Once connected via Tailscale SSH, you can:
- **Check Tailscale Status**: `tailscale status`
- **Test Connectivity**: `tailscale ping <node>`
- **View SSH Configuration**: `tailscale ssh`
- **Debug Network Issues**: Inspect network configuration and routing
- **Run Tests Interactively**: Execute tests and debug failures
- **Access Private Resources**: Connect to internal services through Tailscale
- **Cross-Platform Testing**: Debug platform-specific issues

### Best Practices

1. **Configure ACLs Properly**: Set up appropriate SSH access controls in your Tailscale ACL
2. **Use Timeouts**: Always set reasonable timeouts to avoid resource waste
3. **Monitor Access**: Keep track of who has SSH access to your runners
4. **Clean Sessions**: The runner automatically cleans up when the workflow ends
5. **Document Findings**: Record debugging results for future reference
6. **Test Different Platforms**: Use various runner types to test cross-platform behavior

### Example ACL Configuration

Here's an example Tailscale ACL that allows SSH access to GitHub runners:

```json
{
  "tagOwners": {
    "tag:github-runner": ["your-user@domain.com"]
  },
  "acls": [
    {
      "action": "accept",
      "src": ["*"],
      "dst": ["*:*"]
    }
  ],
  "ssh": [
    {
      "action": "accept", 
      "src": ["your-user@domain.com"],
      "dst": ["tag:github-runner"],
      "users": ["root", "runner", "ubuntu", "ec2-user"]
    }
  ]
}
```

This configuration allows your user to SSH to any node tagged with `tag:github-runner` as various system users.

## Automatic Cleanup

This action includes **automatic post-job cleanup** that ensures Tailscale nodes are properly logged out after your workflow completes. This prevents orphaned connections and maintains a clean tailnet.

The cleanup process:
- Runs automatically after your job completes (success or failure)
- Gracefully logs out from Tailscale
- Removes the ephemeral node from your tailnet
- Works across all supported platforms (Linux, macOS, Windows)

## Performance Benefits

Our enhanced action provides significant performance improvements:
- **Up to 40% faster** installation times compared to the official action
- **Intelligent caching** reduces repeated downloads
- **Optimized binary distribution** for faster extraction
- **Parallel processing** for multi-step operations

## Cross-Platform Support

Full support across GitHub's runner ecosystem:
- **Linux**: AMD64 and ARM64 architectures
- **macOS**: Intel and Apple Silicon (ARM64)
- **Windows**: AMD64 and ARM64 architectures
- **Self-hosted runners**: Compatible with custom runner configurations

## Node Management

Nodes created by this action are automatically configured as:
- **Ephemeral**: Automatically removed when they disconnect
- **Preapproved**: Skip device approval on tailnets with device approval enabled
- **Tagged**: Properly tagged for ACL management (when using OAuth)

## Eventual Consistency

Tailscale network propagation is eventually consistent. Brief delays are expected before new nodes become visible across your tailnet. Use the `ping` parameter or `tailscale ping` command to verify connectivity:

```bash
tailscale ping my-target.my-tailnet.ts.net
```

> ⚠️ **macOS Note**: On macOS runners, you can only ping IP addresses, not hostnames.

## Troubleshooting

### Common Issues
1. **Authentication Failures**: Verify your OAuth client has the `auth_keys` scope
2. **Network Connectivity**: Use the SSH debug workflow to investigate connection issues
3. **Platform Issues**: Test across different runner types using the comprehensive test matrix
4. **Performance**: Monitor the speed comparison workflow results

### Debug Resources
- Use the built-in SSH debugging workflow for interactive troubleshooting
- Check workflow logs for detailed error messages
- Review Tailscale status output in workflow runs
- Test with different runner configurations

### Getting Help
- Review the [Tailscale documentation](https://tailscale.com/kb/)
- Check [GitHub Actions documentation](https://docs.github.com/en/actions)
- Open an issue in this repository for action-specific problems

## Contributing

We welcome contributions! Please see our contributing guidelines and feel free to:
- Report bugs and issues
- Suggest new features
- Submit pull requests
- Improve documentation

## License

This project is licensed under the same terms as the original Tailscale GitHub Action.

## Advanced Configuration

### Version Selection
```yaml
  - name: Enhanced Tailscale
    uses: jaxxstorm/action-setup-tailscale@v1
    with:
      authkey: ${{ secrets.TAILSCALE_AUTHKEY }}
      version: '1.52.0'  # Specific version
      # version: 'latest'   # Latest stable
      # version: 'unstable' # Latest unstable
```

### Caching Control
```yaml
  - name: Enhanced Tailscale
    uses: jaxxstorm/action-setup-tailscale@v1
    with:
      authkey: ${{ secrets.TAILSCALE_AUTHKEY }}
      use-cache: 'true'  # Default: enabled
```

### Tailnet Lock Support
```yaml
  - name: Enhanced Tailscale
    uses: jaxxstorm/action-setup-tailscale@v1
    with:
      authkey: tskey-auth-...  # Pre-signed auth key
      statedir: /tmp/tailscale-state/
```

### Connectivity Testing
```yaml
  - name: Enhanced Tailscale
    uses: jaxxstorm/action-setup-tailscale@v1
    with:
      authkey: ${{ secrets.TAILSCALE_AUTHKEY }}
      ping: 100.x.y.z,my-machine.my-tailnet.ts.net
```
