# Gitea Agent Connector — Produktanforderungen

**Status:** Verbindliche fachliche Zielbeschreibung für v1, noch keine Implementierung  
**Version:** 1.0  
**Repository:** `gitea-agent-connector`  
**Zielgruppe:** Implementierungs-KI, Maintainer und spätere Betreiber

> **Dokumentenrollen:** Die bereits vorhandene `AGENTS.md` enthält die Arbeitsregeln für die **Coding-KI, die dieses Repository entwickelt**. Diese Datei beschreibt dagegen das **gewünschte Verhalten der später laufenden Software**. `GITEA_AGENT_IMPLEMENTIERUNGSPLAN.md` legt die konkrete technische Umsetzung und Reihenfolge fest. `AGENTS.md` darf nicht durch diese Dateien ersetzt werden. Wenn eine Anweisung zur Arbeit der Coding-KI widersprüchlich erscheint, gilt die bestehende `AGENTS.md`; offene Widersprüche sind zu melden statt eigenmächtig umzudeuten.

## 1. Ziel

Ein Docker-Container soll **genau einen** eigenständig handelnden Gitea-Agenten betreiben, der über einen **beliebigen OpenAI-kompatiblen LLM-Endpunkt** Entscheidungen trifft. Der Agent soll auf exakte Erwähnungen seines Gitea-Namens reagieren, den jeweiligen Kontext rekonstruieren, vorhandene Arbeit prüfen und anschließend abhängig von seinen Berechtigungen handeln.

```text
OpenAI-kompatibler Endpoint
           ⇅
  Gitea Agent Connector
           ⇅
          Gitea
```

**Prinzip:** Das LLM wählt erlaubte Aktionen aus. Der Connector erzwingt unabhängig vom LLM Berechtigungen, Grenzen, Sicherheitsregeln, Queue-Reihenfolge und Fehlerbehandlung.

### 1.1 Beispielszenario

1. Unter Issue #42 erscheint der Kommentar `@coder bitte prüfe diesen Fehler`.
2. Der Poller erkennt die Erwähnung, sofern ihr Autor nicht `coder` ist.
3. Eine dauerhafte Queue speichert diese Aufgabe genau einmal.
4. Der einzige Worker bearbeitet die Aufgabe, sobald ältere Aufgaben beendet sind.
5. Er liest die aktuelle Issue samt Kommentaren und relevanten PRs.
6. Vor neuen Änderungen sucht er nach bereits vorhandenen Issues, PRs oder Fixes.
7. Ist das Problem gelöst, dokumentiert er die Lösung und erstellt **keinen** überflüssigen PR.
8. Andernfalls kann das LLM zulässige Tools anfordern: Repository klonen, Code lesen/ändern, Tests ausführen, Branch pushen, PR erstellen.
9. Vor **jeder** Aktion überprüft die Policy Engine die Berechtigung und Sicherheitsbedingungen.
10. Der Agent kommentiert Fortschritt und Ergebnis in Gitea. Der Arbeitsordner wird standardmäßig gelöscht.

## 2. Verbindliche Grundentscheidungen

| ID | Entscheidung | Anforderung |
|---|---|---|
| G-01 | Identität | Jeder Agent verwendet einen **eigenen Gitea-Benutzer und ein eigenes Access Token**. |
| G-02 | Laufzeit | **Ein Agent pro Container**, unabhängig von anderen Agent-Containern. |
| G-03 | Auslöser | In v1 nur **exakte `@agentname`-Erwähnungen**; keine automatische Ausführung durch Labels oder Assignments. |
| G-04 | Ereignisabfrage | **Polling**, Intervall und Scan-Parameter konfigurierbar; zunächst keine Webhooks. |
| G-05 | Aufgaben | **Persistente FIFO-Queue**, genau **eine aktive Aufgabe** pro Container; neue Erwähnungen unterbrechen die aktuelle Aufgabe nicht. |
| G-06 | KI | Frei wählbare OpenAI-kompatible `base_url`, `api_key`, `model` und unterstützte Modellparameter. |
| G-07 | Werkzeuge | Git-, Datei-, Test- und optionale Shell-Aktionen werden **im eigenen Container** ausgeführt. |
| G-08 | Duplikate | Vor dem Erstellen neuer Lösungen vorhandene Issues, PRs und Fixes prüfen; Prüftiefe konfigurierbar. |
| G-09 | Autonomie | Fortschritt nach Erst-Mention auf Wunsch eigenständig fortsetzen; konfigurierbar. |
| G-10 | Rechte | Berechtigungs-**Presets + einzelne Overrides**. |
| G-11 | Merge | **Nur wenn `permissions.allow_merge: true`** und die Aktion zusätzlich ausdrücklich berechtigt ist. |
| G-12 | Kommandos | Konfigurierbare Allowlist und Denylist; keine unkontrollierte Shell als Standard. |
| G-13 | Workspace | Standardmäßig nach Taskende löschen, Verhalten konfigurierbar. |
| G-14 | Ressourcen | Iterations-, Tool-, LLM-, Laufzeit-, Retry- und soweit messbar Tokenlimits. |
| G-15 | Kommunikation | Start-, Ergebnis- und Fehlerkommentare in Gitea, soweit `comment` berechtigt. |
| G-16 | Fehler | Nach begrenzten Wiederholungen abbrechen, verständlich kommentieren, keine Endlosschleife. |
| G-17 | Einstellungen | `config.yaml` mit Environment-Overrides und Secrets aus Umgebungsvariablen. |
| G-18 | Gedächtnis | **Kein dauerhaftes LLM-Gedächtnis**: Kontext vor jeder Aufgabe neu aus Gitea holen. Nur technische Queue-, Cursor- und Idempotenzdaten dauerhaft speichern. |

## 3. Gitea-Identität und Zugang

- Token und Gitea-URL dürfen niemals in Git-Dateien, Logausgaben, LLM-Prompts oder Kommentaren erscheinen.
- Beim Start die Gitea-Identität abrufen und mit `agent.name` abgleichen. Bei Abweichung Start abbrechen.
- Der Token benötigt ausschließlich die tatsächlich erforderlichen Gitea-Rechte; unzureichende Rechte führen zu einer nachvollziehbaren Fehlermeldung.
- Verarbeitete Repositories müssen in einer Allowlist stehen. Niemals blind über alle für das Token erreichbaren Repositories arbeiten.
- Git-Authentisierung muss ohne Token in Clone-URL, Prozessargumenten, Git-Konfiguration oder dauerhaftem Git-Credential-Store erfolgen. Sichere kurzlebige Authentisierung ist im Implementierungsplan verbindlich vorzusehen.

## 4. Exakte Erwähnungen

### 4.1 Quellen

In v1 gelten als mögliche Quellen: Issue-Titel, Issue-Beschreibungen, gewöhnliche Issue-/PR-Kommentare sowie nach Erweiterung PR-Reviews und Inline-Review-Kommentare.

| Beispiel | Bei Agent `coder` auslösen? |
|---|---|
| `@coder prüfe das` | Ja |
| `(@coder)` | Ja |
| `@coder2` | Nein |
| `@coder-test` | Nein |
| `mail@coder.example` | Nein |
| `` `@coder` `` | Nein |
| Fenced Codeblock mit `@coder` | Nein |
| Markdown-Blockzitat mit `@coder` | Nein |
| Kommentar von Gitea-Benutzer `coder` | Nein |

- Mehrere identische Erwähnungen in **einem** Quellobjekt ergeben eine Aufgabe.
- Unterschiedliche Quellobjekte sind unabhängig und ergeben separate Aufgaben.
- Das wiederholte Einlesen desselben Inhalts darf keine zusätzlichen Aufgaben erzeugen.
- Falls eine Mention entfernt und später neu hinzugefügt wird, darf eine neue Aufgabe entstehen.
- Bei Bearbeitung eines Kommentars, der weiterhin dieselbe Mention enthält, darf keine zweite Aufgabe entstehen.
- Mentions sind **keine direkten Shell-Befehle**. Sie liefern Daten und Anliegen, deren Ausführung durch LLM und Policy vermittelt wird.

### 4.2 Grenzen von Polling

Polling kann Mentions, die zwischen zwei Scans hinzugefügt und wieder entfernt werden, nicht garantiert erkennen. Der Connector muss überlappende Scans, Pagination und periodische Rückschau einsetzen, darf aber keine lückenlose Ereigniserfassung versprechen. Bei einem fehlgeschlagenen Scan wird der Cursor nicht fortgeschrieben.

## 5. Persistente FIFO-Queue

Der Agent verarbeitet Aufgaben nach ihrem gespeicherten Eintrittszeitpunkt; bei Gleichstand nach Datenbank-ID.

**Zustände:** `queued` → `running` → `completed` / `skipped` / `failed`.

- Der Poller kann beim Laufen des Workers weitere Aufgaben einreihen.
- Es läuft niemals mehr als ein Task zugleich innerhalb desselben Containers.
- Ein Neustart darf keine unerledigten Aufgaben verlieren.
- Doppelte Events dürfen nicht zu doppelten Jobs werden.
- Bei Wiederaufnahme müssen bereits erfolgte Remote-Schreibaktionen erkannt werden, statt sie blind zu wiederholen.
- Die Reihenfolge und Recovery müssen mit SQLite-Transaktionen abgesichert werden.
- In v1 gibt es **keine Priorisierung und kein Unterbrechen** laufender Tasks.

## 6. Lebenszyklus einer Aufgabe

Für jeden Task gilt die feste Sequenz:

```text
Neue Mention
  → Idempotenz prüfen / enqueue
  → worker.claim_next
  → Repository freigeben oder verweigern
  → Gitea-Kontext neu abrufen
  → ggf. Startkommentar
  → bestehende Issues/PRs/Fixes prüfen
  → KI entscheidet in begrenztem Agent-Loop
  → jedes Tool: Schema → Policy → Ausführung → Ergebnis
  → Ergebnis oder begründeten Abbruch kommentieren
  → Task abschließen / Workspace aufräumen
```

**Mögliche Task-Ergebnisse:**

- `completed`: Anliegen erfolgreich bearbeitet; ggf. PR-Link liefern.
- `skipped`: Problem schon gelöst oder nach Prüfung keine sinnvolle Aktion notwendig.
- `failed`: technische Fehler, fehlende Rechte, Limits oder unauflösbare Unsicherheit; keine Erfolge erfinden.

Ein einzelnes Tool-Ergebnis darf die KI nicht zu unbefugter Weiterarbeit verleiten. Bei Policy-Ablehnung darf die KI nur erlaubte Alternativen versuchen.

## 7. Kontext und Suche nach bestehender Arbeit

Der Agent muss vor jeder neuen Umsetzung Informationen über Repository, Issue/PR, Kommentare, Status, relevante Branches und vorhandene Lösungen sammeln.

**Konfigurierbare Modi:**

| Modus | Verhalten |
|---|---|
| `off` | Keine automatische Suche. Nur mit expliziter Betreiberkonfiguration. |
| `basic` (**Standard**) | Offene und geschlossene Issues, offene, geschlossene und gemergte PRs im aktuellen Repository suchen. |
| `thorough` | Wie `basic`, zusätzlich relevante Branches, Commit-Nachrichten und Code prüfen, soweit erreichbar. |

- Ein Suchtreffer ist zunächst nur ein Kandidat, nicht automatisch ein Duplikat.
- Ist der Fix nachweislich vorhanden, **keine neue Implementierung und keinen weiteren PR erstellen**.
- Ist bereits eine aktive Umsetzung vorhanden, deren Kontext lesen und nicht ohne Grund konkurrierende Änderungen erzeugen.
- Bei unklarer Lage zuerst weiter untersuchen und im Zweifel kommentieren statt vorschnell doppelte Arbeit anzulegen.
- Bei Suchfehler und `duplicate_check.require_success: true` darf keine neue inhaltliche Schreibaktion begonnen werden.
- Das System muss die **Suche vor der ersten inhaltlichen Schreibaktion** technisch erzwingen, nicht nur per Prompt empfehlen.

## 8. Berechtigungen und Autonomie

### 8.1 Presets

| Recht | `observer` | `reviewer` | `developer` | `maintainer` |
|---|:---:|:---:|:---:|:---:|
| Gitea lesen | Ja | Ja | Ja | Ja |
| Kommentare schreiben / Reviews | Nein | Ja | Ja | Ja |
| Repo klonen und Dateien lesen | Nein | Ja | Ja | Ja |
| Dateien ändern / vordefinierte Tests | Nein | Nein | Ja | Ja |
| Branch, Commit, Push, PR erstellen | Nein | Nein | Ja | Ja |
| Issues erstellen / schließen | Nein | Nein | Nein | Ja |
| PR mergen | Nein | Nein | Nein | Nur mit Zusatzfreigabe |
| Branch löschen / freie Kommandos | Nein | Nein | Nein | Nein |

### 8.2 Overrides und harte Regeln

- `permissions.preset` setzt die Grundrechte.
- `permissions.overrides` kann jede einzelne Aktion an- oder abschalten.
- `permissions.allow_merge: false` **überstimmt stets** Preset und Overrides.
- Auch bei `allow_merge: true` sind `merge_pr`-Berechtigung, Gitea-Serverrechte und Branch-Protection erforderlich.
- Branch-Pushes sind nur auf vom Agent angelegte Arbeitsbranches zulässig, nicht direkt auf den Default-/Protected-Branch.
- Allow-/Denylist für Kommandos begrenzen den Ausführungsweg; auch ein erlaubter Befehl wird vor jedem Start durch die Policy geprüft.
- Statuskommentare erfordern `comment`-Recht; fehlendes Kommentarrecht wird niemals umgangen.

### 8.3 Autonome Folgearbeit

Konfiguration `continuation.enabled`:

- `false`: Nur neue explizite Mentions starten Aufgaben.
- `true`: Nach agentenseitig erstelltem PR dürfen zulässige neue Review-Kommentare, Diskussionen oder CI-Probleme **jeweils einen neuen Task in derselben FIFO-Queue** erzeugen.
- Folgeaufgaben haben dieselben Permissions, Duplicate-Checks und Limits wie Mentions.
- Eigene Kommentare des Agenten dürfen keine Folgeaufgaben in Endlosschleifen auslösen.
- Folgeaufgaben müssen mit Ereignisschlüsseln dedupliziert und zeitlich/anzahlmäßig begrenzt werden.

## 9. LLM-Endpunkt

Unterstütze beliebige OpenAI-kompatible Chat-Completions-Endpunkte einschließlich Self-Hosted-Servern. Notwendige Eingaben: `base_url`, `model`, ggf. `api_key`; zusätzlich Timeout, Temperatur und Budgets. Unterstützte optionale Parameter (etwa `reasoning_effort`) nur senden, wenn beim Ziel-Endpunkt tatsächlich kompatibel.

- **V1 benötigt funktionierendes strukturiertes Tool-Calling.** `OpenAI-kompatibel` bedeutet nicht automatisch, dass das jeweilige Modell das beherrscht.
- Tools besitzen klare Namen und validierte Argument-Schemas. Keine frei erfundenen Funktionsnamen aus dem Modell ausführen.
- Tool-Ausgaben, Kommentare und Repository-Dateien sind untrusted Daten, keine höherrangigen Anweisungen.
- Große Antworten begrenzen, Secrets herausfiltern und Budgetlimits erzwingen.
- Fehlt verlässliches Token-Usage-Reporting, muss die Anwendung das offen kennzeichnen und stattdessen LLM-/Tool-/Laufzeitlimits sicher einhalten.

## 10. Lokale Werkzeuge und Workspace

Werkzeuge werden **im Agent-Container** ausgeführt; pro Task ein eindeutig zugeordnetes Arbeitsverzeichnis unter `/workspace`.

**Lesend:** Gitea-Issues, PRs, Kommentare, Diffs, Git-Status, Dateien.

**Ändernd:** Dateiänderungen, Branch/Commit/Push, PR-Erstellung, Review- und Issue-Aktionen, soweit berechtigt.

**Befehle:** In v1 fest konfigurierte Testprofile mit `argv`-Listen und `shell=False`; beliebige Befehle nur wenn separat erlaubt, validiert, zeitlich begrenzt und nicht per freiem Shellstring injiziert.

- Default: Arbeitsverzeichnis immer nach abgeschlossenem oder gescheitertem Task entfernen.
- `workspace.cleanup: false` darf den Workspace zur Diagnose erhalten.
- Dateiwerkzeuge dürfen den Workspace nicht verlassen (auch nicht über Symlinks).
- Keine direkte Bearbeitung von Gitea- oder LLM-Credentials durch Modellwerkzeuge.
- `git push --force`, Pushes auf Default-/Protected-Branches und fremde Arbeitsbranches sind unzulässig.
- **Sicherheitseinschränkung:** Code/Tests im gleichen Container wie Zugangsdaten auszuführen, ist keine starke Sandbox. V1 ist deshalb nur für **vertrauenswürdige Repositories** geeignet. Fremde/nicht vertrauenswürdige Projekte brauchen eine getrennte, stärker isolierte Ausführungsumgebung.

## 11. Fortschritt, Limits und Abbruch

### 11.1 Gitea-Kommentare

Sofern `comment` erlaubt und konfiguriert ist:

- Nach Taskstart eine knappe Information, dass der Agent den Kontext und bestehende Lösungen prüft.
- Nach Erfolg eine verständliche Zusammenfassung und Links zu Issue/PR/Commit.
- Bei bereits gelöstem Problem die vorhandene Lösung verlinken.
- Bei Abbruch Ursache nennen, ohne Tokens, Stacktraces, Systemprompts oder vertraulichen Code offenzulegen.

### 11.2 Limits

Pro Task konfigurierbar: `max_iterations`, `max_llm_calls`, `max_tool_calls`, `max_runtime_seconds`, `max_error_retries`, optional Tokenlimits, Anzahl von Folgeaufgaben. **Bei Überschreitung nicht unendlich weiterlaufen.**

### 11.3 Fehler

- Temporäre Gitea-/LLM-Fehler: begrenzte Wiederholungen mit Backoff.
- Dauerhafte Auth-/Konfigurationsfehler: klar abbrechen.
- Tests schlagen wiederholt fehl: Arbeit nach Limit beenden und Ursache kommentieren.
- Externe Schreibaktion unklar nach Crash/Timeout: erst Remote-Zustand prüfen; keine blinden Wiederholungen.
- Konnte ein Ergebnis-Kommentar nicht zugestellt werden: in einer dauerhaften Outbox speichern und später versuchen.

## 12. Konfiguration

Zentrale `config.yaml`, überschreibbare Werte über erlaubte Environment-Variablen. Geheimnisse gehören in die Umgebung, nicht ins Repository. Alle Einstellungen beim Start validieren.

Minimaler Ausschnitt:

```yaml
agent:
  name: coder

gitea:
  base_url: ${GITEA_BASE_URL}
  token: ${GITEA_TOKEN}
  repositories: ["team/project"]

llm:
  base_url: ${LLM_BASE_URL}
  api_key: ${LLM_API_KEY}
  model: ${LLM_MODEL}

polling:
  interval_seconds: 30

permissions:
  preset: developer
  allow_merge: false
  overrides:
    create_issue: false
    close_issue: false
    merge_pr: false

continuation:
  enabled: true

duplicate_check:
  mode: basic
  require_success: true

workspace:
  cleanup: true

limits:
  max_iterations: 40
  max_runtime_seconds: 3600
```

Der Implementierungsplan enthält das vollständige Konfigurationsschema und die konkreten Modulverträge.

## 13. Nicht Teil von v1

Keine Webhooks, keine eigene Weboberfläche, keine Agenten-Orchestrierung innerhalb eines Containers, keine parallel ausgeführten Tasks, kein Langzeit-LLM-Gedächtnis, keine Kubernetes-Pflicht, kein Redis-/Messagebroker-Zwang, keine garantierte sichere Ausführung beliebiger fremder Repositories, keine umfassende Kostenerfassung ohne nachvollziehbare Provider-Usage.

## 14. Abnahmekriterien

**A-01** Eine neue gültige Mention erzeugt genau einen Task.  
**A-02** Falsche Erwähnungen (`@coder2`, Code, Bot-Text) erzeugen keinen Task.  
**A-03** Mehrere Mentions in verschiedenen Kommentaren werden in FIFO-Reihenfolge verarbeitet.  
**A-04** Nach Neustart geht keine noch ausstehende Aufgabe verloren.  
**A-05** Nach wiederholtem Poll werden keine Duplikataufgaben angelegt.  
**A-06** Vor Codeänderungen erfolgt die konfiguriert erforderliche Suche nach bestehender Arbeit.  
**A-07** Bereits gelöste Probleme erzeugen keinen konkurrierenden PR.  
**A-08** `allow_merge: false` verhindert Merge auch bei Preset `maintainer`.  
**A-09** Unbekannte oder verbotene Tools werden technisch abgelehnt.  
**A-10** Erlaubte Coding-Aufgaben können nach Git-Änderung in einem PR enden.  
**A-11** Ein vorhandener PR wird nach Crash erkannt, statt einen zweiten zu erstellen.  
**A-12** Neue Reviews oder CI-Probleme können bei aktivierter Fortsetzung Folgeaufgaben erzeugen.  
**A-13** Fehler und Limits führen zu beendetem Task und sachlicher Statusmeldung, sofern Kommentarrecht und Gitea-Verfügbarkeit bestehen.  
**A-14** Der Workspace wird standardmäßig gelöscht.  
**A-15** API-Token und Schlüssel erscheinen nicht in Modell-Nachrichten, Logs, Kommentaren oder Git-Historie.  
**A-16** Mehrere Container können unabhängig mit unterschiedlichen Gitea-Benutzern und Modellen betrieben werden.

**Nachweis:** Unit- und Mock-Integrationstests plus ausgewählte Live-Tests gegen ein separates Gitea-Testrepository. Die **Ausführung** dieser Tests durch die Coding-KI unterliegt der vorhandenen `AGENTS.md` und bedarf bei deren aktueller Fassung einer ausdrücklichen Testfreigabe. Die Softwarefunktion `run_tests` des späteren Agenten ist dagegen eine eigene, über die Runtime-Konfiguration geregelte Berechtigung.

## 15. Noch unbekannte Betriebsdaten

Konkrete Gitea-Version: **Unbekannt**. Gitea-Host: **Unbekannt**. Modellanbieter und konkrete Tool-Calling-Fähigkeiten: **Unbekannt**. Repository-Lizenz: **Unbekannt**. Diese Angaben dürfen nicht erfunden werden; die Software muss soweit möglich über Konfiguration damit umgehen können. Vor echten Integrationstests Gitea-Version und tatsächlich unterstützte API-Endpunkte feststellen.

---

**Anweisung für Implementierer:** Implementiere nichts direkt aus dieser Produktbeschreibung ohne Abgleich mit dem detaillierten Implementierungsplan und der vorhandenen `AGENTS.md`. Bei fachlichen Unklarheiten zuerst diese Dateien vergleichen, bei tatsächlich kritischen Entscheidungen entsprechend `AGENTS.md` nachfragen.
