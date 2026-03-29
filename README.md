# CEKINMEZ — SAST Engine
> **Hunt What Others Cannot See.**

A deterministic, multi-pass static analysis engine purpose-built to detect cross-file WAF bypass chains, business logic flaws, and obfuscated attack vectors that file-isolated SAST tools structurally cannot reach.

## What CEKINMEZ Detects

- **IDOR / BOLA** — Insecure Direct Object Reference across single and multi-file chains
- **Mass Assignment** — req.body flowing unfiltered into ORM update/create operations
- **SSRF** — Server-Side Request Forgery via axios, fetch, http.get with unvalidated input
- **JWT Bypass** — decode-only patterns with no signature verification
- **Cross-File Taint Chains** — user input traced through 6+ layers across separate modules
- **Fragmented Exploits** — obfuscated payloads split across multiple files and constants
- **HQL / JPQL Injection** — Java Hibernate ORM query injection
- **SpEL RCE** — Spring Expression Language remote code execution
- **Thymeleaf SSTI** — Server-Side Template Injection in Spring controllers

---

## Architecture

### Global Memory Pool
Before scanning begins, CEKINMEZ pre-loads the entire codebase into an immutable in-memory map. Every file is instantly accessible to every analysis component simultaneously — the engine sees the full architecture as one unified system, not isolated files.

### Dynamic Chunking & Parallel Worker Pool
The codebase is split into chunks and fed into a dynamically scaling worker pool. Workers steal tasks the moment they are free — no CPU cycle is wasted. RAM stays flat even on 15,000+ file codebases.

### Inter-Procedural AST Linker
A dedicated Babel AST engine builds a Call Graph across every file: a complete map of every function, what it calls, and whether it reaches a critical sink. Resolved recursively via fixed-point algorithm. If user-controlled input reaches any function that leads to a sink — in any file — CEKINMEZ flags the confirmed chain.

### Fragment Analyzer
Reconstructs string values split across multiple files. Constants exported from separate modules are resolved, concatenated, and matched against sink patterns. Detects backdoors where the dangerous payload never appears as readable text in any single file.

### Fusion Correlator
Fuses weak signals into confirmed attack chains. Unvalidated input in a controller correlated with a missing auth boundary in a router produces one Distributed WAF Bypass Chain finding — not noise.

### Gatekeeper — Zero Noise Policy
Every finding passes an exploitability filter. Only vulnerabilities with a confirmed, traceable execution path terminating at a critical sink are reported. If there is no confirmed attack vector, CEKINMEZ remains silent.

---

## Installation

> ⚡ **Commercial release coming soon. Pricing will be accessible.**
> Early access available — see below.

**Requirements:**
- Node.js >= 18
- Semgrep >= 1.50 (`pip install semgrep`)

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/cekinmez.git
cd cekinmez

# Install dependencies
npm install

# Run against a target directory
node cekinmez.js /path/to/target/project
```

---

## Usage

```bash
# Scan a directory
node cekinmez.js ./my-project

# Scan a single file
node cekinmez.js ./src/controllers/userController.js
```

**Output files generated:**
- `master_report.json` — All confirmed, exploitable findings sorted by risk
- `dataflow_report.json` — Detailed input-to-sink flow analysis

---

## Output Example

```json
{
  "id": 1,
  "file": "/src/controllers/userController.js",
  "line": 24,
  "risk": "CRITICAL",
  "rule": "WAF_BLIND_AST_CROSS_FILE",
  "message": "[AST-CROSS-FILE] Tainted input reaches sink via 'processRequest' across file boundary.",
  "cwe": "CWE-639"
}
```

---

## Supported Languages

| Language | Semgrep Rules | AST Analysis |
|---|---|---|
| JavaScript | ✅ | ✅ |
| TypeScript | ✅ | ✅ |
| Java | ✅ | — |
| Python | ✅ | — |
| Go | ✅ | — |
| Rust | ✅ | — |
| C# | ✅ | — |

---

## Roadmap

- [ ] npm global package (`npm install -g cekinmez`)
- [ ] License system & commercial tier
- [ ] CLI flags for custom rule paths and output formats
- [ ] HTML report output
- [ ] Solidity / smart contract support

---

## Early Access

Commercial release is imminent. Pricing will be straightforward and accessible for individual researchers and small teams.

**For early access or licensing inquiries, open an Issue or start a Discussion.**

---

## License

Proprietary. All rights reserved.
Commercial use requires a valid license.
Contact for pricing.

---

*CEKINMEZ — Hunt What Others Cannot See.*
