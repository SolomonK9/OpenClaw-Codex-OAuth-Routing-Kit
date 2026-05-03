# Release Notes — v2.3.0

## Headline

OAuth expiry alerts now respect automatic refresh. The router no longer treats a local access-token expiry timestamp as immediate proof that a human browser reauth is required.

## Changed

- Added explicit expiry states in `scripts/oauth_pool_router.py`:
  - `access_expired_refresh_pending`
  - `auto_refresh_recovered`
  - `refresh_grace_failed`
  - `manual_reauth_required`
- Added configurable auto-refresh grace window:
  - `oauthAutoRefresh.graceMinutes`
  - fallback: `alerts.expiryRefreshGraceMinutes`
  - default: `90` minutes
- Suppressed expiry-only operator alerts while refresh is still pending.
- Replaced immediate `PROFILE_EXPIRED` alert behavior with `PROFILE_MANUAL_REAUTH_REQUIRED` only after refresh/probe failure or grace expiry.
- Internal auto-refresh recovery events are logged but do not interrupt the operator.

## Safety model

This release does not include live auth stores, tokens, gateway config, session stores, private account names, private project labels, or operator state.

## Verification run

Validated locally with:

```bash
python3 -m py_compile scripts/oauth_pool_router.py
python3 scripts/verify_auto_refresh_states.py   # temporary local verification helper, not included in release
git diff --check -- scripts/oauth_pool_router.py
```

The synthetic verification covered:

- recently expired access token -> `access_expired_refresh_pending`
- expired beyond grace -> `refresh_grace_failed`
- explicit auth dead/expired -> `manual_reauth_required`
- usage success after expiry -> `auto_refresh_recovered`
