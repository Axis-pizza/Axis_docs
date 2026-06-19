# DTF Market Issues

## Issue: Implement DTFMarket account

Requirement IDs:

```txt
DTF-010
DTF-011
DTF-012
```

Acceptance criteria:

```txt
- stores creator
- stores DTF mint
- stores asset count
- stores status
- stores reserve authority
```

## Issue: Implement CreateMarket validation

Requirement IDs:

```txt
DTF-001
DTF-002
DTF-003
DTF-004
DTF-005
DTF-006
DTF-007
DTF-008
DTF-009
```

Acceptance criteria:

```txt
- reject 1 asset DTF
- reject >5 assets
- reject duplicate assets
- reject invalid total weights
- reject weight below 100 bps
- reject weight above asset max
- reject unregistered asset
- reject creation_disabled asset
```

## Issue: Implement reserve account validation

Requirement IDs:

```txt
DTF-010
NFR-SEC-004
```

Acceptance criteria:

```txt
- reserve account mint matches asset mint
- reserve authority is Axis-controlled
- invalid reserve account fails
```
