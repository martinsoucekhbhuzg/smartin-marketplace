---
name: smartin-self-test
description: Smoke test plugin instalace — connectors, MCP, Drive, prefetch, API. <1 min.
---

# /smartin-self-test

Ověř, že plugin smartin-beta v Cowork prostředí funguje. Spusť po nové instalaci nebo po upgradu, nebo když něco nejde a chceš zjistit kde je problém.

## Workflow (cca 45-60 sekund)

Plugin spustí 8 testů paralelně kde to jde:

1. **Cowork verze + plugin verze** — PASS pokud Cowork ≥ 0.6.0 a plugin_version z poslední published verze
2. **HubSpot connector** — PASS pokud vrátí non-empty profile
3. **Google Analytics connector** — SKIP pokud GA není napojeno (volitelně)
4. **Google Drive connector** — PASS pokud vrátí soubory
5. **Slack connector** — PASS pokud vrátí libovolný response, SKIP pokud `communication_channel: email`
6. **Smartin MCP** — PASS pokud vrátí workspaces (konzultant only)
7. **Smartin app login** — PASS pokud 200 + email match user identity
8. **Drive write probe** — PASS pokud `create_file` projde

## Output

```
═══════════════════════════════════════════
  Smartin Self-Test
═══════════════════════════════════════════

✅ 1. Cowork v0.7.2 + plugin v0.11.0
✅ 2. HubSpot connector
⏭️  3. Google Analytics (not connected)
✅ 4. Google Drive connector
✅ 5. Slack connector
⏭️  6. Smartin MCP (klient mode = correct)
✅ 7. Smartin app login (martin@smartin.work)
✅ 8. Drive write probe (folder writable)

PASS: 6/8 (2 skipped) — plugin OK
```

## Co plugin dělá při FAIL

Per test, plugin nabídne nejčastější opravu:

- HubSpot 401 → Cowork → Settings → Integrations → HubSpot → re-connect
- Drive 401 → Google Drive re-connect
- Slack search 0 results → pozvat plugin do Slack workspace
- Smartin MCP not available → klient OK, konzultant zapni v Settings
- Smartin app 401 → otevři smartin.work, přihlas se, refresh
- Drive write probe 403 → folder není nasdílený s Edit permissions

## Kdy nespouštět

- Při běžné session (zbytečně utratí čas)
- Klient bez Smartin app login (Phase 0 onboarding)

## Tip

Pošli klientovi link s instrukcí: „Spusť `/smartin-self-test`, pošli screenshot. Pokud cokoliv FAIL, opravíme společně."
