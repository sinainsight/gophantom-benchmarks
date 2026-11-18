# GoPhantom Benchmarks

Measured throughput benchmarks for [GoPhantom](https://www.sinainsight.pro/sinaapp/), the pure-Go browser engine by [SINAINSIGHT](https://www.sinainsight.pro).

GoPhantom is a browser engine written entirely in Go — **no Chromium, no WebKit, no browser binary**. HTML is parsed with [goquery](https://github.com/PuerkitoBio/goquery), JavaScript is executed with [goja](https://github.com/nickthereis/goja) (ES5.1), and HTTP is handled by `net/http`. Each scenario runs as a goroutine, not a process.

## API Surface

GoPhantom implements **70 endpoints/primitives**:

- **56 W3C WebDriver endpoints** — full specification coverage
- **14 TN3270 mainframe primitives** — CICS, IMS, TSO monitoring via s3270

Scenarios can mix web and mainframe primitives in the same script.

### W3C WebDriver API — 56 Endpoints

| Category | Count | Endpoints |
|----------|-------|-----------|
| Session | 3 | create, delete, status |
| Navigation | 5 | navigate, get URL, back, forward, refresh |
| Page | 2 | get title, get source |
| Find elements | 4 | find element, find elements, find from element, find elements from element |
| Element properties | 9 | text, attribute, property, displayed, enabled, selected, tag name, rect, CSS value |
| Interaction | 3 | click, clear, send keys |
| Execute JS | 2 | execute sync, execute async (via goja) |
| Cookies | 4 | get, add, delete one, delete all |
| Frames | 2 | switch to frame, switch to parent frame |
| Windows | 8 | get, close, switch, handles, maximize, minimize, fullscreen, get/set rect |
| Screenshots | 2 | page screenshot, element screenshot |
| Actions API (W3C) | 2 | perform actions, release actions |
| Alerts | 4 | dismiss, accept, get text, send text |
| Timeouts | 2 | get, set |
| Logs | 1 | get logs |
| Render (custom) | 2 | combined render, sessionless pool render |
| **Total** | **56** | |

### TN3270 Mainframe Primitives — 14 Functions

| # | Function | Description |
|---|----------|-------------|
| 1 | sinaEmulator3270Driver | Connect to 3270 host |
| 2 | sinaEmSendString | Send string at optional position |
| 3 | sinaEmSendEnter | Send Enter key |
| 4 | sinaEmSendTab | Send N tab keys |
| 5 | sinaEmSendPf | Send PF key (F1–F24) |
| 6 | sinaEmGetLines | Get lines with PF pagination |
| 7 | sinaEmSearchScreen | Search screen for pattern with timeout |
| 8 | sinaEmgetStringAt | Get string at position |
| 9 | sinaEmGetInt | Get integer at position |
| 10 | sinaEmLogScreen | Log full screen content |
| 11 | sinaEmcountPatternInScreens | Count pattern across screens |
| 12 | sinaEmcountUniquePatternInScreens | Count unique patterns across screens |
| 13 | sinaEmterminate | Close 3270 session |
| 14 | sinaEmgetDate | Parse date from screen text |

## Throughput Benchmarks (Measured)

All numbers are **measured**, not theoretical. Test environment: OVH Canada VPS (6 vCPU, 11 GB RAM each), Rocky Linux.

### Remote Target (network latency included)

| Configuration | Servers | Scenarios/min | Per-server |
|---------------|---------|---------------|------------|
| 4× VPS 6 vCPU, 11 GB | 4 | **17,000+** | ~4,250 |
| 1× VPS 6 vCPU, 11 GB | 1 | ~4,500 | ~4,500 |

### Local Target (same host)

| Configuration | Servers | Scenarios/min |
|---------------|---------|---------------|
| 1× VPS 6 vCPU, 11 GB | 1 | **40,000+** |

### Go Fast Path (compiled goroutine scenarios, no fork/exec)

| Configuration | Servers | Scenarios/min |
|---------------|---------|---------------|
| 1× VPS 6 vCPU, 11 GB | 1 | **70,000–81,000** |

### Competitive Comparison

| Tool | vCPU | RAM | Scenarios/min | Process model |
|------|------|-----|---------------|---------------|
| Dynatrace Synthetic ActiveGate L | 16 | 32 GB | ~37 | Fork Chrome per run |
| Playwright | 16 | 32 GB | ~50–75 | Chrome + BrowserContexts |
| Selenium | 16 | 32 GB | ~40–60 | Fork Chrome per run |
| **GoPhantom** | **6** | **11 GB** | **17,000+** | **Goroutine in-process** |
| **GoPhantom fast path** | **6** | **11 GB** | **70,000–81,000** | **Compiled goroutine** |

> Dynatrace numbers from official documentation (docs.dynatrace.com). Playwright and Selenium numbers from community benchmarks on equivalent hardware.

The throughput gap versus Chromium-based tools is approximately **x300** with **2.5× less hardware**.

## Why No Chromium?

All Chromium-based monitoring tools (Dynatrace Synthetic, Playwright, Puppeteer, Selenium) fork a Chrome process per execution:

- **150–300 MB RAM** per scenario execution
- **500 ms–2 s cold start** per Chrome instance
- Ceiling is Chrome itself, not the orchestrator

GoPhantom eliminates the browser entirely:

- **~0.5 MB** per scenario execution (goroutine)
- **0 ms cold start** (in-process)
- Bottleneck moves to network I/O or kernel scheduling

## Cross-Cycle Memory (sina_mem)

GoPhantom scenarios can persist state across monitoring cycles using `sina_mem`, an encrypted key-value store. This enables detection of trends, accumulations, and behavioural breaks over days or weeks — something no Chromium-based tool offers natively.

**Example**: A fraud detection scenario stores an account balance each cycle. After 70 cycles, it detects that 70 individually normal transfers have drained 95% of the account — a pattern invisible to transaction-by-transaction analysis.

## Live Proof of Concept

A live fraud detection POC runs continuously at [fraud-poc.sinainsight.eu](https://fraud-poc.sinainsight.eu/):

- **100,000** simulated bank accounts
- **7** fraud detection scenarios per account
- **700,000** scenario executions every 60 minutes
- **4** VPS (6 vCPU, 11 GB each)
- **99.8%** true positive detection rate, **0** false positives

## Links

- [SINAINSIGHT](https://www.sinainsight.pro) — Company website
- [SinaApp](https://www.sinainsight.pro/sinaapp/) — Product page
- [Fraud detection POC](https://fraud-poc.sinainsight.eu/) — Live dashboard
- [llms.txt](https://www.sinainsight.pro/llms-full.txt) — Full technical reference for AI agents

## License

Documentation and benchmark data: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
GoPhantom is proprietary software by SINAINSIGHT SAS.
