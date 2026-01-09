# Claude Code MCP Server Setup Guide

This guide explains how to connect the talos-mcp-server to Claude Code.

---

## 🔧 Configuration Steps

### 1. Locate Claude Code Configuration File

Claude Code stores MCP server configurations in one of these locations:

**macOS/Linux:**
```bash
~/.claude/claude_code/mcp_servers.json
# or
~/.config/claude-code/mcp_servers.json
```

**Windows:**
```
%APPDATA%\Claude Code\mcp_servers.json
```

### 2. Add Server Configuration

Add this configuration to your `mcp_servers.json` file:

```json
{
  "mcpServers": {
    "talos-mcp": {
      "command": "/home/fedora/nexuro/talos-mcp-server/target/release/talos-mcp-server",
      "args": [],
      "env": {
        "TALOSCONFIG": "/path/to/your/talosconfig"
      }
    }
  }
}
```

**Replace these values:**
- `/home/fedora/nexuro/talos-mcp-server/target/release/talos-mcp-server` → Path to the compiled binary on your system
- `/path/to/your/talosconfig` → Path to your Talos configuration file (usually `~/.talos/config`)

### 3. Create Configuration File (if it doesn't exist)

If the configuration file doesn't exist, create it:

**macOS/Linux:**
```bash
# Create directory if needed
mkdir -p ~/.config/claude-code

# Create the config file with the JSON above
cat > ~/.config/claude-code/mcp_servers.json << 'EOF'
{
  "mcpServers": {
    "talos-mcp": {
      "command": "/home/fedora/nexuro/talos-mcp-server/target/release/talos-mcp-server",
      "args": [],
      "env": {
        "TALOSCONFIG": "/home/fedora/.talos/config"
      }
    }
  }
}
EOF
```

---

## ✅ Prerequisites

Before setting up the connection, ensure:

### 1. TALOSCONFIG is Set

```bash
# Check if TALOSCONFIG environment variable is set
echo $TALOSCONFIG

# Or verify the file exists
ls -la ~/.talos/config

# Set it if needed
export TALOSCONFIG=~/.talos/config
```

### 2. talosctl CLI is Installed

```bash
# Verify talosctl is installed
which talosctl

# Check version
talosctl version --client
```

### 3. Talos Cluster Access Works

```bash
# Test that talosctl can reach your cluster
talosctl health

# Or list nodes
talosctl get nodes
```

### 4. Binary is Built

Verify the compiled binary exists:

```bash
ls -lh /home/fedora/nexuro/talos-mcp-server/target/release/talos-mcp-server
```

If not, build it first:

```bash
cd /home/fedora/nexuro/talos-mcp-server
cargo build --release
```

---

## 🧪 Verify Configuration

### Test the Binary Directly

```bash
export TALOSCONFIG=~/.talos/config

# Test MCP initialization
echo '{"jsonrpc":"2.0","method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}},"id":1}' | /home/fedora/nexuro/talos-mcp-server/target/release/talos-mcp-server
```

Expected output:
```json
{"jsonrpc":"2.0","result":{"capabilities":{"tools":{"listChanged":true}},"protocolVersion":"2025-06-18","serverInfo":{"name":"talos-mcp-server","title":"Talos OS MCP Server","version":"1.0.0"}},"id":1}
```

### Check Configuration File

```bash
# View your Claude Code MCP configuration
cat ~/.config/claude-code/mcp_servers.json
```

### List Available Tools

```bash
# Test tools/list endpoint
echo '{"jsonrpc":"2.0","method":"tools/list","params":{},"id":1}' | /home/fedora/nexuro/talos-mcp-server/target/release/talos-mcp-server
```

---

## 🚀 Using with Claude Code

Once configured:

1. **Restart Claude Code** - The MCP server will auto-discover on startup
2. **Open Claude Code** - The talos-mcp tools should appear in suggestions
3. **Use the tools** - You can now ask Claude Code to:
   - List containers: `containers` tool
   - Check health: `get_health` tool
   - Read files: `read` tool
   - View logs: `get_logs` tool
   - And 33 more tools...

### Example Usage

```
User: "Show me the running containers on node 192.168.1.77"

Claude Code will use the 'containers' tool with:
{
  "method": "containers",
  "params": {
    "node": "192.168.1.77"
  }
}
```

---

## 📚 Available Tools (37 Total)

The server exposes 37 tools for Talos cluster management:

| Category | Tools | Count |
|----------|-------|-------|
| **System Inspection** | containers, stats, processes, memory_verbose, get_cpu_memory_usage | 5 |
| **File Operations** | list, read, copy, get_usage, get_mounts | 5 |
| **Network** | interfaces, routes, get_netstat, capture_packets, get_network_io_cgroups, list_network_interfaces | 6 |
| **Services & Logs** | dmesg, service, restart, get_logs, get_events | 5 |
| **Storage** | disks, list_disks | 2 |
| **Cluster Management** | get_health, get_version, get_time | 3 |
| **Node Management** | reboot_node, shutdown_node, reset_node, upgrade_node, upgrade_k8s | 5 |
| **Configuration** | apply_config, validate_config | 2 |
| **etcd Management** | get_etcd_status, get_etcd_members, bootstrap_etcd, defrag_etcd | 4 |

See `README.md` for detailed tool documentation.

---

## 🔧 Troubleshooting

### Issue: Server not appearing in Claude Code

**Solution:**
1. Verify configuration file exists: `~/.config/claude-code/mcp_servers.json`
2. Check JSON syntax is valid
3. Restart Claude Code completely
4. Check logs (if available) in Claude Code settings

### Issue: "TALOSCONFIG env var not set"

**Solution:**
```bash
# Set environment variable in config file
export TALOSCONFIG=~/.talos/config

# Or add to mcp_servers.json:
"env": {
  "TALOSCONFIG": "/home/fedora/.talos/config"
}
```

### Issue: Binary fails to execute

**Solution:**
```bash
# Rebuild the binary
cd /home/fedora/nexuro/talos-mcp-server
cargo build --release

# Verify it works
./target/release/talos-mcp-server < /dev/null
# Should exit cleanly
```

### Issue: talosctl command not found

**Solution:**
```bash
# Install talosctl
# macOS
brew install talosctl

# Linux
curl -L https://github.com/siderolabs/talos/releases/latest/download/talosctl-linux-amd64 -o talosctl
chmod +x talosctl
sudo mv talosctl /usr/local/bin/
```

---

## 📝 Example Full Configuration

```json
{
  "mcpServers": {
    "talos-mcp": {
      "command": "/home/fedora/nexuro/talos-mcp-server/target/release/talos-mcp-server",
      "args": [],
      "env": {
        "TALOSCONFIG": "/home/fedora/.talos/config"
      }
    }
  }
}
```

---

## 🎯 Next Steps

1. ✅ Build the binary: `cargo build --release`
2. ✅ Verify TALOSCONFIG is set
3. ✅ Create/update Claude Code MCP configuration
4. ✅ Restart Claude Code
5. ✅ Verify tools appear in suggestions
6. ✅ Start using Talos management tools!

---

## 📖 Additional Resources

- **README.md** - Full project documentation
- **CLAUDE.md** - Development guidance
- **PROJECT_INDEX.md** - Code structure reference

---

**Happy cluster management with Claude Code! 🚀**
