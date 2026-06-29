---
name: smartin-failure-handlers
description: Branch guidance when Smartin onboarding or daily operation hits a known failure mode. Use this skill whenever the user reports a problem — Aktualizace nedetekovala nic, klient nemá HubSpot, vrací 401, token expired, workspace je prázdný, klient se zasekl, detekce nesedí, owner odmítá oblast, nemá co odpovědět v maturity. Also use proactively when the onboarding-guide skill reaches a phase output that can not be produced. Don not push past a failure — divert into the right branch.
---

# Smartin Failure Handlers

When the playbook can't progress, don't improvise. Pick the matching branch. Every branch has: symptom, root cause, action, escalation.

## 1. Aktualizace nic nedetekovala

**Symptom.** Po prvním "Aktualizovat" je Mapa firmy prázdná nebo všechny oblasti OK.

**Root cause.** Buď napojení nemá data za relevantní období, nebo detekční model nemá dost signálu.

**Action.**

1. Zkontrolovat dataset každého connectoru: kolik záznamů za posledních 30 dní. Méně než 50 v CRM = nedostatek dat.
2. Pokud data jsou: spustit Aktualizaci podruhé. Detekce může být stochastická (zejména SALES).
3. Pokud druhý běh také prázdný: zkontrolovat, že napojení je aktivní (token neexpiroval).
4. Pokud i poté: změnit Phase 2 plán — místo "klient vidí mapu" → "klient vyplní 5 manuálních otázek o své realitě a detekce se podle nich přizpůsobí".

**Escalation.** Pokud i s manuálními otázkami nedetekuje, eskalovat na product (internal mode): "Detection failure pro segment X — kandidát na metodiku revize."

## 2. Klient nemá HubSpot (nebo jiný očekávaný nástroj)

**Symptom.** Klient používá Pipedrive, Salesforce, vlastní CRM, nebo CRM nemá vůbec.

**Root cause.** Smartin detection má sourceTypes navázané na konkrétní enums (HUBSPOT, GOOGLE_ANALYTICS).

**Action.**

1. Zjistit, co klient reálně používá. Zaznamenat do plugin config.
2. Pokud existuje MCP connector pro jejich CRM: napojit ho jako alternativní zdroj. Detection musí mapovat fields ručně (jiné názvy).
3. Pokud MCP connector neexistuje: zkusit Google Sheets export jako fallback. Klient export udržuje, plugin čte.
4. Pokud i to není: dočasně Marketing/Sales oblast vést jen z emailové komunikace + manuální vstup, dokud nepostavíme custom integration.

**Escalation.** Custom integration = product story, ne onboarding task. Předat do internal product layer.

## 3. API vrátilo 401

**Symptom.** Token expired, plugin přestal vidět workspace data.

**Root cause.** SPA navigace nebo časový limit. Token žije v React fiber, ne v localStorage.

**Action.**

1. Re-extract token (token extraction recipe v smartin-api-access SKILL.md).
2. Pokud opakovaně selhává: ověřit, že uživatel je stále přihlášený v Smartin app (smartin.work/chat nebo dev.mjobs.blog).
3. Pokud token je dev a API call míří na prod (nebo naopak): mapping chyba. Vrátit se k explicitnímu výběru prostředí.

**Escalation.** Pokud token nelze získat ani po reloadu, klient je odhlášený. Požádat o re-login.

## 4. Workspace je prázdný (žádné case types, žádné šablony)

**Symptom.** GET `/case-config/all` vrací prázdný seznam pro klientův workspace.

**Root cause.** Klient pravděpodobně používá fresh prod instanci, ale prod ještě nemá nasazené všechny oblasti (od 19.5.2026 by měla mít všech 6 = 50 case types).

**Action.**

1. Zkontrolovat aktuální stav prod přes `/case-config/all` — kolik je tam case types celkem.
2. Pokud chybí, je třeba spustit dev → prod sync. Postup v Smartin docs: `04_Produkt/Architektura/smartin_api_reference.md`.
3. Pro onboarding klienta: pokud sync nestihneme, dočasně použít DEV instanci pro demo (s vědomím, že po prod ready přejdeme).

**Escalation.** Operations responsibility — internal mode.

## 5. Klient se zasekl ve fázi N

**Symptom.** Plugin v konzultant view ukazuje "Days in phase: 4" pro klienta, který by měl být dál.

**Root cause.** Klient ztratil kontext, ztratil motivation, nebo narazil na nepojmenovanou friction.

**Action.**

1. Plugin generuje nudge text per fáze:
   - Phase 0/1: "Připomeneme si: zbývá nainstalovat connector X."
   - Phase 2: "První Aktualizace ještě neproběhla. Stačí 30 sekund — zkusme to teď."
   - Phase 3: "Mapa firmy je hotová, ale ještě jsme spolu neudělali společný read-out. Kdy se vejdeš na 30 min?"
   - Phase 4–7: situational.
2. Konzultant pošle nudge v Slacku (#klient-{slack_channel_name}).
3. Pokud nudge bez odezvy 48h: konzultant zavolá osobně.
4. Pokud klient v Phase 0/1 7+ dní: pravděpodobně IT/onboarding bottleneck. Eskalovat na sponzora.

**Escalation.** Phase 5+ stagnation = klient ztrácí důvěru. Stojí za to udělat 30 min schůzku, ne tlačit přes nudge.

## 6. Owner odmítá vlastnit oblast

**Symptom.** V Phase 4 allocation nikdo nechce být ownerem konkrétní oblasti (typicky Operations nebo C&M).

**Root cause.** Buď je oblast nejasně definovaná v kontextu klienta, nebo nikdo z týmu k ní organicky nepatří.

**Action.**

1. Vyjasnit s sponzorem, co je v dané oblasti reálně co dělat (např. "Operations = cash + tým + tools").
2. Pokud po vyjasnění stále nikdo: ownership = sponzor sám (dočasně). Re-eval za 4 týdny.
3. Nikdy nenutit někoho vlastnit oblast. Bez vlastníka = oblast se nehýbe, ale není to katastrofa.

**Escalation.** Pokud dvě a více oblastí bez ownera: tým je moc malý na 6 oblastí. Redukovat na 2–3 prioritní a zbytek odložit.

## 7. Maturity audit otázky bez odpovědi ("Nevím")

**Symptom.** Owner v Phase 6 odpovídá na 80%+ otázek "Nevím".

**Root cause.** Buď otázky jsou pro tu fázi firmy nerelevantní, nebo owner k oblasti není dostatečně blízko.

**Action.**

1. "Nevím" je validní odpověď. Není chyba.
2. Pokud "Nevím" převažuje nad 50%: oblast je pro klienta v early stage (Level 1 ještě není naplněn).
3. Plugin to interpretuje jako "Mapa firmy = Level 0 v dané oblasti". To je užitečný insight, ne selhání.
4. Vrátit se k otázkám za 4 týdny.

**Escalation.** Žádná — to je legitimní stav baseline.

## 8. Detekce nesedí ("překvapilo nás")

**Symptom.** V Phase 3 read-out tým označuje oblast jako "tohle není pravda, my X nemáme jako problém".

**Root cause.** Buď detection model nemá dost dat o klientovi, nebo metodika sama je pro segment klienta nepřesná.

**Action.**

1. Zapsat exception do decision mema (Phase 3 output).
2. Plugin v dalším běhu Aktualizace váhuje signál níž.
3. Po 3+ takových exception ze stejné oblasti: úprava plugin config per klient (tj. customizace promptu).
4. Po N klientech se stejnou exception: signál pro internal mode = metodika potřebuje revizi.

**Escalation.** Cross-tenant pattern = product responsibility.

## 9. Slack #klient-{slack_channel_name} bez aktivity

**Symptom.** Klient kanál existuje, ale klient tam nepíše. Plugin nemá feedback signál.

**Root cause.** Klient buď zapomněl, že kanál existuje, nebo nemá zvyk psát do něj.

**Action.**

1. Plugin generuje rotující "nudge" — jednou týdně přes konzultanta: "Co tento týden ne­fungovalo nebo bys měnil?"
2. Pokud 4+ týdny bez aktivity: kanál pravděpodobně mrtvý. Eskalovat osobním kontaktem.

**Escalation.** Nepoužívat plugin na nahrazení lidského vztahu.

## 10. Klient žádá feature, kterou Smartin neumí

**Symptom.** "Šlo by, aby Smartin uměl X?"

**Root cause.** Reálná feature gap nebo špatně komunikované omezení.

**Action.**

1. Plugin klasifikuje: existuje-li feature → odkázat na dokumentaci. Pokud ne → zapsat jako Product signal.
2. Plugin generuje strukturovaný feedback ticket (Reality → Interpretace → Next step) a posílá do interního Smartin systému.
3. Klient dostane potvrzení: "Zapsali jsme, klient X bude první kdo dostane vědět při launch."

**Escalation.** Akumulace 3+ stejných requestů = priorita pro Product roadmap. Internal mode.

## 11. Workspace už má prior data (není to truly první Aktualizace)

**Symptom.** Phase 2 by měla být "první Aktualizace", ale workspace už má `case-result` data z předchozích běhů (typické u internal SMARTIN COMPANY workspace, nebo když klient byl onboardován dvakrát).

**Root cause.** Smartin neresetuje case-result data při nové onboarding session. Prior data zůstává viditelná v Mapě firmy.

**Action.**

1. GET `/case-result/overview/{ws}` a zkontrolovat `lastUpdated` per area.
2. Pokud snapshot je čerstvý (<7 dní) a connector setup se nezměnil: přijmout existující data jako Phase 2 output, pokračovat na Phase 3.
3. Pokud snapshot stale (>7 dní) nebo přibyl/odebral connector: re-trigger.
4. Re-trigger options:
   - **Subset** — top 1 case per oblast = 6 PUT volání, ~2–3 min, ověří svěžest LLM detekce.
   - **Full** — všechny case-configs = 60 PUT volání, ~10–30 min, kompletní fresh map.
5. Decision memo z Phase 3 vždy zachytit, jestli běžela fresh detekce nebo accept-snapshot.

**Escalation.** Žádná — to je workflow rozhodnutí, ne chyba.

## 12. Hypothesis creation přes API selhává (neexistuje endpoint)

**Symptom.** Plugin chce vytvořit DRAFT hypothesis programaticky, ale `POST /hypothesis` neexistuje nebo vrací 404/405.

**Root cause.** Backend nemá direct hypothesis creation endpoint. Vytvoření = UI flow přes `/chat/case-result/{id}` → "Ověřit" tlačítko → variant pick → "Zaplatit". LLM call generuje hypothesis text.

**Action.**

1. Pro sólo / interaktivní onboarding: driving přes Claude in Chrome (postup v smartin-api-access SKILL.md sekce "Hypothesis creation"). 4 clicks + ~15–20 s wait per hypothesis.
2. Pro batch onboarding (víc klientů, víc oblastí): naplánovat manuálně, user dokliká za 5 min per case.
3. Nikdy POST naslepo na `/hypothesis` — riziko malformed instance, případně auto-trigger validation.

**Escalation.** Když batch creation je opakovaný požadavek: product story — přidat `POST /hypothesis/from-template/{templateId}` endpoint.

## 11. Plugin nereaguje strukturovaně

**Symptom.** Klient napsal otázku, plugin odpovídá obecně bez Smartin struktury (žádné Reality → Interpretace → Next step, žádné krátké odstavce). Plugin se zachoval jako generický Claude.

**Root cause.** Skills se v Cowork session neaktivovaly — buď plugin není správně nainstalovaný, není enabled v Settings, nebo konverzace startovala před načtením skills.

**Action.**

1. Klient otevře Cowork Settings (⌘+,) → Skills. Ověří, že všechny 4 smartin-beta skills jsou enabled.
2. Klient zavře aktuální konverzaci a otevře novou („New chat" vlevo nahoře).
3. V nové konverzaci napíše explicitní trigger: „Načti si skill smartin-onboarding-guide a popiš mi Fázi 0."
4. Pokud Claude odpoví strukturovaně, plugin se aktivoval. Pokud ne, restartuj Cowork app.
5. Pokud i po restartu ne, eskaluj do Slacku  — Martin pomůže do 4h.

**Escalation.** Plugin instalace selhala. Martin pošle Tomovi screen-share session 15 min, společně reinstalujeme.

## 12. Memory bloat (klient má příliš mnoho memos)

**Symptom.** Plugin u jednoho klienta vidí > 30 memos za 4 týdny v HubSpot Notes. Klient se v tom nevyzná, nový člen týmu nedokáže najít, co je důležité.

**Root cause.** Plugin (nebo konzultant) zapisuje per session místo weekly digest. Default „uložit vše" v meta-awareness §2 byl porušen, nebo onboarding-guide Phase 7 weekly digest pravidlo nebylo dodrženo.

**Action.**

1. Plugin upozorní: „Tento klient má za 4 týdny [N] memos. To je hodně. Chceš zkonsolidovat? Většinou stačí 4 weekly digesty + 2–3 ad-hoc memos = ~7 záznamů/měsíc."
2. Pokud klient kývne, plugin projde memos chronologicky, navrhne sloučení podle týdne (pondělí–neděle = 1 weekly digest).
3. Sloučené digesty zapíše jako nový Note s tagem `[CONSOLIDATED]`. Původní memos NEMAŽE — přidá k nim tag `[ARCHIVED]` a poznámku „Sloučeno do digestu {date}".
4. Future zápisy: weekly digest pravidlo (viz meta-awareness §6 Pravidlo 2) — 1 memo/týden.

**Escalation.** Pokud konsolidace vyžaduje > 30 min manuálního judgement (memos jsou nesourodá), eskaluj na konzultanta s návrhem strukturní úpravy. Konzultant projde s klientem v páteční review.

## 13. Slack Connect — send_message failuje v klientském kanálu

**Symptom.** Plugin zkusí `slack_send_message` do `#klient-{slack_channel_name}` a dostane `missing_scope` / `channel_not_found` / `not_in_channel`. Nebo zápis projde, ale memo se v kanálu neobjeví.

**Root cause.** `#klient-{slack_channel_name}` je Slack Connect (sdílený kanál mezi Smartin workspace a klientovým workspace). Slack na úrovni platformy omezuje, co může app instalovaná v jednom workspace v Connect kanálu dělat. Funguje pouze: `slack_send_message_draft`, `slack_read_channel`, `slack_read_thread`. Failuje: `slack_send_message`, `slack_add_reaction`.

**Action.**

1. Před prvním send_message do `#klient-{slack_channel_name}` zavolat `slack_search_channels` s `channel_types="public_channel,private_channel"` a zkontrolovat, zda `is_shared = true` (Slack Connect indikátor).
2. Pokud ano, plugin přepíná na `slack_send_message_draft` pro všechny posty v tomto kanálu (nejen jeden, ne fallback po failure — preventivně).
3. Po vytvoření draftu plugin postuje notifikaci konzultantovi (DM nebo `#smartin-ops`):
   ```
   [DRAFT READY] #klient-{slack_channel_name} — {memo type} čeká na odeslání
   ```
4. Konzultant ráno (5 min denně) projde drafts a odešle ručně.

**Escalation.** Pokud i `slack_send_message_draft` selže, kanál pravděpodobně není Connect a problém je jinde (scope, oprávnění). Eskaluj na Slack workspace admin.

## 14. Klient nemá přístup do Smartin gdrive folderu

**Symptom.** Plugin v klientově Coworku (Matula / Tom / Karel / Naty) při startu zkusí číst `09_Klienti/{klient}/smartin_config.yaml` a dostane `403 Forbidden` nebo `404 Not Found`.

**Root cause.** Buď konzultant folder nenasdílil (smartin-d1-prep krok 3c), nebo klient ho nasdílel, ale klient nepřijal pozvánku, nebo email v `team.daily_owners` se nezhoduje s emailem, který klient používá v Drive.

**Action.**

1. Plugin se klienta zeptá: „Vidíš ve své Drive složku `09_Klienti/{klient}/`? Pokud ne, řekni mi to a já napíšu konzultantovi."
2. Pokud klient řekne „nevidím", plugin pošle do Slack `#klient-{slack_channel_name}` notifikaci konzultantovi (přes draft mode pokud Connect):
   ```
   [BLOCKER] {klient_email} nevidí Smartin Drive folder. Možný důvod:
   - folder není nasdílený
   - email v configu se nezhoduje s Drive emailem
   - klient nepřijal pozvánku
   Můžeš ho prosím nasdílet znovu / ověřit email?
   ```
3. Plugin u klienta drží **read-only fallback** — funguje bez Drive přístupu, ale memos a config drží jen v lokální session memory. Po obnovení přístupu plugin nabídne sync (zapsat lokální drafts do gdrive).
4. Plugin ve `session_state.yaml` (jakmile bude přístup) zapíše `blocked_on: "matula nemá Drive přístup, eskalováno {date}"`.

**Escalation.** Pokud 24 hodin po nasdílení klient stále nevidí, konzultant zavolá osobně. Drive sharing někdy neaktivuje email notifikaci, klient ji nedostane.

## 15. Oblast bez napojených dat — UNAVAILABLE místo placeholder STUCK

**Symptom.** V Phase 2 Aktualizace plugin chce běžet detekci na oblast, která **nemá napojený žádný relevantní connector** (např. CSM bez Intercom / Crisp / Help Scout / Zendesk). Pokud plugin pustí backend `/case-evaluation/`, detection vrací placeholder STUCK signaly (typicky 3 generické fráze: „Spokojenost zákazníků klesá", „Odchod zákazníků roste", „Příliš mnoho eskalací"). Klient to v Phase 3 read-outu okamžitě profláknul jako halucinaci (LS D-day 26. 5. 2026 — Karel: „CSM detection nemůže být přesná, nemáme napojená data").

**Root cause.** Detection LLM nemá vstupy → fabrikuje výstup. Backend tu nehlídá.

**Action.**

1. **Před** voláním `/case-evaluation/workspace?smartinField={X}` plugin ověří, jestli klient má connector pro tu oblast. Mapování v `smartin-api-access` §16: MARKETING → GA4/HubSpot/Mailchimp, SALES → HubSpot/Pipedrive, CUSTOMER_SUCCESS → Intercom/Crisp/Help Scout/Zendesk, OPERATIONS → Stripe/calendar, CUSTOMER_MARKET → bez specifického connectoru (always available), PRODUCT → GitHub/Linear/Jira.
2. Pokud connector chybí, plugin **nepustí** case-evaluation pro tu oblast. Místo toho oblast přejde do statusu `UNAVAILABLE` a v Mapě firmy se ukáže jako ⚪ s jediným signálem: „Chybí napojení {tool} — Smartin tu oblast zatím nevidí. Pojďme to nechat jako neznámé, dokud se nepřipojí."
3. Plugin v Phase 3 read-outu pro UNAVAILABLE oblast sám říká: „Nehrajeme si na to, že vidíme, když nevidíme. Tady je čisté znamení, že čeká na data, ne na pozornost."
4. V Phase 1 (Connect data) plugin tu oblast neoznačuje jako blocker — onboarding pokračuje s 5 oblastmi, UNAVAILABLE 6. se přidá až po napojení.

**Escalation.** Žádná — to je legitimní design choice, ne chyba. Pokud klient namítá „proč v Smartin app vidím STUCK?" — to je backend bug (zachycený v LS D-day handoff F2), eskalovat do dev týmu. Plugin sám placeholder STUCK nikdy nehraje.

## 16. Klient odpověděl 0× ANO ze 3 startovních otázek

**Symptom.** V Phase 0.7 klient odpověděl „nevím" nebo „ne" na všechny 3 outside-in otázky. Threshold: 0× ANO ze 3 (od v0.9; dříve bylo < 3× ANO z 9 — viz [[reference-rf-single-field-engine]]).

**Root cause.** Klient je v discovery fázi firmy, ne v evidence-based fázi. Nemá zatím dokumenty, procesy zachycené, customer interviews — to je legitimní stav pre-product / early-stage.

**Action.**

1. Plugin **nepustí** re-evaluation. Mapa firmy by byla skoro prázdná, klient by řekl „k čemu to bylo dobré".
2. Místo toho plugin rámcuje:

```
Máš zatím {N}× ano, {M}× ne, {K}× nevím ze 3 otázek. To je dobrý signál
— některé oblasti potřebují discovery víc než ověření. Mapa firmy by teď
byla skoro prázdná, což by ti nedalo info.

Navrhuji alternativu: pojďme se v příštím týdnu věnovat discovery.
Konkrétně:

  1. Tento týden si zorganizuj 3 rozhovory s klienty (kdokoli z posledních
     30 dní). Cíl: zjisti, jakou bolest formulují vlastními slovy.
  2. Po rozhovorech zachyť kratce v decision memos/{date}_discovery.md.
  3. Vrátíme se k 3 otázkám, část odpovědí se objeví sama.

Není to selhání. Je to znamení, že teď je čas hledat, ne ověřovat.
```

3. Plugin zapíše do `session_state.yaml`: `current_phase: "0.7_discovery_loop"`, `next_milestone: "{date+7d} — vrátíme se k 3 otázkám"`.
4. Pošle Slack notifikaci konzultantovi: „[INFO] {klient} v discovery loop, 0 ANO ze 3 na startu. Vrátí se za týden po 3 rozhovorech."

**Escalation.** Pokud klient po 2. discovery loop stále 0 ANO, eskalovat na konzultanta. Možná že Smartin pro něj **není správný nástroj v tomto stadiu** — discovery framework jako Y Combinator startup school nebo Lenny's bootcamp by mu mohl pomoct víc než operating system.

**Threshold rationale.** Smartin engine v2.2 (13. 6. 2026) běží single-field per case_config. Stačí 1 ANO pro spuštění re-evaluation cyklu (= 1 oblast má alespoň jednu pozitivní evidenci). Multi-field threshold (≥3 ANO z 9) byla legacy logika.

## 17. Write na maturity API selhalo — 403 missing role

**Symptom.** Plugin v klientově Coworku zkusil `PUT /researchFields/workspace/{ws}/answer/detail/{rfId}` a dostal 401 / 403 — klient nemá `WORKSPACE_ADMIN` roli v Smartin Keycloak. Karel z Legal Systems narazil 5. 6. 2026 při Sales maturity backfill.

**Root cause.** Klient v jeho vlastním workspace by **měl** mít roli zapisovat maturity answers — je to jeho data. Že nemá, je dnes auth gap v Smartin role modelu. Standardní onboarding přiděluje role, která dovoluje číst, ale ne zapisovat.

**Action — žádný retry kruh, hned 2 čisté cesty:**

Plugin po prvním 403 NEZKOUŠÍ retry. Místo toho honest message + 2 volby:

```
Hej, narazil jsem na zeď. Tvoje role v tomto workspace dnes nedovoluje
zápis maturity odpovědí přes API. Není to chyba u tebe — je to gap
v Smartin role modelu, který opravujeme.

Dvě cesty, jak to dokončit:

  1. Pošlu tvému konzultantovi (Martinovi) hotový mapping souboru
     a on ti to nahraje za tebe. Cca 5 minut jeho času, tvoje práce
     se neztratí. Doporučená cesta.
  2. Klikneš si to ručně v Smartin app UI s mappingem jako tahákem.
     ~30 minut tvého času. Funguje, ale je to nudné.

Co volíš?
```

**Cesta 1 — konzultantský handoff (default):**

1. Plugin vytvoří strukturovaný handoff soubor v Drive: `09_Klienti/Smartin × {Klient}/handoff/{date}_{oblast}_maturity_backfill.md` — obsahuje:
   - Workspace ID + API base URL
   - Per RF: ID, název, navržená odpověď (DONE/NOT_DONE/PENDING), důkaz/poznámka
   - PENDING konflikty (otázky, kde klient si nebyl jistý) — vyžadují konzultantovo yes/no
   - MVA assets (links na soubory, které klient nahrál jako důkazy)
   - Plus aktuální stav před zápisem (z `smartin_get_research_field_answer` per RF, ať konzultant nepřepíše existující data)
2. Plugin pošle notifikaci konzultantovi:
   - Slack DM (z `team.consultant.slack_handle` v configu)
   - Plus pinned message do `#klient-{slack_channel_name}` se shrnutím
   - Plus draft email pokud klient používá email channel
3. Plugin uloží pending state do `09_Klienti/Smartin × {Klient}/pending_writes/{date}_{oblast}.yaml` — žádná práce se neztratí
4. V dalším weekly review (oba) plugin připomene: „před týdnem klient X poslal Y odpovědí, status: zapsáno / čeká"

**Cesta 2 — klient klikne sám:**

1. Plugin otevře Smartin app Maturity Audit pro danou oblast (přes Chrome MCP)
2. Klient klikne mapping pole po poli, plugin sleduje progress
3. Po dokončení plugin trigne `smartin_evaluate_workspace` per oblast (z MCP) nebo PUT case-eval přes token

**Konzultantský workflow (Martin přijme handoff):**

1. Plugin v Martinově Coworku detekuje nový soubor v `handoff/` folderu klientského subadresáře
2. Plugin: „Karel ti poslal 23 odpovědí na Sales maturity. 3 PENDING konflikty čekají na tvoje yes/no. Otevřít?"
3. Martin schválí 3 PENDING (nebo upraví), plugin přes `smartin_upsert_research_field_answer` (NEW v MCP) zapíše batchem
4. Plugin trigne `smartin_evaluate_workspace(workspace_id=LS, smartin_field=SALES)` pro reevaluaci
5. Plugin pošle Karlovi notifikaci „Sales maturity zapsáno, mapa firmy se aktualizovala, otevři Smartin app"
6. Plugin zapíše decision memo do `decision_memos/{date}_sales_maturity_backfill.md`

**Escalation.** Pokud klient + konzultant po 7 dnech nepokračují (plugin sleduje v pending_writes), eskaluj na Smartin tým s product signal „klient X čeká 7+ dní na maturity backfill".

**Product signal.** Akumulace 3+ klientů s tímto failure handlerem = priorita pro backend tým: auto-grant `WORKSPACE_ADMIN` role klientovi po D-1 prep, ať plugin v jeho Coworku zapisuje sám.

## 18. RF zápis prošel, ale UI dropdown zůstal prázdný

**Symptom.** Plugin (nebo konzultant přes MCP) zapsal RF answer přes `upsert_research_field_answer`, API vrátilo `status: updated`, klient ale v UI vidí v dropdown "Stav" prázdné políčko. Text v "Odpověď" je tam (DONE / NOT_DONE), badge nad ním ne. Objeveno 7. 6. 2026 při LS Sales backfill — Martin to chytil na screenshotu, jinak by Karel viděl bordel.

**Root cause.** RF answer má v API **dvě nezávislá pole**:

- `answer` (text label) — backend detection ho čte
- `answerOption` (enum) — UI dropdown ho čte

Když plugin zapsal jen `answer` a vynechal `answerOption` (předchozí verze skillu říkala že je optional), backend detection fungovala, UI dropdown ne. Vznikl rozpor: Mapa firmy ukázala změnu (Sales overview 3 OK + 7 STUCK místo 10 STUCK), ale klient v RF detail viděl prázdný dropdown a ztratil důvěru ve výsledek.

**Action.**

```python
# CHYBA — UI bude prázdné
smartin_upsert_research_field_answer(
    workspace_id=ws, research_field_id=rf, answer="DONE", answer_option=None
)

# SPRÁVNĚ — UI ukáže "Hotovo" badge
smartin_upsert_research_field_answer(
    workspace_id=ws, research_field_id=rf, answer="DONE", answer_option="DONE"
)
```

Pravidlo: `answer == answer_option`, vždy. Bez výjimky pro CHECKLIST RFs.

**Mapování dropdown ↔ enum:**

| UI label | answer + answerOption |
|---|---|
| Hotovo | `DONE` |
| Nezodpovězeno | `NOT_DONE` |
| Čeká | `PENDING` |
| Přeskočeno | `SKIPPED` |

**Rollback hot path.** Pokud zápis už proběhl bez `answerOption`, plugin re-zapíše batchem se shodným answer + answer_option. Žádná data se neztratí — overwrite je idempotentní, `previousAnswer` se zachová v response pro audit.

**Detekce.** Po každém batch zápisu plugin volá `smartin_get_research_field_answer` na náhodně vybraném RF a kontroluje, jestli `answerOption` není null. Pokud null → re-fire batch s `answer_option`.

## 19. Drive create_file vytváří duplikát (update_file neexistuje)

**Symptom.** Plugin zápis session_state.yaml přes Drive MCP `create_file` vytvoří duplikát místo overwrite. Drive MCP nemá `update_file`. Karel z LS narazil 26.5.2026 (LS handoff F1).

**Root cause.** Drive MCP read-write je create-only. Žádný update endpoint.

**Action — naming strategy + read latest:**

1. Plugin při zápisu session_state, decision_memo, smartin_config NEPŘEPISUJE existující soubor. Místo toho:
   - Konfigurace (singleton, hodně se mění): `smartin_config.yaml` zůstává jednobodový, ale plugin při zápisu nejdřív `search_files` na stejné jméno, pak smaže staré (DELETE přes Drive MCP funguje), pak `create_file` nový.
   - State (rychle mění): `session_state_{YYYY-MM-DDTHH-MM}.yaml` = timestamped. Plugin při čtení vytáhne všechny `session_state_*.yaml`, seřadí sestupně dle názvu (lexicographic), čte nejnovější.
   - Memos (immutable): `decision_memos/{date}_{slug}.md` jednou napsáno, neměnit.

2. V Slack notifikaci konzultantovi po každém větším zápisu: „[INFO] state aktualizován → {filename}". Konzultant ví, co je aktuální.

3. Pokud Drive MCP přidá `update_file` v budoucnu, přepnout na in-place overwrite (jednodušší). Signál pro Drive MCP dev tým: `update_file` je P0 missing.

**Escalation.** Pokud Drive obsahuje > 10 session_state souborů, plugin v Phase 7 weekly review nabídne konzultantovi cleanup (smazat všechny kromě nejnovějších 3).

**Extension — `smartin_config.yaml` backup před každým update (od v0.10):**

Config je jednobodový dokument, ale obsahuje strukturní data (team, slack, drive, gmail, schema). Riziko: klient nebo plugin přepíše config, vznikne data loss.

Pravidlo: před každým write na `smartin_config.yaml`:

1. Plugin přečte aktuální verzi.
2. Plugin zachová kopii do `09_Klienti/Smartin × {klient}/_plugin_temp/smartin_config_{YYYY-MM-DD-HHMM}.yaml.bak`.
3. Plugin teprve potom volá `create_file` na nový obsah.
4. Plugin v Phase 7 weekly review nabídne cleanup: „máš {N} config backupů, nech nejnovější 3?".

Pokud `_plugin_temp/` neexistuje, plugin ho vytvoří (per failure handler #21 honest write probe).

## 20. Klient nemá placený Slack workspace (email-only path)

**Symptom.** V D-1 prep nebo Phase 0.6 plugin chce založit `#klient-{slack_channel_name}` v Slacku, klient ale nemá placený Slack workspace (jen free, kde Slack Connect nefunguje s externími workspaces). Meta IT (Laco Ruttkay) narazil 27.5.2026 (learnings §2).

**Root cause.** Free Slack workspace neumí Slack Connect channels s externími workspaces (Smartin × LS pattern). Klient musí mít Pro+ nebo Business.

**Action — email fallback:**

1. Plugin v `smartin_config.yaml` zapíše `communication_channel: email` místo `slack`.
2. Místo Slack channel založení: plugin přes Gmail MCP nastaví:
   - Label `Smartin × {Klient}` v konzultantově Gmail (filter rule: from:{klient_email_domain} → label)
   - První email „Vítej v Smartinu" z konzultantovy adresy klientovi (přes Gmail MCP `create_draft` → konzultant reviduje → sám odešle per draft-first rule)
3. V `smartin-api-access` §18 (Email fallback) recipes — viz tam.
4. Slack notifikace konzultantovi se nemění (interní Slack tým Martin používá Smartin Slack, ne klientský).
5. Decision memos zůstávají v Drive `09_Klienti/Smartin × {Klient}/decision_memos/` (Drive je primary canonical storage per v0.6+).

**Escalation.** Pokud klient později pořídí Pro Slack, plugin přepne `communication_channel: both` a založí channel.

## 21. Drive `canAddChildren: false` ale create_file funguje (false negative)

**Symptom.** Plugin v Phase 0.5 voláme `get_file_permissions` na klientův Drive folder, vrátí `canAddChildren: false`. Plugin neprávem usoudí, že folder není writable, a nepustí onboarding. LS narazil 26.5.2026 (handoff F6).

**Root cause.** Drive permission propagation z parent na child folder má lag. `canAddChildren: false` znamená „nepotvrzeno", ne „zakázáno". Reálné `create_file` často funguje.

**Action — honest write probe:**

1. Plugin po `get_file_permissions` NEVĚŘÍ `canAddChildren: false` napřímo. Místo toho:
2. Provede honest write probe: `create_file` malého text souboru do `{klient_folder}/_plugin_temp/{date}_probe.txt` s obsahem „Smartin write probe — můžeš smazat".
3. Pokud create_file projde (vrátí file_id) → folder JE writable, pokračuj v onboardingu.
4. Pokud create_file vrátí 403 nebo permission denied → folder skutečně není writable, zastaví workflow a notifikuje konzultanta (failure handler #14).
5. Probe souborem se zachová (klient sám smaže). Plugin v Phase 7 weekly review nabídne cleanup `_plugin_temp/` folderu.

**Signál pro Drive MCP dev tým:** `canAddChildren` má být reliable. Pokud má lag, dokumentovat.

## 22. Klient nemá web — prefetch coverage 0/3

**Symptom.** V Phase 0.7 plugin spočítal `prefetch.coverage = 0`. Klient nemá veřejný web (čistě B2B partnerství, freelancer pracující přes LinkedIn, konzultant s 1:1 referrally). Plugin nemá z čeho tipovat odpovědi na 3 startovní otázky.

**Root cause.** Phase 0.7 design (od v0.9) předpokládá, že plugin má alespoň jeden datový bod z prefetchu (web scrape, HubSpot deals, GA detection). Bez webu plugin neumí pre-fill.

**Action — Cesta B s prázdnými tipy:**

1. Plugin přepne Phase 0.7 z Cesta A na Cesta B (default pro coverage < 3 už dělá).
2. V Cesta B otázky stojí bez kontextu PŘED otázkou (kontextová věta zmizí). Otázky:

```
1/3   Máte v CRM otevřené dealy?
       [ano]   [ne]   [nevím]

2/3   Měříte návštěvnost a konverze na webu nebo aplikaci?
       [ano]   [ne]   [nevím]

3/3   Máte popsaného ideálního zákazníka?
       [ano]   [ne]   [nevím]
```

3. Po dokončení Phase 0.7 (≥ 1 ANO ze 3 → Mapa firmy, 0 ANO → discovery loop), plugin v intra k Phase 1 doporučí:

```
Mimochodem — nevidím tvoji firmu na webu. To není blocker, ale dva
důvody, proč to zmínit:

  1. Smartin se na web dívá pro pre-fill maturity dat (analytics, ICP
     signál, lead capture). Bez webu některé oblasti rozjedeme pomaleji.
  2. Lead generation přes web je standardní cesta růstu pro B2B SaaS.
     Pokud máš jiný (referrally, partnerství, outbound), je to OK —
     ale často stojí za to webový kanál později přidat.

Žádný next step teď, jen poznámka. Pokud chceš web později, zmín to —
strčíme to do Kompasu jako kostičku (Distribuce / kanál).
```

4. Plugin si v `session_state.yaml` zapíše `prefetch.no_web: true` → v Phase 7 weekly review připomene jednou.

**Pre-segment varianta:** pokud `prefetch.segment` obsahuje „konzultant / poradenství / freelancer", plugin přepne Phase 0.7 layout na C&T + Sales + Product variantu (per onboarding-guide). Pro tyto klienty je „bez webu" typický stav, ne anomálie.

**Escalation.** Pokud klient řekne „web mám, ale prefetch nezachytil" — plugin re-spustí proactive-fetch s podle URL (manuál input). Pokud zase 0 dat → web má technický problém (gtag.js missing, robots.txt blokuje scrape, atd.) — handover na konzultanta.

## 23. Existující klient má legacy Slack channel kód (před v0.9 naming konvenci)

**Symptom.** Plugin v Phase 0.5 čte `smartin_config.yaml` existujícího klienta a vidí `slack.channel: "#klient-ls"` (krátká zkratka, ne nová `#klient-legal-systems` konvence z v0.9).

**Root cause.** Před v0.9 plugin používal volné `slack_channel_code` (2-3 letter zkratky). Od v0.9 = `slack_channel_name` = celé jméno firmy lowercase. Existující klienti (LS = `klient-ls`, Meta IT = `klient-metait`) mají legacy kódy.

**Action — neměnit existující, jen flagnout:**

1. Plugin při startu session detekuje legacy pattern regex: `^#klient-[a-z0-9-]{2,8}$` (krátký kanál, < 8 znaků za prefixem).
2. Pokud match, plugin si zapíše do session `slack.naming_legacy: true` a do logu poznámku: „Channel `{název}` používá legacy naming (před v0.9). Zachovávám, klient si zvykl na link."
3. **Auto-rename = NIKDY.** Slack channel rename mění URL, ruší stávající thready, klient ztratí kontext. Plugin to nedělá samostatně.
4. Pokud konzultant explicitně řekne „přejmenuj `#klient-ls` na `#klient-legal-systems`", plugin:
   - Spustí `mcp__*__slack_search_channels` pro ověření, že nová verze ještě neexistuje
   - Vytvoří draft message do starého kanálu „[INFO] Tento kanál se přesouvá do `#klient-legal-systems`. Tady mě najdeš poslední 30 dní pro historii."
   - Vytvoří nový kanál `#klient-legal-systems` se stejnými members
   - Update `smartin_config.yaml` na nový name
   - Konzultant ručně archivuje starý kanál v Slack UI (plugin to neumí samostatně, plus chce explicit lidský klik)
5. Plugin v Phase 7 weekly review nepřipomíná (není to friction, jen drift).

**Pre-build pravidlo (od v0.10):** Noví klienti od v0.10 dostávají `slack.channel` per nový konvent (`#klient-{full_lowercase_name}`). Žádné nové legacy kódy.

**Klienti s legacy kódy (k 15. 6. 2026):**
- Legal Systems: `#klient-ls` (od 22. 5. 2026)
- Meta IT: `#klient-metait` (od 27. 5. 2026)

Tito klienti zůstávají na legacy názvech. Pokud se rozhodnou později rename, použít workflow výše.

## 24. Snapshot capture selhal

**Symptom.** Scheduled task `smartin-snapshot-weekly` (neděle 23:55) nedoběhl nebo `build_full_snapshot` vrátil error. Žádný `{YYYY-WW}.yaml` v `snapshots/`.

**Root cause.** Možnosti:
- `evaluate_workspace` selhal (Smartin API 500, viz handler #1)
- 60 paralelních `get_case_result` calls timeout (> 120s)
- Drive write 403 (viz handler #14, #21)
- Plugin Cowork app nebyla spuštěná v 23:55 (klient ji zavřel)

**Action.**

1. Plugin při pondělním 7:30 trigger pro Monday brief čte snapshots. Pokud chybí poslední week file, plugin re-trignne snapshot **synchronně teď** (před brief generation).
2. Pokud i synchronní snapshot selže, plugin použije baseline + diff vůči předposlednímu weekly. Brief obsahuje warning: „Snapshot z {datum staršího ZP} — re-running."
3. Plugin notifikuje konzultanta v Slack `#klient-smartin`: „[WARN] Snapshot {iso_week} {klient} se nepodařil zachytit. Brief generated z předchozího."
4. Exponential backoff retry (max 3× v intervalech 5/15/45 min)

**Escalation.** Pokud 2 týdny po sobě snapshot selže pro stejného klienta, eskalovat na konzultanta. Pravděpodobně problém s API, Drive permissions, nebo Cowork instalace.

## 25. Monday brief generation selhal

**Symptom.** Scheduled task `smartin-monday-brief-weekly` (pondělí 7:30) doběhl, ale draft writer selhal nebo brief content je prázdný.

**Root cause.**

- Slack token expired (`slack_send_message_draft` 401)
- Gmail token expired (`create_draft` 401)
- `communication_channel` v configu chybí (nebyl set v D-1 prep)
- Snapshot existuje, ale `decision_memos/` folder nepřístupný

**Action.**

1. Plugin re-extract token (per handler #3 API 401)
2. Pokud `communication_channel` chybí, plugin defaultuje na `slack` pokud má `slack.channel_name`, jinak `email` pokud má `team.sponsor.email`, jinak vůbec nepošle a notifikuje konzultanta
3. Pokud `decision_memos/` nepřístupný, sekce „Paměť" se vynechá, brief jde dál
4. Plugin notifikuje konzultanta: „[ERROR] Monday brief {klient} {iso_week} nešel poslat. {důvod}. Re-auth nebo manual send."

**Escalation.** Pokud 2 týdny po sobě brief nedojde klientovi, eskalovat — možná klient přestal používat Slack/email kanál (změnil práci, switched workspaces).
