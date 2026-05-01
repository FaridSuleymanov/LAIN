# QRAE — Quantum Readiness Assessment Engine

**QRAE** is an early-stage Python toolkit for authorized post-quantum cryptography readiness assessment.

It inspects selected systems or communication channels, identifies visible cryptographic primitives where possible, classifies their quantum-readiness risk, and records the assessment process in a tamper-evident local audit log.

> Status: alpha / research prototype.  
> Current focus: TLS endpoint inspection, primitive classification, authorization scope control, structured findings, and local audit integrity.

---

## Why this exists

Post-quantum migration is difficult because many organizations do not have a clear inventory of where classical cryptography is used, which primitives are exposed, and which systems should be prioritized.

QRAE is intended to support practical review questions:

- Which cryptographic primitives are visible on a given endpoint or channel?
- Which primitives are affected by Shor's algorithm or Grover's algorithm?
- Which findings require post-quantum migration planning?
- Was the assessment performed inside an explicit authorization scope?
- Can the assessment trail be reviewed later?

QRAE is not a certification tool and does not claim that a system is compliant or secure. It is a first-pass assessment and training aid.

---

## Current capabilities

| Area | Implementation |
|---|---|
| Authorization scope | Operator, targets, authorization source, expiry, and reference tracking |
| Scope matching | Exact hostname/IP/channel matching, CIDR matching, wildcard hostname matching |
| Audit trail | Append-only JSONL log with SHA-256 hash chaining |
| TLS assessment | Active TLS connection, negotiated cipher metadata, certificate public-key classification |
| Primitive classification | Conservative classification for Shor/Grover/PQC/unprotected/unknown cases |
| Generic channel assessment | Records known unprotected data/RF channels as findings |
| Structured output | JSON findings for reports, dashboards, or later SIEM ingestion |
| Tests | Unit tests for scope, audit, classifier, findings, and channel assessment |

---

## Classification model

QRAE uses a conservative triage model:

| Classification | Meaning |
|---|---|
| `unprotected` | No cryptographic protection was identified |
| `broken` | Primitive is affected by Shor's algorithm once a CRQC exists |
| `weakened` | Primitive is affected by Grover's algorithm; effective security is reduced |
| `unknown` | Primitive is not recognized by the classifier and needs manual review |
| `resistant` | Primitive is recognized as post-quantum or hash-based in the current classifier table |

Examples:

- RSA, DSA, DH, ECDSA, ECDH, Ed25519, X25519 → `broken` / `shor`
- AES, ChaCha20, SHA-2/SHA-3 families → `weakened` / `grover`
- ML-KEM, ML-DSA, SLH-DSA, XMSS, LMS → `resistant` / `none`
- Unknown primitives are never treated as safe by default

---

## Repository layout

```text
qrae/
├── cli.py                  # Command-line interface
├── __main__.py             # Enables python -m qrae
├── core/
│   ├── audit.py            # Tamper-evident local audit log
│   ├── classifier.py       # Quantum-readiness primitive classifier
│   ├── models.py           # Finding and primitive data models
│   └── scope.py            # Authorization scope model
└── protocols/
    ├── channel.py          # Generic unprotected-channel assessment
    └── tls.py              # TLS endpoint assessment

tests/
├── test_audit.py
├── test_channel.py
├── test_classifier.py
├── test_models.py
└── test_scope.py
