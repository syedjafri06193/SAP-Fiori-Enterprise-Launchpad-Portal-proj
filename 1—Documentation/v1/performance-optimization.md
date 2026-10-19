# Performance Optimization – BINARY

## 1. Targets

| Metric | Baseline | Target | Achieved |
|---|---|---|---|
| Launchpad initial load, cold (P90) | ~9.5 s | ≤ 4 s | ~3.2 s |
| Launchpad warm load (P90) | ~5 s | ≤ 2 s | ~1.6 s |
| Visualizations per user (median) | 62 | ≤ 25 | 18 |
| Dynamic tiles per user (median) | 11 | ≤ 4 | 3 |
| Slowest dynamic tile `$count` | 4.8 s | ≤ 0.5 s | 0.3 s |

---

## 2. Where Launchpad Time Goes

```
Logon/SSO ─▶ Shell bootstrap (UI5 core) ─▶ Start-up service (content) ─▶ Page render ─▶ Dynamic tile calls
```

| Phase | Typical issue | Lever |
|---|---|---|
| SSO | Redirect chains | Correct SAML config, keep sessions |
| Bootstrap | UI5 resources not cached | Cache headers, cache buster, CDN/Web Dispatcher caching |
| Start-up service | Many catalogs/tiles per user | Fewer, leaner catalogs; spaces mode |
| Page render | Too many tiles | ≤ 25 visualizations, links for rare apps |
| Dynamic tiles | Slow `$count` services, short refresh | Optimize services, refresh ≥ 300 s, fewer dynamic tiles |

---

## 3. Optimization Measures Implemented

### 3.1 Content
- Consolidated 47 overlapping catalogs into 16; removed duplicate tiles.
- Switched to spaces and pages: only the active page renders.
- Converted occasional apps to links.
- Removed tiles with no usage in two quarters.

### 3.2 Dynamic tiles
- Replaced dynamic with static tiles where the number didn't drive action.
- Rewrote two custom `$count` services to use CDS aggregations with selective filters.
- Set refresh intervals to 300 s minimum.

### 3.3 Caching
- Verified UI5 cache buster (`/sap/bc/ui5_ui5/...` with cache tokens) active, allowing long browser caching.
- App index calculated nightly so the launchpad resolves app resources quickly.
- Launchpad content cache enabled; invalidated only after imports.

### 3.4 Gateway & backend
- Analyzed `/IWFND/STATS` for the top 20 services by total time.
- Traced slow calls with `ST05` (SQL trace) and `SAT`; added selective filters and fixed one CDS view join.
- Ensured `$batch` used by UI5 apps; no unnecessary `$expand`.

### 3.5 Network & infrastructure
- HTTP/2 and gzip compression enabled on Web Dispatcher/ICM.
- Logon group for HTTP to balance load across application servers.

---

## 4. Measurement Method

1. Clear browser cache (cold) or reuse (warm).
2. Open the launchpad with Chrome DevTools → Network, "Disable cache" for cold runs.
3. Record time to the page being interactive and the start-up service duration.
4. Repeat 10 times per persona; compute P90.
5. Server side: `/IWFND/STATS` for service times; `STAD` for HTTP steps.

For UI5-level analysis, the UI5 diagnostics tool (`Ctrl+Alt+Shift+S`) and `sap-ui-debug=true` help identify slow components.

---

## 5. Ongoing Monitoring

| Check | Frequency | Threshold |
|---|---|---|
| Top services by response time (`/IWFND/STATS`) | Weekly | Avg > 1 s investigated |
| Launchpad load spot check per persona | Monthly, and before close | P90 > 4 s investigated |
| Gateway error log volume | Daily | Spike > 2× baseline |
| New dynamic tiles | Each change | Design Authority approval |

---

## 6. Guidelines for New Content

- Prefer static tiles; justify dynamic tiles.
- New services must support `$count`, filtering and paging.
- Keep pages ≤ 25 visualizations.
- Test performance in QAS with realistic data volume before go-live.

---

## Related Documents

- [Spaces & Pages Strategy](spaces-and-pages-strategy.md)
- [Backend Integration](integration-with-backend.md)
- [Troubleshooting Guide](troubleshooting-guide.md#6-performance-issues)
