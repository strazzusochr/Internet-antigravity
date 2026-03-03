# ULTRA-MEGA PROJEKTPLAN & IMPLEMENTIERUNGSPLAN
# HyperMega C2 OSINT Search Engine

**Version:** 1.0.0  
**Erstellt:** 2026-03-03T05:38:16Z  
**Status:** Aktiv  
**Klassifizierung:** Intern  

---

## Executive Summary

Das **HyperMega C2 OSINT Search Engine** Projekt ist eine hochkomplexe, KI-gestützte Intelligence-Plattform für die Extraktion, semantische Verarbeitung und visuelle Darstellung von Surface-Web- und Dark-Web-Inhalten. Das System kombiniert verteiltes Headless-Browser-Crawling, NLP-Pipeline, hybride Vektor-/Volltextsuche und ein modernes 3D-Next.js-Frontend zu einem entkoppelten Microservice-Ökosystem.

Quellen: `architecture.md.resolved`, `bedienungsanleitung.md.resolved`, `protokollbericht.md.resolved`, `hyper_mega_c2_systembuch.md.resolved`, `master_3d_web_plan.md.resolved`, `task.md.resolved`, `walkthrough.md.resolved`, `walkthrough_ultimate.md.resolved`

### Aktueller Implementierungsstand (Aus Dokumenten extrahiert)

| Modul | Implementierungsgrad | Quelle |
|---|---|---|
| API Backend (FastAPI) | ~100 % | `protokollbericht.md.resolved`, Abschnitt 4.3 |
| Semantic Search / Embedding | ~100 % (lokal) | `hyper_mega_c2_systembuch.md.resolved`, Kap. 4 |
| Crawler / Spider | ~100 % (lokal) | `protokollbericht.md.resolved`, Abschnitt 2 |
| 3D Matrix Frontend | ~68 % | `master_3d_web_plan.md.resolved`, Abschnitt 4.1 |
| Mission Control Hub | ~25 % | `master_3d_web_plan.md.resolved`, Abschnitt 4.1 |
| Targeting / Crawler Control UI | ~22 % | `master_3d_web_plan.md.resolved`, Abschnitt 4.1 |
| System Health Dashboard | ~25 % | `master_3d_web_plan.md.resolved`, Abschnitt 4.1 |
| Auto-Deploy / 1-Click Cloud Link | 0 % | `master_3d_web_plan.md.resolved`, Abschnitt 5 |
| CI/CD Pipeline | 0 % | `task.md.resolved`, Abschnitt 8 |

---

## Abkürzungsverzeichnis

| Kürzel | Bedeutung |
|---|---|
| C2 | Command & Control |
| OSINT | Open Source Intelligence |
| NER | Named Entity Recognition |
| OCR | Optical Character Recognition |
| kNN | k-Nearest-Neighbors |
| SPA | Single Page Application |
| TBD | To Be Defined |
| VORSCHLAG | Zeitangabe ist geschätzt, nicht aus Dokumenten extrahiert |

---

## Meilensteinübersicht

| ID | Meilenstein | Zieldatum | Status |
|---|---|---|---|
| M-01 | Infrastruktur & Basisservices produktionsreif | 2026-03-31 (VORSCHLAG) | Offen |
| M-02 | Crawler-Layer vollständig & skaliert | 2026-04-30 (VORSCHLAG) | Offen |
| M-03 | NLP/KI-Pipeline vollständig integriert | 2026-05-15 (VORSCHLAG) | Offen |
| M-04 | API-Layer vollständig & dokumentiert | 2026-05-31 (VORSCHLAG) | Offen |
| M-05 | Frontend vollständig & cloud-deployed | 2026-06-30 (VORSCHLAG) | Offen |
| M-06 | Monitoring, CI/CD & Security | 2026-07-15 (VORSCHLAG) | Offen |
| M-07 | Beta-Test & Abnahme | 2026-07-31 (VORSCHLAG) | Offen |

---

## M-01: Infrastruktur & Basisservices

### T-01 – Docker-Compose Produktionskonfiguration

| Feld | Wert |
|---|---|
| **ID** | T-01 |
| **Titel** | Docker-Compose Produktionskonfiguration erstellen |
| **Beschreibung** | Erstelle eine vollständige `docker-compose.yml` für alle Kernservices: Elasticsearch 8.x (3 Nodes), Redis, Kafka + Zookeeper, PostgreSQL 15, FastAPI Backend, Next.js Frontend. Volumes, Healthchecks, Netzwerke und Environment-Variables definieren. |
| **Quelle** | `architecture.md.resolved` (Persistence Layer, Storage); `task.md.resolved`, Abschnitt 8; `hyper_mega_c2_systembuch.md.resolved`, Kap. 2 |
| **Owner** | TBD |
| **Priorität** | P0 |
| **Schätzung** | 3 PT (VORSCHLAG) |
| **Startdatum** | 2026-03-03 (VORSCHLAG) |
| **Enddatum** | 2026-03-07 (VORSCHLAG) |
| **Abhängigkeiten** | – |
| **Akzeptanzkriterien** | `docker compose up -d` startet alle Services ohne Fehler; alle Healthchecks grün; Elasticsearch Cluster-Status = green; Kafka Topic `raw_articles` existiert. |
| **Risiko** | Ressourcenmangel auf Zielserver (hoch × mittel); Gegenmaßnahme: Ressourcen-Limits in compose konfigurieren, Min. 16 GB RAM empfohlen. |
| **Testfälle** | `curl http://localhost:9200/_cluster/health` → `green`; `kafka-topics --list` zeigt `raw_articles`; Redis PING → PONG |
| **Deliverables** | `docker-compose.yml`, `.env.example`, `README-infra.md` |
| **CI/CD** | GitHub Actions Workflow: `docker compose config` + `docker compose up --dry-run` |
| **Checkpoint** | Vor Commit: alle Secrets aus `.env` in GitHub Secrets lagern; kein Plaintext-Passwort in compose. |
| **Rollback** | `docker compose down -v` entfernt alle Volumes; git revert auf letzten stabilen compose-Stand. |
| **Security** | Elasticsearch Auth aktivieren; Kafka SASL; Redis requirepass setzen; alle Ports nur intern exponieren. |
| **Monitoring** | Container-Restart-Counter, CPU/RAM pro Container in Prometheus |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

### T-02 – Elasticsearch Index-Schema definieren

| Feld | Wert |
|---|---|
| **ID** | T-02 |
| **Titel** | Elasticsearch Index-Schema für Artikel-Index |
| **Beschreibung** | Definiere das Mapping für den primären Artikel-Index. Pflichtfelder: `url` (keyword), `title` (text, analyzed), `content` (text, analyzed mit ngram), `published_date` (date), `crawled_at` (date), `author` (text), `category` (keyword), `entities` (nested: type, value), `sentiment_score` (float), `language` (keyword), `content_hash` (keyword), `vector` (dense_vector, 384 dims). Time-based Index-Rotation täglich. 2 Replicas pro Shard. |
| **Quelle** | `task.md.resolved`, Abschnitt 3 (Search-Engine); `hyper_mega_c2_systembuch.md.resolved`, Kap. 5 |
| **Owner** | TBD |
| **Priorität** | P0 |
| **Schätzung** | 2 PT (VORSCHLAG) |
| **Startdatum** | 2026-03-06 (VORSCHLAG) |
| **Enddatum** | 2026-03-10 (VORSCHLAG) |
| **Abhängigkeiten** | T-01 |
| **Akzeptanzkriterien** | PUT-Mapping ohne Fehler akzeptiert; Beispiel-Dokument korrekt indiziert; kNN-Query liefert Ergebnis. |
| **Risiko** | Mapping-Inkompatibilität nach Index-Neuerstellung (mittel × hoch); Gegenmaßnahme: Index-Templates und Aliases nutzen. |
| **Testfälle** | Testdokument indizieren; Volltextsuche auf `content`; kNN-Suche auf `vector`-Feld; Faceted Filter auf `category`. |
| **Deliverables** | `es_mapping.json`, `es_index_template.json` |
| **CI/CD** | Schema-Migration Script in GitHub Actions |
| **Checkpoint** | Schema-Backup via `_snapshot` vor jeder Mapping-Änderung. |
| **Rollback** | Alias auf alten Index zurückschalten; Snapshot-Restore. |
| **Security** | Index-Level Security (ILS) aktivieren; kein anonymer Zugriff. |
| **Monitoring** | Index-Size, Shard-Count, Query-Latenz (P95) |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

### T-03 – PostgreSQL Source-Registry Schema

| Feld | Wert |
|---|---|
| **ID** | T-03 |
| **Titel** | PostgreSQL Source-Registry anlegen |
| **Beschreibung** | Erstelle das DB-Schema für die Source-Registry: Tabellen `sources` (id, url, name, country, language, category, active, last_crawled, domain_reputation_score), `crawl_history` (id, source_id, status, started_at, finished_at, docs_count, errors), `user_searches` (saved searches). Migrations mit Alembic. |
| **Quelle** | `task.md.resolved`, Abschnitt 6 (Data-Storage); `architecture.md.resolved` (Storage Layer) |
| **Owner** | TBD |
| **Priorität** | P0 |
| **Schätzung** | 2 PT (VORSCHLAG) |
| **Startdatum** | 2026-03-06 (VORSCHLAG) |
| **Enddatum** | 2026-03-10 (VORSCHLAG) |
| **Abhängigkeiten** | T-01 |
| **Akzeptanzkriterien** | Alle Migrationen laufen fehlerfrei; CRUD-Operationen auf `sources` funktionieren; 1000 Quellen als Seed-Daten importiert. |
| **Risiko** | Schema-Änderungen während Produktion (mittel × mittel); Gegenmaßnahme: Alembic Migrations, Blue-Green Deployment. |
| **Testfälle** | Insert / Select / Update / Delete auf `sources`; Crawl-History-Eintrag erstellen; Pagination-Test. |
| **Deliverables** | `migrations/`, `seed_sources.sql`, `schema.dbml` |
| **CI/CD** | `alembic upgrade head` in CI nach Merge |
| **Checkpoint** | DB-Dump vor Migrations: `pg_dump -Fc`. |
| **Rollback** | `alembic downgrade -1` |
| **Security** | Prepared Statements; kein direkter DB-Zugriff aus Frontend; Passwort in Vault. |
| **Monitoring** | DB-Connection-Pool-Auslastung, Query-Zeit (P95), Disk-Usage |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

## M-02: Crawler-Layer

### T-04 – Scrapy-Redis Distributed Crawler

| Feld | Wert |
|---|---|
| **ID** | T-04 |
| **Titel** | Scrapy-Redis verteilten Crawler implementieren |
| **Beschreibung** | Implementiere den Basis-Crawler mit Scrapy + Scrapy-Redis für verteilte Ausführung. Minimaler Durchsatz: 10.000 Seiten/Minute. Delta-Crawling via Content-Hash (SHA-256). Deduplication-Queue in Redis. Error-Handling mit Exponential Backoff. Dead-Letter-Queue für dauerhaft fehlgeschlagene URLs. Output: strukturierte JSON-Dokumente in Kafka-Topic `raw_html_payloads`. |
| **Quelle** | `architecture.md.resolved` (Crawler Layer); `task.md.resolved`, Abschnitt 1; `walkthrough_ultimate.md.resolved`, Abschnitt 2 |
| **Owner** | TBD |
| **Priorität** | P0 |
| **Schätzung** | 5 PT (VORSCHLAG) |
| **Startdatum** | 2026-03-10 (VORSCHLAG) |
| **Enddatum** | 2026-03-20 (VORSCHLAG) |
| **Abhängigkeiten** | T-01, T-03 |
| **Akzeptanzkriterien** | 10.000 Seiten/Minute Durchsatz messbar; Duplikate werden erkannt und übersprungen; Dead-Letter-Queue befüllt sich bei Fehler; Kafka-Topic enthält JSON-Payloads. |
| **Risiko** | Rate-Limiting durch Zielsites (hoch × mittel); Gegenmaßnahme: Proxy-Rotation, User-Agent-Wechsel, Crawl-Delay. |
| **Testfälle** | 100 URLs crawlen, Durchsatz messen; URL doppelt einspeisen, Dedup prüfen; Fehlerhafte URL → Dead-Letter-Queue; Kafka-Consumer Topic prüfen. |
| **Deliverables** | `crawler/spider.py`, `crawler/middlewares.py`, `crawler/settings.py` |
| **CI/CD** | Unit-Tests + Integration-Test gegen lokales Kafka |
| **Checkpoint** | Redis-Snapshot vor Restart; Dead-Letter-Queue regelmäßig prüfen. |
| **Rollback** | Spider stoppen; Redis-Queue leeren; letzten stabilen Commit deployen. |
| **Security** | User-Agent mit Kontakt-Email (robots.txt konform); kein Passwort im Code. |
| **Monitoring** | `crawler_throughput_docs_per_sec`, `crawler_error_rate`, `dead_letter_queue_size` |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

### T-05 – Playwright Headless Browser Integration

| Feld | Wert |
|---|---|
| **ID** | T-05 |
| **Titel** | Playwright JS-Rendering und Stealth-Layer implementieren |
| **Beschreibung** | Integriere Playwright für JavaScript-Rendering von SPAs. Stealth-Modus via `playwright-stealth`: Canvas-Fingerprint-Noise, WebGL/GPU-Spoofing, Navigator-Mutation, `navigator.webdriver = false`. Bezier-Mausbewegungen, heuristisches Scrolling. User-Agent-Rotation via `fake_useragent`. Einsatz nur für Sites, die JS-Rendering benötigen (Whitelist in `sources.json`). |
| **Quelle** | `protokollbericht.md.resolved`, Abschnitt 2.1–2.3; `hyper_mega_c2_systembuch.md.resolved`, Kap. 3.1 |
| **Owner** | TBD |
| **Priorität** | P1 |
| **Schätzung** | 4 PT (VORSCHLAG) |
| **Startdatum** | 2026-03-18 (VORSCHLAG) |
| **Enddatum** | 2026-03-25 (VORSCHLAG) |
| **Abhängigkeiten** | T-04 |
| **Akzeptanzkriterien** | Playwright-Seiten werden gerendert; `navigator.webdriver` ist `false` im Test; User-Agent wechselt bei jedem Request; keine Bot-Detection-Blockade auf Test-Sites. |
| **Risiko** | Playwright erzeugt hohen RAM-Verbrauch in Cloud (hoch × hoch); Gegenmaßnahme: Browser-Pool mit Max-Concurrency begrenzen, Browser nach n Pages recyceln. |
| **Testfälle** | SPA-Seite crawlen; DevTools-Check auf webdriver-Flag; 10 verschiedene User-Agents überprüfen; Cloudflare-Site testen (falls erlaubt). |
| **Deliverables** | `crawler/playwright_engine.py`, `crawler/stealth_config.json` |
| **CI/CD** | Integration-Tests mit Playwright-Test-Site |
| **Checkpoint** | Browser-Instanzen nach jedem Run schließen; Memory-Leak prüfen. |
| **Rollback** | Playwright-Feature-Flag deaktivieren; Fallback auf HTTP-Request. |
| **Security** | Kein Credential-Caching im Playwright-Context; Sandboxed Browser-Instanzen. |
| **Monitoring** | `playwright_active_browsers`, `playwright_page_load_time_ms`, `playwright_error_rate` |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

### T-06 – Tor/SOCKS5 Proxy für .onion-URLs

| Feld | Wert |
|---|---|
| **ID** | T-06 |
| **Titel** | Tor-Proxy-Integration für Dark-Web-Crawling |
| **Beschreibung** | Implementiere automatische Erkennung von `.onion`-Domains. Bei .onion-URL: Route Traffic über SOCKS5-Proxy (`127.0.0.1:9050`). Automatische Graph-Expansion: extrahiere neue .onion-Links aus gescraptem DOM und füge sie der Crawl-Queue hinzu. Tor-Circuit-Rotation alle N Requests. |
| **Quelle** | `protokollbericht.md.resolved`, Abschnitt 3.1; `hyper_mega_c2_systembuch.md.resolved`, Kap. 3.2 |
| **Owner** | TBD |
| **Priorität** | P2 |
| **Schätzung** | 3 PT (VORSCHLAG) |
| **Startdatum** | 2026-03-24 (VORSCHLAG) |
| **Enddatum** | 2026-03-28 (VORSCHLAG) |
| **Abhängigkeiten** | T-05 |
| **Akzeptanzkriterien** | .onion-URL wird via Tor geroutet (verifiziert durch Exit-IP-Check); neue .onion-Links werden in Queue eingefügt; Crawl-History enthält Tor-Flag. |
| **Risiko** | Tor auf Hugging Face gesperrt (hoch × hoch); Gegenmaßnahme: Externen Tor-Relay-Service konfigurieren oder Feature für On-Premise-Deploy vorbehalten. |
| **Testfälle** | Test-Onion-Site crawlen; IP-Leak-Test; neue Links aus DOM extrahiert; Rollback auf HTTPS falls Tor nicht verfügbar. |
| **Deliverables** | `crawler/tor_proxy.py`, Tor-Konfiguration in `docker-compose.yml` |
| **CI/CD** | Tor-Integration-Test in separatem Pipeline-Step |
| **Checkpoint** | Tor-Daemon-Status prüfen; Circuit-Counter loggen. |
| **Rollback** | `USE_TOR=false` Environment-Variable; .onion-URLs in Dead-Letter-Queue. |
| **Security** | Keine Klartextidentifikation durch Tor-Exit-Nodes; kein Credential-Handling über Tor. |
| **Monitoring** | `tor_active_circuits`, `onion_crawl_success_rate`, `tor_circuit_rotations_total` |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

### T-07 – PDF/DOCX Forensic-Extraktion

| Feld | Wert |
|---|---|
| **ID** | T-07 |
| **Titel** | PDF & DOCX Deep-Text-Extraktion implementieren |
| **Beschreibung** | Erkenne Links zu `.pdf` und `.docx` im gescraptem DOM. Lade Dateien als Byte-Stream in RAM (max. 50 MB Limit). Extrahiere Text via `PyPDF2` (PDF) und `python-docx` (DOCX). Hänge extrahierten Text an das Haupt-Dokument an. EXIF-Metadaten mit `exifread` extrahieren (GPS, Autor, Kamera, Software). |
| **Quelle** | `protokollbericht.md.resolved`, Abschnitt 3.2; `hyper_mega_c2_systembuch.md.resolved`, Kap. 4.1 |
| **Owner** | TBD |
| **Priorität** | P2 |
| **Schätzung** | 2 PT (VORSCHLAG) |
| **Startdatum** | 2026-03-26 (VORSCHLAG) |
| **Enddatum** | 2026-03-28 (VORSCHLAG) |
| **Abhängigkeiten** | T-04 |
| **Akzeptanzkriterien** | PDF-Text korrekt extrahiert und indiziert; DOCX-Text extrahiert; EXIF-GPS-Daten im Dokument-Feld vorhanden; Dateien > 50 MB werden übersprungen und geloggt. |
| **Risiko** | Malformed PDFs verursachen Crashes (mittel × mittel); Gegenmaßnahme: Try/Except, Timeouts, Sandbox. |
| **Testfälle** | 5 Beispiel-PDFs verarbeiten; 3 DOCX verarbeiten; Datei mit EXIF-GPS-Daten verarbeiten; 60-MB-PDF testen (muss übersprungen werden). |
| **Deliverables** | `api/metadata.py`, Tests in `tests/test_metadata.py` |
| **CI/CD** | Unit-Tests mit Sample-Dateien |
| **Checkpoint** | Dateigröße prüfen vor Download; Timeout für Extraktion. |
| **Rollback** | Feature-Flag `ENABLE_DOC_EXTRACTION=false`. |
| **Security** | Keine persistente Speicherung der Rohdateien; Malware-Scan via ClamAV optional. |
| **Monitoring** | `doc_extraction_success_rate`, `doc_extraction_avg_ms` |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

## M-03: NLP / KI-Pipeline

### T-08 – Kafka Consumer & Indexer Worker

| Feld | Wert |
|---|---|
| **ID** | T-08 |
| **Titel** | Kafka-Consumer und Indexer-Worker implementieren |
| **Beschreibung** | Implementiere FastAPI/Celery Indexer-Worker, der Kafka-Topic `raw_html_payloads` konsumiert. Für jeden Payload: HTML-Parsing (BeautifulSoup4/lxml), Textbereinigung, dann NLP-Pipeline (T-09 bis T-12), dann Elasticsearch-Bulk-Indexing (Batches à 100 Dokumente). Consumer-Group für horizontale Skalierung. |
| **Quelle** | `architecture.md.resolved` (Pipeline Layer); `task.md.resolved`, Abschnitt 2; `walkthrough_ultimate.md.resolved`, Abschnitt 2 |
| **Owner** | TBD |
| **Priorität** | P0 |
| **Schätzung** | 3 PT (VORSCHLAG) |
| **Startdatum** | 2026-04-01 (VORSCHLAG) |
| **Enddatum** | 2026-04-05 (VORSCHLAG) |
| **Abhängigkeiten** | T-01, T-02, T-04 |
| **Akzeptanzkriterien** | 100.000 Dokumente in < 5 Minuten indiziert; Consumer-Lag in Grafana sichtbar; Batch-Fehler werden in Dead-Letter-Topic geloggt. |
| **Risiko** | Kafka-Consumer-Lag wächst unbegrenzt (mittel × hoch); Gegenmaßnahme: Auto-Scaling Worker, Lag-Alerting. |
| **Testfälle** | 1000 Mock-Payloads in Kafka; Consumer verarbeitet alle; ES-Index enthält 1000 Dokumente; Consumer-Lag = 0 nach Verarbeitung. |
| **Deliverables** | `indexer/consumer.py`, `indexer/pipeline.py` |
| **CI/CD** | Integration-Test mit Test-Kafka-Instance |
| **Checkpoint** | Consumer-Offset committen nur nach erfolgreichem ES-Index. |
| **Rollback** | Consumer-Offset zurücksetzen auf letzten stabilen Offset. |
| **Security** | Keine sensiblen Daten in Kafka-Logs; SASL-Authentifizierung. |
| **Monitoring** | `consumer_lag`, `indexing_rate_docs_per_sec`, `indexing_error_rate` |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

### T-09 – spaCy NER Integration

| Feld | Wert |
|---|---|
| **ID** | T-09 |
| **Titel** | spaCy Named Entity Recognition (NER) implementieren |
| **Beschreibung** | Integriere `spaCy en_core_web_sm` in die Indexer-Pipeline. Extrahiere PERSON, ORG, GPE (Orte) aus jedem Dokument. Zusätzlich: Regex-basierte Extraktion von BTC-Wallets (Strings beginnend mit 1/3/bc1, 25–39 Zeichen) und XMR-Wallets (Strings beginnend mit 4/8). Ergebnis als nested Array im `entities`-Feld des ES-Dokuments. |
| **Quelle** | `protokollbericht.md.resolved`, Abschnitt 6.3; `hyper_mega_c2_systembuch.md.resolved`, Kap. 4.3; `architecture.md.resolved` (Pipeline) |
| **Owner** | TBD |
| **Priorität** | P1 |
| **Schätzung** | 2 PT (VORSCHLAG) |
| **Startdatum** | 2026-04-03 (VORSCHLAG) |
| **Enddatum** | 2026-04-07 (VORSCHLAG) |
| **Abhängigkeiten** | T-08 |
| **Akzeptanzkriterien** | NER erkennt PERSON/ORG/GPE in 5 Beispielsätzen korrekt; BTC/XMR-Regex matcht gültige Wallet-Adressen; Performance: < 100 ms pro Dokument. |
| **Risiko** | spaCy-Modell zu groß für Cloud-Container (mittel × mittel); Gegenmaßnahme: `en_core_web_sm` (kleinstes Modell) nutzen. |
| **Testfälle** | Unit-Tests mit bekannten Testsätzen; 10 echte BTC-Wallet-Adressen im Regex-Test; Performance-Test 1000 Dokumente. |
| **Deliverables** | `nlp/ner.py`, `nlp/wallet_extractor.py`, `tests/test_ner.py` |
| **CI/CD** | Unit-Tests in GitHub Actions |
| **Checkpoint** | Modell-Version in `requirements.txt` pinnen. |
| **Rollback** | Feature-Flag `ENABLE_NER=false`. |
| **Security** | Wallet-Daten werden nur im Rahmen von OSINT-Analyse verwendet; kein Export ohne Autorisierung. |
| **Monitoring** | `ner_entities_extracted_total`, `ner_processing_time_ms` |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

### T-10 – Sentiment-Analyse (VADER)

| Feld | Wert |
|---|---|
| **ID** | T-10 |
| **Titel** | VADER Sentiment-Analyse implementieren |
| **Beschreibung** | Integriere `vaderSentiment` in die Indexer-Pipeline. Berechne Compound-Score für jedes Dokument. Speichere `sentiment_score` (float, -1.0 bis +1.0) und `sentiment_label` (positive/negative/neutral) im ES-Index. |
| **Quelle** | `architecture.md.resolved` (Pipeline, VADER Sentiment Analysis); `task.md.resolved`, Abschnitt 2 |
| **Owner** | TBD |
| **Priorität** | P2 |
| **Schätzung** | 1 PT (VORSCHLAG) |
| **Startdatum** | 2026-04-05 (VORSCHLAG) |
| **Enddatum** | 2026-04-07 (VORSCHLAG) |
| **Abhängigkeiten** | T-08 |
| **Akzeptanzkriterien** | Score für positiven/negativen Test-Text korrekt; Feld `sentiment_score` in ES-Dokument vorhanden; Performance: < 10 ms pro Dokument. |
| **Risiko** | VADER schlecht bei Fremdsprachen (mittel × niedrig); Gegenmaßnahme: Sentiment nur nach Übersetzung (T-11) anwenden. |
| **Testfälle** | 5 positiver / 5 negativer Testsätze; Überprüfung des Compound-Scores; Benchmark 10.000 Dokumente. |
| **Deliverables** | `nlp/sentiment.py`, `tests/test_sentiment.py` |
| **CI/CD** | Unit-Tests |
| **Checkpoint** | VADER-Lexikon-Version pinnen. |
| **Rollback** | Feature-Flag `ENABLE_SENTIMENT=false`. |
| **Security** | Keine personenbezogenen Daten in Sentiment-Logs. |
| **Monitoring** | `sentiment_processing_time_ms`, `sentiment_distribution` (Histogram) |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

### T-11 – Cross-Lingual Translation Pipeline

| Feld | Wert |
|---|---|
| **ID** | T-11 |
| **Titel** | Cross-Linguale Übersetzungs-Pipeline (deep-translator) |
| **Beschreibung** | Integriere `langdetect` für Sprachdetection. Bei Nicht-Englischen Dokumenten: Übersetzung via `deep-translator` auf Englisch vor der Embedding-Generierung. Ursprungssprache im Feld `language` speichern. Originaltext im Feld `content_original` erhalten. |
| **Quelle** | `protokollbericht.md.resolved`, Abschnitt 6.2; `hyper_mega_c2_systembuch.md.resolved`, Kap. 4.2; `architecture.md.resolved` (Pipeline, langdetect) |
| **Owner** | TBD |
| **Priorität** | P1 |
| **Schätzung** | 2 PT (VORSCHLAG) |
| **Startdatum** | 2026-04-05 (VORSCHLAG) |
| **Enddatum** | 2026-04-09 (VORSCHLAG) |
| **Abhängigkeiten** | T-08 |
| **Akzeptanzkriterien** | Russischer Text wird als `ru` erkannt und korrekt übersetzt; Englischer Text bleibt unverändert; `language`-Feld im ES-Dokument vorhanden; Performance-Overhead < 200 ms. |
| **Risiko** | Translation-API Rate-Limits (mittel × hoch); Gegenmaßnahme: Caching, Batching, Fallback auf Offline-Modell. |
| **Testfälle** | 5 Sprachen testen (DE/FR/RU/ZH/AR); Originaltext erhalten; API-Rate-Limit-Test. |
| **Deliverables** | `nlp/translator.py`, `nlp/lang_detector.py` |
| **CI/CD** | Unit-Tests mit Mock-Translation |
| **Checkpoint** | API-Key in Secrets; Rate-Limit-Counter in Redis. |
| **Rollback** | `ENABLE_TRANSLATION=false`; Text ohne Übersetzung indizieren. |
| **Security** | Kein Plaintext-API-Key; HTTPS zu Translation-API. |
| **Monitoring** | `translation_api_calls_total`, `translation_errors`, `translation_latency_ms` |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

### T-12 – Sentence-Transformer Embeddings (all-MiniLM-L6-v2)

| Feld | Wert |
|---|---|
| **ID** | T-12 |
| **Titel** | Sentence-Transformer Vektorembeddings generieren |
| **Beschreibung** | Integriere `sentence-transformers/all-MiniLM-L6-v2` in die API. Wandle jeden gescrapten Text (nach Übersetzung) in einen 384-dimensionalen Float32-Vektor um. Speichere Vektor im Elasticsearch `dense_vector`-Feld (T-02). Beim Start der API Modell vorladen (Warm-Up). |
| **Quelle** | `protokollbericht.md.resolved`, Abschnitt 4.1; `hyper_mega_c2_systembuch.md.resolved`, Kap. 4 + Kap. 5; `architecture.md.resolved` |
| **Owner** | TBD |
| **Priorität** | P0 |
| **Schätzung** | 2 PT (VORSCHLAG) |
| **Startdatum** | 2026-04-08 (VORSCHLAG) |
| **Enddatum** | 2026-04-11 (VORSCHLAG) |
| **Abhängigkeiten** | T-08, T-11 |
| **Akzeptanzkriterien** | Embedding für Text generiert; Vektor hat genau 384 Dimensionen; kNN-Suche liefert semantisch ähnliche Dokumente; Warm-Up < 30 s. |
| **Risiko** | Modell-Download bei Start zu langsam in Cloud (mittel × mittel); Gegenmaßnahme: Modell im Docker-Image vorbaken. |
| **Testfälle** | `len(embedding) == 384`; Cosinus-Ähnlichkeit zweier ähnlicher Texte > 0.8; Performance: < 50 ms pro Dokument. |
| **Deliverables** | `api/embeddings.py`, `Dockerfile` (Modell inkludiert) |
| **CI/CD** | Unit-Test: Dimension prüfen; Smoke-Test nach Deploy |
| **Checkpoint** | Modell-Version in `requirements.txt` pinnen. |
| **Rollback** | BM25-Only-Suche als Fallback. |
| **Security** | Modell aus vertrauensw. Quelle (Hugging Face Official); kein externer API-Call zur Laufzeit. |
| **Monitoring** | `embedding_generation_time_ms`, `embedding_queue_length` |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

### T-13 – OCR (Tesseract) Bildtext-Extraktion

| Feld | Wert |
|---|---|
| **ID** | T-13 |
| **Titel** | Tesseract OCR für Bild-zu-Text-Konvertierung |
| **Beschreibung** | Implementiere OCR-Modul: BeautifulSoup parst Bild-URLs aus HTML; Bilder werden per `requests` heruntergeladen; `pytesseract` + OpenCV extrahieren Text. Extrahierter Text wird an Dokument-Content angehängt. OCR-Flag im Dokument-Metadaten-Feld setzen. |
| **Quelle** | `protokollbericht.md.resolved`, Abschnitt 6.1; `hyper_mega_c2_systembuch.md.resolved`, Kap. 4.1 |
| **Owner** | TBD |
| **Priorität** | P2 |
| **Schätzung** | 2 PT (VORSCHLAG) |
| **Startdatum** | 2026-04-10 (VORSCHLAG) |
| **Enddatum** | 2026-04-14 (VORSCHLAG) |
| **Abhängigkeiten** | T-08 |
| **Akzeptanzkriterien** | Text aus Test-Screenshot korrekt extrahiert; OCR-Flag `ocr_used=true` im Dokument; Performance: < 2 s pro Bild; Bilder > 5 MB übersprungen. |
| **Risiko** | Tesseract schlägt bei niedrig-auflösenden Bildern fehl (mittel × niedrig); Gegenmaßnahme: Mindestresoltion-Filter, Fallback leer string. |
| **Testfälle** | Screenshot mit bekanntem Text; leeres Bild (Fallback); Bild > 5 MB (übersprungen). |
| **Deliverables** | `nlp/ocr.py`, `tests/test_ocr.py` |
| **CI/CD** | Unit-Tests mit Sample-Images |
| **Checkpoint** | Tesseract-Version in Docker pinnen. |
| **Rollback** | `ENABLE_OCR=false`. |
| **Security** | Bilder nur im RAM; keine persistente Speicherung von Bilddaten. |
| **Monitoring** | `ocr_success_rate`, `ocr_processing_time_ms` |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

### T-14 – Video-Transkription (YouTube / yt-dlp)

| Feld | Wert |
|---|---|
| **ID** | T-14 |
| **Titel** | Ghost-Media Modul: YouTube-Untertitel-Extraktion |
| **Beschreibung** | Erkenne Video-Plattform-URLs (youtube.com, rumble.com etc.) vor Playwright-Rendering. Nutze `youtube-transcript-api` und `yt-dlp` zum Abrufen von Closed-Captions/Untertiteln ohne Video-Playback. Extrahierten Text als Dokument-Content verarbeiten. Video-Flag im Metadaten-Feld setzen. |
| **Quelle** | `protokollbericht.md.resolved`, Abschnitt 6.4; `hyper_mega_c2_systembuch.md.resolved`, Kap. 4.4 |
| **Owner** | TBD |
| **Priorität** | P2 |
| **Schätzung** | 2 PT (VORSCHLAG) |
| **Startdatum** | 2026-04-12 (VORSCHLAG) |
| **Enddatum** | 2026-04-16 (VORSCHLAG) |
| **Abhängigkeiten** | T-04 |
| **Akzeptanzkriterien** | YouTube-URL → Transkript extrahiert ohne Video-Playback; `video_transcript=true` im Dokument-Feld; Performance: < 5 s. |
| **Risiko** | YouTube ändert API (hoch × mittel); Gegenmaßnahme: yt-dlp regelmäßig aktualisieren, Fallback auf leeren Content. |
| **Testfälle** | Bekannte YouTube-URL mit verfügbaren Untertiteln; URL ohne Untertitel (Fallback); nicht-YouTube-URL (übersprungen). |
| **Deliverables** | `crawler/video_module.py` |
| **CI/CD** | Unit-Tests mit Mock-API |
| **Checkpoint** | yt-dlp-Version pinnen; wöchentliches Update prüfen. |
| **Rollback** | `ENABLE_VIDEO_TRANSCRIPTION=false`. |
| **Security** | Keine Cookies oder Credentials für YouTube API. |
| **Monitoring** | `video_transcription_success_rate`, `video_transcription_time_ms` |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

### T-15 – MinHash LSH Deduplication & TF-IDF Trending

| Feld | Wert |
|---|---|
| **ID** | T-15 |
| **Titel** | MinHash-Deduplication und TF-IDF Trending-Analyzer |
| **Beschreibung** | Implementiere Story-Deduplication via MinHash LSH Clustering (erkennt ähnliche, aber nicht identische Artikel). TF-IDF Trend-Analyzer über 24-Stunden-Fenster: Extrahiere Top-N Begriffe nach Häufigkeitszuwachs. Ergebnisse in Redis Hot-Cache speichern. API-Endpunkt `GET /api/trending` liefert Top-20-Trending-Topics. |
| **Quelle** | `architecture.md.resolved` (Pipeline: MinHash, TF-IDF); `task.md.resolved`, Abschnitt 2 + 7 |
| **Owner** | TBD |
| **Priorität** | P2 |
| **Schätzung** | 3 PT (VORSCHLAG) |
| **Startdatum** | 2026-04-14 (VORSCHLAG) |
| **Enddatum** | 2026-04-18 (VORSCHLAG) |
| **Abhängigkeiten** | T-08 |
| **Akzeptanzkriterien** | Zwei fast-identische Artikel → Dedup erkannt; Trending-Topics nach 24 h plausibel; API-Endpunkt liefert JSON. |
| **Risiko** | False-Positive-Dedup bei ähnlichen, aber inhaltlich anderen Artikeln (mittel × mittel); Gegenmaßnahme: Schwellwert konfigurierbar. |
| **Testfälle** | 5 Duplikat-Paare; 5 ähnliche aber verschiedene Paare; Trending-API nach Seeding 100 Artikel. |
| **Deliverables** | `nlp/dedup.py`, `nlp/trending.py` |
| **CI/CD** | Unit-Tests |
| **Checkpoint** | MinHash-Schwellwert in Config externalisieren. |
| **Rollback** | Dedup deaktivierbar per Flag. |
| **Security** | Keine PII in Trending-Daten. |
| **Monitoring** | `dedup_rate_percent`, `trending_refresh_interval_sec` |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

## M-04: API-Layer

### T-16 – FastAPI Backend vollständig implementieren

| Feld | Wert |
|---|---|
| **ID** | T-16 |
| **Titel** | FastAPI Backend: alle Endpunkte, Rate-Limiting, Auth |
| **Beschreibung** | Vollständige Implementierung des FastAPI Backends mit allen Endpunkten: `POST /api/search` (Hybrid BM25+kNN), `GET /api/sources`, `GET /api/trending`, `GET /api/stats`, `GET /api/health`, `WebSocket /api/live`. Rate-Limiting via Redis (1000 req/min pro IP). CORS-Konfiguration für Frontend. Optional: JWT-Authentifizierung. Gunicorn/Uvicorn Worker. |
| **Quelle** | `task.md.resolved`, Abschnitt 5; `protokollbericht.md.resolved`, Abschnitt 4.3; `hyper_mega_c2_systembuch.md.resolved`, Kap. 2.2; `architecture.md.resolved` (API Layer) |
| **Owner** | TBD |
| **Priorität** | P0 |
| **Schätzung** | 4 PT (VORSCHLAG) |
| **Startdatum** | 2026-04-18 (VORSCHLAG) |
| **Enddatum** | 2026-04-25 (VORSCHLAG) |
| **Abhängigkeiten** | T-02, T-03, T-08, T-12 |
| **Akzeptanzkriterien** | Alle Endpunkte antworten korrekt; Rate-Limiting blockiert nach 1000 req/min; WebSocket liefert Live-Updates; Swagger-Docs unter `/docs` verfügbar; Response-Time < 100 ms (P95). |
| **Risiko** | Semantic-Search-Latenz zu hoch (mittel × hoch); Gegenmaßnahme: Query-Cache in Redis (TTL 5 min), Approximate kNN. |
| **Testfälle** | Jeder Endpunkt mit gültigen + ungültigen Inputs; Rate-Limit-Test; WebSocket-Connect-Test; Performance-Test 1000 gleichzeitige Requests. |
| **Deliverables** | `api/main.py`, `api/routes/`, `api/middleware.py`, OpenAPI-Spec |
| **CI/CD** | Integration-Tests mit Testcontainer (ES + Redis) |
| **Checkpoint** | API-Keys in GitHub Secrets; kein Hard-Coding. |
| **Rollback** | Blue-Green Deployment; alten Pod behalten. |
| **Security** | Input-Validation (Pydantic); SQL-Injection-Prevention; XSS-Protection; HTTPS erzwingen. |
| **Monitoring** | `api_request_latency_ms` (P50/P95/P99), `api_error_rate`, `websocket_connections_active` |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

### T-17 – Quellen-Management: 1000 Nachrichtenquellen

| Feld | Wert |
|---|---|
| **ID** | T-17 |
| **Titel** | 1000 Nachrichtenquellen kuratieren und importieren |
| **Beschreibung** | Erstelle eine kuratierte Liste von 1000 Nachrichtenquellen aus diversen Ländern (DE, US, UK, FR, ES, IT, RU, CN, JP) und Kategorien (Politik, Wirtschaft, Tech, etc.). Felder: Name, URL, Land, Sprache, Kategorie. Importiere in PostgreSQL Source-Registry (T-03). Ergänze automatisches Hinzufügen neuer Quellen via NewsAPI.org. |
| **Quelle** | `task.md.resolved`, Abschnitt (Liste 1000 Nachrichtenseiten); `task.md.resolved`, Abschnitt Spezielle Anforderungen |
| **Owner** | TBD |
| **Priorität** | P1 |
| **Schätzung** | 3 PT (VORSCHLAG) |
| **Startdatum** | 2026-04-20 (VORSCHLAG) |
| **Enddatum** | 2026-04-24 (VORSCHLAG) |
| **Abhängigkeiten** | T-03 |
| **Akzeptanzkriterien** | 1000 Quellen in DB importiert; API `/api/sources` liefert vollständige Liste; mind. 10 Quellen pro Sprache; NewsAPI-Integration funktioniert. |
| **Risiko** | Quellen veralten schnell (mittel × niedrig); Gegenmaßnahme: Automatisches Health-Check der Quellen wöchentlich. |
| **Testfälle** | Alle 1000 Quellen in DB; Pagination der Sources-API; Filterung nach Sprache. |
| **Deliverables** | `data/sources_1000.json`, `scripts/import_sources.py` |
| **CI/CD** | Import-Script als Setup-Step |
| **Checkpoint** | Backup der Source-Liste vor Massenimport. |
| **Rollback** | `TRUNCATE sources; pg_restore` |
| **Security** | Keine Quellen aus Spam/Malware-Domains (Domain-Reputation-Check). |
| **Monitoring** | `sources_active_count`, `sources_last_crawled_age_hours` |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

## M-05: Frontend

### T-18 – Next.js Mission Control Hub (Landing)

| Feld | Wert |
|---|---|
| **ID** | T-18 |
| **Titel** | Next.js Mission Control Hub implementieren |
| **Beschreibung** | Implementiere das zentrale Einstiegsportal (Hub) als Next.js SPA. Design: Glassmorphism Dark Mode, WebGL-Shader-Hintergrund (CSS/Three.js Partikel-Netz in Dunkelviolett/Cyan). Boot-Sequence-Intro (2 s). Drei schwebende 3D-Kacheln für Targeting, 3D Matrix Search, System Health. Framer Motion Kachel-Tilt auf Hover. Festes HTTPS-Deployment (Vercel oder Hugging Face Static Space). |
| **Quelle** | `master_3d_web_plan.md.resolved`, Abschnitt 2.1, 6.1, 6.2; `hyper_mega_c2_systembuch.md.resolved`, Kap. 7 |
| **Owner** | TBD |
| **Priorität** | P1 |
| **Schätzung** | 4 PT (VORSCHLAG) |
| **Startdatum** | 2026-05-01 (VORSCHLAG) |
| **Enddatum** | 2026-05-08 (VORSCHLAG) |
| **Abhängigkeiten** | T-16 |
| **Akzeptanzkriterien** | Hub lädt in < 3 s; drei Kacheln sichtbar; Hover-Tilt-Effekt funktioniert; Link zur App ist öffentlich via HTTPS. |
| **Risiko** | WebGL-Performance auf mobilen Geräten (mittel × niedrig); Gegenmaßnahme: Fallback auf CSS-Animation, reduced-motion Media Query. |
| **Testfälle** | Desktop / Mobile Viewport; Lighthouse Score > 85; Kacheln klickbar; Dark Mode; Accessibility (WCAG 2.1 AA). |
| **Deliverables** | `frontend/src/app/page.tsx`, `frontend/src/components/Hub3D.tsx` |
| **CI/CD** | Vercel Deploy via GitHub Actions auf Merge in main |
| **Checkpoint** | `NEXT_PUBLIC_API_URL` in Vercel Environment konfiguriert. |
| **Rollback** | Vercel Instant Rollback auf letzten Deploy. |
| **Security** | CSP-Header; kein eval() im Frontend; HTTPS erzwungen. |
| **Monitoring** | Vercel Web Analytics; Lighthouse CI; Core Web Vitals |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

### T-19 – Targeting & Crawler Control Frontend

| Feld | Wert |
|---|---|
| **ID** | T-19 |
| **Titel** | Targeting & Crawler Control 3D-Frontend |
| **Beschreibung** | Implementiere die Targeting-UI als Next.js-Seite. Smart-Paste: URL-Extraktion aus Clipboard. URL-Pills mit Favicon und Domain-Name. Deep-Mode-Schalter (rotes Beleuchtungsthema für .onion). "God-Tier Launch" Button mit Warpdrive-Animation. API-Call zu `POST /api/crawl`. Status-Feedback nach Launch. Gradio-Admin durch diese UI ersetzen. |
| **Quelle** | `master_3d_web_plan.md.resolved`, Abschnitt 2.2, 6.3, 6.4, 7.1; `bedienungsanleitung.md.resolved`, Teil 1 |
| **Owner** | TBD |
| **Priorität** | P1 |
| **Schätzung** | 4 PT (VORSCHLAG) |
| **Startdatum** | 2026-05-06 (VORSCHLAG) |
| **Enddatum** | 2026-05-13 (VORSCHLAG) |
| **Abhängigkeiten** | T-16, T-18 |
| **Akzeptanzkriterien** | Smart-Paste extrahiert URLs aus beliebigem Text; URL erscheint als Pill mit Favicon; Launch-Button feuert API-Call; Warpdrive-Animation läuft; Fehlerfall zeigt Fehlermeldung. |
| **Risiko** | Clipboard-API auf manchen Browsern eingeschränkt (mittel × niedrig); Gegenmaßnahme: Fallback auf manuelle Textarea. |
| **Testfälle** | Paste 10 URLs aus Text; Deep-Mode aktivieren; Launch-Button; API-Mock für Fehlerfall. |
| **Deliverables** | `frontend/src/app/targeting/page.tsx`, `frontend/src/components/TargetPill.tsx` |
| **CI/CD** | Vercel Deploy; E2E-Test mit Playwright |
| **Checkpoint** | API-URL in Umgebungsvariable; kein Hard-Coding. |
| **Rollback** | Vercel Instant Rollback. |
| **Security** | URL-Validierung vor API-Call; kein XSS in URL-Pills. |
| **Monitoring** | Crawl-Launch-Events, Pill-Count pro Session |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

### T-20 – 3D Matrix Search Frontend

| Feld | Wert |
|---|---|
| **ID** | T-20 |
| **Titel** | 3D Matrix Search: Suchinterface + 3D Force Graph |
| **Beschreibung** | Implementiere die Haupt-Suchseite. Zentrale Suchleiste mit Glassmorphism. Semantic-AI-Toggle (Neon-Blau-Schalter). Suchergebnisse als animierte Karten (Framer Motion Stagger). 3D Force Graph via `react-force-graph-3d` (WebGL): Knoten = PERSON (blau), ORG (grün), Wallets (rot), Domain (weiß). Zoom, Rotation, Label-Hover. Card-View: Metadaten als Icons (EXIF, Deep Web Badge, OCR Badge, Übersetzungs-Icon). Match-Score (Semantic Similarity %) auf jeder Karte. Geo-Icon öffnet Karten-Overlay. |
| **Quelle** | `master_3d_web_plan.md.resolved`, Abschnitt 2.3, 7.2, 7.3, 7.4; `protokollbericht.md.resolved`, Abschnitt 5; `hyper_mega_c2_systembuch.md.resolved`, Kap. 7 |
| **Owner** | TBD |
| **Priorität** | P0 |
| **Schätzung** | 6 PT (VORSCHLAG) |
| **Startdatum** | 2026-05-10 (VORSCHLAG) |
| **Enddatum** | 2026-05-22 (VORSCHLAG) |
| **Abhängigkeiten** | T-16, T-18 |
| **Akzeptanzkriterien** | Suche liefert Ergebnisse; 3D Graph rendert Knoten und Links korrekt; Karten zeigen Metadaten-Icons; Semantic-Toggle ändert Query-Mode; Deep-Web-Badge bei .onion-Quellen. |
| **Risiko** | `react-force-graph-3d` SSR-Inkompatibilität mit Next.js (hoch → bekannt); Gegenmaßnahme: Dynamic Import `{ssr: false}`. |
| **Testfälle** | Suche "hack" → Ergebnisse; Graph rendert; Deep-Web-Ergebnis → Badge; Semantic-Toggle; Mobile-Viewport. |
| **Deliverables** | `frontend/src/app/search/page.tsx`, `frontend/src/components/ForceGraph3D.tsx`, `frontend/src/components/ResultCard.tsx` |
| **CI/CD** | Vercel Deploy; Playwright E2E |
| **Checkpoint** | Three.js Version auf 0.170.0 via overrides pinnen (bekannte Kompatibilität). |
| **Rollback** | Vercel Instant Rollback; Fallback-Liste ohne 3D-Graph. |
| **Security** | XSS-Schutz für angezeigte URLs/Inhalte (sanitize-html); kein eval. |
| **Monitoring** | `search_requests_total`, `search_latency_ms`, `graph_render_time_ms` |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

### T-21 – System Health Dashboard Frontend

| Feld | Wert |
|---|---|
| **ID** | T-21 |
| **Titel** | System Health & Analytics Dashboard |
| **Beschreibung** | Implementiere das System-Health-Dashboard: Live-Feed-Ticker (SSE/WebSocket), "Brain Size" Gauge-Chart (Anzahl indizierter Vektoren), Quantum-Stealth-Status-Schild (grün), Service-Status-Indikatoren (ES/Redis/Kafka/API). Nuclear-Reset-Button hinter Glas-Klappe mit Passwort-Dialog. API-Calls zu `/api/stats` und `/api/health`. |
| **Quelle** | `master_3d_web_plan.md.resolved`, Abschnitt 2.4, 7; `bedienungsanleitung.md.resolved`, Teil 2 |
| **Owner** | TBD |
| **Priorität** | P1 |
| **Schätzung** | 4 PT (VORSCHLAG) |
| **Startdatum** | 2026-05-20 (VORSCHLAG) |
| **Enddatum** | 2026-05-27 (VORSCHLAG) |
| **Abhängigkeiten** | T-16, T-18 |
| **Akzeptanzkriterien** | Live-Feed zeigt neue Crawl-Events in < 2 s; Gauge aktualisiert sich; Nuclear-Reset nur nach Passwort; alle Service-Statusindikatoren korrekt. |
| **Risiko** | WebSocket-Verbindung bricht auf mobilen Geräten oft ab (mittel × niedrig); Gegenmaßnahme: Auto-Reconnect-Logik. |
| **Testfälle** | Health-API mock; Live-Feed-Event simulieren; Nuclear-Reset mit falschem Passwort (ablehnen); Mobil-Viewport. |
| **Deliverables** | `frontend/src/app/health/page.tsx`, `frontend/src/components/GaugeChart.tsx` |
| **CI/CD** | Vercel Deploy |
| **Checkpoint** | Nuclear-Reset-Passwort in Secrets; nie im Code. |
| **Rollback** | Vercel Instant Rollback. |
| **Security** | Reset-Passwort via HTTPS + JWT; Rate-Limiting für Reset-Endpoint. |
| **Monitoring** | WebSocket-Reconnect-Rate, Live-Feed-Latenz |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

### T-22 – Cloud-Deployment (Vercel / Hugging Face)

| Feld | Wert |
|---|---|
| **ID** | T-22 |
| **Titel** | Automatisches 1-Click Cloud-Deployment |
| **Beschreibung** | Konfiguriere automatisches Deployment des Next.js Frontends auf Vercel (oder Hugging Face Static Space). `NEXT_PUBLIC_API_URL` als Umgebungsvariable setzen. GitHub Actions Workflow: bei Push auf `main` → `vercel deploy --prod`. API-Backend auf Hugging Face Docker Space deployen. Ergebnis: ein öffentlicher HTTPS-Link. |
| **Quelle** | `master_3d_web_plan.md.resolved`, Abschnitt 5; `bedienungsanleitung.md.resolved`, Teil 1, Schritt 1; `hyper_mega_c2_systembuch.md.resolved`, Kap. 2 |
| **Owner** | TBD |
| **Priorität** | P0 |
| **Schätzung** | 3 PT (VORSCHLAG) |
| **Startdatum** | 2026-05-28 (VORSCHLAG) |
| **Enddatum** | 2026-06-02 (VORSCHLAG) |
| **Abhängigkeiten** | T-18, T-19, T-20, T-21 |
| **Akzeptanzkriterien** | Öffentlicher HTTPS-Link erreichbar; Deploy-Zeit < 5 min nach Push; API-URL korrekt konfiguriert; kein Terminal nötig für Nutzer. |
| **Risiko** | Hugging Face Free-Tier-Limits (hoch × mittel); Gegenmaßnahme: Pro-Account oder Backup auf Vercel. |
| **Testfälle** | Push auf main → Auto-Deploy; Link öffnen auf Desktop+Mobil; API-Calls funktionieren. |
| **Deliverables** | `.github/workflows/deploy.yml`, `vercel.json` |
| **CI/CD** | GitHub Actions Workflow für Auto-Deploy |
| **Checkpoint** | Deploy-URL in Release-Notes dokumentieren. |
| **Rollback** | Vercel Instant Rollback; Hugging Face Space zurückrollen. |
| **Security** | Deployment-Tokens in GitHub Secrets; keine API-Keys im Frontend-Code. |
| **Monitoring** | Vercel Deploy-Status, Uptime-Check alle 5 min |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

## M-06: Monitoring, CI/CD & Security

### T-23 – Prometheus & Grafana Monitoring

| Feld | Wert |
|---|---|
| **ID** | T-23 |
| **Titel** | Prometheus Metriken + Grafana Dashboards |
| **Beschreibung** | Implementiere Prometheus-Metriken in allen Services (Crawler, API, Indexer). Grafana-Dashboards für: Crawler-Throughput, ES-Index-Size, Query-Latenz (P50/P95/P99), Error-Rates, Consumer-Lag. Alerting via Discord/Email bei Service-Ausfall oder Crawler-Throughput < 1000 docs/min. |
| **Quelle** | `architecture.md.resolved` (Ops Layer: Prometheus, Grafana); `task.md.resolved`, Abschnitt 7 |
| **Owner** | TBD |
| **Priorität** | P1 |
| **Schätzung** | 3 PT (VORSCHLAG) |
| **Startdatum** | 2026-06-01 (VORSCHLAG) |
| **Enddatum** | 2026-06-08 (VORSCHLAG) |
| **Abhängigkeiten** | T-01, T-08, T-16 |
| **Akzeptanzkriterien** | Alle definierten Metriken in Prometheus sichtbar; Grafana-Dashboards zeigen Live-Daten; Alert ausgelöst bei Simulation eines Service-Ausfalls. |
| **Risiko** | Zu viele Metriken verursachen Prometheus-Speicherproblem (niedrig × mittel); Gegenmaßnahme: Retention-Policy, nur kritische Metriken. |
| **Testfälle** | Prometheus `/metrics` erreichbar; Grafana-Dashboard lädt; Alert-Test (artificielle Fehlerrate erhöhen). |
| **Deliverables** | `monitoring/prometheus.yml`, `monitoring/grafana_dashboards/`, `monitoring/alerting_rules.yml` |
| **CI/CD** | Dashboard-as-Code via Grafana API |
| **Checkpoint** | Alert-Routing konfigurieren; Oncall-Kontakt hinterlegen. |
| **Rollback** | Monitoring ist read-only; kein Rollback nötig. |
| **Security** | Prometheus + Grafana nicht öffentlich exponieren; Basic Auth. |
| **Monitoring** | Prometheus Self-Monitoring (meta-metrics) |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

### T-24 – GitHub Actions CI/CD Pipeline

| Feld | Wert |
|---|---|
| **ID** | T-24 |
| **Titel** | GitHub Actions CI/CD Pipeline einrichten |
| **Beschreibung** | Erstelle GitHub Actions Workflows: (1) CI: Lint, Unit-Tests, Integration-Tests bei jedem Push/PR. (2) CD: Auto-Deploy Frontend auf Vercel + API auf Hugging Face bei Push auf `main`. (3) Security-Scan: Dependency-Check (pip-audit, npm audit), SAST (Bandit), Container-Scan. |
| **Quelle** | `task.md.resolved`, Abschnitt 8 (Deployment, CI/CD) |
| **Owner** | TBD |
| **Priorität** | P1 |
| **Schätzung** | 2 PT (VORSCHLAG) |
| **Startdatum** | 2026-06-05 (VORSCHLAG) |
| **Enddatum** | 2026-06-10 (VORSCHLAG) |
| **Abhängigkeiten** | T-22 |
| **Akzeptanzkriterien** | CI läuft bei jedem PR; CD deployed automatisch; Security-Scan blockiert bei kritischen CVEs. |
| **Risiko** | Lange CI-Laufzeiten (mittel × niedrig); Gegenmaßnahme: Caching, parallele Jobs. |
| **Testfälle** | PR mit fehlgeschlagenem Test → CI rot; Push auf main → Deploy grün; Dependency mit CVE → Scan rot. |
| **Deliverables** | `.github/workflows/ci.yml`, `.github/workflows/cd.yml`, `.github/workflows/security.yml` |
| **CI/CD** | Selbstreferenziell |
| **Checkpoint** | Secrets in GitHub Secrets; kein Token im Code. |
| **Rollback** | Workflow deaktivieren; manuelle Deploy-Befehle. |
| **Security** | `GITHUB_TOKEN` minimal scoped; Secrets rotation. |
| **Monitoring** | GitHub Actions Job-Success-Rate |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

### T-25 – Security & Compliance Checks

| Feld | Wert |
|---|---|
| **ID** | T-25 |
| **Titel** | Security-Härtung: Input-Validation, Auth, HTTPS |
| **Beschreibung** | Umfassende Security-Härtung: Input-Validation via Pydantic (API); SQL-Injection-Prevention (ORM, Prepared Statements); XSS-Schutz Frontend (sanitize-html, CSP-Header); HTTPS für alle externen Verbindungen; JWT-Auth für Admin-Endpoints; Rate-Limiting für alle öffentlichen Endpoints; Dependency-Audit in CI. Hinweis: "Nur für persönlichen/edukativen Gebrauch" in README. |
| **Quelle** | `task.md.resolved`, Abschnitt Code-Qualität + Lizenz; `task.md.resolved`, Abschnitt 5 (Rate-Limiting) |
| **Owner** | TBD |
| **Priorität** | P0 |
| **Schätzung** | 3 PT (VORSCHLAG) |
| **Startdatum** | 2026-06-08 (VORSCHLAG) |
| **Enddatum** | 2026-06-13 (VORSCHLAG) |
| **Abhängigkeiten** | T-16, T-22 |
| **Akzeptanzkriterien** | OWASP Top 10 adressiert; `bandit` + `npm audit` sauber; HTTPS erzwungen; Admin-Reset hinter Auth. |
| **Risiko** | Unbekannte Vulnerabilities in Abhängigkeiten (mittel × hoch); Gegenmaßnahme: Wöchentlicher Dependency-Scan via Dependabot. |
| **Testfälle** | SQL-Injection-Test; XSS-Test; Auth-Test (ungültiger JWT); Rate-Limit-Test. |
| **Deliverables** | Security-Checklist, `SECURITY.md` |
| **CI/CD** | Security-Scan in GitHub Actions |
| **Checkpoint** | Penetration-Test vor Go-Live. |
| **Rollback** | Hotfix-Branch; schneller Deploy via CD. |
| **Security** | Dieser Task ist selbst ein Security-Task. |
| **Monitoring** | `auth_failed_attempts_total`, `rate_limit_blocks_total` |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

## M-07: Beta-Test & Abnahme

### T-26 – End-to-End Beta-Test

| Feld | Wert |
|---|---|
| **ID** | T-26 |
| **Titel** | End-to-End Beta-Test und Abnahme |
| **Beschreibung** | Vollständiger End-to-End-Test des gesamten Systems: Targeting → Crawl → Indexierung → Suche → 3D-Graph. Manuelle Verifikation der bedienungsanleitung.md.resolved Schritte. Performance-Benchmarks: 10.000 URLs in < 10 min; 100.000 Dokumente in < 5 min; Query < 50 ms (P95); 1000 concurrent users. Lighthouse-Score > 85. |
| **Quelle** | `bedienungsanleitung.md.resolved` (alle Teile); `task.md.resolved`, Abschnitt Spezielle Anforderungen (Performance) |
| **Owner** | TBD |
| **Priorität** | P0 |
| **Schätzung** | 3 PT (VORSCHLAG) |
| **Startdatum** | 2026-07-15 (VORSCHLAG) |
| **Enddatum** | 2026-07-22 (VORSCHLAG) |
| **Abhängigkeiten** | T-01 bis T-25 |
| **Akzeptanzkriterien** | Alle Bedienungsanleitung-Schritte erfolgreich; Performance-Benchmarks erreicht; keine kritischen Bugs; Lighthouse > 85. |
| **Risiko** | Performance-Benchmarks nicht erreichbar (mittel × hoch); Gegenmaßnahme: Horizontal Scaling, Query-Optimierung. |
| **Testfälle** | Vollständiger Durchlauf nach Bedienungsanleitung; Load-Test mit k6; Lighthouse-CI. |
| **Deliverables** | Beta-Test-Report, Performance-Benchmark-Report |
| **CI/CD** | Automatisierter E2E-Test-Report nach Deploy |
| **Checkpoint** | Go/No-Go-Entscheidung nach Beta-Test. |
| **Rollback** | Beta-Phase verlängern; kritische Bugs fixen. |
| **Security** | Final Security-Review; Penetration-Test-Ergebnisse. |
| **Monitoring** | Alle definierten Metriken aktiv; Alerting getestet. |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

### T-27 – Dokumentation finalisieren

| Feld | Wert |
|---|---|
| **ID** | T-27 |
| **Titel** | Vollständige Dokumentation erstellen |
| **Beschreibung** | README.md mit Setup-Anleitung (3-Schritte-Install), Lizenzhinweis ("Nur für persönlichen/edukativen Gebrauch"), robots.txt-Compliance, User-Agent mit Kontakt-Email. Inline-Kommentare, Type-Hints (Python), TypeScript im Frontend. Docstrings. OpenAPI/Swagger vollständig. `CONTRIBUTING.md`. |
| **Quelle** | `task.md.resolved`, Abschnitt Code-Qualität + Lizenz |
| **Owner** | TBD |
| **Priorität** | P1 |
| **Schätzung** | 2 PT (VORSCHLAG) |
| **Startdatum** | 2026-07-22 (VORSCHLAG) |
| **Enddatum** | 2026-07-28 (VORSCHLAG) |
| **Abhängigkeiten** | T-26 |
| **Akzeptanzkriterien** | README vollständig; Lizenzhinweis vorhanden; OpenAPI-Spec vollständig; alle öffentlichen Funktionen dokumentiert. |
| **Risiko** | Dokumentation veraltet schnell (niedrig × mittel); Gegenmaßnahme: Docs-as-Code, automatisch aus Code generiert. |
| **Testfälle** | README-Schritt-für-Schritt-Durchlauf; OpenAPI-Spec validieren. |
| **Deliverables** | `README.md`, `SECURITY.md`, `CONTRIBUTING.md`, OpenAPI-Spec |
| **CI/CD** | Docs-Validierung in CI |
| **Checkpoint** | Lizenzhinweis vor Go-Live prüfen. |
| **Rollback** | n/a |
| **Security** | Keine Credentials in Dokumentation. |
| **Monitoring** | n/a |
| **Audit-Log** | Erstellt: 2026-03-03T05:38:16Z; Genehmigt: TBD |

---

## Widersprüche & Abweichungen zwischen Dokumenten

| ID | Konflikt | Dok 1 | Dok 2 | Bewertung | Empfehlung |
|---|---|---|---|---|---|
| W-01 | Datenbank-Backend: Elasticsearch vs. LanceDB | `architecture.md.resolved`: Elasticsearch 8.x | `hyper_mega_c2_systembuch.md.resolved`: LanceDB (serverless) | Beide Ansätze sind implementiert in verschiedenen Architektur-Stufen. LanceDB war ein Hugging-Face-spezifisches Deployment-Kompromiss. | Empfehlung: Elasticsearch für Produktions-Volumen (>1M Dokumente), LanceDB als Fallback für Cloud-Free-Tier. Beide parallel unterstützen via Abstraktionsschicht. |
| W-02 | Frontend-Deployment: Lokal vs. Cloud | `bedienungsanleitung.md.resolved`: `localhost:3000` | `master_3d_web_plan.md.resolved`: Vercel/Hugging Face Cloud | Lokal war der Status Quo, Cloud ist das Ziel. | Empfehlung: Cloud-Deployment (Vercel) als primäres Ziel; lokaler Dev-Mode (`npm run dev`) bleibt für Entwickler. |
| W-03 | Port-Konfiguration | `walkthrough.md.resolved`: 3005/8005 | `walkthrough_ultimate.md.resolved`: 3000/8000 | Beide Portkonfigurationen wurden in verschiedenen Entwicklungsphasen verwendet. | Empfehlung: 3000/8000 als Standard; 3005/8005 als Alternative (dokumentieren in `.env.example`). |
| W-04 | Admin-Interface: Gradio vs. Next.js | `hyper_mega_c2_systembuch.md.resolved`, Kap. 6: Gradio | `master_3d_web_plan.md.resolved`, Abschnitt 2.2: Next.js-Targeting-UI | Gradio war Interim-Lösung; Next.js ist das strategische Ziel. | Empfehlung: Next.js-Targeting-UI als Ziel; Gradio kurzfristig als Fallback. |

---

## Git-Branching-Schema

```
main (produktions-stabil)
  └── develop (integrations-branch)
       ├── feature/T-01-docker-compose
       ├── feature/T-04-scrapy-crawler
       ├── feature/T-08-kafka-consumer
       ├── feature/T-12-embeddings
       ├── feature/T-16-fastapi-backend
       ├── feature/T-18-hub-frontend
       ├── feature/T-20-3d-search
       └── fix/...

releases:
  v0.1.0  → M-01 (Infrastruktur)
  v0.2.0  → M-02 (Crawler)
  v0.3.0  → M-03 (NLP)
  v0.4.0  → M-04 (API)
  v0.5.0  → M-05 (Frontend)
  v1.0.0  → M-07 (Beta-Abnahme)
```

---

## Empfehlungen (Prioritäten)

1. **[P0] Sofortmaßnahme:** `docker-compose.yml` (T-01) und Elasticsearch-Schema (T-02) als Fundament aufsetzen.
2. **[P0] Kritisch:** FastAPI Backend (T-16) und Embedding-Pipeline (T-12) sind der Kern des Systems.
3. **[P0] Cloud-Deploy** (T-22) ist der entscheidende Schritt für das "1-Click"-Ziel aus `master_3d_web_plan.md.resolved`.
4. **[P1] Parallel:** Frontend-Entwicklung (T-18–T-21) kann parallel zu Backend-Tasks laufen, sobald API-Mocks bereit sind.
5. **[P1] CI/CD** (T-24) früh einrichten, um Qualität sicherzustellen.
6. **[P2] Später:** Tor-Proxy (T-06), OCR (T-13), Video-Transkription (T-14) sind wertvolle Features, aber nicht kritisch für MVP.

---

## Kontrollbericht

### Zusammenfassung

| Kategorie | Anzahl | Kommentar |
|---|---|---|
| Gesamtaufgaben | 27 | T-01 bis T-27 |
| P0-Aufgaben | 8 | Kritischer Pfad |
| P1-Aufgaben | 9 | Wichtig |
| P2-Aufgaben | 10 | Erweiterungen |
| Meilensteine | 7 | M-01 bis M-07 |
| Widersprüche dokumentiert | 4 | W-01 bis W-04 |
| Offene Zeitangaben | 27 | Alle als VORSCHLAG markiert |

### Checkliste

| ID | Aufgabe | Status |
|---|---|---|
| T-01 | Docker-Compose | Offen |
| T-02 | ES-Schema | Offen |
| T-03 | PostgreSQL Schema | Offen |
| T-04 | Scrapy Crawler | Offen |
| T-05 | Playwright Stealth | Offen |
| T-06 | Tor Proxy | Offen |
| T-07 | PDF/DOCX Extraktion | Offen |
| T-08 | Kafka Consumer | Offen |
| T-09 | spaCy NER | Offen |
| T-10 | VADER Sentiment | Offen |
| T-11 | Translation Pipeline | Offen |
| T-12 | Embeddings | Offen |
| T-13 | OCR Tesseract | Offen |
| T-14 | Video Transkription | Offen |
| T-15 | MinHash + TF-IDF | Offen |
| T-16 | FastAPI Backend | Offen |
| T-17 | 1000 Quellen | Offen |
| T-18 | Hub Frontend | Offen |
| T-19 | Targeting Frontend | Offen |
| T-20 | 3D Search Frontend | Offen |
| T-21 | Health Dashboard | Offen |
| T-22 | Cloud-Deployment | Offen |
| T-23 | Prometheus/Grafana | Offen |
| T-24 | CI/CD Pipeline | Offen |
| T-25 | Security Checks | Offen |
| T-26 | Beta-Test | Offen |
| T-27 | Dokumentation | Offen |

### Changelog

| Datum | Änderung | Autor | Genehmigt von |
|---|---|---|---|
| 2026-03-03 | Initialer Projektplan erstellt aus Repository-Dokumenten | GitHub Copilot AI Agent | TBD |

### Prüfprotokoll

| Prüfer | Rolle | Datum | Ergebnis |
|---|---|---|---|
| TBD | Projektleiter | TBD | Ausstehend |
| TBD | Tech Lead | TBD | Ausstehend |

### Signaturen

| Name | Rolle | Datum |
|---|---|---|
| TBD | Projektleiter | TBD |
| TBD | Tech Lead | TBD |
| TBD | Security Officer | TBD |

---

*Dieses Dokument wurde automatisch generiert aus den Repository-Quelldokumenten am 2026-03-03T05:38:16Z.*  
*Alle Zeitangaben ohne Quellenangabe sind als **VORSCHLAG** zu verstehen.*
