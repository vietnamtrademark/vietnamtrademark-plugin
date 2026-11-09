# VietnamTrademark plugin for Claude

Look up trademarks filed and registered in Vietnam, and notices in the Vietnam Industrial Property Gazette, from Claude Code. Data comes from [VietnamTrademark.net](https://vietnamtrademark.net) through the VietnamTrademark MCP server and is updated with each gazette issue.

The plugin contains:

- **MCP server** `vietnamtrademark` (`https://mcp.vietnamtrademark.net/mcp`): read-only tools `search_trademarks`, `get_trademark_detail`, `search_gazettes`, `get_gazette_detail`, `get_reference_categories`, `get_search_guide`, and 3 prompts (`check_trademark`, `trademark_status`, `monitor_gazette`).
- **Skill** `vietnamtrademark-search`: how to route a question to the right tool, a narrow-to-broad workflow for similar-mark clearance checks, a 6-level registrability report for clients (`references/registrability-report.md`), call budgets and answer format.

## Install

```
/plugin marketplace add vietnamtrademark/vietnamtrademark-plugin
/plugin install vietnamtrademark@vietnamtrademark
```

Then run `/mcp`, choose `vietnamtrademark` and sign in with your VietnamTrademark.net account. Each successful `search_*` or `get_*_detail` call uses one call from your account quota; `get_reference_categories` and `get_search_guide` are free. Plans and quotas: https://vietnamtrademark.net/mcp-guide

## Example questions

- "Is the mark PHỞ THÌN registered in Vietnam, and who owns it?"
- "Check whether the name MINH AN is safe to file for coffee in class 30."
- "What is the status of application 4-2023-123456?"
- "List new applications published for Công ty ABC in the gazette this month."

## Tiếng Việt

Plugin tra cứu nhãn hiệu đã nộp đơn và đã đăng ký tại Việt Nam và công báo sở hữu công nghiệp qua VietnamTrademark MCP. Sau khi cài, chạy `/mcp` và đăng nhập tài khoản vietnamtrademark.net.

## License

Apache License 2.0, see [LICENSE](LICENSE). The license covers the files in this repository. It does not grant rights to the VietnamTrademark name or to the data served by VietnamTrademark MCP.

Support: contact@vietnamtrademark.net
