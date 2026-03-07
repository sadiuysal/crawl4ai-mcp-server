# Crawl4AI MCP Server

🕷️ **A lightweight Model Context Protocol (MCP) server that exposes [Crawl4AI](https://docs.crawl4ai.com/) web scraping and crawling capabilities as tools for AI agents.**

Similar to Firecrawl's API but self-hosted and free. Perfect for integrating web scraping into your AI workflows with OpenAI Agents SDK, Cursor, Claude Code, and other MCP-compatible tools.

## Features

- **🔧 MCP Tools**: Exposes 4 powerful tools: `scrape`, `crawl`, `crawl_site`, `crawl_sitemap` via stdio MCP server
- **🌐 Web Scraping**: Single-page scraping with markdown extraction
- **🕷️ Web Crawling**: Multi-page breadth-first crawling with depth control
- **🧠 Adaptive Crawling**: Smart crawling that stops when sufficient content is gathered
- **🛡️ Safety**: Blocks internal networks, localhost, and private IPs
- **📱 Agent Ready**: Works with OpenAI Agents SDK, Cursor, and Claude Code
- **⚡ Fast**: Powered by Playwright and Crawl4AI's optimized extraction

## 🚀 Quick Start

Choose between **Docker** (recommended) or **manual installation**:

### Option A: Docker (Recommended) 🐳

Docker eliminates all setup complexity and provides a consistent environment:

#### Option A1: Use Pre-built Image (Fastest) ⚡

```bash
# Pull from GitHub Container Registry (recommended - auto-built for AMD64 and ARM64)
docker pull ghcr.io/uysalsadi/crawl4ai-mcp-server:latest

# Or from Docker Hub
docker pull uysalsadi/crawl4ai-mcp-server:latest

# Test it works
python test-config.py

# Use directly in MCP configurations (see examples below)
```

#### Using GitHub Container Registry (ghcr.io)

This repository uses GitHub Actions to automatically build and push multi-architecture Docker images (AMD64 + ARM64) to the GitHub Container Registry.

**Image location:** `ghcr.io/uysalsadi/crawl4ai-mcp-server`

**Available tags:**
- `latest` - Latest build from main branch
- `v1.0.0`, `v1.0.1`, etc. - Versioned releases
- `1.0`, `1` - Major/minor version tags
- `<sha>` - Commit SHA tag

**Pull specific architecture:**
```bash
# Auto-selects your system architecture
docker pull ghcr.io/uysalsadi/crawl4ai-mcp-server:latest

# Explicitly pull AMD64 (Intel/AMD)
docker pull --platform linux/amd64 ghcr.io/uysalsadi/crawl4ai-mcp-server:latest

# Explicitly pull ARM64 (Apple Silicon, Raspberry Pi, ARM servers)
docker pull --platform linux/arm64 ghcr.io/uysalsadi/crawl4ai-mcp-server:latest
```

#### Option A2: Build Yourself
```bash
# Clone the repository
git clone https://github.com/uysalsadi/crawl4ai-mcp-server.git
cd crawl4ai-mcp-server

# Quick build and test (simplified)
docker build -f Dockerfile.simple -t crawl4ai-mcp .
echo '{"jsonrpc": "2.0", "id": 1, "method": "initialize", "params": {"protocolVersion": "2024-11-05", "capabilities": {}, "clientInfo": {"name": "test", "version": "1.0"}}}' | docker run --rm -i crawl4ai-mcp

# Or use helper script (full build with Playwright)
./docker-run.sh build
./docker-run.sh test
./docker-run.sh run
```

**Docker Quick Commands:**
- `./docker-run.sh build` - Build the image
- `./docker-run.sh run` - Run MCP server (stdio mode)
- `./docker-run.sh test` - Run smoke tests
- `./docker-run.sh dev` - Development mode with shell access

### Option B: Manual Installation

```bash
# Clone and setup
git clone https://github.com/uysalsadi/crawl4ai-mcp-server.git
cd crawl4ai-mcp-server
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
pip install -r requirements.txt

# Install Playwright browsers
python -m playwright install chromium

# Test basic functionality
python -m crawler_agent.smoke_client

# Test adaptive crawling
python test_adaptive.py
```

### Use with OpenAI Agents SDK

```bash
# Set your OpenAI API key
export OPENAI_API_KEY="your-key-here"

# Docker: Run the example agent
docker-compose run --rm -e OPENAI_API_KEY crawl4ai-mcp python -m crawler_agent.agents_example

# Manual: Run the example agent
python -m crawler_agent.agents_example
```

## 🛠️ Tools Reference

The MCP server exposes 4 production-ready tools with content hiding features:

### `scrape`

Fetch a single URL and return markdown content.

**Arguments:**
- `url` (required): The URL to scrape
- `output_dir` (optional): If provided, persists content to disk and returns metadata only
- `crawler`: Optional Crawl4AI crawler config overrides  
- `browser`: Optional Crawl4AI browser config overrides
- `script`: Optional C4A-Script for page interaction
- `timeout_sec`: Request timeout (default: 45s, max: 600s)

**Returns (without output_dir):**
```json
{
  "url": "https://example.com",
  "markdown": "# Page Title\n\nContent...",
  "links": ["https://example.com/page1", "..."],
  "metadata": {}
}
```

**Returns (with output_dir):**
```json
{
  "run_id": "scrape_20250815_122811_4fb188",
  "file_path": "/path/to/output/pages/example.com_index.md",
  "manifest_path": "/path/to/output/manifest.json",
  "bytes_written": 230
}
```

### `crawl`

Multi-page breadth-first crawling with filtering and adaptive stopping.

**Arguments:**
- `seed_url` (required): Starting URL for the crawl
- `max_depth`: Maximum link depth to follow (default: 1, max: 4)
- `max_pages`: Maximum pages to crawl (default: 5, max: 100)
- `same_domain_only`: Stay within the same domain (default: true)
- `include_patterns`: Regex patterns URLs must match
- `exclude_patterns`: Regex patterns to exclude URLs
- `adaptive`: Enable adaptive crawling (default: false)
- `output_dir` (optional): If provided, persists content to disk and returns metadata only
- `crawler`, `browser`, `script`, `timeout_sec`: Same as scrape

**Returns (without output_dir):**
```json
{
  "start_url": "https://example.com",
  "pages": [
    {
      "url": "https://example.com/page1",
      "markdown": "Content...",
      "links": ["..."]
    }
  ],
  "total_pages": 3
}
```

**Returns (with output_dir):**
```json
{
  "run_id": "crawl_20250815_122828_464944",
  "pages_ok": 3,
  "pages_failed": 0,
  "manifest_path": "/path/to/output/manifest.json",
  "bytes_written": 690
}
```

### `crawl_site`

Comprehensive site crawling with persistence (always requires output_dir).

**Arguments:**
- `entry_url` (required): Starting URL for site crawl
- `output_dir` (required): Directory to persist results
- `max_depth`: Maximum crawl depth (default: 2, max: 6)
- `max_pages`: Maximum pages to crawl (default: 200, max: 5000)
- Additional config options for filtering and performance

**Returns:**
```json
{
  "run_id": "site_20250815_122851_0e2455",
  "output_dir": "/path/to/output",
  "manifest_path": "/path/to/output/manifest.json",
  "pages_ok": 15,
  "pages_failed": 2,
  "bytes_written": 45672
}
```

### `crawl_sitemap`

Sitemap-based crawling with persistence (always requires output_dir).

**Arguments:**
- `sitemap_url` (required): URL to sitemap.xml
- `output_dir` (required): Directory to persist results  
- `max_entries`: Maximum sitemap entries to process (default: 1000)
- Additional config options for filtering and performance

**Returns:**
```json
{
  "run_id": "sitemap_20250815_123006_667d71",
  "output_dir": "/path/to/output",
  "manifest_path": "/path/to/output/manifest.json", 
  "pages_ok": 25,
  "pages_failed": 0,
  "bytes_written": 123456
}
```

## 💡 Usage Examples

### Standalone MCP Server

```python
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

params = StdioServerParameters(
    command="python",
    args=["-m", "crawler_agent.mcp_server"]
)

async with stdio_client(params) as (read, write):
    async with ClientSession(read, write) as session:
        await session.initialize()
        
        # Scrape a single page
        result = await session.call_tool("scrape", {
            "url": "https://example.com"
        })
        
        # Crawl with adaptive stopping
        result = await session.call_tool("crawl", {
            "seed_url": "https://docs.example.com",
            "max_pages": 10,
            "adaptive": True
        })
```

### OpenAI Agents SDK Integration

```python
from agents import Agent, Runner
from agents.mcp.server import MCPServerStdio

async with MCPServerStdio(
    params={
        "command": "python", 
        "args": ["-m", "crawler_agent.mcp_server"]
    },
    cache_tools_list=True
) as server:
    
    agent = Agent(
        name="Research Assistant",
        instructions="Use scrape and crawl tools to research topics.",
        mcp_servers=[server],
        mcp_config={"convert_schemas_to_strict": True}
    )
    
    result = await Runner.run(
        agent, 
        "Research the latest AI safety papers"
    )
```

### Cursor/Claude Code Integration

This project supports both **Cursor** and **Claude Code** with synchronized configuration:

#### Cursor Setup
- Uses `.cursorrules` for AI assistant behavior and project guidance
- Automatic MCP tool detection and integration
- Project-specific rules and workflow guidance

#### Claude Code Setup  
- Uses `CLAUDE.md` for comprehensive project context and memory bank
- Native MCP integration via `mcp__crawl4ai-mcp__*` tool calls
- Global config: `~/.claude/claude_desktop_config.json` or project config: `.mcp.json`

#### Docker Integration for Both Environments
Both Cursor and Claude Code can use the Dockerized MCP server:

```json
// .mcp.json (project-scoped configuration)
// Using GitHub Container Registry (ghcr.io) - recommended
{
  "mcpServers": {
    "crawl4ai-mcp": {
      "command": "docker",
      "args": [
        "run", 
        "--rm", 
        "-i",
        "--volume", 
        "./crawls:/app/crawls",
        "ghcr.io/uysalsadi/crawl4ai-mcp-server:latest"
      ],
      "env": {
        "CRAWL4AI_MCP_LOG": "INFO"
      }
    }
  }
}
```

**Alternative: Docker Hub (ARM Only)**
```json
{
  "mcpServers": {
    "crawl4ai-mcp": {
      "command": "docker",
      "args": [
        "run", 
        "--rm", 
        "-i",
        "--volume", 
        "./crawls:/app/crawls",
        "uysalsadi/crawl4ai-mcp-server:latest"
      ],
      "env": {
        "CRAWL4AI_MCP_LOG": "INFO"
      }
    }
  }
}
```

#### Dual Environment Features
- **Synchronized Documentation**: Changes to documentation are maintained across both environments
- **Shared MCP Configuration**: Both use the same MCP server and tool schemas
- **Docker Compatibility**: Consistent environment across both Cursor and Claude Code
- **Cross-Compatible**: Projects work seamlessly whether using Cursor or Claude Code

## 🐳 Docker Usage

### Quick Start with Docker

The Docker approach eliminates all manual setup and provides a consistent environment:

```bash
# 1. Clone and build
git clone https://github.com/uysalsadi/crawl4ai-mcp-server.git
cd crawl4ai-mcp-server
./docker-run.sh build

# 2. Test the installation
./docker-run.sh test

# 3. Run MCP server
./docker-run.sh run
```

### Docker Commands Reference

| Command | Description |
|---------|-------------|
| `./docker-run.sh build` | Build the Docker image |
| `./docker-run.sh run` | Run MCP server in stdio mode |
| `./docker-run.sh test` | Run smoke tests |
| `./docker-run.sh dev` | Development mode with shell access |
| `./docker-run.sh stop` | Stop running containers |
| `./docker-run.sh clean` | Remove containers and images |
| `./docker-run.sh logs` | Show container logs |

### Docker Compose Services

The `docker-compose.yml` provides multiple service configurations:

- **`crawl4ai-mcp`**: Production MCP server
- **`crawl4ai-mcp-dev`**: Development container with shell access  
- **`crawl4ai-mcp-test`**: Test runner for smoke tests

### MCP Integration with Docker

#### Global MCP Configuration (Claude Code)

Add to `~/.claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "crawl4ai-mcp": {
      "command": "docker",
      "args": [
        "run", 
        "--rm", 
        "-i", 
        "--volume", 
        "/tmp/crawl4ai-crawls:/app/crawls",
        "ghcr.io/uysalsadi/crawl4ai-mcp-server:latest"
      ],
      "env": {
        "CRAWL4AI_MCP_LOG": "INFO"
      }
    }
  }
}
```

**Alternative: Docker Hub**
```json
{
  "mcpServers": {
    "crawl4ai-mcp": {
      "command": "docker",
      "args": [
        "run", 
        "--rm", 
        "-i", 
        "--volume", 
        "/tmp/crawl4ai-crawls:/app/crawls",
        "uysalsadi/crawl4ai-mcp-server:latest"
      ],
      "env": {
        "CRAWL4AI_MCP_LOG": "INFO"
      }
    }
  }
}
```

**✅ Copy this exact config** - it uses the published Docker image!

#### Global MCP Configuration (Cursor)

Add to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "crawl4ai-mcp": {
      "command": "docker",
      "args": [
        "run", 
        "--rm", 
        "-i",
        "--volume", 
        "/tmp/crawl4ai-crawls:/app/crawls", 
        "ghcr.io/uysalsadi/crawl4ai-mcp-server:latest"
      ],
      "env": {
        "CRAWL4AI_MCP_LOG": "INFO"
      }
    }
  }
}
```

**Alternative: Docker Hub**
```json
{
  "mcpServers": {
    "crawl4ai-mcp": {
      "command": "docker",
      "args": [
        "run", 
        "--rm", 
        "-i",
        "--volume", 
        "/tmp/crawl4ai-crawls:/app/crawls", 
        "uysalsadi/crawl4ai-mcp-server:latest"
      ],
      "env": {
        "CRAWL4AI_MCP_LOG": "INFO"
      }
    }
  }
}
```

**✅ This configuration is tested and working!**

#### Project-Scoped Configuration

Add to your project's `.mcp.json`:

```json
{
  "mcpServers": {
    "crawl4ai-mcp": {
      "command": "docker",
      "args": [
        "run",
        "--rm", 
        "-i",
        "--volume", 
        "./crawls:/app/crawls",
        "ghcr.io/uysalsadi/crawl4ai-mcp-server:latest"
      ],
      "env": {
        "CRAWL4AI_MCP_LOG": "INFO"
      }
    }
  }
}
```

**Alternative: Docker Hub**
```json
{
  "mcpServers": {
    "crawl4ai-mcp": {
      "command": "docker",
      "args": [
        "run", 
        "--rm", 
        "-i",
        "--volume", 
        "./crawls:/app/crawls",
        "uysalsadi/crawl4ai-mcp-server:latest"
      ],
      "env": {
        "CRAWL4AI_MCP_LOG": "INFO"
      }
    }
  }
}
```

**✅ This project already includes this configuration** - see `.mcp.json`

#### Configuration Validation

After setting up any MCP configuration:

1. **Test the Docker image works**:
   ```bash
   python test-config.py
   ```

2. **Restart your editor** (Cursor/Claude Code) to reload MCP configuration

3. **Verify tools are available**:
   - Look for `crawl4ai-mcp` in the MCP tools panel
   - Should see 4 tools: `scrape`, `crawl`, `crawl_site`, `crawl_sitemap`

If tools don't appear, check:
- Docker is running and image is accessible
- MCP configuration file syntax is valid JSON
- Editor has been restarted after config changes

### Docker Advantages

✅ **Zero Setup**: No need for Python venv, pip, or Playwright installation  
✅ **Consistent Environment**: Same behavior across all platforms  
✅ **Isolated Dependencies**: No conflicts with your system Python  
✅ **Easy Updates**: `docker pull` to get latest version  
✅ **Portable**: Works anywhere Docker runs  
✅ **Volume Persistence**: Crawl outputs saved to host filesystem

### Environment Variables

Set these when running Docker containers:

```bash
# Using docker-compose
OPENAI_API_KEY=your-key docker-compose up crawl4ai-mcp

# Using docker run directly
docker run --rm -i \
  -e OPENAI_API_KEY=your-key \
  -e CRAWL4AI_MCP_LOG=DEBUG \
  -v ./crawls:/app/crawls \
  crawl4ai-mcp-server:latest
```

## ⚙️ Configuration

### Environment Variables

- `TARGET_URL`: Default URL for smoke testing (default: https://modelcontextprotocol.io/docs)
- `RESEARCH_TASK`: Custom research task for agents example
- `OPENAI_API_KEY`: Required for OpenAI Agents SDK

### Safety Settings

The server blocks these URL patterns by default:
- `localhost`, `127.0.0.1`, `::1`
- Private IP ranges (RFC 1918)
- `file://` schemes
- `.local`, `.internal`, `.lan` domains

## 🚀 Advanced Features

### Adaptive Crawling

When `adaptive: true` is set, the crawler uses a simple content-based stopping strategy:

- Stops when total content exceeds 5,000 characters
- Prevents over-crawling for information gathering tasks
- More sophisticated LLM-based strategies can be added

### Custom Filtering

Use regex patterns to control which URLs are crawled:

```python
await session.call_tool("crawl", {
    "seed_url": "https://docs.example.com",
    "include_patterns": [r"/docs/", r"/api/"],
    "exclude_patterns": [r"/old/", r"\.pdf$"]
})
```

### Browser Configuration

Pass custom browser/crawler settings:

```python
await session.call_tool("scrape", {
    "url": "https://example.com",
    "browser": {"headless": True, "viewport": {"width": 1280, "height": 720}},
    "crawler": {"verbose": True}
})
```

## 🏗️ Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   AI Agent      │───▶│   MCP Server     │───▶│   Crawl4AI      │
│ (Cursor/Agents) │    │ (stdio/stdio)    │    │ (Playwright)    │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌──────────────────┐
                       │  Safety Guards   │
                       │ (URL validation) │
                       └──────────────────┘
```

## ⚙️ GitHub Actions - Auto-Build

This repository uses GitHub Actions to automatically build and push multi-architecture Docker images.

### What It Builds

- **linux/amd64** - For Intel and AMD processors (most common)
- **linux/arm64** - For Apple Silicon (M1/M2/M3), Raspberry Pi, and ARM servers

### When It Builds

- On every push to `main` or `master` branch
- On every pull request to `main` or `master`
- On every new tag starting with `v` (e.g., `v1.0.0`)

### How It Works

The workflow (located at `.github/workflows/docker-build.yml`):
1. Checks out the code
2. Sets up QEMU for ARM64 emulation
3. Sets up Docker Buildx
4. Logs in to GitHub Container Registry (ghcr.io)
5. Builds and pushes multi-arch images with caching

### Image Location

Images are pushed to: `ghcr.io/uysalsadi/crawl4ai-mcp-server`

### Usage

```bash
# Pull latest (auto-selects architecture)
docker pull ghcr.io/uysalsadi/crawl4ai-mcp-server:latest

# Pull specific version
docker pull ghcr.io/uysalsadi/crawl4ai-mcp-server:v1.0.0

# Pull specific architecture
docker pull --platform linux/amd64 ghcr.io/uysalsadi/crawl4ai-mcp-server:latest
docker pull --platform linux/arm64 ghcr.io/uysalsadi/crawl4ai-mcp-server:latest
```

## 📦 Publishing to Docker Hub

For maintainers who want to publish updates to the Docker registry:

### Publishing Process

```bash
# 1. Login to Docker Hub (one time setup)
./docker-push.sh login

# 2. Build, push, and test everything
./docker-push.sh all

# Or do steps individually:
./docker-push.sh build    # Build and tag image
./docker-push.sh push     # Push to Docker Hub
./docker-push.sh test     # Test the published image
```

### Docker Hub Repository

The image is published at: **`uysalsadi/crawl4ai-mcp-server`**
- Latest: `uysalsadi/crawl4ai-mcp-server:latest`
- Versioned: `uysalsadi/crawl4ai-mcp-server:v1.0.0`

### Usage Statistics

Users can pull and use the image without any local setup:
```bash
docker pull uysalsadi/crawl4ai-mcp-server:latest
```

## 🔧 Development

### Project Structure

```
crawler_agent/
├── __init__.py
├── mcp_server.py         # Main MCP server
├── safety.py            # URL safety validation
├── adaptive_strategy.py  # Adaptive crawling logic
├── smoke_client.py       # Basic testing client
└── agents_example.py     # OpenAI Agents SDK example
```

### Testing

```bash
# Test MCP server
python -m crawler_agent.smoke_client

# Test adaptive crawling
python test_adaptive.py

# Test with agents (requires OPENAI_API_KEY)
python -m crawler_agent.agents_example
```

### Contributing

1. **Environment Setup**: Always activate `.venv` before development work
2. **Documentation**: Follow the Documentation Memory Rule - synchronize changes across CLAUDE.md, .cursorrules, and README.md
3. **Code Style**: Follow `.cursorrules` for Cursor or `CLAUDE.md` for Claude Code
4. **Testing**: Use `python -m crawler_agent.smoke_client` for validation
5. **Safety**: Ensure security guards are maintained for all new features
6. **Dual Compatibility**: Verify changes work in both Cursor and Claude Code environments

## 📚 References

- [Crawl4AI Documentation](https://docs.crawl4ai.com/)
- [Model Context Protocol](https://modelcontextprotocol.io/introduction)
- [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/mcp/)
- [MCP Tools Reference](https://openai.github.io/openai-agents-python/tools/)

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details.

### 🙏 Acknowledgments

This project uses [Crawl4AI](https://github.com/unclecode/crawl4ai) by [UncleCode](https://github.com/unclecode) for web scraping capabilities. Crawl4AI is an excellent open-source LLM-friendly web crawler and scraper.

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request. For major changes, please open an issue first to discuss what you would like to change.

## ⭐ Support

If this project helped you, please give it a star! It helps others discover the project.

## 🐛 Issues

Found a bug or have a feature request? Please [open an issue](https://github.com/uysalsadi/crawl4ai-mcp-server/issues).
