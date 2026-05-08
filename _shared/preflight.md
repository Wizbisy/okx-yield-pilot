# Shared Pre-flight Checks

> This file is shared across all onchainos skills. Follow these steps before the first `onchainos` command each session.

Every time before running any `onchainos` command, always follow these steps in order. Do not echo routine command output to the user; only provide a brief status update when installing, updating, or handling a failure.

## 1. System Integrity (CLI Setup)

1.  **Resolve latest stable version**: Fetch the latest stable release tag from the GitHub API:
    ```bash
    curl -sSL "https://api.github.com/repos/okx/onchainos-skills/releases/latest"
    ```
    Extract the `tag_name` field (e.g., `v1.0.5`) into `LATEST_TAG`.
    If the API call fails and `onchainos` is already installed locally, skip steps 2-3 and continue with step 4.

2.  **Install or update**: If `onchainos` is not found, or if the check cache is older than 12 hours:
    - Download the installer from the latest release tag:
      - **Windows**:
        `Invoke-WebRequest -Uri "https://raw.githubusercontent.com/okx/onchainos-skills/${LATEST_TAG}/install.ps1" -OutFile "$env:TEMP\onchainos-install.ps1"`
      - **macOS/Linux**:
        `curl -sSL "https://raw.githubusercontent.com/okx/onchainos-skills/${LATEST_TAG}/install.sh" -o /tmp/onchainos-install.sh`
    - Execute the installer:
      - **Windows**: `& "$env:TEMP\onchainos-install.ps1"`
      - **macOS/Linux**: `sh /tmp/onchainos-install.sh`
    - **Note**: The installer handles version comparison internally and places the binary in `$env:USERPROFILE\.local\bin\onchainos.exe` (Windows) or `~/.local/bin/onchainos` (macOS/Linux).

3.  **Verify binary integrity** (once per session):
    Run `onchainos --version` to get the version. Verify the SHA256 hash of the binary.
    - **Windows Path**: `$env:USERPROFILE\.local\bin\onchainos.exe`
    - **macOS/Linux Path**: `~/.local/bin/onchainos`

4.  **Version drift check**:
    - Run `onchainos --version` → CLI version (e.g., `2.2.9`)
    - Read `version` field from the active skill's YAML frontmatter.
    - If CLI version > skill version → warn: **"⚠️ Skill outdated. Re-install skills to get the latest features."**

## 2. Wallet State (Auth & Connectivity)

1.  **Wallet Login**:
    ```bash
    onchainos wallet status
    ```
    - If `loggedIn: false` → tell user: "You need to log in first. Run `onchainos wallet login` or say 'log me in'."
    - If `loggedIn: true` → continue

2.  **Address Resolution**:
    ```bash
    onchainos wallet addresses
    ```
    - Resolve and store EVM address for Ethereum/BSC/Base/X Layer and Solana address for Solana.
    - **CRITICAL**: Never pass an EVM address to `--chains solana` or vice versa.
    - Once addresses are resolved → continue

3.  **Connectivity Check**:
    ```bash
    onchainos token price-info --address 0xeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee --chain ethereum
    ```
    - Verify API access. If error 50011 (rate limit), wait 5 seconds and retry.
    - If returns price data → API is healthy, continue ✅

4.  **Active Positions**:
    ```bash
    onchainos defi positions --address <addr> --chains "xlayer,ethereum,bsc,base"
    onchainos defi positions --address <sol_addr> --chains "solana"
    ```
    - If zero positions found → inform user: "No active yield farming positions found."
    - If positions found → continue to requested workflow
