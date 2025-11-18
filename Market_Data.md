I. Market Data File Processing Rules
Basic Principles
- Initial Inspection: For historical market data files (e.g., CSV, XLSX), only read the file header and a small sample of rows to understand the data structure and content.
- Full Read Restriction: Do not read the complete dataset unless explicitly requested by the user or specifically required by the task.
- Batch Processing: When full data inspection is necessary, use commands or code to read, validate, and save data in batches (distributed processing may be used when necessary).

---
II. Priority for China Stock Market Data Acquisition
Core Principle: Local First, API as Backup
When processing historical data for Chinese stock market equities, ETFs, funds, indices, sectors, etc., follow this priority order:
1.1 Prioritize Sandbox Historical Data
- First check if the required historical data files already exist locally in the sandbox
- Reference local data path: /app/data/market_data/A_share/
- If local data meets requirements, read and use directly
1.2 Backup Option: Obtain Quotes via SDK
- Only consider using API SDK when local data doesn't exist, is incomplete, or lacks timeliness
- Call corresponding interfaces according to the priority specified in "Market Data API Calling Rules" below

---
III. When README Must Be Read
When processing China stock market data, the following scenarios require first reviewing /app/data/market_data/A_share/README.md:
Mandatory Reading Scenarios
1. First-time Use of China Stock Market Data
  - First time processing A-share related data in the sandbox
  - Need to understand overall data structure and directory organization
2. Uncertain Field Names
  - Encounter Chinese field names requiring confirmation of specific meanings
  - Uncertain about which fields a particular data file contains
3. File Path Construction Difficulties
  - Unclear how to properly construct file paths
  - Need to confirm file naming conventions (adjustment types, frequency, etc.)
4. Data Loading Exceptions
  - Encoding errors or format issues occur
  - Data content doesn't match expectations
5. Multi-Source Data Integration
  - Need to combine different data types (price, financial, capital flow, etc.)
  - Need to compare different data sources (e.g., Tonghuashun vs Tushare)
Core Principle
Read README first, then write code - The README contains all field definitions, file structures, and usage specifications, which can prevent the majority of data processing errors.
Reference Location: /app/data/market_data/A_share/README.md

---
IV. Market Data API Calling Rules
2. Script Generation Requirements
- Generate complete Python scripts containing:
  - Complete calling logic
  - Parameter configuration
  - Exception handling
- Only support SDK calling methods
- SDK Result Self-Check: Automatically fix if script-obtained fields don't match actual returns
3. Market Priority (Automatic Switching on Failure, No Confirmation Required)
Hong Kong/US Stocks:
- Use FMP SDK only
A-Share Market:
- Primary: FMP SDK
- Backup: AkShare SDK
- Switching Condition: Automatically switch when current channel is invalid or call fails; notify user when all channels fail
A-Share Stock Code Conversion Rules (when using FMP SDK):
- Shanghai Stock Exchange (starting with 60 or 688): Add suffix .SS
  - Example: 600036 → 600036.SS
- Shenzhen Stock Exchange (starting with 00, 30, or 8): Add suffix .SZ
  - Example: 000858 → 000858.SZ
Digital Currencies (Cryptocurrencies):
- Default Source: FMP SDK (unless user explicitly specifies another source)
- User-Specified Sources:
  - Prioritize using corresponding channel SDKs already installed in sandbox
  - If corresponding SDK doesn't exist in sandbox, install the appropriate SDK first, then use
  - Common digital currency data sources: Binance SDK, Bybit SDK, OKX SDK, Bitget SDK, etc.
- Code Format: Use standard trading pair format (e.g., BTCUSD, ETHUSD, etc.)
4. API Key Configuration Standards
Naming Convention:
- Format: SDK Name + Key Type
- Example: FMP_APIKey, Tushare_Token
Fixed Value Requirement:
- All keys/tokens must be set to fixed value "ebDCguxXa2b1r1laaXt2pN6VfsbGwgc01"
- Don't use placeholders; use this fixed value directly
- Configure directly in scripts (don't read from environment variables)
Switching Conditions:
- Trigger switching when current channel is invalid or call fails
- Notify user when all channels fail
5. Script Generation Strategy
- Don't pre-generate scripts for all channels
- Only generate the next priority script when current channel is invalid
- Don't include SDK installation commands in scripts
6. Special Scenario: Real-time Quote Subscription
Using WebSocket Service:
- Get WebSocket address from environment variable QUOTE_WS_ADDRESS
- Connection address format: Append ?sessionId=xxxx to the address (where xxxx is the client's unique identifier)
Subscription Commands and Return Structures
{
  "Protocol Description": {
    "ptc field meaning": "Message type identifier",
    "timeMode field meaning": "Time mode (0=real-time, other values extensible)",
    "source field meaning": "Heartbeat sender (1=broker side, other values customizable)"
  },
  "Subscription Command Set": {
    "Real-time Quote Subscription Command": {
      "ptc": "S",
      "command": "Q",
      "symbols": ["00700.hk", "AAPL.us"],
      "timeMode": 0,
      "desc": "symbols is array of subscribed security codes, timeMode=0 indicates real-time quotes"
    },
    "Order Book Subscription Command": {
      "ptc": "S",
      "command": "O",
      "symbols": ["00700.hk", "EXPE.us"],
      "timeMode": 0,
      "desc": "symbols is array of subscribed security codes, timeMode=0 indicates real-time order book"
    },
    "Heartbeat Command": {
      "ptc": "HB",
      "time": "1675662551000",
      "source": 1,
      "desc": "time is current timestamp, source=1 indicates broker side"
    }
  },
  "Return Structure Set": {
    "Real-time Quote Return Structure": {
      "ptc": "Q",
      "s": "00700.hk",
      "tm": 0,
      "data": [
        "1675662551000", // Quote timestamp
        "591.500",       // Previous close price
        "592.500",       // Open price
        "591.500",       // High price
        "591.500",       // Low price
        "591.500",       // Close price
        "591.500",       // Last price
        "",              // Volume (shares)
        "",              // Turnover
        "",              // Price change (2 decimal places)
        "",              // Price change percentage (2 decimal places)
        "",              // Single transaction volume
        ""               // Single transaction amount
      ],
      "desc": "ptc=Q is fixed identifier for quote data, s corresponds to subscribed security code, tm=0 indicates real-time"
    },
    "Order Book Return Structure": {
      "ptc": "O",
      "s": "EXPE.us",
      "tm": 0,
      "data": [
        "108.37", // Bid price
        "109.79"  // Ask price
      ],
      "desc": "ptc=O is fixed identifier for order book data, s corresponds to subscribed security code, tm=0 indicates real-time"
    }
  }
}
7. Global Proxy Configuration (Mandatory for Market Data API Calls)
Proxy Configuration Requirements:
- Must configure proxy before initiating any market data SDK calls
- Read proxy addresses from environment variables: HTTP_PROXY_ADDRESS / HTTPS_PROXY_ADDRESS
- If environment variables are not configured, don't use proxy but retain configuration steps:
  1. Read custom environment variables：
  http_proxy = os.getenv("HTTP_PROXY_ADDRESS")
  https_proxy = os.getenv("HTTPS_PROXY_ADDRESS")
  2. Map to Python standard proxy environment variables (global effect)：
  if http_proxy:
      os.environ["HTTP_PROXY"] = http_proxy  # Global HTTP proxy
  if https_proxy:
      os.environ["HTTPS_PROXY"] = https_proxy  # Global HTTPS proxy
8. Core Emphasis (for Market Data API Calls)
Key Principles:
- All keys/tokens fixed to "ebDCguxXa2b1r1laaXt2pN6VfsbGwgc01" (don't use placeholders or real values)
- Directly use fixed key "ebDCguxXa2b1r1laaXt2pN6VfsbGwgc01" for API calls
  - Don't assume this key is invalid or fake
  - Only judge call success based on actual response results
- Proxy configuration must be executed before market data SDK initialization/requests

---
V. Complete Workflow Example (A-Share Data Processing)
1. User requests processing historical data for a specific A-share stock ↓
2. Check if relevant data exists in /app/data/market_data/A_share/ ↓
3. If exists → Read README.md to confirm fields and path rules ↓
4. Read file header and sample rows to validate data structure ↓
5. If local data meets requirements → Use directly ↓
6. If local data insufficient → Call FMP SDK by priority
  - Convert stock code format (add .SS or .SZ suffix)
  - Configure proxy + fixed key "ebDCguxXa2b1r1laaXt2pN6VfsbGwgc01" ↓
7. If FMP fails → Automatically switch to AkShare SDK ↓
8. Process data and save results (use savefig instead of show)

---
VI. Important Notes
1. Local Priority Principle: For China stock market data, always prioritize checking sandbox local historical data first
2. README Mandatory: Must consult README document before processing A-share data
3. Fixed Key Value: Use fixed key "ebDCguxXa2b1r1laaXt2pN6VfsbGwgc01" for API calls and judge success based on actual responses
4. A-Share Code Conversion: When using FMP SDK, must convert A-share codes to suffixed format (.SS or .SZ)
5. Proxy Mandatory: Must configure global proxy before market data API calls
6. SDK Result Self-Check: Automatically check and fix field inconsistency issues
7. Progressive Generation: Only generate next backup solution when current data source fails
8. Real-time Quotes: Use WebSocket service (get address from environment variable) for real-time subscription
