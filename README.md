# ChatGPT Automation MCP

A Model Context Protocol (MCP) server for automating ChatGPT web interface using Playwright. Provides programmatic control over ChatGPT conversations including model selection, message sending, and response retrieval.

## Features

- 🌐 **Browser Automation**: Controls ChatGPT web interface via Playwright
- 🤖 **Model Selection**: Switch between GPT-5.1, GPT-5.1 Instant, GPT-5.1 Thinking, and legacy models (o3, o4-mini, GPT-4.5, etc.)
- 💬 **Chat Management**: Create chats, send messages, get responses with explicit chat ID safety
- 🔄 **Session Persistence**: Maintain login state across sessions using Chrome profiles
- 🛡️ **Chat ID Safety**: All operations require explicit chat ID to prevent cross-chat contamination
- 🔍 **Auto Web Search**: Automatically enables web search for research-related queries
- 📤 **Export Chats**: Save chats in markdown or JSON format
- 🔧 **Comprehensive Error Recovery**: Automatic handling of network errors, timeouts, browser crashes
- ⚡ **Batch Operations**: Execute multiple operations in sequence for efficiency

## Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/chatgpt-automation-mcp.git
cd chatgpt-automation-mcp

# Install with uv (recommended)
uv sync

# Or install with pip
pip install -e .

# Install Playwright browsers
uv run playwright install chromium
```

### Browser Configuration

The MCP uses Chrome with a separate automation profile via Chrome DevTools Protocol (CDP). This allows:
- Maintaining ChatGPT login across sessions
- Browser automation without interfering with your personal Chrome instance
- Automatic profile creation from your default Chrome profile

**Profile Location**: `~/Library/Application Support/Google/Chrome-Automation` (macOS)

If you encounter issues, simply delete the automation profile - it will be recreated automatically.

## Configuration

1. Copy `.env.example` to `.env`:
```bash
cp .env.example .env
```

2. Edit `.env` with your settings:
```env
# Optional - browser may already be logged in
CHATGPT_EMAIL=your_email@example.com
CHATGPT_PASSWORD=your_password

# Browser settings
HEADLESS=false
BROWSER_TIMEOUT=30000

# Session persistence
PERSIST_SESSION=true
SESSION_NAME=default
```

## Usage

### As MCP Server

Add to your Claude desktop configuration:

```json
{
  "mcpServers": {
    "chatgpt": {
      "command": "uv",
      "args": ["run", "chatgpt-mcp"],
      "cwd": "/path/to/chatgpt-automation-mcp"
    }
  }
}
```

### Available Tools

#### Core Operations
- `chatgpt_launch` - Launch ChatGPT in browser
- `chatgpt_status` - Check if ChatGPT is ready
- `chatgpt_get_model` - Check current model
- `chatgpt_select_model` - Switch to a different model (gpt-5.1, gpt-5.1-instant, gpt-5.1-thinking, etc.)

#### Chat Management (require chat_id)
- `chatgpt_new_chat` - **Start a new chat** (returns chat_id + first response)
- `chatgpt_send_message` - Send a message to a specific chat
- `chatgpt_send_and_get_response` - Send message and wait for response (auto-enables web search)
- `chatgpt_wait_response` - Wait for response completion in a specific chat
- `chatgpt_get_last_response` - Get the most recent response from a specific chat
- `chatgpt_get_chat` - Get full chat history
- `chatgpt_upload_file` - Upload a file to a specific chat
- `chatgpt_edit_message` - Edit a previous user message in a specific chat

#### Export & Organization
- `chatgpt_export_chat` - Export chat as markdown or JSON
- `chatgpt_save_chat` - Save chat to file
- `chatgpt_list_chats` - List all available chats
- `chatgpt_delete_chat` - Delete a chat

#### Advanced Features
- `chatgpt_enable_think_longer` - Enable Think Longer mode for enhanced reasoning
- `chatgpt_enable_deep_research` - Enable Deep Research mode (250/month quota)
- `chatgpt_batch_operations` - Execute multiple operations in sequence

### Chat ID Safety

**IMPORTANT**: All chat operations now require explicit `chat_id` parameter. This prevents AI agents from accidentally sending messages to the wrong chat.

**Workflow**:
1. Create a new chat with `chatgpt_new_chat` → get `chat_id`
2. Use that `chat_id` for all subsequent operations

```python
# Create a new chat
result = await chatgpt_new_chat(message="Hello!")
# Returns: "Chat ID: 12345-67890-abcdef\n\nResponse: Hi there!"

# Extract chat_id
chat_id = "12345-67890-abcdef"

# Use chat_id for all operations
await chatgpt_send_message(
    message="What's 2+2?",
    chat_id=chat_id
)

response = await chatgpt_get_last_response(chat_id=chat_id)
```

### Example Usage

```python
# Basic conversation flow with chat ID safety
await chatgpt_launch()
await chatgpt_select_model(model="gpt-5.1-instant")

# Create new chat and get chat_id
result = await chatgpt_new_chat(
    message="Explain quantum computing",
    project_name="MCP-Automation"
)
# Returns: "Chat ID: abc123...\n\nResponse: Quantum computing is..."

# Extract chat_id from result
chat_id = "abc123..."  # Parse from result

# Send follow-up message
response = await chatgpt_send_and_get_response(
    message="How does quantum entanglement work?",
    chat_id=chat_id,
    timeout=120
)

# Export the chat
await chatgpt_export_chat(
    chat_id=chat_id,
    format="markdown"
)

# Enhanced reasoning with Think Longer
await chatgpt_enable_think_longer()
response = await chatgpt_send_and_get_response(
    message="Solve this complex logic puzzle...",
    chat_id=chat_id,
    timeout=600  # Extended timeout for thinking
)

# Deep Research for comprehensive information
await chatgpt_enable_deep_research()
response = await chatgpt_send_and_get_response(
    message="Research the history and impact of quantum computing",
    chat_id=chat_id,
    timeout=3600  # Deep research can take up to an hour
)

# Batch operations for efficiency
batch_result = await chatgpt_batch_operations(operations=[
    {"operation": "new_chat", "args": {"message": "Start analysis"}},
    {"operation": "select_model", "args": {"model": "gpt-5.1-thinking"}},
    {"operation": "enable_think_longer"},
    {"operation": "send_and_get_response", "args": {
        "message": "Analyze this complex problem step by step",
        "chat_id": chat_id,
        "timeout": 900
    }},
    {"operation": "save_chat", "args": {
        "chat_id": chat_id,
        "filename": "analysis"
    }},
])
```

## Smart Features

### Auto-Enable Web Search

The `chatgpt_send_and_get_response` tool automatically enables web search when your message contains research-related keywords:

**Trigger Keywords:**
- `research`, `latest`, `current`, `recent`, `2025`, `2024`, `2026`
- `update`, `new`, `find`, `search`, `discover`, `investigate`
- `what's new`, `recent changes`, `current state`, `up to date`

**Example:**
```python
# This will automatically enable web search
response = await chatgpt_send_and_get_response(
    message="What are the latest developments in AI?",
    chat_id=chat_id
)

# This will not trigger auto-enable
response = await chatgpt_send_and_get_response(
    message="Write a Python function to sort a list",
    chat_id=chat_id
)
```

## Available Models (November 2025)

### Current Models

#### GPT-5.1 Family (Latest)
- **gpt-5.1** / **5.1** - Latest flagship model
- **gpt-5.1-instant** / **instant** - Fast responses
- **gpt-5.1-thinking** / **thinking** - Extended reasoning (15 min timeout)
- **auto** - Automatically choose best model

#### GPT-5 Family
- **gpt-5** / **5** - Previous flagship model
- **gpt-5-instant** - Fast responses
- **gpt-5-thinking** - Extended reasoning
- **gpt-5-t-mini** / **thinking-mini** - Lightweight reasoning
- **gpt-5-pro** - Research-grade intelligence (30 min timeout)

#### Legacy Models
- **o3** - Advanced reasoning model (60+ min timeout)
- **o3-pro** - Expert reasoning (can take hours)
- **o4-mini** - Lightweight model
- **gpt-4.5** - Previous generation
- **gpt-4.1** / **4.1** / **4-1** - Large context window
- **gpt-4.1-mini** - Lightweight GPT-4
- **gpt-4o** / **4o** - Multimodal model

### Model Selection Tips
- **Default choice**: `gpt-5.1-instant` for fast responses
- **Complex reasoning**: `gpt-5.1-thinking` or `o3` for deep analysis
- **Critical research**: `gpt-5-pro` or `o3-pro` for sophisticated reasoning
- **Quick access**: Use shortcuts like `instant`, `thinking`, `auto`

## Development

### Installing Dependencies

This project uses UV for dependency management:

```bash
# Install all dependencies including dev dependencies
uv sync --dev
```

### Running Tests

```bash
# Run unit tests
uv run pytest tests/test_browser_controller.py -v

# Run all tests with coverage
uv run pytest --cov=src/chatgpt_automation_mcp --cov-report=html

# Run specific test
uv run pytest -k test_sidebar_handling -v

# Run functional test (requires browser)
uv run python tests/test_functional.py

# Or use the test runner script
uv run run-tests --unit        # Unit tests only
uv run run-tests --integration # Integration tests (requires browser)
uv run run-tests --auto-enable # Auto-enable web search tests
uv run run-tests --all         # All tests
uv run run-tests --coverage    # With coverage report
```

### Test Structure

- `tests/test_browser_controller.py` - Unit tests with mocked browser
- `tests/test_integration.py` - Integration tests with real browser
- `tests/test_functional.py` - Quick smoke test for basic functionality
- `tests/test_auto_enable_search.py` - Auto-enable web search keyword detection
- `tests/test_server_auto_enable.py` - Server integration tests
- `tests/test_web_search_verification.py` - Visual verification with screenshots
- `tests/test_deep_research_verification.py` - Deep Research feature verification
- `tests/test_think_longer_verification.py` - Think Longer feature verification
- `run_tests.py` - Test runner with various options

### Code Quality

```bash
# Format code
uv run ruff format .

# Check linting
uv run ruff check .

# Fix linting issues
uv run ruff check . --fix
```

### Development Best Practices

See [docs/DEVELOPMENT_BEST_PRACTICES.md](docs/DEVELOPMENT_BEST_PRACTICES.md) for important lessons learned about:
- Using Playwright methods (`get_by_role`, `get_by_test_id`) instead of `locator()`
- Visual verification testing with screenshots
- Handling ChatGPT UI changes
- Chrome profile management and troubleshooting
- Common pitfalls and debugging tips

## Error Handling & Recovery

The MCP server includes comprehensive error handling and automatic recovery:

### Automatic Recovery Scenarios
- **Network Errors**: Automatic retry with exponential backoff
- **Browser Crashes**: Automatic browser restart and session restoration
- **Session Expiration**: Automatic re-authentication when needed
- **Element Not Found**: Page refresh and element waiting
- **Timeout Errors**: Extended waiting and page responsiveness checks

### Error Types Handled
- Connection failures and network instability
- ChatGPT UI changes and element location issues
- Browser crashes and unexpected closures
- Authentication token expiration
- Page loading timeouts and delays

### Recovery Strategies
- **Exponential Backoff**: Progressively longer delays between retries
- **Multiple Selectors**: Fallback element selectors for UI changes
- **Session Restoration**: Automatic login when sessions expire
- **Context Preservation**: Maintain conversation state during recovery
- **Graceful Degradation**: Continue operation even with partial failures

## Troubleshooting

### Chrome Profile Issues

**Problem**: Chrome opens folder listing instead of ChatGPT
- **Cause**: Automation profile is corrupted or command line args are wrong
- **Fix**: Delete the automation profile
  ```bash
  rm -rf ~/Library/Application\ Support/Google/Chrome-Automation
  ```
  The MCP will recreate it automatically from your default Chrome profile.

**Problem**: "Not logged in to ChatGPT" after profile recreation
- **Expected behavior**: First run with new profile requires login
- Login to ChatGPT in the automation browser once
- Session will persist for all future runs

### Browser Session Issues

**Problem**: Browser won't launch
- Ensure Playwright Chromium is installed: `uv run playwright install chromium`
- Try with `HEADLESS=false` to see the browser window
- Check Chrome is installed and working
- Delete automation profile and let it recreate

### Response Detection
- The tool waits for ChatGPT's thinking animation to complete
- Increase timeout for complex queries or reasoning models
  - GPT-5.1 Thinking: 15+ minutes
  - GPT-5 Pro: 30+ minutes
  - o3: 60+ minutes
  - o3-pro: Can take hours
- Enable debug mode with `CHATGPT_LOG_LEVEL=DEBUG`

### Model Selection
- Not all models may be available to your account
- Some models require ChatGPT Plus/Pro subscription
- Model names are case-sensitive
- Use shortcuts when available: `instant`, `thinking`, `auto`

### Chat ID Issues
- **Error**: "Failed to navigate to chat {chat_id}"
  - Check the chat_id is valid (from `chatgpt_new_chat` or `chatgpt_list_chats`)
  - Chat may have been deleted
  - Use `chatgpt_list_chats` to see available chats

### Auto-Enable Web Search Issues
- Feature only works with `chatgpt_send_and_get_response` tool
- Check message contains research keywords: `latest`, `current`, `research`, etc.
- Web search may already be enabled (not an error)

### Error Recovery
- Error recovery is automatic and logged at INFO level
- Set `CHATGPT_LOG_LEVEL=DEBUG` for detailed recovery information
- Recovery attempts are limited to prevent infinite loops
- Browser restart is the ultimate fallback for persistent issues

## Security

- Credentials are stored in `.env` (never commit this file)
- Browser sessions are isolated in separate Chrome profile
- Screenshots on error are stored locally only
- All temporary files are gitignored
- Chat IDs are required to prevent cross-chat access

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Run tests and linting
5. Submit a pull request

## License

MIT License - see LICENSE file for details
