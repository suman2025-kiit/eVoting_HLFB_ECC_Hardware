# ECC-HLFB-eVOTING-FPGA-Validation (Vivado/Vitis + Hyperledger Fabric v2.2)

This repository contains a **reference implementation** of a **hardware-assisted validation gate**
for **ECC-HLFB-eVOTING** on a **permissioned Hyperledger Fabric (HLF) v2.2** blockchain.

The goal is to enforce **fast pre-validation** of e-Voting protocol messages using an FPGA datapath
that outputs:

- **Decision**: `ACCEPT` / `REJECT`
- **Reason code (REC)**: `CERT_FAIL`, `SIG_FAIL`, `TIME_FAIL`, `REPLAY_FAIL`, `FMT_FAIL`, `EP_FAIL` (etc.)

Only `ACCEPT` transactions are forwarded to Fabric for endorsement/ordering/commit; `REJECT` attempts
are logged (world-state or events) for **fraud tracing**, rate limiting, and audit analytics.

---

## 1. Repository structure

```
ECC-HLFB-eVOTING-FPGA-Validation/
├─ README.md
├─ LICENSE
├─ .gitignore
├─ hardware/
│  ├─ rtl/
│  │  ├─ validator_core.v
│  │  └─ validator_codes.vh
│  ├─ vivado/
│  │  └─ build_notes.md
│  └─ screenshots/
│     ├─ valid_output.png
│     └─ invalid_output.png
├─ software/
│  ├─ vitis_app/
│  │  ├─ src/
│  │  │  ├─ main.c
│  │  │  ├─ fpga_mmio.c
│  │  │  ├─ fpga_mmio.h
│  │  │  ├─ validator_codes.h
│  │  │  └─ http_client.c
│  │  └─ Makefile
│  └─ gateway_service/
│     ├─ server.js
│     ├─ package.json
│     ├─ README_gateway.md
│     └─ crypto/   (ignored)
└─ chaincode/
   └─ evotecc-go/
      ├─ evotecc.go
      └─ go.mod
```

---

## 2. Reason codes (REC) and decision logic

### 2.1 Input checks (from ECC-HLFB-eVOTING validation table)
The FPGA core evaluates these checks (mapped to signals):

- `CERT_OK`: certificate/MSP validity (Fabric CA / MSP chain)
- `SIG_OK`: ECDSA/Idemix signature/proof verification result
- `TIME_OK`: timestamp freshness within `ΔT`
- `NONCE_OK`: nonce/txID uniqueness (replay detection)
- `FMT_OK`: protocol/format correctness (IDs, fields, tags, policy fields)
- `EP_OK`: endorsement/policy satisfaction (minimum endorsements / policy match)

### 2.2 Output mapping (REC)
`REC` is a 4-bit code:

| REC | Meaning |
|---:|---|
| 0 | OK |
| 1 | CERT_FAIL |
| 2 | SIG_FAIL |
| 3 | TIME_FAIL |
| 4 | REPLAY_FAIL |
| 5 | FMT_FAIL |
| 6 | EP_FAIL |

**Priority**: the first failed check determines the reason code (CERT → SIG → TIME → NONCE → FMT → EP).

---

## 3. Standard baseline environment (recommended)

### 3.1 Blockchain side
- Hyperledger Fabric: **v2.2.x**
- Docker + Docker Compose
- Node.js: **18+**
- Go: **1.20+** (for chaincode build)
- Fabric Gateway client (Node): `@hyperledger/fabric-gateway`

### 3.2 FPGA side
- Vivado: **2020.2+**
- Vitis: **2020.2+**
- Board: **Xilinx PYNQ / Zynq / ZynqMP** (Linux preferred)

---

## 4. Hardware build (Vivado)

See: `hardware/vivado/build_notes.md`

### 4.1 Recommended integration method (AXI GPIO)
Use AXI GPIO to pass a compact “check mask” (PS→PL) and read decision + reason code (PL→PS).

**Example (8-bit PS→PL):**
- bit0 `cert_ok`, bit1 `sig_ok`, bit2 `time_ok`, bit3 `nonce_ok`, bit4 `fmt_ok`, bit5 `ep_ok`, bit6 `start`, bit7 reserved

**Example (8-bit PL→PS):**
- bit0 `accept`, bits1..4 `rec[3:0]`, bit5 `done`

---

## 5. Software build (Vitis app)

### 5.1 Compile
```bash
cd software/vitis_app
make
```

### 5.2 Configure
Edit `software/vitis_app/src/main.c`:
- `FPGA_BASE_ADDR` : physical MMIO base address (from Vivado Address Editor)
- `GW`             : Gateway service URL (e.g., `http://192.168.1.10:8080`)

### 5.3 Run (Linux on board)
```bash
sudo ./validator_app
```

---

## 6. Gateway service (Node.js → Fabric v2.2)

This service receives HTTP calls from the Vitis app and submits chaincode transactions via Fabric Gateway.

### 6.1 Configure crypto (local only)
Place Fabric TLS + user credentials locally at:
`software/gateway_service/crypto/`

Files expected:
- `peer0-tls-ca.pem`
- `user-cert.pem`
- `user-key.pem`

**These are ignored** and must not be committed.

### 6.2 Install and run
```bash
cd software/gateway_service
npm install
node server.js
```

Endpoints:
- `POST /api/submitVote`
- `POST /api/logReject`

---

## 7. Chaincode (Go) — vote + fraud logging

Location: `chaincode/evotecc-go/`

Functions:
- `SubmitVote(voterId, txId, ballotHash, ts)`
- `LogReject(voterId, txId, reason, ts)`

---

## 8. Validation workflow (end-to-end)

1) Protocol message arrives
2) Checks are evaluated → FPGA core outputs `ACCEPT` or `REJECT+REC`
3) Vitis app forwards:
   - `ACCEPT` → Fabric gateway `submitVote`
   - `REJECT` → Fabric gateway `logReject`
4) Chaincode commits votes and/or reject logs to ledger/world state

---

## 9. Fraud tracking and invalid voter identification

Typical signals:
- `REPLAY_FAIL`: duplicate nonce/txID → replay/double voting attempt
- `SIG_FAIL` / `CERT_FAIL`: forged identity/credential misuse
- `TIME_FAIL`: stale submission
- `EP_FAIL`: endorsement/policy manipulation attempt

Maintain:
- `fail_count[voterId]`, `last_reason[voterId]`, `last_ts[voterId]`

---

## 10. Screenshots (sample)
See `hardware/screenshots/`.

---

## 11. License
Apache-2.0
