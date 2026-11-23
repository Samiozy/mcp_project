# Build and Audit an MCP Server

This mini-project extends a **Model Context Protocol (MCP)** cybersecurity server and evaluates how reliably its tools retrieve vulnerability information. It uses a simulated, offline network environment, a local CVE knowledge base, and an MCP client in a Jupyter notebook. No real hosts are scanned.

## What I built

- Extended `project_mcp_server.py` with simulated host **`10.0.0.30`**, exposing FTP (`vsftpd 2.3.4`), SSH (`OpenSSH 7.2p2 Ubuntu`), and HTTP (`Apache Tomcat 9.0.30` on port 8080).
- Added **`summarize_host_security(target)`**, an MCP tool that summarizes exposed ports, service banners, potential concerns, and defensive recommendations. Its findings are heuristic, not proof of exploitability.
- Implemented **`lookup_cve_v2(service)`**, a product-and-version-aware lookup, while retaining the original `lookup_cve(service)` for comparison.
- Exposed the Markdown CVE knowledge base as the MCP resource **`kb://cve`** and added a Tomcat entry for **CVE-2020-1938**. See `SOURCES.md` for references.
- Used an MCP client over **stdio** to discover tools, call them, and read the resource.

## Audit results

The experiment evaluated **14 simulated service banners across five hosts** using both CVE lookup tools. After recording the baseline, the knowledge-base heading `Apache httpd 2.4.49` was temporarily changed to the vaguer `Apache httpd`, and the sweep was repeated. The original knowledge base was restored afterward.

| Experiment | Lookup tool | Grounded | Blind spot | Wrong version | True gap |
|---|---|---:|---:|---:|---:|
| Before | `lookup_cve` | 9 | 2 | 1 | 2 |
| Before | `lookup_cve_v2` | 12 | 0 | 0 | 2 |
| After vague entry | `lookup_cve` | 9 | 2 | 1 | 2 |
| After vague entry | `lookup_cve_v2` | 11 | 0 | 1 | 2 |

Each row represents 14 service evaluations. The improved matcher reduced the original tool's blind spots and wrong-version responses in the baseline, but vague source knowledge introduced a wrong-version response even with the improved matcher. Detailed per-service results are in `results_mine.csv`.

## Files

| File | Contents |
|---|---|
| `mcp_notebook.ipynb` | Client implementation, MCP interactions, audit, and saved outputs |
| `project_mcp_server.py` | Simulated MCP server and custom security tool |
| `project_cve_kb.md` | Local vulnerability knowledge base |
| `expected_cves.json` | Instructor-provided expected results |
| `expected_mine.json` | Expected results for the custom simulated host |
| `SOURCES.md` | Citations for the added vulnerability knowledge |
| `results_mine.csv` | Before-and-after per-service audit results |

## How to run

1. Open `mcp_notebook.ipynb` in **Google Colab**.
2. Upload `project_mcp_server.py`, `project_cve_kb.md`, `expected_cves.json`, and `expected_mine.json` to the Colab working directory (`/content/`).
3. Install the MCP SDK in a notebook cell: `%pip install "mcp>=1,<2"`.
4. Run the notebook from top to bottom. The MCP client launches the server as a subprocess using **stdio**; there is no API key or real network scan.
5. Inspect the tool discovery, simulated scans, CVE comparisons, and audit outputs. Save the notebook with its outputs before committing.

## Scope and limitations

All target hosts and scans are **simulated**. The knowledge base is small and educational, so a missing advisory does **not** mean a service is secure. Version banners alone cannot establish exploitability; for example, the Tomcat Ghostcat advisory depends on AJP exposure and configuration. The results illustrate tool-output trust, version matching, and knowledge-base coverage rather than real-world scanner accuracy.
