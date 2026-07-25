# Ignored — Verified Test Results

This folder contains real-world test results for techniques in recon-skills.
Every entry was tested against live targets and confirmed working.
If it's here, it's proven useful.

## Structure
```
ignored/
├── README.md           # This file
├── tests/              # Test scripts and raw output
├── findings/           # Confirmed useful techniques with evidence
└── dead-ends/          # Things that looked promising but failed in practice
```

## Rules
- Everything must be tested on real targets
- No theoretical "should work" entries
- Document the target, method, result, and why it's useful
- If it doesn't work in practice, it goes to dead-ends/
