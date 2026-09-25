# AGENTS.md — ZipShip Affiliate Engine

> **Status:** v3.0 · 2026-09-20 · Single Source of Truth für alle Agenten und Menschen in diesem Monorepo.
>
> Dieses Dokument gilt für jedes Repo, jede Site und jeden Branch. Bei Widersprüchen gilt die Präzedenzkette: **Recht > Sicherheit > Harte Regeln > Site-Pack-Konfiguration > Modul-Defaults > Agenten-Ermessen**.

## 0. Verwendung

Zu Beginn jeder Session in dieser Reihenfolge lesen:

1. Mission und harte Regeln (Kapitel 1–2).
2. Agenten-Autoritätsstufen (Kapitel 21), bevor ein Merge oder Deploy vorbereitet wird.
3. Sicherheit und Recht (Kapitel 13–14), sobald Daten, Prompts, Texte oder Trust-Bausteine betroffen sind.
4. Site-Pack-Vertrag bei Änderungen an einer Site.
5. Das Kapitel des betroffenen Moduls.
6. Arbeitsweise vor dem ersten Commit.
7. Freigabekapazität vor jedem Vorschlag, der zusätzliche menschliche Freigaben einführt.

Nicht raten: Unklare Annahmen unter `## Annahmen` im PR dokumentieren. Betrifft eine Annahme Geld, Recht oder Daten, wird der Task blockiert. Regeln werden nicht umgangen oder abgeschwächt; Änderungen an geschützten Regeln benötigen einen ADR.

## 1. Mission und Grenzen

ZipShip ist ein Baukasten für belegte, monetarisierte Nischenseiten, der mit jeder Site Wissen akkumuliert. Die drei geteilten Speicher sind:

- **Merchant Graph:** EPC, Conversion, Freigabe- und Stornoquote je Merchant/Netzwerk.
- **Component Ledger:** gemessene EPC/RPM je UI-Komponente und Variante.
- **Fact Store:** stabile, quellenbelegte Fakten mit Erhebungs- und Ablaufdatum.

Kein Massen- oder Thin-Content ohne Daten, kein Dark Pattern, keine Empfehlung gegen die Passung, kein unkontrolliertes Cross-Linking und kein geteilter Runtime-Origin für alle Sites.

## 2. Harte Regeln

1. Jede faktische Aussage braucht eine `fact_id`, Quelle und Erhebungsdatum (`check:grounding`).
2. Eine Route wird nur bei vorhandenen, deklarierten Datenquellen gerendert (`check:silos`).
3. Affiliate-Links kommen ausschließlich aus dem zentralen LinkBuilder und tragen `rel="sponsored nofollow"` sowie `target="_blank"`; die Kennzeichnung steht vor dem ersten Link (`check:links`).
4. Preise tragen `capturedAt`; veraltete Preise werden ausgeblendet und bei kaufkritischen Seiten aus der Sitemap genommen (`check:freshness`).
5. Aktive Sites unterscheiden sich in mindestens zwei von Theme, URL-Grammatik und Textbausteinen (`check:variance`).
6. Cross-Linking zwischen Portfolio-Domains braucht einen begründeten Eintrag in `portfolio.links.allow` (`check:portfolio`).
7. Primär-Keywords dürfen nicht kannibalisieren (`check:cannibalization`).
8. Ähnlichkeitsgrenzen des Portfolios sind einzuhalten (`check:similarity`).
9. Jeder LLM-Endpunkt braucht Rate-Limit, Kostendeckel und Kill-Switch (`check:llm-guards`).
10. Fremddaten sind feindlich: an jeder Außengrenze Zod-Schema rein und raus; kein ungeprüfter Feed-, Nutzer-, Scrape-, Tool- oder LLM-Inhalt in Prompt, HTML oder Entscheidungen (`check:boundaries`).
11. Secrets bleiben aus Repository und Client-Bundle (`check:secrets`).
12. Die Rechtsklasse steuert Trust-Bausteine automatisch (`check:trust`).
13. Entitlements kommen aus der Datenbank; Cookies, JWTs und Query-Parameter sind nur kurzlebige Caches (`check:entitlements`).
14. Kein Publish ohne grünes Release-Gate.
15. Tier-B- und Tier-C-Änderungen benötigen die Freigaben aus Kapitel 21 (`check:authority`).
16. Merchant Graph und Component Ledger werden über Staging, Mindeststichprobe und Anomalieprüfung fortgeschrieben (`check:ledger-anomaly`).
17. Fakten haben `valid_until`; abgelaufene Fakten gehen in Review, nicht still in die Löschung (`check:fact-freshness`).
18. Kontrollpfade sind Tier C: `ops/gates/**`, `.github/workflows/**`, Gate-/Check-Skripte, Branch Protection und prüfungsrelevantes Caching. Die Prüfung darf sich nicht selbst abschalten (`check:authority --self`).
19. Widerrufene oder korrigierte Fakten bleiben mit Status, Nachfolger und sichtbarer Korrekturnotiz erreichbar (`check:fact-continuity`).
20. Merchant Graph, Component Ledger und Fact Store brauchen PITR, dokumentiertes RPO/RTO und einen getesteten Restore (`check:backup`).
21. Menschliche Freigaben haben Obergrenzen; bei Überlauf stoppt die Publikation, es gibt kein Auto-Approve (`check:queues`).
22. Embeds sind cookiefrei, ohne `localStorage` oder Fingerprinting, sandboxed und verwenden nur einen sichtbaren Marken-Anker mit `nofollow` (`check:embeds`).
23. Lieferkette: frozen Lockfile, Postinstall-Allowlist, Audit-Schwelle „high“ für Geld-/Trust-Pfade und exakte Versionen (`check:supplychain`).
24. Zeitstempel werden als UTC/`timestamptz` verarbeitet; Ortszeit entsteht ausschließlich im Rendering (`check:time`).

Kein Agent deaktiviert Checks per Kommentar, `eslint-disable`, Workflow-Änderung oder Konfigurationskniff.

## 3. Repo-Layout

```text
AGENTS.md
portfolio.config.ts
packages/{core,ui,linkmagnet,advisor,feeds,catalog,affiliate,facts,seo,schema,geo,trust,commerce,analytics,variance,testing,cli}
sites/{_template,_archive,<site-id>}
apps/web
data/{facts,snapshots}
etl/
ops/{gates,runbooks,restore}
```

Site Packs enthalten keinen React-Code außerhalb von `overrides/` und dort höchstens drei Dateien. Gemeinsame Logik gehört in ein Paket. `etl/` unterliegt ebenfalls Typ-, Lockfile-, Snapshot- und CI-Regeln.

## 4. Befehle und Betriebsregeln

```bash
pnpm i
pnpm dev --filter <site>
pnpm build --filter <site>
pnpm lint && pnpm typecheck
pnpm test
pnpm test:e2e
pnpm new-site <id>
pnpm doctor <id>
pnpm gate <id>
pnpm gate:attribution <id>
pnpm gate:sunset <id>
pnpm check:all
pnpm feeds:pull <id>
pnpm facts:import <file>
pnpm facts:correct <id>
pnpm ledger:sync
pnpm ledger:promote
pnpm cli:block-merchant <network:merchant> --reason "..."
pnpm ops:restore-test
pnpm ops:queues
```

`feeds:pull`, `ledger:promote`, `facts:correct` und `cli:block-merchant` niemals gegen Produktion ausführen, außer der Task verlangt dies ausdrücklich. `ledger:sync` schreibt nur nach Staging. Produktionsdaten werden nicht überschrieben; Migrationen sind additiv und werden niemals editiert.

## 5. Site-Pack-Vertrag

Jede Site ist deklarativ und enthält mindestens:

- `id`, `domain`, `locale`, `risk` und die Persona mit Offenlegung, Verbotsliste und `aiDisclosure`.
- Theme, Layout und URL-Grammatik.
- Datenquellen für Produkte, Fakten und Geo-Daten inklusive Preisalter und Fakten-TTL.
- Taxonomie mit Silo-Pattern, benötigten Daten, Mindestfakten und Wortbudget.
- Advisor mit Intake, Fit-/Preis-/Verfügbarkeitsgewichtung, `fitFloor`, Tool-Allowlist und Turn-Limit.
- Affiliate-Merchants, Stripe-Konfiguration (standardmäßig `null`) und `subidPrefix`.
- Primär-Keyword und Publish-Budget.
- Operator, Trust- und Accessibility-Einstufung sowie eigene Analytics-Property.

Keine API-Keys, Secrets, Preise, Produkttexte oder Merchant-Sperrliste im Site Pack. Die Sperrliste ist Laufzeitzustand in der Datenbank und wird höchstens 30 Sekunden gecacht, zusätzlich CLI-seitig invalidiert.

## 6. Module

Pakete haben einen öffentlichen Einstiegspunkt `src/index.ts`, keine Import-Seiteneffekte, keine direkten Writes in geteilte Speicher und pure, testbare Kerne. Erwartbare Fehler werden als `Result<T, E>` behandelt. Zeit wird über eine injizierte `Clock` bezogen, nicht über ungeprüfte `new Date()`-Vergleiche.

Jedes Paket dokumentiert seine Degradation. Bei Feed-Ausfall werden Preise ausgeblendet, bei LLM-Ausfall wird auf Kaskade/Regeln zurückgefallen, bei Analytics-Ausfall wird der Redirect nicht blockiert und bei unbekanntem Affiliate-Token auf die interne Produktseite geleitet.

## 7. Affiliate und Money Path

Affiliate-URLs dürfen nie als Strings gebaut werden:

```ts
import { affiliateLink } from "@zs/affiliate";

const href = affiliateLink({
  merchant: "network:merchant",
  target: product.deeplink,
  ctx: { pageId, component: "Component", variant: "a", position: 1 },
});
```

`/go/*` verwendet 302, `X-Robots-Tag: noindex, nofollow`, keinen offenen Redirect und keinen Cookie-/`localStorage`-Zugriff. Der Klick wird vor dem Redirect nicht länger als 150 ms blockierend erfasst. Gesperrte oder unbekannte Merchants führen zur internen Produktseite.

Routing erfolgt nach Erwartungswert, nicht nach Provision: `provision × conversion_rate × freigabequote × (1 − stornoquote)`. Kill-Switch, Fit-Floor und Offenlegung stehen über dem EV. Neue Ledger-Werte gehen ausschließlich über Staging und R16 in den geteilten Speicher.

## 8. Content, SEO und Fakten

Pipeline: Brief → Retrieve → deterministisches Outline → belegter Draft → Grounding/Verbotsliste/Wortbudget/Ähnlichkeit → Freigabe → gedrosseltes Publish. Keine Treffer bedeuten Stop statt Platzhalterseite. Jede Seite hat `author_ref`; vor generierten Silo-Chargen sind mindestens zehn redaktionelle Seiten erforderlich.

Fakten werden mit stabilem Zitier-Anker auf der Trägerseite veröffentlicht (`/<route>#fakt-<id>`). `/fakt/<id>` ist nur ein `302`-Umleiter und `noindex`. Korrekturen bleiben sichtbar; Fakten werden nicht still gelöscht.

## 9. Commerce und Trust

Stripe ist standardmäßig aus. Bei Aktivierung gelten signierte, idempotente Webhooks, DB-basierte Entitlements und die Pflichtbausteine für Verkauf, Datenschutz und Barrierefreiheit. Rechtstexte sind keine Rechtsberatung und vor YMYL-Launch bzw. echtem Verkauf anwaltlich zu prüfen.

## 10. Gates und Definition of Done

Vor einem PR müssen mindestens `lint`, `typecheck`, `test` und `check:all` grün sein. Neue Logik bekommt Tests, neue Adapter Contract-Tests gegen Snapshots, keine neuen `any` oder `eslint-disable`, Änderungen an Verträgen ein Changeset und aktualisierte Templates/Doctor-Prüfungen. Annahmen und Autoritätsstufe gehören in den PR.

Das Launch-Gate prüft Site-Pack, Trust, Consent, redaktionelle Basis, Fakten, Feeds, `/go/`, Attribution, GSC/Sitemaps, Varianz, LLM-Guards, Monitoring, Backup/Restore, KI-Offenlegung, A11y und Review-Warteschlangen. Der Money-Path und die Negativtests sind blockierend. Das Attributions-Gate hat eine Frist von sieben Tagen; bei Verfehlung wird das Publish-Budget auf null gesetzt.

## 11. Arbeitsweise für Agenten

1. Erst lesen, dann planen, dann schreiben; bei größeren Änderungen einen Plan im PR dokumentieren.
2. Ein PR, ein Zweck; kleine Diffs; kein Refactoring mit Feature vermischen.
3. Geldlogik testgetrieben entwickeln.
4. Keine stillen Vertragsänderungen.
5. Neue Dependencies begründen und R23 prüfen.
6. Conventional Commits verwenden.
7. Kein Force-Push, keine Produktionsschreibvorgänge, keine Secrets, keine Gate-Umgehung.
8. Checks reparieren vorschlagen, nicht abschalten; Kontrollpfade sind Tier C.
9. Nicht getestete Aspekte ehrlich als „nicht getestet“ melden.
10. Vor Merge die höchste Autoritätsstufe aller geänderten Pfade prüfen.
11. Nicht parallel am selben Paket wie ein anderer offener Agenten-PR arbeiten.
12. Bei Unsicherheit die höhere Autoritätsstufe annehmen.

## 12. Autoritätsstufen

| Tier | Beispiele | Merge-Bedingung |
|---|---|---|
| A | UI, SEO, Tests, allgemeine Dokumentation, ETL ohne Produktionsschreibzugriff | CI grün |
| B | Affiliate, Commerce, Facts, Site-Konfiguration, `portfolio.config.ts`, Geldpfad-Dependencies, Migrationen, Produktions-ETL | CI grün + Human-Approval-Label |
| C | Harte Regeln, Sicherheits-/Rechts-/Authority-Kapitel, `AGENTS.md`, `ops/gates/**`, Workflows, Check-/Gate-Skripte, Branch Protection, Produktions-Secrets/Live-Stripe-Keys | CI grün + Human-Approval-Label + ADR in Kapitel 20 |

Fehlt ein Pfad in der Authority-Konfiguration, gilt Tier B. Ein Agent darf Tier-C-Änderungen vorbereiten, aber nicht selbst mergen oder deployen.

## 13. ADR-Grundsatz

Änderungen an Kapitel 2, 7, 13, 14, 21 oder 22 sowie an der Kontrollebene benötigen einen ADR mit Nummer, Datum, Kontext, Entscheidung und Konsequenz. Der aktuelle Regelstand umfasst ADRs zu getrennten Deploys, EV-Routing, Grounding, Autoritätsstufen, Ledger-Staging, Faktenverfall und -korrektur, Citation Anchors, Restore, Review-Budget, Embeds, Lieferkette, UTC-Zeit, zweistufigen Gates, Datenschutz/A11y und dem blockierten Site-Pack `holy`.

## 14. Betrieb und Wiederherstellung

P1 sind insbesondere rote Money-Paths, `/go/`-5xx, falsch geroutete gesperrte Merchants, Secrets, DB-Ausfall, unerreichbare Pflichtseiten und dauerhafte Stripe-Webhook-Fehler. P2 sind veraltete Feeds, verfehlte Attribution, Kostendeckel, Review-Rückstau, falsche YMYL-Fakten und Indexierungsquote unter 40 %. P3 werden wöchentlich gesammelt.

Zielwerte: Money-Path 99,5 %, `/go/` p95 unter 300 ms, Preise in mindestens 95 % der Angebote aktuell, erste Advisor-Ausgabe unter 3 s. PITR, tägliche verschlüsselte Backups, RPO 15 Minuten, RTO 4 Stunden und monatlicher Testrestore sind verpflichtend.

Bei Degradation darf eine Site langweilig werden, aber nicht falsch: lieber „Preis prüfen“, Regel-Fallback oder ungezählter Klick als veralteter Preis, erfundene Empfehlung oder blockierter Redirect.
