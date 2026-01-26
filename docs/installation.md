# Installation Guide

This guide walks you through downloading the binaries, obtaining a demo license, and configuring Claude Desktop.

## 1) Download Files

- Download **`ofmcp.exe`** and **`metadata.db`** from the latest release.
- Save both files in the same folder (e.g., `C:\Users\User\ofmcp\`).

## 2) Generate a Machine ID

Open a terminal in the folder where `ofmcp.exe` lives and run:

```powershell
ofmcp.exe --print-machine-id
```

Copy the printed machine ID. You will use it to request a license.

## 3) Request a Demo License (14 Days)

- Open: https://license.oraclefusionsql.com/
- Enter your **email** and **machine ID**.
- Submit the form.
- You will receive a verification email. Click the link.
- You will receive a second email with your license.

Save the license content as `license.json` in the same folder as `ofmcp.exe`
or set `LICENSE_PATH` in Claude Desktop (see below).

## 4) Configure Claude Desktop

Open Claude Desktop and go to Settings:

![Open Claude Desktop Settings](https://github.com/krokozyab/ofjdbc_claudie_mcp/blob/main/pics/w_setup_1.png)

Navigate to Configuration:

![Claude Desktop Configuration](https://github.com/krokozyab/ofjdbc_claudie_mcp/blob/main/pics/w_setup_2.png)

Edit `claude_desktop_config.json` with the correct paths.

### SSO Variant

```json
{
  "mcpServers": {
    "fusion-metadata": {
      "command": "C:\\Users\\User\\ofmcp\\ofmcp.exe",
      "args": ["-db", "C:\\Users\\User\\ofmcp\\metadata.db"],
      "env": {
        "FUSION_HOST": "https://server.oraclecloud.com",
        "LICENSE_PATH": "C:\\Users\\User\\ofmcp\\license.json"
      }
    }
  }
}
```

### Basic Authentication Variant

```json
{
  "mcpServers": {
    "fusion-metadata": {
      "command": "C:\\Users\\User\\ofmcp\\ofmcp.exe",
      "args": ["-db", "C:\\Users\\User\\ofmcp\\metadata.db"],
      "env": {
        "FUSION_HOST": "https://server.oraclecloud.com",
        "FUSION_USER": "user",
        "FUSION_PASSWORD": "password",
        "LICENSE_PATH": "C:\\Users\\User\\ofmcp\\license.json"
      }
    }
  }
}
```

## Notes

- The demo license expires after 14 days. Request a new license or purchase a full license to continue.
- Make sure the paths match your actual folder structure.
