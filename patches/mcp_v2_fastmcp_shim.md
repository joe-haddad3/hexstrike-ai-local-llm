# Patch: make HexStrike AI work with the mcp v2 library (for ollmcp)

## The problem

`ollmcp` (MCP Client for Ollama) needs the Python `mcp` library **version 2**.
HexStrike AI was written for `mcp` **version 1**: its MCP client imports
`FastMCP` from `mcp.server.fastmcp`, and version 2 renamed that to `MCPServer`.
So the two could not run on the same Python at the same time. With mcp v1
HexStrike worked but ollmcp broke, and with mcp v2 ollmcp worked but HexStrike
crashed on startup with:

```
ModuleNotFoundError: No module named 'mcp.server.fastmcp'
```

## The check

HexStrike only uses the standard parts of the library: the `FastMCP(...)`
constructor and the `@mcp.tool()` decorators. Those are the same in version 2
under the new name, so a small import shim is enough. No other code needs to
change.

## The fix

In `/usr/share/hexstrike-ai/hexstrike_mcp.py`, the single import line:

```python
from mcp.server.fastmcp import FastMCP
```

was replaced with a shim that works on both versions:

```python
try:
    # MCP SDK 2.x: FastMCP was renamed to MCPServer
    from mcp.server.mcpserver import MCPServer as FastMCP
except ImportError:
    # MCP SDK 1.x keeps the original path
    from mcp.server.fastmcp import FastMCP
```

Then upgrade the library:

```bash
pip install "mcp>=2" --break-system-packages
```

After this, the patched `hexstrike_mcp.py` still loads fine, and `ollmcp`
and HexStrike run on the same mcp v2 library. Keep a backup of the original
file before editing it.
