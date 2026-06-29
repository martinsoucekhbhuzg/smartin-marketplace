---
name: smartin-monday-brief
description: Pondělní 5-min brief pro klienta. Use this skill pondělí 7:30 (scheduled task) nebo on-demand. Plugin přečte nejnovější weekly snapshot, vygeneruje brief per šablona Dnes → Co se hnulo → Co stojí → Paměť, a pošle jako draft do klientova kanálu (Slack draft nebo Gmail draft per communication_channel). Konzultant reviduje a odesílá, nebo auto-send pro důvěryhodné klienty.
---

# Smartin Monday Brief

Pondělní 5-minutový brief pro klienta. Cíl: klient v 8:00 při kávě přečte 30 sekund a ví co dnes dělat, vidí že firma se hýbe, pamatuje co rozhodl minulý týden. Bez nutnosti otevírat Cowork.

## Trigger

| Trigger | Kdy |
|---|---|
| Scheduled task `smartin-monday-brief-weekly` | Pondělí 07:30 |
| On-demand klient | „pošli mi brief", „co mi přijde v pondělí" |
| Po Phase 7 weekly review | Konzultant + klient hotov, plugin nabídne „mám tě zítra ráno upozornit?" |

## Workflow

07:30 (po) — plugin čte snapshot z neděle 23:55 + decision_memos posledních 7 dní, generuje brief, zapíše draft do klientova kanálu, notif konzultantovi.
07:45 — konzultant v Cowork projde drafts, edituje, odesílá ručně (per draft-first rule).
08:00 — klient v telefonu při kávě dostane brief, čte 30 sec, pokud chce drill-down klikne link → /smartin-status.

AUTO-SEND mode (po 3+ úspěšných týdnech): `config monday_brief.auto_send: true` → plugin pošle bez konzultanta v 7:30.

## Šablona briefu

```
Pondělí {DD.M.} · Smartin · {company_short_name}


Dnes
{1 sloveso} {1 konkrétní akce + oblast}.
{Time estimate}. {Co plugin pošle/poskytne}.

Co se za týden hnulo
{Oblast 1}    {konkrétní číslo old → new} ({% change})
{Oblast 2}    {konkrétní akce nebo metrika}

Co stojí
{Oblast 1}    {metrika + jak dlouho stojí}
{Oblast 2}    {konkrétní stav, jak dlouho}

Paměť
→ {DD. M.} {1-věta o rozhodnutí}
→ {DD. M.} {další}

Detail: {cowork_link}#smartin-status
```

## Draft writer per channel

- `slack` → `slack_send_message_draft` do `#klient-{slack_channel_name}`
- `email` → Gmail `create_draft` na `team.sponsor.email`
- `both` → oboje paralelně

## Co plugin NEDĚLÁ

- Nepošle brief sám bez review konzultanta (default).
- Nezahltí klienta detaily. Brief = 4 sekce, max 12 řádků.
- Nepřidá motivační fráze. Žádné „Super práce!".
- Nepošle brief při 0 změnách (extrémní stagnace).

## Reference

- Spec: `04_Produkt/smartin-beta-plugin/signaly/plugin_v0_11_snapshots_timeseries_15062026.md`
- Value prop: `value_prop_v1_marketing_board_15062026.md`
- `smartin-snapshots` skill — zdroj dat
- `smartin-decision-memo` skill — zdroj sekce „Paměť"
- `smartin-api-access` §13 (draft-first) + §18 (email) + §20 (draft formats)
