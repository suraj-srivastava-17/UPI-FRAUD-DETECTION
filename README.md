# UPI FraudScope

A rule-based fraud-risk scoring tool for UPI transactions. Checks a
transaction (or batch of transactions) against fraud indicators and
returns a 0–100 risk score, a verdict, and — where the transaction
matches a documented scam pattern — a plain-language explanation of
which one and why.


## What's real here

UPI fraud in India is a large, well-documented, and rapidly growing
problem. This tool's rules and thresholds are grounded in real public
sources rather than invented:

- **'fraud_patterns.json'** contains a 'regulatory_context' section and
  a 'scam_categories' section, each entry cited to a real, checkable
  source (RBI circulars, RBI's own discussion paper figures, NPCI
  advisories, I4C public alerts). For example: the "amount just below
  the UPI transaction limit" rule is a documented evasion pattern, the
  screen-sharing-app rule comes from an NPCI advisory, and the
  "₹1 verification payment" rule comes from a specific I4C Q2 2026
  advisory about fake customer-care pages.
- There is **no public, queryable database of scam UPI handles** to pull
  from live — unlike malware threat-intel feeds (which the security
  research community shares openly), banks and I4C don't publish
  fraud-account identifiers for obvious reasons. So unlike the companion
  project's live ThreatFox integration, this project's "known fraud UPI
  ID" list is explicitly labeled in the file itself as **illustrative,
  not a real database** — it demonstrates the rule engine, not a usable
  blocklist.
- Transaction data is synthetic, since real UPI transaction logs aren't
  public (for obvious privacy reasons) — built only to demonstrate the
  detector working end to end.

This is the same honest pattern used in the companion project, **APT
Hunter India**: ground every rule in a real, checkable source where one
exists, and clearly label what's illustrative where it doesn't.

## Files

| File | Purpose |
| 'upi_fraudscope.py' | main script |
| 'fraud_patterns.json' | cited regulatory context + scam categories + sample fraud-ID list |
| 'sample_txn_fraud.json' | demo — a single transaction matching a KBC/lottery scam pattern |
| 'sample_txn_safe.json' | demo — a single ordinary, low-risk transaction |
| 'sample_txn_batch.json' | demo — six transactions analyzed together (batch mode) |


## Running the demos

'''bash
# Single fraudulent transaction
python upi_fraudscope.py sample_txn_fraud.json

# Single safe transaction
python upi_fraudscope.py sample_txn_safe.json

# Batch of six transactions, with exports
python upi_fraudscope.py sample_txn_batch.json --batch --export json --export csv
'''

## All CLI options

'''bash
python upi_fraudscope.py <transactions_file.json> [options]

  --batch              treat the input as a list of transactions
                       (default: a single transaction object)
  --patterns PATH      use a different fraud patterns file
                       (default: fraud_patterns.json)
  --export json        also write fraud_report.json
  --export csv         also write fraud_alerts.csv
  --quiet              skip the rule-by-rule scan output
'''

## Running on your own transaction

Any JSON object with this shape works:

'''json
{
  "amount": 49999,
  "recipient_upi": "someone@bank",
  "timestamp": "2026-06-14 02:30:45",
  "payment_note": "free text",
  "is_new_recipient": true,
  "new_device": true,
  "device_name": "device label",
  "transactions_last_hour": 5,
  "location_mismatch": true,
  "home_city": "Delhi",
  "current_city": "Mumbai"
}
'''

All fields except 'amount' and 'recipient_upi' are optional — missing
fields just mean the related rule won't fire.

## What it checks, and why

| Rule | Source / rationale |

| Unusual transaction time | Heuristic (late-night/early-morning activity is a common social-engineering window) |
| High value transaction | Heuristic threshold |
| New recipient | Heuristic — first-time payees carry more risk |
| Amount just below the UPI limit | Documented evasion pattern — staying under a threshold avoids extra verification |
| High transaction frequency | Heuristic — rapid repeated transfers suggest urgency/pressure tactics |
| Unrecognized device | Heuristic, aligned with RBI's broader push for device-based risk signals |
| Location mismatch | Heuristic |
| Suspicious UPI ID keyword | Built from documented scam-category keywords (prize, customs, helpdesk, etc.) |
| Known scam payment note | Same — matched against the same cited scam categories |
| Known fraud UPI ID | Illustrative sample list only — see note above |

Full citations for the scam categories and regulatory figures are in
'fraud_patterns.json'.

## A note on scope and honesty

This tool demonstrates a detection *pipeline* — rule scoring, scam-
category attribution, batch processing, exportable reports — built on
real regulatory context. It is not, and doesn't claim to be, a
production fraud-detection system: real systems use far more signals
(device fingerprinting, behavioral biometrics, network-graph analysis
for mule accounts — see RBI's "MuleHunter.AI", cited in
'fraud_patterns.json') than a single-transaction rule engine can capture.
