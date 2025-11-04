# Dönerguide Stuttgart - Projekt Backlog

## Projektmanagement-Methodik

Dieses Projekt nutzt einen **hybriden Ansatz** aus Kanban, Scrum und klassischem Projektmanagement:

### Prinzipien
- **Vorab-Planung:** Alle Anforderungen und Tasks werden initial definiert (wie Wasserfall/PRINCE2)
- **Flexibilität:** Tasks können bei Bedarf angepasst, priorisiert oder verschoben werden (agile Prinzipien)
- **Milestone-basiert:** Statt festen Sprints arbeiten wir mit Milestones ohne fixe Zeitbeschränkung
- **Kanban-Flow:** Jeder Task durchläuft das Board (To Do → In Progress → Review → Done)

### Task-Struktur
Jeder Task enthält:
- **Titel:** Kurze, prägnante Beschreibung
- **Kategorie:** Fachliche Zuordnung (Frontend, Backend, Infra, DevOps, etc.)
- **Priorisierung:** 
  - **Hoch:** Kritisch für MVP-Funktionalität, keine Überbrückung möglich
  - **Mittel:** Wichtig für MVP, aber temporär überbrückbar
  - **Niedrig:** Nice-to-have, kann verschoben werden
- **Schätzung:** Personenstunden (PH)
- **Abhängigkeiten:** Welche Tasks müssen vorher abgeschlossen sein
- **Beschreibung:** Detaillierte Aufgabenbeschreibung
- **Akzeptanzkriterien:** Wann ist der Task erfolgreich abgeschlossen
- **Test-Anforderungen:** Erforderliche Unit-Tests
- **Doku-Anforderungen:** Notwendige Dokumentation

### Definition of Done
Ein Task gilt als "Done", wenn:
1. ✅ Alle Akzeptanzkriterien erfüllt sind
2. ✅ Unit-Tests geschrieben und bestanden
3. ✅ Dokumentation erstellt (README, Code-Kommentare, etc.)
4. ⚠️ Code Review (optional, aber empfohlen)

---

## Milestone 0: Setup & Foundations

**Ziel:** Entwicklungsumgebung, Infrastruktur-Grundlagen und CI/CD-Pipeline etablieren

**Geschätzte Dauer:** 2 Wochen  
**Geschätzter Aufwand:** 71 Personenstunden  
**Team-Kapazität:** 35-70 PH/Woche bei 7 Personen

---

### INFRA-001: Repository-Struktur anlegen

**Kategorie:** Infrastructure  
**Priorisierung:** Hoch  
**Schätzung:** 2 PH  
**Abhängigkeiten:** Keine

**Beschreibung:**
Monorepo auf GitHub erstellen mit grundlegender Verzeichnisstruktur für Frontend, Backend (Functions) und Infrastructure (Terraform).

**Akzeptanzkriterien:**
- Repository `doenerguide-stuttgart` ist auf GitHub erstellt
- Branch-Strategie definiert (`main` für Production, Feature-Branches)
- Folgende Verzeichnisstruktur existiert:
  ```
  /
  ├── frontend/          # Next.js Anwendung
  ├── functions/         # Azure Functions
  ├── infrastructure/    # Terraform Configuration
  ├── docs/             # Projekt-Dokumentation
  ├── .github/          # GitHub Actions Workflows
  ├── .gitignore
  └── README.md
  ```
- `.gitignore` enthält Node, Terraform und Azure-spezifische Einträge

**Test-Anforderungen:**
- Keine Unit-Tests erforderlich
- Manuelle Überprüfung der Struktur

**Doku-Anforderungen:**
- Root-README.md mit Projektübersicht
- Beschreibung der Repository-Struktur
- Branch-Strategie dokumentiert

---

### INFRA-002: Azure Subscription & Resource Group erstellen

**Kategorie:** Infrastructure  
**Priorisierung:** Hoch  
**Schätzung:** 3 PH  
**Abhängigkeiten:** Keine

**Beschreibung:**
Azure Subscription für das Projekt einrichten und initiale Resource Group manuell erstellen (wird später von Terraform verwaltet).

**Akzeptanzkriterien:**
- Azure Subscription ist aktiv und zugänglich
- Service Principal für Terraform/GitHub Actions erstellt mit Contributor-Rechten
- Resource Group `rg-doenerguide-prod` manuell angelegt
- Alle Team-Mitglieder haben Zugriff auf die Subscription

**Test-Anforderungen:**
- Keine Unit-Tests erforderlich
- Manuelle Verifikation: `az login` und `az group show --name rg-doenerguide-prod`

**Doku-Anforderungen:**
- `docs/azure-setup.md` mit Subscription-Details (ohne Secrets!)
- Anleitung für Team-Mitglieder zum Subscription-Zugriff
- Service Principal Name und Application ID dokumentiert

---

### INFRA-003: Terraform Backend Storage einrichten

**Kategorie:** Infrastructure  
**Priorisierung:** Hoch  
**Schätzung:** 4 PH  
**Abhängigkeiten:** INFRA-002

**Beschreibung:**
Azure Storage Account für Terraform Remote State manuell erstellen und konfigurieren. State Locking aktivieren.

**Akzeptanzkriterien:**
- Storage Account `stdoenerguidestate` in `rg-doenerguide-prod` erstellt
- Container `tfstate` mit Private Access Level existiert
- State Locking über Blob Lease aktiviert
- Backend-Konfiguration in `infrastructure/backend.tf` definiert
- Terraform kann erfolgreich initialisiert werden (`terraform init`)

**Test-Anforderungen:**
- Test: `terraform init` läuft ohne Fehler
- Test: State-Datei wird nach `terraform apply` im Blob Storage sichtbar

**Doku-Anforderungen:**
- `infrastructure/README.md` mit Backend-Setup-Anleitung
- Erklärung des State Management
- Troubleshooting-Guide bei Backend-Problemen

---

### INFRA-004: GitHub Secrets konfigurieren

**Kategorie:** DevOps  
**Priorisierung:** Hoch  
**Schätzung:** 2 PH  
**Abhängigkeiten:** INFRA-002, INFRA-003

**Beschreibung:**
Erforderliche Secrets in GitHub Repository Settings hinterlegen für CI/CD-Pipeline.

**Akzeptanzkriterien:**
- Folgende Secrets sind in GitHub hinterlegt:
  - `AZURE_CREDENTIALS` (Service Principal JSON)
  - `AZURE_SUBSCRIPTION_ID`
  - `AZURE_TENANT_ID`
  - `TF_BACKEND_STORAGE_ACCOUNT`
  - `TF_BACKEND_CONTAINER_NAME`
  - `TF_BACKEND_KEY` (z.B. `prod.terraform.tfstate`)
- Secrets sind für alle Workflows zugänglich
- Test-Workflow kann Secrets abrufen

**Test-Anforderungen:**
- Test-Workflow erstellen, der Secrets ausgibt (maskiert)
- Verifizieren, dass keine Secrets im Log sichtbar sind

**Doku-Anforderungen:**
- `docs/github-secrets.md` mit Liste aller benötigten Secrets (ohne Werte!)
- Anleitung zum Rotieren von Secrets
- Zuständigkeiten für Secret-Management

---

### INFRA-005: Terraform Base Configuration erstellen

**Kategorie:** Infrastructure  
**Priorisierung:** Hoch  
**Schätzung:** 6 PH  
**Abhängigkeiten:** INFRA-003

**Beschreibung:**
Terraform-Konfiguration für grundlegende Azure-Ressourcen erstellen (Provider, Resource Group, Naming Convention).

**Akzeptanzkriterien:**
- `infrastructure/main.tf` mit Azure Provider konfiguriert (v3.x)
- `infrastructure/variables.tf` mit zentralen Variablen (location, environment, project_name)
- `infrastructure/outputs.tf` für wichtige Resource-IDs
- `infrastructure/terraform.tfvars.example` als Template
- Resource Group wird von Terraform verwaltet
- `terraform plan` läuft ohne Fehler
- Naming Convention: `<resource-type>-doenerguide-<environment>` (z.B. `func-doenerguide-prod`)

**Test-Anforderungen:**
- `terraform validate` erfolgreich
- `terraform plan` zeigt erwartete Ressourcen
- Manuelle Prüfung: `terraform apply` erstellt Resource Group

**Doku-Anforderungen:**
- `infrastructure/README.md` mit Terraform-Nutzung
- Variablen-Beschreibungen in `variables.tf`
- Kommentare in `main.tf` für Module-Struktur

---

### DEVOPS-001: GitHub Actions Workflow - Checks

**Kategorie:** DevOps  
**Priorisierung:** Hoch  
**Schätzung:** 5 PH  
**Abhängigkeiten:** INFRA-001, INFRA-004

**Beschreibung:**
Workflow für automatische Checks bei jedem Push (Compile, Lint, Terraform Validate).

**Akzeptanzkriterien:**
- Workflow `.github/workflows/checks.yml` existiert
- Triggert bei Push auf alle Branches
- Jobs:
  1. **Frontend:** `npm install`, `npm run lint`, `npm run build`
  2. **Functions:** `npm install`, `npm run lint`, `npm run build`
  3. **Terraform:** `terraform fmt -check`, `terraform validate`
- Alle Jobs laufen parallel
- Badge im README zeigt Workflow-Status

**Test-Anforderungen:**
- Workflow läuft erfolgreich bei gültigem Code
- Workflow schlägt fehl bei Lint-Errors
- Workflow schlägt fehl bei ungültigem Terraform

**Doku-Anforderungen:**
- `docs/ci-cd.md` mit Workflow-Beschreibungen
- Kommentare im Workflow-File
- Troubleshooting-Guide für häufige Fehler

---

### DEVOPS-002: GitHub Actions Workflow - Deploy

**Kategorie:** DevOps  
**Priorisierung:** Hoch  
**Schätzung:** 12 PH  
**Abhängigkeiten:** DEVOPS-001, INFRA-005, INFRA-106

**Beschreibung:**
Workflow für automatisches Deployment von Infrastruktur und allen 4 Function Apps bei Push auf `main` Branch.

**Akzeptanzkriterien:**
- Workflow `.github/workflows/deploy.yml` existiert
- Triggert nur bei Push auf `main`
- Deployment-Schritte:
  1. **Terraform Apply:** Infrastruktur aktualisieren
  2. **Build Shared Package:** `cd functions/shared && npm ci && npm run build`
  3. **Build & Deploy Function Apps** (parallel möglich):
     - `place-search`: Build → ZIP → Deploy zu `func-doenerguide-placesearch-prod`
     - `image-classifier`: Build → ZIP → Deploy zu `func-doenerguide-imageclassifier-prod`
     - `llm-analyzer`: Build → ZIP → Deploy zu `func-doenerguide-llmanalyzer-prod`
     - `data-persistence`: Build → ZIP → Deploy zu `func-doenerguide-datapersistence-prod`
  4. **Frontend Deploy:** Next.js zu Azure Static Web App deployen (später in Milestone 3)
- Build-Prozess pro Function:
  - `npm ci` (installiert dependencies inkl. shared package)
  - `npm run build` (TypeScript → JavaScript)
  - ZIP: `dist/`, `node_modules/`, `host.json`, `package.json`
- Deployment via Azure CLI: `az functionapp deployment source config-zip`
- Secrets werden sicher verwendet (keine Logs)
- Deployment schlägt bei Terraform- oder Build-Fehlern fehl
- Rollback-Strategie dokumentiert (manuell via Azure Portal)

**Test-Anforderungen:**
- Dry-Run Test mit `terraform plan` in Workflow
- Build-Test für alle 4 Functions
- Test: ZIP-Artefakte enthalten alle notwendigen Files
- Deployment auf Test-Resource-Group vor Production (optional)

**Doku-Anforderungen:**
- `docs/ci-cd.md` mit Deployment-Prozess für separate Apps
- Build-Script-Dokumentation
- Paralleles vs. Sequenzielles Deployment
- Rollback-Anleitung pro Function App
- Troubleshooting für häufige Deployment-Fehler

---

### DEV-001: Lokale Entwicklungsumgebung - Azurite Setup

**Kategorie:** DevOps  
**Priorisierung:** Mittel  
**Schätzung:** 3 PH  
**Abhängigkeiten:** INFRA-001

**Beschreibung:**
Azurite (Azure Storage Emulator) für lokale Entwicklung einrichten und dokumentieren.

**Akzeptanzkriterien:**
- `docker-compose.yml` im Root mit Azurite-Service
- Azurite exponiert Blob Storage (Port 10000), Queue (10001), Table (10002)
- Connection String in `.env.example` für lokale Entwicklung
- Script `scripts/start-local-env.sh` startet Azurite
- Entwickler können lokal gegen Azurite entwickeln

**Test-Anforderungen:**
- Test-Script: Blob hochladen und abrufen von Azurite
- Verifizieren mit Azure Storage Explorer

**Doku-Anforderungen:**
- `docs/local-development.md` mit Azurite-Setup
- Anleitung zur Installation von Docker
- Troubleshooting für häufige Azurite-Probleme

---

### DEV-002: Lokale Entwicklungsumgebung - Cosmos DB Emulator

**Kategorie:** DevOps  
**Priorisierung:** Mittel  
**Schätzung:** 4 PH  
**Abhängigkeiten:** INFRA-001

**Beschreibung:**
Cosmos DB Emulator für lokale Entwicklung einrichten (Docker-basiert).

**Akzeptanzkriterien:**
- Cosmos DB Emulator in `docker-compose.yml` integriert
- Emulator läuft auf localhost:8081
- Connection String in `.env.example`
- Datenbank und Container werden beim Start automatisch erstellt (Init-Script)
- Entwickler können lokal CRUD-Operationen ausführen

**Test-Anforderungen:**
- Test-Script: Dokument in Cosmos DB schreiben und lesen
- Verifizieren mit Data Explorer (https://localhost:8081/_explorer/index.html)

**Doku-Anforderungen:**
- `docs/local-development.md` mit Cosmos DB Emulator-Setup
- Bekannte Limitationen des Emulators dokumentiert
- SSL-Zertifikat-Handling für localhost erklärt

---

### DEV-003: Lokale Entwicklungsumgebung - Service Bus Emulator

**Kategorie:** DevOps  
**Priorisierung:** Mittel  
**Schätzung:** 4 PH  
**Abhängigkeiten:** INFRA-001

**Beschreibung:**
Offiziellen Azure Service Bus Emulator (Docker-basiert) für lokale Entwicklung einrichten und konfigurieren.

**Akzeptanzkriterien:**
- Service Bus Emulator in `docker-compose.yml` integriert
- Emulator läuft auf localhost:5672 (AMQP)
- Connection String in `.env.example` für lokale Entwicklung
- Benötigte Queues werden beim Start automatisch erstellt:
  - `places-found`
  - `images-classified`
  - `reviews-generated`
- Functions können lokal Nachrichten senden/empfangen
- Umschalten zwischen Emulator und Azure Service Bus via Environment Variable

**Test-Anforderungen:**
- Test-Script: Nachricht in Queue senden und empfangen
- Test: Queue-Erstellung beim Start verifizieren
- Test: Connection String Switching (lokal vs. Azure)

**Doku-Anforderungen:**
- `docs/local-development.md` mit Service Bus Emulator Setup
- Bekannte Limitationen des Emulators dokumentiert
- Unterschiede zu echtem Azure Service Bus erklärt
- Troubleshooting für häufige Probleme

---

### DEV-004: Projektstruktur - Frontend Bootstrap

**Kategorie:** Frontend  
**Priorisierung:** Mittel  
**Schätzung:** 4 PH  
**Abhängigkeiten:** INFRA-001

**Beschreibung:**
Next.js-Projekt im `/frontend` Verzeichnis initialisieren mit TypeScript, Tailwind, DaisyUI.

**Akzeptanzkriterien:**
- Next.js 14+ mit App Router initialisiert
- TypeScript konfiguriert (strict mode)
- Tailwind CSS + DaisyUI installiert und konfiguriert
- ESLint + Prettier Setup
- Starter-Layout mit Header/Footer
- `npm run dev` startet Entwicklungsserver
- Build läuft ohne Fehler

**Test-Anforderungen:**
- Build-Test: `npm run build` erfolgreich
- Lint-Test: `npm run lint` ohne Errors

**Doku-Anforderungen:**
- `frontend/README.md` mit Setup-Anleitung
- Verzeichnisstruktur erklärt (app/, components/, lib/)
- Styling-Konventionen (Tailwind + DaisyUI)

---

### DEV-005: Projektstruktur - Functions mit Shared Package Bootstrap

**Kategorie:** Backend  
**Priorisierung:** Hoch  
**Schätzung:** 8 PH  
**Abhängigkeiten:** INFRA-001, DEV-001

**Beschreibung:**
Azure Functions Struktur mit separaten Function Apps und Shared Package für gemeinsamen Code initialisieren.

**Akzeptanzkriterien:**
- Verzeichnisstruktur:
  ```
  /functions
  ├── shared/                          # Shared Code Package
  │   ├── src/
  │   │   ├── services/               # Shared Services
  │   │   ├── models/                 # TypeScript Interfaces
  │   │   └── utils/                  # Helper Functions
  │   ├── package.json                # "@doenerguide/shared"
  │   ├── tsconfig.json
  │   └── README.md
  ├── place-search/                    # Function App 1
  │   ├── src/
  │   │   └── functions/
  │   │       └── placeSearch/
  │   │           └── index.ts
  │   ├── host.json
  │   ├── package.json                # depends on "../shared"
  │   ├── tsconfig.json
  │   └── local.settings.json.example
  ├── image-classifier/                # Function App 2
  │   └── ...
  ├── llm-analyzer/                    # Function App 3
  │   └── ...
  ├── data-persistence/                # Function App 4
  │   └── ...
  └── scripts/
      ├── build-all.sh                # Build alle Functions
      └── deploy-all.sh               # Deploy alle Functions
  ```
- Shared Package als lokales npm Package (npm workspaces oder file: dependency)
- Alle 4 Function Apps können lokal starten
- Hello-World Function in place-search als Test
- Root package.json mit workspace-Konfiguration (optional)

**Test-Anforderungen:**
- Unit-Test im Shared Package
- Unit-Test für Hello-World Function (Jest)
- Test: `npm test` in jedem Function-Verzeichnis

**Doku-Anforderungen:**
- `functions/README.md` mit Architektur-Erklärung
- Shared Package Konzept dokumentiert
- Debugging-Guide für jede Function App
- Anleitung: Neue Function App hinzufügen

---

### DEV-006: Build & Deployment Scripts für separate Function Apps

**Kategorie:** DevOps  
**Priorisierung:** Hoch  
**Schätzung:** 6 PH  
**Abhängigkeiten:** DEV-005

**Beschreibung:**
Scripts zum Bauen und Deployen der separaten Function Apps lokal und in CI/CD.

**Akzeptanzkriterien:**
- Scripts in `functions/scripts/`:
  - `build-shared.sh`: Baut Shared Package
  - `build-function.sh <function-name>`: Baut einzelne Function App
  - `build-all.sh`: Baut alle Function Apps sequenziell
  - `package-function.sh <function-name>`: Erstellt Deployment-ZIP
  - `deploy-function.sh <function-name>`: Deployed ZIP zu Azure
  - `deploy-all.sh`: Deployed alle Functions
- Build-Prozess pro Function:
  ```bash
  cd functions/<function-name>
  npm ci                    # Install dependencies
  npm run build            # TypeScript → dist/
  ```
- ZIP-Packaging:
  ```bash
  zip -r ../deployments/<function-name>.zip \
    dist/ \
    node_modules/ \
    host.json \
    package.json
  ```
- Deployment via Azure CLI:
  ```bash
  az functionapp deployment source config-zip \
    --resource-group rg-doenerguide-prod \
    --name func-doenerguide-<function>-prod \
    --src deployments/<function-name>.zip
  ```
- Scripts haben Error Handling und Logging
- Alle Scripts sind ausführbar (`chmod +x`)

**Test-Anforderungen:**
- Test: build-all.sh baut alle 4 Functions ohne Fehler
- Test: package-function.sh erstellt gültige ZIP-Datei
- Test: ZIP enthält alle notwendigen Files (dist/, node_modules/, etc.)
- Manuelle Verifikation: deploy-function.sh funktioniert

**Doku-Anforderungen:**
- `functions/scripts/README.md` mit Script-Dokumentation
- Verwendung jedes Scripts erklärt
- CI/CD Integration beschrieben
- Troubleshooting für Build-Fehler

---

### EXT-001: Google Places API Key beantragen

**Kategorie:** External Services  
**Priorisierung:** Hoch  
**Schätzung:** 2 PH  
**Abhängigkeiten:** INFRA-002

**Beschreibung:**
Google Cloud Project anlegen, Places API aktivieren, API Key erstellen und absichern.

**Akzeptanzkriterien:**
- Google Cloud Project `doenerguide-stuttgart` erstellt
- Places API (New) aktiviert
- API Key erstellt mit:
  - Application Restriction: HTTP Referrers (Azure Function Domain)
  - API Restriction: Nur Places API
- Billing Account verknüpft (Kreditkarte hinterlegt)
- API Key in Azure Key Vault gespeichert (wird in späterem Task erstellt)
- Testabfrage erfolgreich

**Test-Anforderungen:**
- Test-Request mit curl gegen Places API
- Verifizieren, dass Restriktionen greifen

**Doku-Anforderungen:**
- `docs/google-api-setup.md` mit Setup-Schritten
- API-Quotas und Kosten dokumentiert
- Key-Rotation-Prozess beschrieben

---

### INFRA-108: Azure Key Vault Terraform Module

**Kategorie:** Infrastructure  
**Priorisierung:** Hoch  
**Schätzung:** 5 PH  
**Abhängigkeiten:** INFRA-005, EXT-001

**Beschreibung:**
Key Vault via Terraform erstellen und Google API Key sowie zukünftige Secrets speichern.

**Akzeptanzkriterien:**
- Terraform Module für Key Vault in `infrastructure/modules/keyvault/`
- Key Vault `kv-doenerguide-prod` erstellt
- Soft-Delete und Purge Protection aktiviert
- Managed Identity für alle 4 Function Apps hat Get/List Permissions
- Google API Key als Secret gespeichert
- Terraform Outputs mit Key Vault URI

**Test-Anforderungen:**
- Test: Secret aus Key Vault abrufen via Azure CLI
- Test: Managed Identity kann Secret lesen (simuliert)

**Doku-Anforderungen:**
- `infrastructure/modules/keyvault/README.md`
- Secret-Naming-Convention dokumentiert
- Access Policy Management erklärt

---

### INFRA-109: Application Insights Terraform Module

**Kategorie:** Infrastructure  
**Priorisierung:** Mittel  
**Schätzung:** 4 PH  
**Abhängigkeiten:** INFRA-005

**Beschreibung:**
Application Insights für Monitoring und Logging via Terraform erstellen.

**Akzeptanzkriterien:**
- Terraform Module für App Insights in `infrastructure/modules/monitoring/`
- Log Analytics Workspace `log-doenerguide-prod` erstellt
- Application Insights `appi-doenerguide-prod` verknüpft
- Instrumentation Key in Terraform Outputs
- Retention auf 90 Tage gesetzt
- Sampling auf 100% für MVP (später reduzieren)

**Test-Anforderungen:**
- Test: Dummy-Event an App Insights senden
- Verifizieren in Azure Portal

**Doku-Anforderungen:**
- `infrastructure/modules/monitoring/README.md`
- Wichtige Metriken und Queries dokumentiert
- Kosten-Hinweis (Retention, Sampling)

---

### DOC-001: Entwickler-Onboarding-Dokumentation

**Kategorie:** Documentation  
**Priorisierung:** Mittel  
**Schätzung:** 6 PH  
**Abhängigkeiten:** DEV-001, DEV-002, DEV-004, DEV-005

**Beschreibung:**
Umfassende Dokumentation für neue Entwickler zum Projekt-Setup.

**Akzeptanzkriterien:**
- `docs/onboarding.md` mit Step-by-Step-Guide:
  1. Prerequisites (Node, Docker, Azure CLI, Terraform)
  2. Repository klonen und Setup
  3. Lokale Umgebung starten
  4. Frontend lokal starten
  5. Functions lokal starten und debuggen
  6. Ersten Code-Change machen und testen
- Troubleshooting-Sektion für häufige Probleme
- Kontaktpersonen bei Fragen
- Geschätzte Setup-Zeit: 1-2 Stunden

**Test-Anforderungen:**
- Dokumentation von neuem Team-Mitglied testen lassen
- Feedback einarbeiten

**Doku-Anforderungen:**
- Onboarding-Checklist als Markdown
- Screenshots für kritische Schritte
- Links zu externen Ressourcen (Azure Docs, etc.)

---

### DOC-002: Architektur-Diagramme ins Repository integrieren

**Kategorie:** Documentation  
**Priorisierung:** Niedrig  
**Schätzung:** 1 PH  
**Abhängigkeiten:** INFRA-001

**Beschreibung:**
Bestehende Architektur-Diagramme in das Repository übertragen und in der Dokumentation verlinken.

**Akzeptanzkriterien:**
- Alle vorhandenen Diagramme nach `docs/diagrams/` kopiert
- Diagramme im README.md verlinkt
- Falls Source-Files vorhanden (`.drawio`, `.mermaid`), ebenfalls integriert
- Kurze Beschreibung zu jedem Diagramm in `docs/architecture.md`

**Test-Anforderungen:**
- Keine Unit-Tests erforderlich
- Manuelle Prüfung: Alle Links funktionieren

**Doku-Anforderungen:**
- Liste aller verfügbaren Diagramme in `docs/architecture.md`
- Erklärung wann welches Diagramm relevant ist
- Hinweis auf Aktualisierung bei Architektur-Änderungen

---

## Milestone 0 - Zusammenfassung

**Gesamt-Tasks:** 17  
**Gesamt-Schätzung:** 71 Personenstunden  
**Prioritäten:**
- Hoch: 9 Tasks (37 PH)
- Mittel: 7 Tasks (33 PH)
- Niedrig: 1 Task (1 PH)

**Kritischer Pfad:**
1. INFRA-001 → INFRA-002 → INFRA-003 → INFRA-005 → INFRA-006
2. Parallel: DEVOPS-001 → DEVOPS-002
3. Parallel: DEV-001, DEV-002, DEV-003, DEV-004, DEV-005
4. EXT-001 kann parallel laufen

**Ready for Milestone 1:** 
- ✅ Repository und CI/CD funktionieren
- ✅ Lokale Entwicklungsumgebung läuft (Azurite, Cosmos DB Emulator, Service Bus Emulator)
- ✅ Terraform deployed grundlegende Infrastruktur
- ✅ Team kann produktiv arbeiten

---

**Status:** ⏳ Bereit für Review  
**Nächster Milestone:** Milestone 1 - Data Pipeline

---

## Milestone 1: Data Pipeline

**Ziel:** Automatisierte Datenerfassung, Bildklassifikation und KI-Bewertung implementieren

**Geschätzte Dauer:** 3-5 Wochen  
**Geschätzter Aufwand:** 195 Personenstunden  
**Team-Kapazität:** 35-70 PH/Woche bei 7 Personen

---

### INFRA-101: Cosmos DB Terraform Module

**Kategorie:** Infrastructure  
**Priorisierung:** Hoch  
**Schätzung:** 6 PH  
**Abhängigkeiten:** INFRA-005 (Terraform Base)

**Beschreibung:**
Cosmos DB via Terraform erstellen für Speicherung von Laden-Daten, Bewertungen und Metadaten.

**Akzeptanzkriterien:**
- Terraform Module in `infrastructure/modules/cosmosdb/`
- Cosmos DB Account `cosmos-doenerguide-prod` (Serverless Mode)
- Database `DoenerGuideDB` erstellt
- Container `places` mit:
  - Partition Key: `/stadtbezirk`
  - Indexing Policy optimiert für Abfragen (stadtbezirk, scores)
  - TTL disabled (Daten bleiben permanent)
- Container `grid-state` für Grid-Management (Partition Key: `/city`)
- Throughput: Autoscale (400-4000 RU/s)
- Terraform Outputs: Connection String, Endpoint

**Test-Anforderungen:**
- Test: Dokument über Azure CLI schreiben und lesen
- Test: Query nach Partition Key funktioniert

**Doku-Anforderungen:**
- `infrastructure/modules/cosmosdb/README.md`
- Datenmodell-Schema dokumentiert (JSON-Beispiele)
- Indexing-Strategie erklärt
- Kosten-Schätzung (Serverless vs. Provisioned)

---

### INFRA-102: Blob Storage Terraform Module

**Kategorie:** Infrastructure  
**Priorisierung:** Hoch  
**Schätzung:** 4 PH  
**Abhängigkeiten:** INFRA-005

**Beschreibung:**
Blob Storage für Bilder (Essen, Ambiente) via Terraform erstellen.

**Akzeptanzkriterien:**
- Storage Account `stdoenerguide` erstellt (Hot Tier)
- Container `shop-images` mit Private Access
- Container `shop-thumbnails` für optimierte Versionen
- Lifecycle Management: Bilder älter als 180 Tage zu Cool Tier
- CORS-Regeln für Frontend-Zugriff (später)
- CDN-Integration vorbereitet (Optional)
- Terraform Outputs: Connection String, Container URLs

**Test-Anforderungen:**
- Test: Blob hochladen via Azure CLI
- Test: Lifecycle Policy greift (simuliert)

**Doku-Anforderungen:**
- `infrastructure/modules/storage/README.md`
- Naming Convention für Blobs: `{placeId}/{category}/{imageId}.jpg`
- Thumbnail-Generierung-Strategie (wird in Service implementiert)

---

### INFRA-103: Service Bus Terraform Module

**Kategorie:** Infrastructure  
**Priorisierung:** Hoch  
**Schätzung:** 5 PH  
**Abhängigkeiten:** INFRA-005

**Beschreibung:**
Azure Service Bus mit Queues für Data Pipeline via Terraform erstellen.

**Akzeptanzkriterien:**
- Service Bus Namespace `sb-doenerguide-prod` (Standard Tier)
- Queues erstellt:
  - `places-found` (TTL: 24h, Max Delivery: 10)
  - `images-ready` (TTL: 24h, Max Delivery: 10)
  - `reviews-ready` (TTL: 24h, Max Delivery: 10)
- Dead Letter Queues automatisch aktiviert
- Managed Identity für Functions hat Send/Receive Permissions
- Terraform Outputs: Connection Strings, Queue Names

**Test-Anforderungen:**
- Test: Nachricht in Queue senden via CLI
- Test: DLQ erhält Nachricht nach Max Delivery Count

**Doku-Anforderungen:**
- `infrastructure/modules/servicebus/README.md`
- Queue-Flow-Diagramm (Mermaid)
- Message-Schema für jede Queue dokumentiert
- Retry-Policy und DLQ-Handling erklärt

---

### INFRA-104: Azure AI Vision Terraform Module

**Kategorie:** Infrastructure  
**Priorisierung:** Hoch  
**Schätzung:** 4 PH  
**Abhängigkeiten:** INFRA-005

**Beschreibung:**
Azure AI Vision (Computer Vision) für Bildklassifikation via Terraform erstellen.

**Akzeptanzkriterien:**
- Computer Vision Resource `cv-doenerguide-prod` (S1 Tier)
- Region: West Europe (oder näher an Stuttgart)
- Custom Vision Training möglich (falls später benötigt)
- Managed Identity hat Zugriff
- API Key in Key Vault gespeichert
- Terraform Outputs: Endpoint, Key Reference

**Test-Anforderungen:**
- Test: Dummy-Bild analysieren via REST API
- Test: Rate Limiting verifizieren (20 calls/minute in S1)

**Doku-Anforderungen:**
- `infrastructure/modules/ai-vision/README.md`
- Unterstützte Features dokumentiert (Image Analysis v4.0)
- Kosten-Kalkulation für erwartete Bildanzahl
- API-Limits dokumentiert

---

### INFRA-105: Azure AI Services Setup (OpenAI + Foundry Models)

**Kategorie:** Infrastructure  
**Priorisierung:** Hoch  
**Schätzung:** 6 PH  
**Abhängigkeiten:** INFRA-005, INFRA-108 (Key Vault)

**Beschreibung:**
Azure AI Services für LLM-Bewertungen via Terraform einrichten (Azure OpenAI + Azure AI Foundry für Open Source Modelle).

**Akzeptanzkriterien:**
- **Azure OpenAI Resource** `aoai-doenerguide-prod` erstellt
  - GPT-4o Deployment mit Name `gpt-4o-deployment`
  - Rate Limiting: 10K tokens/minute (anpassen nach Bedarf)
  - Content Filtering: Standard (kann Essen-Bilder verarbeiten)
- **Azure AI Foundry Deployment** (Alternative für Open Source):
  - Unterstützte Modelle: Llama 3.1, Mistral, Phi-3
  - Serverless Endpoint oder Managed Compute (Entscheidung nach Proof of Concept)
- Beide Endpoints verfügbar, Auswahl über Environment Variable
- API Keys in Key Vault gespeichert:
  - `openai-api-key`
  - `ai-foundry-api-key` (falls applicable)
- Managed Identity für Function Apps hat Zugriff
- Terraform Outputs: Endpoints, Deployment Names, Key References
- Proof of Concept durchführen:
  - Kosten pro Review: GPT-4o vs. Llama vs. Mistral
  - Qualität: Bewertungstexte vergleichen
  - Entscheidung dokumentieren

**Test-Anforderungen:**
- Test: Einfacher Prompt via REST API (beide Endpoints)
- Test: JSON-Output mit strukturiertem Schema (beide Modelle)
- Test: Token-Usage-Tracking funktioniert
- Proof of Concept: 10-20 Reviews mit verschiedenen Modellen generieren

**Doku-Anforderungen:**
- `infrastructure/modules/ai-services/README.md`
- Unterstützte Modelle dokumentiert
- Prompt-Engineering-Guidelines für verschiedene Modelle
- Token-Kalkulation (Bilder + Text → ca. 1000-2000 tokens/request)
- Kosten-Vergleich-Tabelle (GPT-4o, Llama, Mistral)
- Proof of Concept Ergebnisse dokumentieren
- Entscheidungs-Empfehlung für MVP

---

### INFRA-106: Function Apps Terraform Module (Separate Apps)

**Kategorie:** Infrastructure  
**Priorisierung:** Hoch  
**Schätzung:** 10 PH  
**Abhängigkeiten:** INFRA-005, INFRA-101, INFRA-102, INFRA-103

**Beschreibung:**
Terraform Module für 4 separate Function Apps erstellen (eine pro Pipeline-Step).

**Akzeptanzkriterien:**
- Terraform Module in `infrastructure/modules/function-apps/`
- 4 Function Apps erstellt:
  - `func-doenerguide-placesearch-prod` (Timer Trigger)
  - `func-doenerguide-imageclassifier-prod` (Queue Trigger)
  - `func-doenerguide-llmanalyzer-prod` (Queue Trigger)
  - `func-doenerguide-datapersistence-prod` (Queue Trigger)
- Alle im Consumption Plan (serverless)
- App Service Plan: Shared zwischen allen Functions (Linux, Consumption)
- Managed Identity für jede Function App
- Application Insights verknüpft
- Common App Settings:
  - `FUNCTIONS_WORKER_RUNTIME=node`
  - `WEBSITE_NODE_DEFAULT_VERSION=~20`
  - Key Vault Referenzen für Secrets
  - Cosmos DB, Service Bus, Storage Account Connection Strings
- RBAC: Managed Identities haben Zugriff auf benötigte Ressourcen
- Terraform Outputs: Function App Names, Managed Identity IDs

**Test-Anforderungen:**
- Test: `terraform plan` zeigt 4 Function Apps
- Test: Managed Identities können auf Key Vault zugreifen
- Test: ZIP-Deployment funktioniert (manuell testen)

**Doku-Anforderungen:**
- `infrastructure/modules/function-apps/README.md`
- Tabelle: Welche Function braucht welche Permissions
- App Settings Dokumentation
- Deployment-Prozess erklärt

---

### INFRA-107: Storage Account für Function Deployments

**Kategorie:** Infrastructure  
**Priorisierung:** Hoch  
**Schätzung:** 3 PH  
**Abhängigkeiten:** INFRA-005

**Beschreibung:**
Separater Storage Account für Function App Deployments (ZIP-Files) via Terraform.

**Akzeptanzkriterien:**
- Storage Account `stfuncdeployments` erstellt
- Container `deployments` mit Private Access
- Container für jede Function App:
  - `place-search-deployments`
  - `image-classifier-deployments`
  - `llm-analyzer-deployments`
  - `data-persistence-deployments`
- Lifecycle Policy: Deployments älter als 30 Tage löschen (nur letzte 5 Versionen behalten)
- Terraform Outputs: Connection String

**Test-Anforderungen:**
- Test: ZIP-Upload via CLI
- Test: Lifecycle Policy (simuliert)

**Doku-Anforderungen:**
- Deployment-Storage-Konzept erklärt
- Retention-Policy dokumentiert

---

### BE-101: Grid Service - Stuttgart Grid Berechnung

**Kategorie:** Backend  
**Priorisierung:** Hoch  
**Schätzung:** 8 PH  
**Abhängigkeiten:** DEV-005 (Functions Bootstrap)

**Beschreibung:**
Service zur Berechnung des initialen Stuttgart-Grids mit kreisförmiger Priorisierung vom Stadtkern.

**Akzeptanzkriterien:**
- TypeScript Service in `functions/src/services/gridService.ts`
- Funktion `generateInitialGrid(city: string): GridCell[]`
  - Input: "Stuttgart"
  - Output: Array von GridCells (id, lat/lng bounds, center, radius, priority)
- Initiales Grid: 4x4 (16 Zellen) über Stuttgart
- Priority-Sortierung: Distanz zur Stadtmitte (48.7758° N, 9.1829° E)
- Zellen-Radius: ~2km (um 20+ Ergebnisse zu garantieren)
- Grid-Bounds basierend auf Stuttgart Stadtgrenzen

**Test-Anforderungen:**
- Unit Test: Grid hat 16 Zellen
- Unit Test: Priority 1 ist Stadtmitte-Zelle
- Unit Test: Alle Zellen decken Stuttgart-Gebiet ab
- Unit Test: Keine Überlappungen zwischen Zellen

**Doku-Anforderungen:**
- JSDoc für alle Funktionen
- Algorithmus-Beschreibung (wie Priority berechnet)
- Visualisierung: Grid als Mermaid-Diagramm oder ASCII-Art

---

### BE-102: Grid Service - Dynamisches Splitting

**Kategorie:** Backend  
**Priorisierung:** Hoch  
**Schätzung:** 10 PH  
**Abhängigkeiten:** BE-101

**Beschreibung:**
Logik zum dynamischen Aufteilen von Grid-Zellen wenn 20 Ergebnisse erreicht werden.

**Akzeptanzkriterien:**
- Funktion `splitGridCell(cell: GridCell): GridCell[]`
  - Teilt Zelle in 4 Sub-Zellen (2x2)
  - Berechnet neue Bounds und Radien
  - Erhält Priority-Reihenfolge (Unterzellen nah an Original-Priority)
- Funktion `shouldSplitCell(cell: GridCell, resultCount: number): boolean`
  - Returns true wenn resultCount >= 20
- Maximum Splitting Depth: 3 (verhindert unendliche Rekursion)
- Minimum Cell Radius: 500m (sinnvolle Untergrenze)

**Test-Anforderungen:**
- Unit Test: Zelle mit 20 Ergebnissen wird gesplittet
- Unit Test: 4 Sub-Zellen decken Original-Zelle vollständig ab
- Unit Test: Splitting stoppt bei Max Depth
- Unit Test: Splitting stoppt bei Min Radius

**Doku-Anforderungen:**
- JSDoc mit Splitting-Algorithmus
- Beispiel-Visualisierung (vorher/nachher)
- Edge Cases dokumentiert

---

### BE-103: Grid State Management Service

**Kategorie:** Backend  
**Priorisierung:** Hoch  
**Schätzung:** 12 PH  
**Abhängigkeiten:** BE-102, INFRA-101

**Beschreibung:**
Service zum Verwalten des Grid-Status in Cosmos DB (welche Zelle als nächstes, Rotation, Splitting-Historie).

**Akzeptanzkriterien:**
- TypeScript Service in `functions/src/services/gridStateService.ts`
- Funktionen:
  - `initializeGrid(city: string)`: Speichert initiales Grid in `grid-state` Container
  - `getNextCell(city: string): GridCell`: Gibt nächste zu verarbeitende Zelle zurück
  - `markCellProcessed(cellId: string, resultCount: number)`: Markiert Zelle als verarbeitet mit Timestamp
  - `handleCellSplitting(cellId: string, newCells: GridCell[])`: Fügt neue Sub-Zellen hinzu nach Split
  - `getAllCells(city: string): GridCell[]`: Gibt alle aktuellen Zellen zurück
- Rotation-Logik: Kreisförmig durch alle Zellen (Priority-basiert)
- State-Dokument Schema:
  ```json
  {
    "id": "stuttgart-grid",
    "city": "Stuttgart",
    "cells": [...],
    "currentCellIndex": 0,
    "lastProcessedAt": "2025-11-04T10:00:00Z"
  }
  ```

**Test-Anforderungen:**
- Unit Test: Initiales Grid wird korrekt gespeichert
- Unit Test: getNextCell rotiert durch alle Zellen
- Unit Test: Nach 16 Durchläufen startet Rotation von vorne
- Unit Test: Splitting fügt neue Zellen korrekt ein
- Integration Test: Cosmos DB CRUD funktioniert

**Doku-Anforderungen:**
- Service-Architektur dokumentiert
- State-Schema mit allen Feldern
- Rotation-Algorithmus erklärt
- Beispiel-Flows (Initial, Normal, Splitting)

---

### BE-104: Google Places API Service

**Kategorie:** Backend  
**Priorisierung:** Hoch  
**Schätzung:** 10 PH  
**Abhängigkeiten:** DEV-005, EXT-001

**Beschreibung:**
Service für Kommunikation mit Google Places API (Nearby Search).

**Akzeptanzkriterien:**
- TypeScript Service in `functions/src/services/googlePlacesService.ts`
- Funktionen:
  - `searchDoenarShops(cell: GridCell): PlaceResult[]`
    - Nearby Search für "Döner" oder "Kebab" in Zelle
    - Returns: Array von Places (name, address, placeId, rating, photos, etc.)
  - `getPlaceDetails(placeId: string): PlaceDetails`
    - Ruft Details ab (Öffnungszeiten, Reviews, etc.)
  - `getPhotoUrls(photoReferences: string[]): string[]`
    - Generiert Photo-URLs aus References
- API Key aus Key Vault laden
- Rate Limiting: Max. 10 requests/second
- Error Handling: Retry bei 503, Throw bei 403 (Quota)
- Caching: 24h für Place Details (in-memory oder Redis später)

**Test-Anforderungen:**
- Unit Test (Mock): searchDoenarShops returns expected format
- Unit Test: Rate Limiting funktioniert
- Unit Test: API Key Loading aus Key Vault
- Integration Test: Echte API-Abfrage (mit Test-Key)

**Doku-Anforderungen:**
- API-Dokumentation verlinken
- Rate Limits und Quotas dokumentiert
- Response-Mapping erklärt (Google → Internal Format)
- Kosten-Kalkulation pro Request

---

### BE-105: Image Download Service

**Kategorie:** Backend  
**Priorisierung:** Hoch  
**Schätzung:** 8 PH  
**Abhängigkeiten:** INFRA-102, BE-104

**Beschreibung:**
Service zum Herunterladen von Bildern aus Google Places und Upload zu Blob Storage.

**Akzeptanzkriterien:**
- TypeScript Service in `functions/src/services/imageService.ts`
- Funktionen:
  - `downloadImage(photoUrl: string): Buffer`
    - Lädt Bild von Google herunter
  - `uploadToBlob(placeId: string, imageBuffer: Buffer, metadata: ImageMetadata): string`
    - Upload zu Blob Storage
    - Naming: `{placeId}/original/{imageId}.jpg`
    - Returns: Blob URL
  - `generateThumbnail(imageBuffer: Buffer, size: number): Buffer`
    - Erstellt Thumbnail (z.B. 300x300px) mit sharp library
- Unterstützt JPEG, PNG, WebP
- Max. Bildgröße: 10MB
- Error Handling: Skip bei Download-Fehlern

**Test-Anforderungen:**
- Unit Test (Mock): Image Download simulieren
- Unit Test: Thumbnail-Generierung (sharp)
- Unit Test: Blob Upload mit korrektem Naming
- Integration Test: Ende-zu-Ende Download → Upload

**Doku-Anforderungen:**
- Unterstützte Formate dokumentiert
- Thumbnail-Größen und Strategie
- Performance-Optimierungen (parallel Downloads)
- Error-Handling-Strategien

---

### BE-106: Azure AI Vision Classifier Service

**Kategorie:** Backend  
**Priorisierung:** Hoch  
**Schätzung:** 10 PH  
**Abhängigkeiten:** INFRA-104, BE-105

**Beschreibung:**
Service zur Bildklassifikation mit Azure AI Vision (Essen, Ambiente, Irrelevant).

**Akzeptanzkriterien:**
- TypeScript Service in `functions/src/services/imageClassifierService.ts`
- Funktionen:
  - `classifyImage(imageUrl: string): ImageCategory`
    - Ruft Azure AI Vision Image Analysis API auf
    - Analysiert Tags, Objects, Categories
    - Returns: "food", "ambience", "irrelevant"
  - Klassifikations-Logik:
    - **Food:** Tags enthalten "food", "dish", "meal", "kebab", "döner", "meat"
    - **Ambience:** Tags enthalten "restaurant", "interior", "dining", "seating"
    - **Irrelevant:** Alles andere (z.B. "person", "car", "receipt")
- Confidence Threshold: 0.6 (60% Sicherheit)
- Batch Processing möglich (max. 10 Bilder parallel)

**Test-Anforderungen:**
- Unit Test (Mock): Verschiedene Tag-Kombinationen → korrekte Kategorie
- Unit Test: Confidence Threshold funktioniert
- Unit Test: API Key Loading
- Integration Test: Echtes Bild klassifizieren

**Doku-Anforderungen:**
- Klassifikations-Regeln dokumentiert
- Azure AI Vision Features genutzt (Image Analysis v4.0)
- Beispiel-Tags pro Kategorie
- Kosten pro Bild

---

### BE-107: LLM Service (Modell-agnostisch)

**Kategorie:** Backend  
**Priorisierung:** Hoch  
**Schätzung:** 12 PH  
**Abhängigkeiten:** INFRA-105

**Beschreibung:**
Service für KI-Bewertungsgenerierung mit flexibler LLM-Wahl (Azure OpenAI GPT-4o, Llama, Mistral, etc.).

**Akzeptanzkriterien:**
- TypeScript Service in `functions/shared/src/services/llmService.ts`
- Interface-basierte Architektur für verschiedene LLM-Provider:
  ```typescript
  interface LLMProvider {
    generateReview(place: Place, images: ClassifiedImage[]): Promise<AIReview>;
  }
  ```
- Provider-Implementierungen:
  - `AzureOpenAIProvider` (GPT-4o)
  - `AzureAIFoundryProvider` (für Open Source Modelle: Llama, Mistral)
  - Weitere Provider einfach hinzufügbar
- Provider-Auswahl via Environment Variable: `LLM_PROVIDER=azure-openai|azure-foundry`
- Funktionen:
  - `generateReview(place: Place, images: ClassifiedImage[]): AIReview`
    - Input: Laden-Daten + klassifizierte Bilder
    - Optional: Google Reviews (Text) aus Place Details
    - Output: Strukturierte Bewertung (JSON)
- Prompt Engineering (Provider-unabhängig):
  - System Prompt: "Du bist ein Döner-Experte..."
  - User Prompt: Bilder als Base64 oder URLs + Laden-Info
  - Output Format: JSON-Schema erzwingen (via Function Calling oder Structured Output)
- JSON-Schema:
  ```typescript
  {
    reviewText: string;      // 150-300 Wörter
    scores: {
      taste: number;         // 1-10
      toppings: number;      // 1-10 (Belag)
      priceValue: number;    // 1-10 (Preis-Leistung)
      overall: number;       // 1-10 (Gesamt)
    }
  }
  ```
- Validation: Scores 1-10, Text min. 100 Wörter
- Retry bei Invalid Output (max. 2x)
- Kosten-Tracking pro Provider (Token-Usage logging)

**Test-Anforderungen:**
- Unit Test (Mock): Provider Interface funktioniert
- Unit Test: Valid JSON Output für beide Provider
- Unit Test: Schema Validation funktioniert
- Unit Test: Scores in Range 1-10
- Integration Test: Echtes Review mit GPT-4o
- Integration Test: Echtes Review mit Open Source Modell (Llama/Mistral)
- Performance-Vergleich: Kosten pro Review (GPT-4o vs. Open Source)

**Doku-Anforderungen:**
- Provider-Architektur dokumentiert (Interface-basiert)
- Prompt-Template dokumentiert
- JSON-Schema mit Beispielen
- Token-Usage pro Request schätzen für jeden Provider
- Kosten-Kalkulation-Tabelle (GPT-4o vs. Llama vs. Mistral)
- Beispiel-Outputs (Good/Bad Cases)
- Entscheidungshilfe: Welcher Provider für MVP?

---

### BE-108: Cosmos DB Repository Service

**Kategorie:** Backend  
**Priorisierung:** Hoch  
**Schätzung:** 8 PH  
**Abhängigkeiten:** INFRA-101

**Beschreibung:**
Repository-Service für CRUD-Operationen auf Cosmos DB (Places, Reviews).

**Akzeptanzkriterien:**
- TypeScript Service in `functions/src/services/cosmosService.ts`
- Funktionen:
  - `savePlaceWithReview(place: PlaceDocument): void`
    - Upsert (Insert or Update) Place-Dokument
    - Idempotent (verhindert Dubletten bei Re-Processing)
  - `getPlaceById(placeId: string): PlaceDocument | null`
  - `getAllPlaces(stadtbezirk?: string): PlaceDocument[]`
  - `updateReview(placeId: string, review: AIReview): void`
  - `deletePlaceById(placeId: string): void` (für Admin später)
- Connection String aus Environment Variable oder Key Vault
- Error Handling: Retry bei 429 (Rate Limit), Throw bei anderen Fehlern
- Bulk Operations für Performance (Batch Insert)

**Test-Anforderungen:**
- Unit Test (Mock): CRUD Operations
- Unit Test: Upsert verhindert Dubletten (gleiche ID)
- Integration Test: Cosmos DB Emulator lokal
- Integration Test: Partition Key Handling

**Doku-Anforderungen:**
- Repository Pattern erklärt
- Cosmos DB Best Practices (Partition Key Strategy)
- Query-Beispiele für häufige Use Cases
- Performance-Tipps (Batch, Cross-Partition Queries)

---

### FUNC-101: PlaceSearchFunction implementieren

**Kategorie:** Backend  
**Priorisierung:** Hoch  
**Schätzung:** 10 PH  
**Abhängigkeiten:** BE-103, BE-104, INFRA-103

**Beschreibung:**
Azure Function mit Timer Trigger für Grid-basierte Dönerläden-Suche.

**Akzeptanzkriterien:**
- Function in `functions/src/functions/placeSearchFunction.ts`
- Timer Trigger: Konfigurierbar (Default: alle 30 Min via CRON)
- Flow:
  1. Grid State Service: getNextCell()
  2. Google Places Service: searchDoenarShops(cell)
  3. Prüfen: resultCount >= 20? → Split Cell
  4. Service Bus: Send Messages zu `places-found` Queue (ein Message pro Place)
  5. Grid State: markCellProcessed(cell, resultCount)
- Message Format:
  ```json
  {
    "placeId": "ChIJ...",
    "name": "...",
    "address": "...",
    "photoReferences": [...],
    "stadtbezirk": "Stuttgart-Mitte"
  }
  ```
- Logging: Application Insights Custom Events
- Error Handling: Catch & Log, Continue mit nächster Zelle

**Test-Anforderungen:**
- Unit Test: Flow-Logik (Mocked Services)
- Unit Test: Service Bus Message Format
- Unit Test: Grid Splitting Trigger
- Integration Test: Timer Trigger lokal (Azurite + Emulators)

**Doku-Anforderungen:**
- Function-Architektur (Flow-Diagramm)
- CRON Expression erklärt
- Environment Variables dokumentiert
- Troubleshooting (häufige Fehler)

---

### FUNC-102: ImageClassifierFunction implementieren

**Kategorie:** Backend  
**Priorisierung:** Hoch  
**Schätzung:** 10 PH  
**Abhängigkeiten:** BE-105, BE-106, INFRA-103

**Beschreibung:**
Azure Function mit Service Bus Queue Trigger für Bildklassifikation.

**Akzeptanzkriterien:**
- Function in `functions/src/functions/imageClassifierFunction.ts`
- Queue Trigger: `places-found`
- Flow:
  1. Parse Message (Place-Daten)
  2. Image Service: Download alle Bilder
  3. Classifier Service: Klassifiziere jedes Bild
  4. Filtere: Nur "food" und "ambience" behalten
  5. Image Service: Upload relevante Bilder zu Blob Storage + Thumbnails
  6. Service Bus: Send Message zu `images-ready` Queue
- Message Format (Output):
  ```json
  {
    "placeId": "ChIJ...",
    "images": [
      {
        "url": "https://...",
        "thumbnailUrl": "https://...",
        "category": "food"
      }
    ]
  }
  ```
- Max. 5 Food-Bilder, 3 Ambience-Bilder pro Place
- Parallel Processing: Max. 3 Bilder gleichzeitig

**Test-Anforderungen:**
- Unit Test: Flow-Logik (Mocked Services)
- Unit Test: Bild-Limit (5 Food, 3 Ambience)
- Unit Test: Message Format
- Integration Test: Queue → Download → Classify → Upload → Queue

**Doku-Anforderungen:**
- Function-Architektur
- Parallelisierungs-Strategie
- Error Handling (fehlende Bilder, Klassifikation schlägt fehl)

---

### FUNC-103: LLMAnalyzerFunction implementieren

**Kategorie:** Backend  
**Priorisierung:** Hoch  
**Schätzung:** 12 PH  
**Abhängigkeiten:** BE-107, INFRA-103

**Beschreibung:**
Azure Function mit Service Bus Queue Trigger für KI-Bewertungsgenerierung.

**Akzeptanzkriterien:**
- Function in `functions/src/functions/llmAnalyzerFunction.ts`
- Queue Trigger: `images-ready`
- Flow:
  1. Parse Message (Place + Images)
  2. Optional: Google Places Service: getPlaceDetails() für Reviews
  3. LLM Service: generateReview(place, images, reviews?)
  4. Validation: Scores 1-10, Text nicht leer
  5. Service Bus: Send Message zu `reviews-ready` Queue
- Message Format (Output):
  ```json
  {
    "placeId": "ChIJ...",
    "aiReview": {
      "reviewText": "...",
      "scores": {...}
    }
  }
  ```
- Proof of Concept Flag: `USE_GOOGLE_REVIEWS` (Environment Variable)
  - true: Include Reviews in Prompt
  - false: Only Images
- Retry Logic: Max. 2x bei Invalid Output

**Test-Anforderungen:**
- Unit Test: Flow mit/ohne Reviews
- Unit Test: Score Validation
- Unit Test: Retry bei Invalid Output
- Integration Test: Echtes Review generieren

**Doku-Anforderungen:**
- Function-Architektur
- Prompt-Strategie (Bilder vs. Bilder+Reviews)
- Proof of Concept Entscheidungs-Kriterien
- Output-Beispiele

---

### FUNC-104: DataPersistenceFunction implementieren

**Kategorie:** Backend  
**Priorisierung:** Hoch  
**Schätzung:** 8 PH  
**Abhängigkeiten:** BE-108, INFRA-103

**Beschreibung:**
Azure Function mit Service Bus Queue Trigger für finale Datenspeicherung in Cosmos DB.

**Akzeptanzkriterien:**
- Function in `functions/src/functions/dataPersistenceFunction.ts`
- Queue Trigger: `reviews-ready`
- Flow:
  1. Parse Message (Place + Images + AI Review)
  2. Konstruiere vollständiges PlaceDocument
  3. Cosmos Service: savePlaceWithReview(document)
  4. Logging: Success Event mit placeId
- PlaceDocument Schema:
  ```json
  {
    "id": "place_ChIJ...",
    "googlePlaceId": "ChIJ...",
    "name": "...",
    "address": "...",
    "stadtbezirk": "Stuttgart-Mitte",
    "images": [...],
    "aiReview": {...},
    "createdAt": "2025-11-04T10:00:00Z",
    "lastUpdatedAt": "2025-11-04T10:00:00Z"
  }
  ```
- Idempotency: Bei Re-Processing nur `lastUpdatedAt` updaten

**Test-Anforderungen:**
- Unit Test: Document Construction
- Unit Test: Upsert Logic (Insert vs. Update)
- Integration Test: Cosmos DB Emulator

**Doku-Anforderungen:**
- Function-Architektur
- Schema-Dokumentation
- Idempotency-Strategie

---

### TEST-101: Integration Test - End-to-End Pipeline

**Kategorie:** Testing  
**Priorisierung:** Mittel  
**Schätzung:** 12 PH  
**Abhängigkeiten:** FUNC-101, FUNC-102, FUNC-103, FUNC-104

**Beschreibung:**
End-to-End Integration Test für die komplette Data Pipeline.

**Akzeptanzkriterien:**
- Test-Suite in `functions/tests/integration/pipeline.test.ts`
- Test-Szenario:
  1. Trigger PlaceSearchFunction manuell (Test-Grid-Zelle mit bekanntem Laden)
  2. Verifiziere: Messages in `places-found` Queue
  3. Warte auf ImageClassifierFunction
  4. Verifiziere: Bilder in Blob Storage, Messages in `images-ready`
  5. Warte auf LLMAnalyzerFunction
  6. Verifiziere: Messages in `reviews-ready`
  7. Warte auf DataPersistenceFunction
  8. Verifiziere: Dokument in Cosmos DB mit allen Daten
- Test läuft gegen lokale Emulatoren (nicht Production)
- Timeout: 5 Minuten max.
- Cleanup: Alle Test-Daten nach Test löschen

**Test-Anforderungen:**
- Integration Test mit echten Emulatoren
- Test-Daten-Setup (Fixtures)
- Cleanup-Mechanismus

**Doku-Anforderungen:**
- Test-Setup-Anleitung
- Wie Test lokal ausführen
- CI/CD Integration (später)

---

### TEST-102: Load Test - Rate Limiting & Performance

**Kategorie:** Testing  
**Priorisierung:** Niedrig  
**Schätzung:** 8 PH  
**Abhängigkeiten:** FUNC-101, FUNC-102, FUNC-103, FUNC-104

**Beschreibung:**
Performance- und Load-Tests für kritische Services (Google API, Azure AI, OpenAI).

**Akzeptanzkriterien:**
- Test-Suite mit Artillery oder K6
- Test-Szenarien:
  - Google Places API: 10 requests/second → Verifiziere Rate Limiting
  - Azure AI Vision: 20 Bilder parallel → Verifiziere Throughput
  - OpenAI: 5 parallele Requests → Verifiziere Token Limits
- Metriken:
  - Response Times (p50, p95, p99)
  - Error Rate
  - Throughput (requests/minute)
- Test läuft gegen Shared Dev-Account (nicht Production)

**Test-Anforderungen:**
- Load Test Scripts
- Performance Benchmarks dokumentiert

**Doku-Anforderungen:**
- Test-Ausführung-Anleitung
- Benchmark-Results
- Bottleneck-Analyse

---

### DOC-101: Data Pipeline Architektur-Dokumentation

**Kategorie:** Documentation  
**Priorisierung:** Mittel  
**Schätzung:** 6 PH  
**Abhängigkeiten:** Alle FUNC-Tasks

**Beschreibung:**
Umfassende Dokumentation der Data Pipeline Architektur.

**Akzeptanzkriterien:**
- Dokument `docs/data-pipeline.md`
- Inhalte:
  - Übersichts-Diagramm (Mermaid oder Bild)
  - Detaillierte Beschreibung jeder Function
  - Queue-Flow und Message-Schemas
  - Grid-Management-Algorithmus
  - Fehlerbehandlung und Retry-Logic
  - Monitoring und Debugging
- Sequence-Diagramm für kompletten Flow
- Troubleshooting-Guide (häufige Probleme)

**Test-Anforderungen:**
- Review durch Team
- Verständlichkeit testen mit neuem Entwickler

**Doku-Anforderungen:**
- Diagramme in Mermaid oder Draw.io
- Code-Beispiele für Message-Handling
- Links zu relevanten Azure Docs

---

### DOC-102: Google Places API Integration Guide

**Kategorie:** Documentation  
**Priorisierung:** Niedrig  
**Schätzung:** 4 PH  
**Abhängigkeiten:** BE-104

**Beschreibung:**
Dokumentation für Google Places API Integration, Kosten und Best Practices.

**Akzeptanzkriterien:**
- Dokument `docs/google-places-integration.md`
- Inhalte:
  - API-Setup und Key-Erstellung
  - Verwendete Endpoints (Nearby Search, Place Details, Photos)
  - Request/Response Beispiele
  - Kosten-Kalkulation (pro 1000 Requests)
  - Rate Limits und Quotas
  - Best Practices (Caching, Error Handling)

**Test-Anforderungen:**
- Keine Tests erforderlich

**Doku-Anforderungen:**
- API-Kosten für erwartetes Volumen (~500 Läden in Stuttgart)
- Monitoring-Empfehlungen (API Usage)

---

### DEVOPS-101: Pipeline Monitoring & Alerting

**Kategorie:** DevOps  
**Priorisierung:** Mittel  
**Schätzung:** 8 PH  
**Abhängigkeiten:** INFRA-109 (App Insights), Alle FUNC-Tasks

**Beschreibung:**
Monitoring und Alerting für Data Pipeline via Application Insights.

**Akzeptanzkriterien:**
- Application Insights Queries (KQL) für:
  - Pipeline Success Rate (% erfolgreiche Durchläufe)
  - Average Processing Time pro Function
  - Error Rate pro Function
  - Dead Letter Queue Message Count
  - API Call Counts (Google, Azure AI, OpenAI)
- Alerts konfiguriert:
  - Error Rate > 10% in 5 Minuten
  - DLQ Messages > 5
  - PlaceSearchFunction läuft nicht (Heartbeat)
  - API Rate Limit erreicht
- Dashboard in Azure Portal mit Widgets
- Optional: Export zu Slack/Teams

**Test-Anforderungen:**
- Test: Alerts triggern bei simulierten Fehlern
- Test: Queries liefern erwartete Ergebnisse

**Doku-Anforderungen:**
- `docs/monitoring.md` mit Query-Beispielen
- Alert-Konfiguration dokumentiert
- Runbook für häufige Alerts (was tun bei Error?)

---

## Milestone 1 - Zusammenfassung

**Gesamt-Tasks:** 27  
**Gesamt-Schätzung:** 195 Personenstunden  
**Prioritäten:**
- Hoch: 21 Tasks (163 PH)
- Mittel: 4 Tasks (24 PH)
- Niedrig: 2 Tasks (8 PH)

**Kritischer Pfad:**
1. Infrastruktur: INFRA-101 → INFRA-109 (teilweise parallel)
2. DEV-005 (Functions Bootstrap) → DEV-006 (Build Scripts)
3. Services: BE-101 → BE-102 → BE-103 → BE-104 → BE-105 → BE-106 → BE-107 → BE-108
4. Functions: FUNC-101 → FUNC-102 → FUNC-103 → FUNC-104
5. Testing & Doku parallel nach Functions fertig

**Parallelisierungs-Möglichkeiten:**
- Infrastruktur-Tasks (INFRA-101 bis INFRA-109) können größtenteils parallel laufen
- Service-Entwicklung teilweise parallel (BE-104, BE-105, BE-106 nach Basics)
- Functions können nach Services parallel entwickelt werden (4 Teams à 1-2 Personen)
- Build Scripts (DEV-006) kann parallel zu Service-Entwicklung laufen

**Separate Function Apps - Vorteile:**
- ✅ Unabhängiges Deployment einzelner Functions
- ✅ Individuelle Skalierung pro Function
- ✅ Isolierte Fehlerbehandlung
- ✅ Granulare Monitoring-Metriken
- ⚠️ Höherer initialer Setup-Aufwand (kompensiert durch Flexibilität)

**Ready for Milestone 2:**
- ✅ Data Pipeline läuft automatisch alle 30 Min
- ✅ Stuttgart Grid wird systematisch durchsucht
- ✅ Bilder werden klassifiziert und gespeichert
- ✅ KI-Bewertungen werden generiert (Modell nach PoC gewählt)
- ✅ Daten in Cosmos DB verfügbar für Frontend
- ✅ Alle 4 Function Apps deployen unabhängig

---

**Status:** ⏳ Bereit für Review  
**Nächster Milestone:** Milestone 2 - API Layer

---

## Milestone 2: API Layer

**Ziel:** REST API für Frontend-Zugriff auf Dönerläden-Daten implementieren

**Geschätzte Dauer:** 2 Wochen  
**Geschätzter Aufwand:** 62 Personenstunden  
**Team-Kapazität:** 35-70 PH/Woche bei 7 Personen

---

### INFRA-201: API Function App Terraform Module

**Kategorie:** Infrastructure  
**Priorisierung:** Hoch  
**Schätzung:** 5 PH  
**Abhängigkeiten:** INFRA-005, INFRA-101 (Cosmos DB)

**Beschreibung:**
Separate Function App für API-Endpoints via Terraform erstellen.

**Akzeptanzkriterien:**
- Function App `func-doenerguide-api-prod` erstellt
- Consumption Plan (serverless, Linux)
- Managed Identity erstellt
- Application Insights verknüpft
- RBAC: Managed Identity hat Read-Only Access auf Cosmos DB
- App Settings:
  - `FUNCTIONS_WORKER_RUNTIME=node`
  - `WEBSITE_NODE_DEFAULT_VERSION=~20`
  - Cosmos DB Connection String (Read-Only)
  - `CORS_ALLOWED_ORIGINS` für Frontend (später konfigurieren)
- CORS konfiguriert (wildcards für Entwicklung, spezifische Domain später)
- Terraform Outputs: Function App Name, Managed Identity ID, API Base URL

**Test-Anforderungen:**
- Test: Function App ist erreichbar
- Test: Managed Identity kann Cosmos DB lesen
- Test: CORS Headers funktionieren

**Doku-Anforderungen:**
- `infrastructure/modules/api-function/README.md`
- API Function App Konfiguration dokumentiert
- CORS-Strategie erklärt
- Read-Only Access Pattern beschrieben

---

### BE-201: API Service Layer - Cosmos DB Repository

**Kategorie:** Backend  
**Priorisierung:** Hoch  
**Schätzung:** 6 PH  
**Abhängigkeiten:** BE-108 (Cosmos Repository aus Milestone 1)

**Beschreibung:**
API-spezifische Repository-Methoden für optimierte Read-Operationen auf Cosmos DB.

**Akzeptanzkriterien:**
- Service in `functions/shared/src/services/apiCosmosService.ts`
- Funktionen:
  - `getPlaces(filters: PlaceFilters, pagination: Pagination): PlacesResult`
    - Filters: stadtbezirk (optional), sortBy, sortOrder
    - Pagination: limit, offset
    - Returns: places[], total count
  - `getPlaceById(id: string): PlaceDocument | null`
  - `getStadtbezirke(): string[]` (unique list)
  - `getPlaceCount(): number` (total)
- Optimierte Cosmos DB Queries:
  - Nutzt Partition Key (stadtbezirk) für Performance
  - Sortierung via ORDER BY
  - Pagination via OFFSET/LIMIT
- Query-Beispiele:
  ```sql
  SELECT * FROM c 
  WHERE c.stadtbezirk = @stadtbezirk 
  ORDER BY c.aiReview.scores.overall DESC
  OFFSET @offset LIMIT @limit
  ```
- Error Handling: Spezifische Fehler (NotFound, InvalidQuery)

**Test-Anforderungen:**
- Unit Test: Query Builder Logic
- Unit Test: Filter & Pagination
- Integration Test: Cosmos DB Emulator
- Performance Test: Query mit 500+ Dokumenten

**Doku-Anforderungen:**
- Query-Optimierung dokumentiert
- Partition Key Strategy erklärt
- Performance-Benchmarks (Latenz, RU-Verbrauch)

---

### BE-202: API Response Models & Validation

**Kategorie:** Backend  
**Priorisierung:** Hoch  
**Schätzung:** 4 PH  
**Abhängigkeiten:** BE-201

**Beschreibung:**
TypeScript Interfaces und Validation für API Request/Response-Formate.

**Akzeptanzkriterien:**
- Models in `functions/shared/src/models/apiModels.ts`
- Request Models:
  ```typescript
  interface PlaceFilters {
    stadtbezirk?: string;
    sortBy?: 'overall' | 'taste' | 'toppings' | 'priceValue';
    sortOrder?: 'asc' | 'desc';
  }
  
  interface Pagination {
    limit: number;    // default: 20, max: 100
    offset: number;   // default: 0
  }
  ```
- Response Models:
  ```typescript
  interface ApiResponse<T> {
    data: T;
    meta?: ResponseMeta;
  }
  
  interface ResponseMeta {
    total?: number;
    page?: number;
    limit?: number;
  }
  
  interface ApiError {
    error: {
      code: string;
      message: string;
    }
  }
  ```
- Validation mit Zod oder class-validator:
  - stadtbezirk: string, optional
  - sortBy: enum, default 'overall'
  - sortOrder: enum, default 'desc'
  - limit: number, 1-100, default 20
  - offset: number, >=0, default 0
- Helper Functions:
  - `validatePlaceFilters(query: any): PlaceFilters`
  - `formatApiResponse<T>(data: T, meta?: ResponseMeta): ApiResponse<T>`
  - `formatApiError(code: string, message: string): ApiError`

**Test-Anforderungen:**
- Unit Test: Validation Logic (valid/invalid inputs)
- Unit Test: Default Values
- Unit Test: Response Formatting

**Doku-Anforderungen:**
- API Request/Response Schemas dokumentiert
- Validation Rules erklärt
- Beispiel-Requests/Responses

---

### BE-203: HTTP Cache Headers Service

**Kategorie:** Backend  
**Priorisierung:** Mittel  
**Schätzung:** 3 PH  
**Abhängigkeiten:** Keine

**Beschreibung:**
Service zur Generierung von HTTP Cache Headers für API-Responses.

**Akzeptanzkriterien:**
- Service in `functions/shared/src/services/cacheHeaderService.ts`
- Funktionen:
  - `getCacheHeaders(endpoint: string): HeadersObject`
- Cache-Strategie:
  - `GET /places`: 
    - `Cache-Control: public, max-age=900, s-maxage=1800` (15 Min Client, 30 Min CDN)
    - `ETag` basierend auf Query + Timestamp der letzten Cosmos DB Änderung
  - `GET /places/{id}`:
    - `Cache-Control: public, max-age=1800, s-maxage=3600` (30 Min Client, 60 Min CDN)
    - `ETag` basierend auf PlaceDocument.lastUpdatedAt
- `Vary: Accept-Encoding` Header setzen
- `Last-Modified` Header basierend auf Cosmos DB Timestamps
- Unterstützung für `If-None-Match` (304 Not Modified)

**Test-Anforderungen:**
- Unit Test: Header-Generierung für verschiedene Endpoints
- Unit Test: ETag-Berechnung konsistent
- Integration Test: 304 Response bei gleichem ETag

**Doku-Anforderungen:**
- Cache-Strategie dokumentiert
- ETag-Berechnung erklärt
- CDN-Integration-Hinweise

---

### FUNC-201: GetPlaces API Function

**Kategorie:** Backend  
**Priorisierung:** Hoch  
**Schätzung:** 8 PH  
**Abhängigkeiten:** BE-201, BE-202, BE-203

**Beschreibung:**
HTTP-Triggered Function für Liste aller Dönerläden mit Filtering, Sorting und Pagination.

**Akzeptanzkriterien:**
- Function in `functions/api-functions/src/functions/getPlaces/index.ts`
- HTTP Trigger: `GET /api/places`
- Query Parameters:
  - `stadtbezirk` (optional): Filter nach Stadtbezirk
  - `sortBy` (optional, default: 'overall'): Sortier-Feld
  - `sortOrder` (optional, default: 'desc'): Sortier-Richtung
  - `limit` (optional, default: 20, max: 100): Anzahl Ergebnisse
  - `offset` (optional, default: 0): Pagination Offset
- Flow:
  1. Parse & Validate Query Parameters
  2. API Cosmos Service: getPlaces(filters, pagination)
  3. Cache Headers Service: getCacheHeaders('places')
  4. Format Response mit Meta (total, page, limit)
  5. Return 200 mit JSON Body + Cache Headers
- Response Format:
  ```json
  {
    "data": [
      {
        "id": "place_ChIJ...",
        "name": "Döner Palast",
        "stadtbezirk": "Stuttgart-Mitte",
        "address": "...",
        "images": [...],
        "aiReview": {
          "reviewText": "...",
          "scores": {...}
        }
      }
    ],
    "meta": {
      "total": 123,
      "limit": 20,
      "offset": 0
    }
  }
  ```
- Error Handling:
  - 400 Bad Request bei Invalid Query Parameters
  - 500 Internal Server Error bei Cosmos DB Fehler
- Logging: Application Insights Custom Events

**Test-Anforderungen:**
- Unit Test: Query Parameter Validation
- Unit Test: Response Formatting
- Integration Test: Verschiedene Filter/Sort-Kombinationen
- Integration Test: Pagination (offset/limit)
- Load Test: 100 parallele Requests

**Doku-Anforderungen:**
- API Endpoint dokumentiert (README oder Swagger später)
- Query Parameter Details
- Response Schema
- Beispiel-Requests mit curl

---

### FUNC-202: GetPlaceById API Function

**Kategorie:** Backend  
**Priorisierung:** Hoch  
**Schätzung:** 6 PH  
**Abhängigkeiten:** BE-201, BE-202, BE-203

**Beschreibung:**
HTTP-Triggered Function für Details eines einzelnen Dönerladens.

**Akzeptanzkriterien:**
- Function in `functions/api-functions/src/functions/getPlaceById/index.ts`
- HTTP Trigger: `GET /api/places/{id}`
- Path Parameter: `id` (z.B. "place_ChIJ...")
- Flow:
  1. Parse & Validate Path Parameter
  2. API Cosmos Service: getPlaceById(id)
  3. If not found → 404 Error
  4. Cache Headers Service: getCacheHeaders('place-detail')
  5. Format Response
  6. Return 200 mit JSON Body + Cache Headers
- Response Format:
  ```json
  {
    "data": {
      "id": "place_ChIJ...",
      "name": "Döner Palast",
      "googlePlaceId": "ChIJ...",
      "stadtbezirk": "Stuttgart-Mitte",
      "address": "...",
      "images": [
        {
          "url": "https://...",
          "thumbnailUrl": "https://...",
          "category": "food"
        }
      ],
      "aiReview": {
        "reviewText": "...",
        "scores": {
          "taste": 8,
          "toppings": 9,
          "priceValue": 7,
          "overall": 8
        }
      },
      "lastUpdatedAt": "2025-11-04T10:00:00Z"
    }
  }
  ```
- Error Handling:
  - 400 Bad Request bei Invalid ID Format
  - 404 Not Found wenn Place nicht existiert
  - 500 Internal Server Error bei Cosmos DB Fehler

**Test-Anforderungen:**
- Unit Test: ID Validation
- Unit Test: 404 Error bei nicht-existentem Place
- Integration Test: Vollständiges Place-Dokument returnen
- Load Test: 100 parallele Requests

**Doku-Anforderungen:**
- API Endpoint dokumentiert
- Path Parameter Details
- Response Schema mit allen Feldern
- Beispiel-Requests

---

### FUNC-203: GetStadtbezirke API Function (Optional)

**Kategorie:** Backend  
**Priorisierung:** Niedrig  
**Schätzung:** 4 PH  
**Abhängigkeiten:** BE-201, BE-202

**Beschreibung:**
HTTP-Triggered Function für Liste aller verfügbaren Stadtbezirke (für Frontend-Filter).

**Akzeptanzkriterien:**
- Function in `functions/api-functions/src/functions/getStadtbezirke/index.ts`
- HTTP Trigger: `GET /api/stadtbezirke`
- Flow:
  1. API Cosmos Service: getStadtbezirke()
  2. Cache Headers: lange Cache-Duration (1 Stunde, ändert sich selten)
  3. Format Response
  4. Return 200 mit JSON Array
- Response Format:
  ```json
  {
    "data": [
      "Stuttgart-Mitte",
      "Stuttgart-Nord",
      "Stuttgart-Ost",
      ...
    ]
  }
  ```
- Cache Strategy: `Cache-Control: public, max-age=3600` (1 Stunde)

**Test-Anforderungen:**
- Unit Test: Response Format
- Integration Test: Alle Stadtbezirke aus DB

**Doku-Anforderungen:**
- Endpoint dokumentiert
- Use Case: Frontend-Filter

---

### TEST-201: API Integration Tests

**Kategorie:** Testing  
**Priorisierung:** Mittel  
**Schätzung:** 8 PH  
**Abhängigkeiten:** FUNC-201, FUNC-202

**Beschreibung:**
Umfassende Integration Tests für alle API-Endpoints.

**Akzeptanzkriterien:**
- Test-Suite in `functions/api-functions/tests/integration/api.test.ts`
- Test-Szenarien:
  - **GET /places:**
    - Ohne Filter → Alle Places
    - Mit stadtbezirk Filter → Gefilterte Liste
    - Sortierung (asc/desc, verschiedene Felder)
    - Pagination (verschiedene limit/offset)
    - Invalid Query Parameters → 400 Error
  - **GET /places/{id}:**
    - Valid ID → Place Details
    - Invalid ID → 404 Error
    - Malformed ID → 400 Error
  - **Cache Headers:**
    - ETag vorhanden
    - Cache-Control korrekt
    - 304 Not Modified bei If-None-Match
- Tests laufen gegen lokale Emulatoren
- Test-Daten-Setup mit Fixtures (mind. 20 Test-Places)
- Cleanup nach Tests

**Test-Anforderungen:**
- Integration Tests mit echten HTTP Requests
- Test-Daten-Management
- Assertions für alle Response-Felder

**Doku-Anforderungen:**
- Test-Setup-Anleitung
- Fixture-Daten dokumentiert
- Test-Execution lokal und in CI/CD

---

### TEST-202: API Performance Tests

**Kategorie:** Testing  
**Priorisierung:** Niedrig  
**Schätzung:** 6 PH  
**Abhängigkeiten:** FUNC-201, FUNC-202

**Beschreibung:**
Performance- und Load-Tests für API-Endpoints.

**Akzeptanzkriterien:**
- Load Test Scripts mit Artillery oder K6
- Test-Szenarien:
  - **Concurrent Requests:**
    - 100 parallele GET /places → Response Time < 500ms (p95)
    - 100 parallele GET /places/{id} → Response Time < 300ms (p95)
  - **High Load:**
    - 1000 Requests über 1 Minute → Error Rate < 1%
  - **Cache Validation:**
    - ETag-basierte Requests → 304 Responses schneller als 200
- Metriken:
  - Response Times (p50, p95, p99)
  - Error Rate
  - Throughput (requests/second)
  - Cosmos DB RU Consumption
- Tests laufen gegen Shared Dev-Account (nicht Production)

**Test-Anforderungen:**
- Load Test Scripts ausführbar
- Performance Benchmarks dokumentiert

**Doku-Anforderungen:**
- Test-Execution-Anleitung
- Benchmark-Results
- Bottleneck-Analyse
- Skalierungs-Empfehlungen

---

### DEVOPS-201: API Function Deployment Integration

**Kategorie:** DevOps  
**Priorisierung:** Hoch  
**Schätzung:** 4 PH  
**Abhängigkeiten:** DEVOPS-002, INFRA-201

**Beschreibung:**
Deployment-Integration für API Function App in GitHub Actions Workflow erweitern.

**Akzeptanzkriterien:**
- Workflow `.github/workflows/deploy.yml` erweitern
- Neue Deployment-Steps:
  - Build API Function: `cd functions/api-functions && npm ci && npm run build`
  - Package: ZIP erstellen
  - Deploy zu `func-doenerguide-api-prod`
- Deploy läuft parallel zu anderen Function Apps (optional)
- Health Check nach Deployment:
  - `GET /api/places` returns 200
  - Verifiziere: Mindestens 1 Place in Response
- Rollback bei fehlgeschlagenem Health Check

**Test-Anforderungen:**
- Dry-Run Deployment
- Health Check funktioniert

**Doku-Anforderungen:**
- `docs/ci-cd.md` erweitern
- API Deployment-Prozess dokumentiert
- Health Check Strategie

---

### DOC-201: API Dokumentation

**Kategorie:** Documentation  
**Priorisierung:** Mittel  
**Schätzung:** 5 PH  
**Abhängigkeiten:** FUNC-201, FUNC-202, FUNC-203

**Beschreibung:**
Vollständige API-Dokumentation für Frontend-Entwickler.

**Akzeptanzkriterien:**
- Dokument `docs/api-documentation.md`
- Inhalte:
  - Base URL: `https://func-doenerguide-api-prod.azurewebsites.net`
  - Authentication: None (öffentliche API)
  - Alle Endpoints mit:
    - HTTP Method
    - Path
    - Query/Path Parameters
    - Request/Response Schemas
    - Beispiel-Requests (curl)
    - Beispiel-Responses (JSON)
    - Status Codes (200, 400, 404, 500)
  - Cache Headers Erklärung
  - Rate Limiting: None (für MVP)
  - Error Format Standardisierung
- CORS-Info für Frontend
- Hinweis: OpenAPI Spec wird separat in PM erstellt

**Test-Anforderungen:**
- Review durch Frontend-Team
- Beispiele testen (curl)

**Doku-Anforderungen:**
- Verständliche Sprache
- Vollständige Beispiele
- Versionierung (v1)

---

### DOC-202: API Performance & Monitoring Guide

**Kategorie:** Documentation  
**Priorisierung:** Niedrig  
**Schätzung:** 3 PH  
**Abhängigkeiten:** TEST-202, DEVOPS-201

**Beschreibung:**
Dokumentation zu API-Performance, Monitoring und Troubleshooting.

**Akzeptanzkriterien:**
- Dokument `docs/api-monitoring.md`
- Inhalte:
  - Application Insights Queries für API-Metriken
  - Performance Benchmarks (aus TEST-202)
  - Cache-Strategie und Hit-Rates
  - Cosmos DB RU-Verbrauch pro Endpoint
  - Häufige Fehler und Lösungen
  - Skalierungs-Empfehlungen
- Alert-Empfehlungen:
  - API Response Time > 1s (p95)
  - Error Rate > 5%
  - Cosmos DB RU Throttling

**Test-Anforderungen:**
- Keine Tests erforderlich

**Doku-Anforderungen:**
- KQL-Queries für App Insights
- Dashboard-Empfehlungen

---

## Milestone 2 - Zusammenfassung

**Gesamt-Tasks:** 13  
**Gesamt-Schätzung:** 62 Personenstunden  
**Prioritäten:**
- Hoch: 7 Tasks (42 PH)
- Mittel: 4 Tasks (16 PH)
- Niedrig: 2 Tasks (4 PH)

**Kritischer Pfad:**
1. INFRA-201 (API Function App)
2. BE-201 → BE-202 → BE-203 (parallel)
3. FUNC-201, FUNC-202 (parallel möglich nach BE-Tasks)
4. TEST-201 → DEVOPS-201
5. Doku parallel

**Parallelisierungs-Möglichkeiten:**
- BE-201, BE-202, BE-203 teilweise parallel
- FUNC-201, FUNC-202, FUNC-203 parallel (3 Entwickler)
- Tests und Doku nach Functions

**API-Features MVP:**
- ✅ GET /api/places (Filter, Sort, Pagination)
- ✅ GET /api/places/{id} (Details)
- ✅ GET /api/stadtbezirke (Optional, für Filter-UI)
- ✅ HTTP Cache Headers (CDN-ready)
- ✅ Standardisiertes Response Format
- ✅ Error Handling
- ⚠️ Keine Admin-Endpoints (V2)
- ⚠️ Kein In-Memory Caching (stateless)
- ⚠️ Kein Rate Limiting (erst bei Bedarf)

**Ready for Milestone 3:**
- ✅ API liefert Daten an Frontend
- ✅ Performance optimiert (Cache Headers, Cosmos Queries)
- ✅ Vollständig dokumentiert
- ✅ Integration Tests grün
- ✅ Deployment automatisiert

---

**Status:** ⏳ Bereit für Review  
**Nächster Milestone:** Milestone 3 - Frontend MVP

---

## Milestone 3: Frontend MVP

**Ziel:** Benutzerfreundliches Frontend mit Listen-, Detail- und Filteransicht implementieren

**Geschätzte Dauer:** 3-4 Wochen  
**Geschätzter Aufwand:** 128 Personenstunden  
**Team-Kapazität:** 35-70 PH/Woche bei 7 Personen

---

### INFRA-301: Azure Static Web Apps Terraform Module

**Kategorie:** Infrastructure  
**Priorisierung:** Hoch  
**Schätzung:** 5 PH  
**Abhängigkeiten:** INFRA-005, INFRA-201 (API Function App)

**Beschreibung:**
Azure Static Web App für Next.js Frontend via Terraform erstellen.

**Akzeptanzkriterien:**
- Terraform Module in `infrastructure/modules/static-web-app/`
- Static Web App `swa-doenerguide-prod` erstellt
- SKU: Free (ausreichend für MVP, später Standard bei Bedarf)
- GitHub Integration konfiguriert (Repository + Branch)
- API Integration: Linked zu `func-doenerguide-api-prod`
  - API Base Path: `/api` → Proxy zu Function App
- Custom Domain: Vorbereitet, aber nicht aktiviert (später)
- Environment Variables:
  - `NEXT_PUBLIC_API_BASE_URL=/api`
- Terraform Outputs: Default Hostname, Resource ID

**Test-Anforderungen:**
- Test: Static Web App ist erreichbar
- Test: `/api` Proxy funktioniert
- Test: Deployment via GitHub Actions (manuell)

**Doku-Anforderungen:**
- `infrastructure/modules/static-web-app/README.md`
- API-Proxy-Konfiguration dokumentiert
- Custom Domain Setup-Anleitung (für später)
- Free vs. Standard Tier Unterschiede

---

### FE-301: Frontend-Architektur & Context Setup

**Kategorie:** Frontend  
**Priorisierung:** Hoch  
**Schätzung:** 6 PH  
**Abhängigkeiten:** DEV-004 (Frontend Bootstrap)

**Beschreibung:**
React Context API Setup für globalen State (Filter, Sort, Loaded Places).

**Akzeptanzkriterien:**
- Verzeichnisstruktur:
  ```
  /frontend
  ├── src/
  │   ├── app/              # Next.js App Router
  │   ├── components/       # React Components
  │   │   ├── layout/
  │   │   ├── places/
  │   │   ├── common/
  │   │   └── ui/
  │   ├── contexts/         # React Contexts
  │   │   ├── PlacesContext.tsx
  │   │   └── FilterContext.tsx
  │   ├── hooks/            # Custom Hooks
  │   ├── lib/              # Utilities
  │   │   ├── api.ts
  │   │   └── types.ts
  │   └── styles/           # Global Styles
  ```
- **PlacesContext:**
  ```typescript
  interface PlacesContextType {
    places: Place[];
    loading: boolean;
    error: string | null;
    hasMore: boolean;
    loadPlaces: () => Promise<void>;
    loadMore: () => Promise<void>;
  }
  ```
- **FilterContext:**
  ```typescript
  interface FilterContextType {
    stadtbezirke: string[];
    sortBy: 'overall' | 'taste' | 'toppings' | 'priceValue';
    sortOrder: 'asc' | 'desc';
    setStadtbezirke: (bezirke: string[]) => void;
    setSortBy: (sortBy: string) => void;
    setSortOrder: (order: string) => void;
    resetFilters: () => void;
  }
  ```
- Contexts verwenden localStorage für Filter-Persistierung
- Custom Hooks: `usePlaces()`, `useFilters()`

**Test-Anforderungen:**
- Unit Test: Context Provider funktioniert
- Unit Test: State Updates korrekt
- Unit Test: localStorage Integration

**Doku-Anforderungen:**
- `frontend/README.md` mit Architektur
- Context-API-Nutzung dokumentiert
- State-Flow-Diagramm

---

### FE-302: API Client Service

**Kategorie:** Frontend  
**Priorisierung:** Hoch  
**Schätzung:** 5 PH  
**Abhängigkeiten:** FE-301, FUNC-201, FUNC-202

**Beschreibung:**
Service-Layer für API-Kommunikation mit Error Handling und TypeScript Types.

**Akzeptanzkriterien:**
- Service in `frontend/src/lib/api.ts`
- TypeScript Interfaces für API-Responses (aus Backend übernehmen)
- Funktionen:
  - `fetchPlaces(filters: FilterOptions, pagination: Pagination): Promise<PlacesResponse>`
  - `fetchPlaceById(id: string): Promise<Place>`
  - `fetchStadtbezirke(): Promise<string[]>`
- Axios oder Fetch API mit:
  - Base URL aus Environment Variable
  - Error Handling & Retry Logic (3x bei Network Error)
  - Request Timeout: 10 Sekunden
  - Response Validation (Zod Schema)
- Error Types:
  ```typescript
  class ApiError extends Error {
    statusCode: number;
    code: string;
  }
  ```

**Test-Anforderungen:**
- Unit Test (Mock): API-Funktionen mit verschiedenen Responses
- Unit Test: Error Handling (404, 500, Network Error)
- Unit Test: Retry Logic funktioniert
- Integration Test: Echte API-Calls (optional)

**Doku-Anforderungen:**
- API Client Nutzung dokumentiert
- Error Handling Strategien
- TypeScript Types exportiert

---

### FE-303: Routing Setup (React Router / Next.js)

**Kategorie:** Frontend  
**Priorisierung:** Hoch  
**Schätzung:** 4 PH  
**Abhängigkeiten:** FE-301

**Beschreibung:**
Next.js App Router Konfiguration mit allen benötigten Routes.

**Akzeptanzkriterien:**
- Routes:
  - `/` - Homepage (Laden-Liste)
  - `/places/[id]` - Detailseite
  - `/about` - Über uns (KI-Transparenz)
  - `/404` - Not Found Page
- Loading States via `loading.tsx`
- Error Handling via `error.tsx`
- Layout mit Sticky Header/Footer (`layout.tsx`)
- Metadata per Route (Basic, keine SEO-Optimierung)
- Navigation zwischen Seiten funktioniert

**Test-Anforderungen:**
- Unit Test: Routing funktioniert
- Test: Back-Button Browser funktioniert
- Test: Direct Links (z.B. /places/xyz) funktionieren

**Doku-Anforderungen:**
- Route-Struktur dokumentiert
- Navigation-Patterns erklärt

---

### FE-304: Toast Notification System

**Kategorie:** Frontend  
**Priorisierung:** Mittel  
**Schätzung:** 4 PH  
**Abhängigkeiten:** FE-301

**Beschreibung:**
Toast Notification System für Fehler und Erfolgsmeldungen.

**Akzeptanzkriterien:**
- Library: react-hot-toast oder Sonner (leichtgewichtig)
- Toast Provider im Root Layout
- Toast-Funktion in Context oder Custom Hook:
  ```typescript
  const { showToast } = useToast();
  showToast('error', 'Failed to load places');
  ```
- Toast Types: success, error, info, warning
- Auto-Dismiss nach 5 Sekunden
- Styling mit Tailwind + DaisyUI
- Position: Top-Right (Desktop), Top-Center (Mobile)
- Max. 3 Toasts gleichzeitig sichtbar

**Test-Anforderungen:**
- Unit Test: Toast wird angezeigt
- Unit Test: Auto-Dismiss funktioniert
- Visual Test: Styling korrekt

**Doku-Anforderungen:**
- Toast-Nutzung dokumentiert
- Wann welcher Toast-Type verwenden

---

### FE-305a: Header-Komponente

**Kategorie:** Frontend  
**Priorisierung:** Hoch  
**Schätzung:** 3 PH  
**Abhängigkeiten:** FE-303

**Beschreibung:**
Sticky Header mit Logo, Navigation und Desktop-Layout.

**Akzeptanzkriterien:**
- Komponente: `Header.tsx` in `components/layout/`
- **Inhalte:**
  - Logo/Titel: "Dönerguide Stuttgart" (Links zu "/")
  - Navigation: Links zu "/" und "/about"
  - Desktop: Horizontal Navigation (Links nebeneinander)
  - Sticky Position (bleibt beim Scrollen sichtbar)
  - Tailwind + DaisyUI Navbar
  - Mobile: Hamburger Icon (triggert Menu in FE-305c)
- **Styling:**
  - Background: DaisyUI Primary Color
  - Text: Kontrastfarbe (weiß/schwarz)
  - Höhe: 64px
  - Shadow beim Scrollen
- Responsive: Desktop & Mobile unterschiedlich (Mobile zeigt Hamburger)

**Test-Anforderungen:**
- Component Test: Header rendert korrekt
- Component Test: Links funktionieren
- Component Test: Sticky Position funktioniert
- Responsive Test: Mobile zeigt Hamburger Icon

**Doku-Anforderungen:**
- Header-Komponente Props dokumentiert
- Styling-Konventionen

---

### FE-305b: Footer-Komponente

**Kategorie:** Frontend  
**Priorisierung:** Mittel  
**Schätzung:** 2 PH  
**Abhängigkeiten:** FE-303

**Beschreibung:**
Footer mit Copyright und Links.

**Akzeptanzkriterien:**
- Komponente: `Footer.tsx` in `components/layout/`
- **Inhalte:**
  - Copyright: "© 2025 Dönerguide Stuttgart"
  - Link zu "/about" ("Über uns")
  - Hinweis: "KI-generierte Bewertungen"
  - Optional: E-Mail für Feedback
- **Styling:**
  - Background: Grau/Neutral
  - Text: Klein (14px)
  - Padding: 20px vertikal
  - Nicht sticky (unten auf Seite)
- Responsive: Mobile & Desktop gleich

**Test-Anforderungen:**
- Component Test: Footer rendert
- Component Test: Links funktionieren

**Doku-Anforderungen:**
- Footer-Content-Management (wie ändern)

---

### FE-305c: Hamburger Menu (Mobile Navigation)

**Kategorie:** Frontend  
**Priorisierung:** Hoch  
**Schätzung:** 3 PH  
**Abhängigkeiten:** FE-305a

**Beschreibung:**
Hamburger Menu für Mobile Navigation mit Slide-In Drawer.

**Akzeptanzkriterien:**
- Komponente: `MobileMenu.tsx` in `components/layout/`
- **Hamburger Icon:**
  - 3 horizontale Striche
  - Position: Rechts oben im Header
  - Nur auf Mobile sichtbar (<768px)
- **Slide-In Drawer:**
  - Von links einsliden (Animation)
  - Overlay: Semi-transparent Background
  - Drawer-Breite: 75% des Viewports (max. 300px)
  - Inhalte:
    - Logo/Titel oben
    - Navigation Links: "Home", "Über uns"
    - Close Button (X) oben rechts
  - Schließen via:
    - Close Button
    - Click Outside (Overlay)
    - ESC-Taste
- State Management: Open/Closed State in Header oder Context
- Accessibility: Keyboard Navigation, Focus Trap

**Test-Anforderungen:**
- Component Test: Menu öffnet/schließt
- Component Test: Overlay Click schließt Menu
- Component Test: ESC schließt Menu
- Accessibility Test: Keyboard Navigation

**Doku-Anforderungen:**
- Mobile Navigation Pattern dokumentiert
- State-Management erklärt

---

### FE-306: Laden-Liste Komponente

**Kategorie:** Frontend  
**Priorisierung:** Hoch  
**Schätzung:** 10 PH  
**Abhängigkeiten:** FE-301, FE-302, FE-308, FE-305c

**Beschreibung:**
Hauptansicht mit Laden-Liste (1 Spalte), Skeleton Screens und Load More Button.

**Akzeptanzkriterien:**
- Komponente: `PlacesList.tsx`
- Layout: 1 Spalte (Desktop & Mobile)
- Initial Load: 20 Läden
- Zeigt für jeden Laden: `PlaceCard` Komponente
- **Load More Button:**
  - Erscheint nur wenn `hasMore === true`
  - Lädt weitere 20 Läden beim Klick
  - Loading State: Button disabled + Spinner
- **Empty State:**
  - Wenn keine Läden gefunden: "Keine Dönerläden gefunden"
  - CTA: "Filter zurücksetzen" Button
- **Error State:**
  - Bei API-Fehler: Toast Notification + Retry Button
- **Skeleton Screens:**
  - Während Initial Load: 5 Skeleton Cards
  - Während Load More: Spinner am Ende der Liste
- Smooth Scroll nach Load More (zu ersten neuen Eintrag)

**Test-Anforderungen:**
- Component Test: Liste rendert korrekt
- Component Test: Load More funktioniert
- Component Test: Empty State
- Component Test: Error State
- Integration Test: API Integration

**Doku-Anforderungen:**
- Komponenten-API dokumentiert
- State-Handling erklärt

---

### FE-307a: Multi-Select Stadtbezirk-Filter

**Kategorie:** Frontend  
**Priorisierung:** Hoch  
**Schätzung:** 5 PH  
**Abhängigkeiten:** FE-301, FE-302

**Beschreibung:**
Multi-Select Dropdown für Stadtbezirk-Filter.

**Akzeptanzkriterien:**
- Komponente: `StadtbezirkFilter.tsx` in `components/places/`
- **Multi-Select Dropdown:**
  - Library: react-select oder DaisyUI Multi-Select
  - Lädt Stadtbezirke von API: `fetchStadtbezirke()`
  - Mehrfachauswahl möglich (Checkboxes)
  - Zeigt Anzahl ausgewählter Bezirke (z.B. "3 ausgewählt")
  - Placeholder: "Stadtbezirk wählen..."
- **Zusatz-Features:**
  - "Alle auswählen" Button
  - "Keine auswählen" / "Zurücksetzen" Button
  - Suche/Filter innerhalb Dropdown (bei vielen Bezirken)
- State-Management: Verwendet FilterContext
- Responsive: Mobile-optimiert (Touch-friendly)

**Test-Anforderungen:**
- Component Test: Dropdown rendert
- Component Test: Mehrfachauswahl funktioniert
- Component Test: "Alle auswählen" funktioniert
- Component Test: State-Update in Context
- Integration Test: API-Aufruf fetchStadtbezirke()

**Doku-Anforderungen:**
- Komponenten-API dokumentiert
- Library-Wahl begründet (react-select vs. DaisyUI)

---

### FE-307b: Sortierung-Dropdown

**Kategorie:** Frontend  
**Priorisierung:** Hoch  
**Schätzung:** 3 PH  
**Abhängigkeiten:** FE-301

**Beschreibung:**
Dropdown-Komponente für Sortierung nach verschiedenen Scores.

**Akzeptanzkriterien:**
- Komponente: `SortSelector.tsx` in `components/places/`
- **Dropdown mit Optionen:**
  - "Gesamtbewertung (Beste zuerst)" [default]
  - "Gesamtbewertung (Schlechteste zuerst)"
  - "Geschmack (Beste zuerst)"
  - "Belag (Bester zuerst)"
  - "Preis-Leistung (Beste zuerst)"
- Mapping zu API-Parametern:
  - sortBy: 'overall' | 'taste' | 'toppings' | 'priceValue'
  - sortOrder: 'asc' | 'desc'
- State-Management: Verwendet FilterContext
- Styling: DaisyUI Select oder Custom Dropdown
- Icon: Sortier-Icon (↑↓)

**Test-Anforderungen:**
- Component Test: Dropdown rendert alle Optionen
- Component Test: Auswahl ändert State
- Component Test: Default-Wert korrekt

**Doku-Anforderungen:**
- Sortier-Optionen dokumentiert
- API-Mapping erklärt

---

### FE-307c: Filter-Actions (Apply & Reset)

**Kategorie:** Frontend  
**Priorisierung:** Hoch  
**Schätzung:** 2 PH  
**Abhängigkeiten:** FE-307a, FE-307b

**Beschreibung:**
Action-Buttons für Filter anwenden und zurücksetzen, Integration der Filter-Komponenten.

**Akzeptanzkriterien:**
- Container-Komponente: `PlacesFilter.tsx` in `components/places/`
- **Layout:**
  - Integriert StadtbezirkFilter (FE-307a) und SortSelector (FE-307b)
  - Desktop: Horizontal nebeneinander
  - Mobile: Vertikal gestapelt, ggf. Collapsible
- **Action-Buttons:**
  - "Filter anwenden" Button (Primary)
    - Triggert neue API-Abfrage mit aktuellen Filtern
    - Loading State während API-Call
  - "Zurücksetzen" Button (Secondary)
    - Setzt Filter auf Defaults (alle Bezirke, Sortierung: overall desc)
    - Triggert API-Abfrage
- **Optionale Features:**
  - Filter-State in URL-Query-Params speichern (für Share-Links)
  - Anzahl Ergebnisse anzeigen ("42 Läden gefunden")
- Verwendet FilterContext und PlacesContext

**Test-Anforderungen:**
- Component Test: Filter-Container rendert Sub-Komponenten
- Component Test: "Anwenden" triggert API-Call
- Component Test: "Zurücksetzen" setzt Defaults
- Integration Test: Filter + API funktioniert End-to-End
- Responsive Test: Layout auf Mobile & Desktop

**Doku-Anforderungen:**
- Filter-Komponenten-Hierarchie
- State-Flow dokumentiert
- URL-Params-Strategie (falls implementiert)

---

### FE-308: Laden-Card Komponente (für Liste)

**Kategorie:** Frontend  
**Priorisierung:** Hoch  
**Schätzung:** 8 PH  
**Abhängigkeiten:** FE-301

**Beschreibung:**
Card-Komponente für einzelnen Laden in der Liste (1 Spalte Layout).

**Akzeptanzkriterien:**
- Komponente: `PlaceCard.tsx`
- Layout: Horizontal (Bild links, Infos rechts)
- **Inhalte:**
  - Thumbnail-Bild (300x300px, object-fit: cover)
  - Name (groß, bold)
  - Stadtbezirk (klein, grau)
  - Gesamt-Score (großer Badge, farblich: Grün >7, Gelb 5-7, Rot <5)
  - Kurztext: Erste 100 Zeichen des Review-Texts + "..." 
  - "Details ansehen" Button/Link
- **Hover/Click:**
  - Hover: Leichter Schatten, Scale-Effekt
  - Click: Navigiert zu `/places/[id]`
- **Responsive:**
  - Desktop: Bild 200px Breite, Rest Infos
  - Mobile: Bild kleiner (100px), Text angepasst
- Accessibility: Klickbare Card (nicht nur Button)

**Test-Anforderungen:**
- Component Test: Card rendert alle Infos
- Component Test: Link funktioniert
- Snapshot Test: Visual Regression
- Accessibility Test: Keyboard Navigation

**Doku-Anforderungen:**
- Komponenten-Props dokumentiert
- Design-Tokens (Farben, Spacing)

---

### FE-309a: Detailseite Layout & API Integration

**Kategorie:** Frontend  
**Priorisierung:** Hoch  
**Schätzung:** 4 PH  
**Abhängigkeiten:** FE-302, FE-303

**Beschreibung:**
Grundstruktur der Detailseite mit API-Integration und Layout.

**Akzeptanzkriterien:**
- Page: `/app/places/[id]/page.tsx` (Next.js App Router)
- **API-Integration:**
  - Lädt Place-Daten: `fetchPlaceById(id)`
  - Loading State: Skeleton Screen (aus FE-310)
  - Error State: 404 Page wenn Place nicht existiert
- **Layout-Struktur:**
  - Hero Section:
    - Name (h1, groß)
    - Adresse (grau, kleiner)
    - Stadtbezirk (Badge)
  - Bild-Bereich (Placeholder, wird in FE-309b gefüllt)
  - Score-Bereich (Placeholder, wird in FE-309c gefüllt)
  - Bewertungstext:
    - Vollständiger AI-Review Text
    - Hinweis: "🤖 KI-generierte Bewertung" (klein, grau)
  - Back Button: "← Zurück zur Liste"
- **Responsive:**
  - Mobile: Fullwidth
  - Desktop: Max-Width Container (1024px)
- **Meta-Tags (Basic, keine SEO-Optimierung):**
  - title: `{place.name} - Dönerguide Stuttgart`

**Test-Anforderungen:**
- Component Test: Seite rendert Place-Daten
- Component Test: API-Integration funktioniert
- Component Test: Loading State zeigt Skeleton
- Component Test: 404 bei ungültiger ID
- Component Test: Back Button navigiert zurück

**Doku-Anforderungen:**
- Detailseite-Struktur dokumentiert
- API-Integration erklärt

---

### FE-309b: Bilder-Carousel-Komponente

**Kategorie:** Frontend  
**Priorisierung:** Hoch  
**Schätzung:** 5 PH  
**Abhängigkeiten:** FE-309a

**Beschreibung:**
Responsive Bilder-Carousel für Food & Ambience Bilder mit Touch-Support.

**Akzeptanzkriterien:**
- Komponente: `ImageCarousel.tsx` in `components/places/`
- **Library-Wahl:**
  - Swiper.js oder Embla Carousel (React-kompatibel)
  - Leichtgewichtig, Touch-optimiert
- **Features:**
  - Zeigt alle Food & Ambience Bilder
  - Navigation: Pfeile Links/Rechts
  - Pagination: Dots unten (1/5, 2/5, etc.)
  - Touch-Swipe auf Mobile
  - Keyboard Navigation (← →)
  - Optional: Thumbnails-Leiste unten
  - Optional: Fullscreen-Modus beim Klick (Lightbox)
- **Bildoptimierung:**
  - Lazy Loading
  - Responsive Images (srcset)
  - Fallback wenn keine Bilder: Placeholder-Image
- **Styling:**
  - Border-Radius
  - Shadow
  - Aspect Ratio: 16:9 oder Square
- Accessibility: Alt-Texte, ARIA-Labels

**Test-Anforderungen:**
- Component Test: Carousel rendert alle Bilder
- Component Test: Navigation funktioniert (Pfeile, Dots)
- Component Test: Touch-Swipe (simuliert)
- Component Test: Keyboard Navigation
- Component Test: Placeholder bei leeren Bildern
- Accessibility Test: Screen Reader kompatibel

**Doku-Anforderungen:**
- Carousel Library dokumentiert
- Props-API
- Accessibility Features

---

### FE-309c: Score-Badges-Komponente

**Kategorie:** Frontend  
**Priorisierung:** Hoch  
**Schätzung:** 3 PH  
**Abhängigkeiten:** FE-309a

**Beschreibung:**
Visuelle Darstellung der 4 Bewertungs-Scores mit Badges/Progress Bars.

**Akzeptanzkriterien:**
- Komponente: `ScoreBadges.tsx` in `components/places/`
- **4 Scores anzeigen:**
  - Geschmack (taste)
  - Belag (toppings)
  - Preis-Leistung (priceValue)
  - Gesamt (overall)
- **Visuelle Darstellung (wählbar):**
  - **Option A:** Progress Bars (1-10 Skala)
  - **Option B:** Große Badge-Zahlen mit Hintergrund-Farbe
  - **Option C:** Kombination (Badge + Bar)
- **Farbcodierung:**
  - Grün: Score >7
  - Gelb: Score 5-7
  - Rot: Score <5
- **Layout:**
  - Desktop: 4 Scores horizontal nebeneinander
  - Mobile: 2x2 Grid oder vertikal gestapelt
- Styling: DaisyUI Badges oder Custom mit Tailwind
- Icon pro Score (optional): 🍖 Geschmack, 🥗 Belag, 💰 Preis, ⭐ Gesamt

**Test-Anforderungen:**
- Component Test: Alle 4 Scores rendern
- Component Test: Farbcodierung korrekt (>7 grün, etc.)
- Component Test: Responsive Layout
- Snapshot Test: Visual Regression

**Doku-Anforderungen:**
- Score-Darstellung dokumentiert
- Farbschema erklärt
- Design-Entscheidung (Option A/B/C)

---

### FE-310: Skeleton Screens

**Kategorie:** Frontend  
**Priorisierung:** Mittel  
**Schätzung:** 5 PH  
**Abhängigkeiten:** FE-306, FE-309a

**Beschreibung:**
Skeleton Screen Komponenten für Loading States.

**Akzeptanzkriterien:**
- Komponenten:
  - `PlaceCardSkeleton.tsx` (für Liste)
  - `PlaceDetailSkeleton.tsx` (für Detailseite)
- Skeleton Design:
  - Graue Boxen mit Shimmer-Animation (Tailwind)
  - Gleiche Dimensionen wie echte Komponenten
  - Accessibility: aria-label="Loading..."
- Verwendung:
  - In `PlacesList` während Initial Load (5 Skeletons)
  - In Detailseite während Laden
- Library: react-loading-skeleton oder Custom mit Tailwind

**Test-Anforderungen:**
- Component Test: Skeletons rendern
- Snapshot Test: Visual Regression

**Doku-Anforderungen:**
- Skeleton Pattern dokumentiert
- Wann welche Skeleton verwenden

---

### FE-311: "Über uns" Seite (KI-Transparenz)

**Kategorie:** Frontend  
**Priorisierung:** Mittel  
**Schätzung:** 6 PH  
**Abhängigkeiten:** FE-303

**Beschreibung:**
Statische Seite mit Projektbeschreibung und KI-Transparenz-Hinweis (NFR-02).

**Akzeptanzkriterien:**
- Route: `/about`
- **Inhalte:**
  - **Überschrift:** "Über Dönerguide Stuttgart"
  - **Projekt-Beschreibung:**
    - Was ist Dönerguide? (ca. 2-3 Absätze)
    - Ziel: Beste Dönerläden in Stuttgart finden
  - **KI-Transparenz:**
    - Abschnitt: "Wie funktionieren die Bewertungen?"
    - Erklärung: KI-generierte Bewertungen basierend auf Bildern
    - Klarstellung: Keine menschlichen Reviews
    - Hinweis: Bewertungen können subjektiv sein
  - **Team/Hochschule:**
    - Hinweis: Hochschulprojekt (Masterkurs System Engineering)
    - Optional: Team-Namen (anonymisierbar)
  - **Kontakt:** (optional)
    - E-Mail oder GitHub Link
  - **Technologie:** (optional)
    - Kurze Auflistung: Next.js, Azure, Google Places API, etc.
- Styling: Lesbare Typografie, max-width für Text (700px)
- Responsive: Mobile-optimiert

**Test-Anforderungen:**
- Component Test: Seite rendert
- Snapshot Test: Content korrekt

**Doku-Anforderungen:**
- Content Management: Wie Text ändern
- Design Guidelines für statische Seiten

---

### FE-312: 404 und Error-Seiten

**Kategorie:** Frontend  
**Priorisierung:** Niedrig  
**Schätzung:** 4 PH  
**Abhängigkeiten:** FE-303

**Beschreibung:**
Custom 404 Not Found Seite und generische Error Page.

**Akzeptanzkriterien:**
- **404 Page:**
  - Route: Next.js `not-found.tsx`
  - Titel: "404 - Seite nicht gefunden"
  - Text: "Diese Seite existiert nicht."
  - CTA: "Zurück zur Startseite" Button
  - Illustration oder Icon (optional)
- **Error Page:**
  - Route: Next.js `error.tsx`
  - Titel: "Etwas ist schief gelaufen"
  - Text: "Bitte versuche es später erneut."
  - CTA: "Neu laden" Button
  - Fehler-Details in Dev-Modus sichtbar
- Styling konsistent mit Rest der App
- Responsive

**Test-Anforderungen:**
- Component Test: Seiten rendern
- Test: 404 wird bei ungültiger Route gezeigt
- Test: Error Page bei Fehler

**Doku-Anforderungen:**
- Error Handling Pattern dokumentiert

---

### TEST-301: Component Unit Tests

**Kategorie:** Testing  
**Priorisierung:** Mittel  
**Schätzung:** 12 PH  
**Abhängigkeiten:** Alle FE-Tasks

**Beschreibung:**
Unit Tests für alle Frontend-Komponenten mit Vitest + React Testing Library.

**Akzeptanzkriterien:**
- Test-Framework: Vitest + React Testing Library
- Test Coverage: Mind. 70% für Komponenten
- Tests für:
  - Layout-Komponenten (Header, Footer, Navigation)
  - PlaceCard, PlacesList
  - Filter-Komponente
  - Detailseite
  - Skeleton Screens
  - API Client (Mocked)
  - Context (PlacesContext, FilterContext)
- Test-Patterns:
  - Rendering Tests
  - User Interaction Tests (Click, Input)
  - State Management Tests
  - API Integration Tests (Mocked)
- Mock Data: Fixtures in `/tests/fixtures/`
- Test-Befehl: `npm run test`

**Test-Anforderungen:**
- Alle Tests grün
- Coverage Report generieren
- CI/CD Integration (Tests laufen bei jedem Push)

**Doku-Anforderungen:**
- Testing-Guide in `frontend/README.md`
- Mock-Data-Struktur dokumentiert
- Wie neue Tests schreiben

---

### TEST-302: Mobile Responsiveness Tests

**Kategorie:** Testing  
**Priorisierung:** Mittel  
**Schätzung:** 6 PH  
**Abhängigkeiten:** Alle FE-Tasks

**Beschreibung:**
Manuelle und automatisierte Tests für Mobile Responsiveness (NFR-01).

**Akzeptanzkriterien:**
- **Manuelle Tests:**
  - Test auf echten Geräten (iOS, Android)
  - Test in Browser DevTools (verschiedene Viewports)
  - Checkliste:
    - Text lesbar auf Mobile (min. 16px)
    - Buttons tappable (min. 44x44px Touch Targets)
    - Hamburger Menu funktioniert
    - Bilder laden optimiert (Thumbnails)
    - Carousel Swipe funktioniert
    - Load More Button gut erreichbar
    - Keine horizontalen Scrollbars
- **Responsive Breakpoints:**
  - Mobile: 320px - 767px
  - Tablet: 768px - 1023px
  - Desktop: 1024px+
- **Performance:**
  - Lighthouse Mobile Score > 80
  - First Contentful Paint < 2s
- Test-Dokumentation: Screenshots verschiedener Viewports

**Test-Anforderungen:**
- Test-Protokoll mit Screenshots
- Lighthouse Report

**Doku-Anforderungen:**
- Responsive Design Guidelines
- Breakpoints dokumentiert
- Mobile-Optimierungen erklärt

---

### DEVOPS-301: Static Web App Deployment Integration

**Kategorie:** DevOps  
**Priorisierung:** Hoch  
**Schätzung:** 6 PH  
**Abhängigkeiten:** INFRA-301, DEVOPS-002

**Beschreibung:**
GitHub Actions Workflow für automatisches Deployment zu Azure Static Web Apps.

**Akzeptanzkriterien:**
- Workflow `.github/workflows/deploy.yml` erweitern (oder neuer Workflow)
- Deployment-Steps:
  - Checkout Code
  - Setup Node.js
  - `cd frontend && npm ci`
  - `npm run build` (Next.js Build)
  - Deploy zu Azure Static Web App via `azure/static-web-apps-deploy@v1` Action
  - Environment Variables setzen:
    - `NEXT_PUBLIC_API_BASE_URL=/api`
- Deployment triggert bei:
  - Push auf `main` (Production)
  - Pull Request (Preview Deployment)
- Preview Deployments:
  - Jeder PR bekommt eigene Preview-URL
  - Kommentar im PR mit Preview-Link
- Health Check nach Deployment:
  - Verifiziere: Homepage lädt (HTTP 200)
  - Verifiziere: API-Proxy funktioniert
- Rollback bei fehlgeschlagenem Health Check

**Test-Anforderungen:**
- Dry-Run Deployment
- Preview Deployment testen
- Health Check funktioniert

**Doku-Anforderungen:**
- `docs/ci-cd.md` erweitern
- Static Web App Deployment dokumentiert
- Preview Deployments erklärt
- Troubleshooting

---

### DOC-301: Frontend-Entwickler-Dokumentation

**Kategorie:** Documentation  
**Priorisierung:** Mittel  
**Schätzung:** 6 PH  
**Abhängigkeiten:** Alle FE-Tasks

**Beschreibung:**
Umfassende Dokumentation für Frontend-Entwicklung und -Wartung.

**Akzeptanzkriterien:**
- Dokument: `frontend/README.md` (erweitern)
- Inhalte:
  - **Getting Started:**
    - Lokales Setup (npm install, npm run dev)
    - Environment Variables
    - API-Verbindung konfigurieren
  - **Architektur:**
    - Verzeichnisstruktur erklärt
    - Context API Pattern
    - API Client Nutzung
  - **Komponenten:**
    - Liste aller Komponenten mit Beschreibung
    - Props-Dokumentation
    - Usage-Beispiele
  - **Styling:**
    - Tailwind + DaisyUI Konventionen
    - Custom Styles (wo/wann verwenden)
    - Design Tokens (Farben, Spacing)
  - **Testing:**
    - Wie Tests schreiben
    - Wie Tests ausführen
  - **Deployment:**
    - Build-Prozess
    - Preview Deployments
  - **Troubleshooting:**
    - Häufige Probleme und Lösungen

**Test-Anforderungen:**
- Review durch Team
- Neuer Entwickler kann Setup durchführen

**Doku-Anforderungen:**
- Verständliche Sprache
- Code-Beispiele
- Screenshots (optional)

---

### DOC-302: Component Library Documentation

**Kategorie:** Documentation  
**Priorisierung:** Niedrig  
**Schätzung:** 4 PH  
**Abhängigkeiten:** Alle FE-Tasks

**Beschreibung:**
Interaktive Component Library mit Storybook (optional) oder Markdown-Dokumentation.

**Akzeptanzkriterien:**
- **Option A (Empfohlen für MVP):** Markdown-Dokumentation
  - Dokument: `frontend/docs/components.md`
  - Für jede Komponente:
    - Name und Beschreibung
    - Props-Tabelle
    - Usage-Beispiel (Code)
    - Screenshot oder Link zu Live-Beispiel
- **Option B (Nice-to-have):** Storybook Setup
  - Storybook installiert
  - Stories für Hauptkomponenten
  - Läuft lokal: `npm run storybook`
  - Deployed zu separater Static Web App (optional)
- Dokumentiert:
  - PlaceCard, PlacesList, Filter, Detailseite
  - Layout-Komponenten
  - Skeleton Screens

**Test-Anforderungen:**
- Dokumentation vollständig
- Beispiele funktionieren

**Doku-Anforderungen:**
- Component Props vollständig
- Beispiele für alle Varianten

---

## Milestone 3 - Zusammenfassung

**Gesamt-Tasks:** 24  
**Gesamt-Schätzung:** 128 Personenstunden  
**Prioritäten:**
- Hoch: 14 Tasks (102 PH)
- Mittel: 9 Tasks (22 PH)
- Niedrig: 1 Task (4 PH)

**Kritischer Pfad:**
1. INFRA-301 (Static Web App)
2. FE-301 (Context) → FE-302 (API Client) → FE-303 (Routing)
3. FE-304 (Toast) parallel zu FE-305a/b/c (Layout-Komponenten)
4. FE-308 (PlaceCard) → FE-306 (Liste) parallel zu FE-307a/b/c (Filter)
5. FE-309a (Detailseite Layout) → FE-309b (Carousel) + FE-309c (Scores) parallel
6. FE-310 (Skeletons), FE-311 (Über uns), FE-312 (Error Pages) parallel
7. TEST-301, TEST-302 nach Frontend-Komponenten
8. DEVOPS-301 parallel zu Tests
9. Doku am Ende

**Parallelisierungs-Möglichkeiten:**
- FE-304, FE-305a, FE-305b parallel zu FE-302, FE-303
- FE-307a, FE-307b parallel entwickeln
- FE-309b, FE-309c parallel nach FE-309a
- FE-306 (Liste) parallel zu FE-307c (Filter Actions)
- FE-310, FE-311, FE-312 parallel (3 Entwickler)
- Tests und DevOps parallel

**Vorteile der Unterteilung:**
- ✅ Mehr kleine Tasks (durchschnittlich 5-6 PH)
- ✅ Bessere Parallelisierung auf 7 Personen
- ✅ Häufigere "Done"-Erfolge
- ✅ Klarere Verantwortlichkeiten
- ✅ Einfacheres Code Review (kleinere PRs)

**Frontend Features MVP:**
- ✅ Responsive Listen-Ansicht (1 Spalte)
- ✅ Multi-Select Stadtbezirk-Filter (mit "Alle auswählen")
- ✅ Sortierung nach Scores (5 Optionen)
- ✅ Filter Apply & Reset Buttons
- ✅ Load More Button (Pagination)
- ✅ PlaceCard (horizontal Layout mit Thumbnail)
- ✅ Detailseite mit kompletter Struktur
- ✅ Bilder-Carousel (Touch-Swipe, Keyboard Navigation)
- ✅ Score-Badges (4 Scores, farbcodiert)
- ✅ Skeleton Screens (Liste & Detailseite)
- ✅ Toast Notifications
- ✅ Sticky Header + Hamburger Menu
- ✅ Footer mit KI-Hinweis
- ✅ "Über uns" Seite (KI-Transparenz)
- ✅ 404 & Error Pages
- ✅ Mobile-optimiert
- ⚠️ Keine SEO/Meta Tags (später)
- ⚠️ Keine Admin-Funktionen (später)

**Ready for Milestone 4:**
- ✅ Funktionales Frontend deployed auf Azure Static Web Apps
- ✅ Alle MVP-Features implementiert (F-05 bis F-08)
- ✅ Mobile Responsiveness (NFR-01)
- ✅ KI-Transparenz (NFR-02)
- ✅ Component Tests grün
- ✅ Automatisches Deployment

---

**Status:** ⏳ Bereit für Review  
**Nächster Milestone:** Milestone 4 - Integration & Launch

---

## Milestone 4: Integration & Launch

**Ziel:** End-to-End-Integration, Performance-Optimierung, Security-Review und Production-Launch

**Geschätzte Dauer:** 2 Wochen  
**Geschätzter Aufwand:** 78 Personenstunden  
**Team-Kapazität:** 35-70 PH/Woche bei 7 Personen

---

### TEST-401: Manuelles End-to-End Testing

**Kategorie:** Testing  
**Priorisierung:** Hoch  
**Schätzung:** 10 PH  
**Abhängigkeiten:** Alle Milestones 0-3 abgeschlossen

**Beschreibung:**
Umfassendes manuelles E2E-Testing der gesamten Anwendung mit detaillierter Checkliste.

**Akzeptanzkriterien:**
- Test-Dokument: `docs/e2e-test-checklist.md`
- **Test-Szenarien (Happy Path):**
  1. **Data Pipeline:**
     - PlaceSearchFunction läuft automatisch
     - Neue Läden werden gefunden
     - Bilder werden klassifiziert
     - AI-Reviews werden generiert
     - Daten landen in Cosmos DB
  2. **API:**
     - GET /api/places liefert Läden
     - Filter nach Stadtbezirk funktioniert
     - Sortierung funktioniert
     - Pagination funktioniert
     - GET /api/places/{id} liefert Details
  3. **Frontend:**
     - Homepage lädt und zeigt Läden
     - Filter funktionieren (Multi-Select)
     - Sortierung funktioniert
     - Load More Button lädt weitere Läden
     - Klick auf Laden öffnet Detailseite
     - Detailseite zeigt alle Infos + Carousel
     - Navigation funktioniert (Header, Back Button)
     - "Über uns" Seite ist erreichbar
  4. **Mobile:**
     - Hamburger Menu funktioniert
     - Touch-Gesten (Swipe im Carousel)
     - Text lesbar, Buttons tappable
     - Keine Layout-Probleme
- **Test-Szenarien (Edge Cases):**
  - Keine Ergebnisse nach Filter → Empty State
  - Ungültige Place-ID → 404 Seite
  - API-Fehler → Toast Notification + Retry
  - Sehr lange Laden-Namen → Layout bricht nicht
  - Läden ohne Bilder → Placeholder
- **Browser-Testing:**
  - Chrome, Firefox, Safari, Edge
  - Mobile: iOS Safari, Android Chrome
- **Performance:**
  - Lighthouse Tests (siehe PERF-401)
- Test-Protokoll mit Screenshots und Ergebnissen
- Bug-Liste mit Severity (Critical, High, Medium, Low)

**Test-Anforderungen:**
- Alle Critical & High Bugs gefixt
- Medium Bugs dokumentiert (optional fixen)
- Test-Protokoll vollständig

**Doku-Anforderungen:**
- E2E-Test-Checkliste
- Test-Protokoll mit Ergebnissen
- Bug-Report-Template

---

### PERF-401: Performance-Optimierung & Lighthouse

**Kategorie:** Performance  
**Priorisierung:** Hoch  
**Schätzung:** 12 PH  
**Abhängigkeiten:** Milestone 3 Frontend abgeschlossen

**Beschreibung:**
Performance-Analyse und -Optimierung für Frontend, API und Datenbank.

**Akzeptanzkriterien:**
- **Lighthouse Scores (Ziel: >85 für alle):**
  - Performance: >85
  - Accessibility: >90
  - Best Practices: >90
  - SEO: >80 (ohne Meta-Tags-Optimierung, nur Basics)
- **Frontend-Optimierungen:**
  - Bilder: WebP-Format oder Next.js Image Optimization
  - Lazy Loading für Bilder
  - Code Splitting (Next.js automatisch)
  - Minification (Next.js automatisch)
  - Font-Loading optimiert (font-display: swap)
  - Unused CSS entfernen
- **API-Optimierungen:**
  - Response Compression (gzip/brotli)
  - HTTP Cache Headers korrekt gesetzt (bereits in BE-203)
  - Cosmos DB Queries optimiert (Indexes nutzen)
- **Cosmos DB Query-Optimierung:**
  - Query Execution Stats analysieren (RU-Verbrauch)
  - Composite Indexes für häufige Queries:
    - `stadtbezirk` + `aiReview.scores.overall`
    - `stadtbezirk` + `aiReview.scores.taste`
  - Unnötige Properties aus SELECT entfernen
- **Bilder-Optimierung:**
  - Thumbnails für Liste (300x300px)
  - Full-Size für Detailseite
  - WebP-Konvertierung in ImageService (Milestone 1)
- **Performance-Benchmarks dokumentieren:**
  - First Contentful Paint (FCP): <2s
  - Largest Contentful Paint (LCP): <2.5s
  - Time to Interactive (TTI): <3.5s
  - API Response Time (p95): <500ms
  - Cosmos DB Query Time (p95): <100ms
- Lighthouse Reports in `/docs/performance/`

**Test-Anforderungen:**
- Lighthouse Tests auf Mobile & Desktop
- Performance Tests mit verschiedenen Netzwerkbedingungen (3G, 4G)
- Load Tests (siehe TEST-202)

**Doku-Anforderungen:**
- Performance-Optimierungen dokumentiert
- Before/After Lighthouse Scores
- Empfehlungen für zukünftige Optimierungen

---

### SEC-401: Security Review & Hardening

**Kategorie:** Security  
**Priorisierung:** Hoch  
**Schätzung:** 8 PH  
**Abhängigkeiten:** Alle Milestones 0-3 abgeschlossen

**Beschreibung:**
Umfassende Security-Review mit Checkliste und Härtung der Anwendung.

**Akzeptanzkriterien:**
- Security-Checkliste in `docs/security-review.md`
- **API-Security:**
  - Keine sensiblen Daten in Responses (z.B. keine internen IDs, Keys)
  - CORS korrekt konfiguriert (nur erlaubte Origins)
  - Rate Limiting dokumentiert (aktuell: none, für V2)
  - Input Validation in allen Endpoints
  - SQL Injection Prevention (Cosmos DB Parameterized Queries)
  - Error Messages nicht zu detailliert (keine Stack Traces in Production)
- **Secrets Management:**
  - Alle Secrets in Azure Key Vault
  - Keine Secrets in Code oder Git
  - Managed Identities für Azure-Ressourcen
  - Function Apps haben nur notwendige Permissions (Least Privilege)
- **HTTPS-Only:**
  - Alle Endpoints nur über HTTPS erreichbar
  - HTTP → HTTPS Redirect
  - HSTS Header gesetzt
- **Frontend-Security:**
  - Content Security Policy (CSP) Header
  - X-Frame-Options: DENY
  - X-Content-Type-Options: nosniff
  - Keine eval() oder dangerouslySetInnerHTML
  - Dependencies: npm audit (keine Critical/High Vulnerabilities)
- **Azure Security:**
  - Network Security Groups (NSGs) Review
  - Storage Accounts: Private Access (außer notwendige)
  - Cosmos DB: Firewall Rules (optional)
  - Application Insights: Keine PII logged
- **DSGVO-Compliance:**
  - Kein User-Tracking (erfüllt)
  - Keine personenbezogenen Daten gespeichert
  - IP-Adressen nicht geloggt (Application Insights Config)
- Security Scan mit Tools:
  - npm audit (Frontend & Functions)
  - OWASP ZAP (optional, falls Zeit)
- Security-Report mit Findings und Remediations

**Test-Anforderungen:**
- Alle Critical & High Security Issues gefixt
- Medium Issues dokumentiert
- npm audit clean (oder Exceptions dokumentiert)

**Doku-Anforderungen:**
- Security-Checkliste
- Security-Report
- Security Best Practices für zukünftige Entwicklung

---

### MON-401: Monitoring Dashboard & Alerting Finalisierung

**Kategorie:** DevOps  
**Priorisierung:** Hoch  
**Schätzung:** 10 PH  
**Abhängigkeiten:** INFRA-109 (App Insights), DEVOPS-101

**Beschreibung:**
Finalisierung von Monitoring, Dashboards und Alert-Integration mit Slack/Teams.

**Akzeptanzkriterien:**
- **Azure Dashboard erstellen:**
  - Dashboard Name: "Dönerguide Stuttgart - Production"
  - Widgets:
    - Request Rate (API & Frontend)
    - Response Times (p50, p95, p99)
    - Error Rate (gesamt & pro Endpoint)
    - Pipeline Success Rate
    - Cosmos DB RU Consumption
    - Function App Execution Count
    - Storage Account Usage
  - Dashboard exportiert als JSON in `/docs/monitoring/`
- **Application Insights Workbooks:**
  - Custom Workbook für detaillierte Analysen
  - KQL Queries für häufige Szenarien:
    - Top 10 häufigste Errors
    - Langsamste API-Requests
    - Pipeline Execution History
    - User Journey (Page Views Flow)
- **Alerts konfigurieren:**
  - **Pipeline Alerts:**
    - PlaceSearchFunction nicht gelaufen (24h)
    - Pipeline Error Rate > 10%
    - Dead Letter Queue Messages > 5
  - **API Alerts:**
    - API Response Time (p95) > 1s
    - API Error Rate > 5%
    - API Downtime (HTTP 503/500)
  - **Frontend Alerts:**
    - Frontend Errors > 10/minute
    - Lighthouse Score drop > 10 points
  - **Infrastructure Alerts:**
    - Cosmos DB RU Throttling
    - Function App Cold Start > 5s
    - Storage Account near capacity
  - **Cost Alerts:**
    - Daily Budget Überschreitung (> 1€/day)
    - Monthly Budget Überschreitung (> 20€/month)
- **Slack/Teams Integration:**
  - Action Group erstellt mit Slack/Teams Webhook
  - Alerts werden an Channel gesendet
  - Alert-Format: Severity, Message, Link zu Azure Portal
  - Test-Alert senden
- Alert-Runbook dokumentiert:
  - Für jeden Alert: Was tun? (Troubleshooting Steps)

**Test-Anforderungen:**
- Test: Alerts triggern bei simulierten Fehlern
- Test: Slack/Teams Notification wird empfangen
- Test: Dashboard zeigt Live-Daten

**Doku-Anforderungen:**
- `docs/monitoring-guide.md`
- Dashboard-Setup dokumentiert
- Alert-Runbook (Troubleshooting pro Alert)
- KQL Query Library

---

### OPS-401: Backup & Disaster Recovery Setup

**Kategorie:** Operations  
**Priorisierung:** Mittel  
**Schätzung:** 6 PH  
**Abhängigkeiten:** INFRA-101 (Cosmos DB), INFRA-003 (Terraform State)

**Beschreibung:**
Backup-Strategie für Cosmos DB konfigurieren und Disaster Recovery Plan dokumentieren.

**Akzeptanzkriterien:**
- **Cosmos DB Backup:**
  - Automatische Backups aktiviert (Continuous Backup Mode)
  - Retention: 7 Tage
  - Point-in-Time-Restore möglich
  - Test: Restore-Prozess dokumentiert (nicht durchgeführt)
- **Terraform State Backup:**
  - Terraform State in Azure Storage (bereits in INFRA-003)
  - Versioning aktiviert (letzte 5 Versionen behalten)
  - Backup-Script für lokale Kopie: `scripts/backup-terraform-state.sh`
- **Disaster Recovery Plan:**
  - Dokument: `docs/disaster-recovery-plan.md`
  - Szenarien:
    - Cosmos DB Datenverlust → Restore from Backup
    - Function App Fehler → Rollback Deployment
    - Terraform State Corruption → Restore from Backup
    - Kompletter Azure Outage → Wartezeit (keine Multi-Region)
  - Recovery Time Objective (RTO): <4 Stunden
  - Recovery Point Objective (RPO): <24 Stunden (Pipeline-Daten)
  - Kontaktpersonen und Eskalationsprozess
  - Schritt-für-Schritt Wiederherstellungs-Anleitungen
- Backup-Test-Protokoll (simuliert, nicht tatsächlich restore)

**Test-Anforderungen:**
- Backup-Konfiguration verifizieren
- Restore-Anleitung durchgehen (Dry-Run)

**Doku-Anforderungen:**
- Disaster Recovery Plan vollständig
- Backup-Konfiguration dokumentiert
- Restore-Anleitungen mit Screenshots

---

### LAUNCH-401: Pre-Launch Checklist

**Kategorie:** Operations  
**Priorisierung:** Hoch  
**Schätzung:** 6 PH  
**Abhängigkeiten:** Alle vorherigen Tasks

**Beschreibung:**
Finale Pre-Launch-Checkliste durchgehen und Go/No-Go Entscheidung.

**Akzeptanzkriterien:**
- Checkliste in `docs/launch-checklist.md`
- **Technical Readiness:**
  - [ ] Alle E2E-Tests bestanden (TEST-401)
  - [ ] Lighthouse Scores >85 (PERF-401)
  - [ ] Security Review abgeschlossen (SEC-401)
  - [ ] Monitoring & Alerts aktiv (MON-401)
  - [ ] Backups konfiguriert (OPS-401)
  - [ ] CI/CD Pipelines laufen fehlerfrei
  - [ ] Alle Critical & High Bugs gefixt
- **Content & Documentation:**
  - [ ] "Über uns" Seite mit korrektem Content
  - [ ] E-Mail für Feedback vorhanden
  - [ ] KI-Transparenz-Hinweis auf allen relevanten Seiten
  - [ ] Known Issues dokumentiert (DOC-401)
  - [ ] API-Dokumentation vollständig (DOC-201)
- **Infrastructure:**
  - [ ] Alle Azure-Ressourcen deployed
  - [ ] Function Apps laufen stabil
  - [ ] Static Web App erreichbar
  - [ ] API-Proxy funktioniert
  - [ ] Cosmos DB enthält Daten (mind. 50 Läden)
- **Monitoring:**
  - [ ] Dashboard zeigt Live-Daten
  - [ ] Alerts funktionieren
  - [ ] Slack/Teams Integration aktiv
- **Legal & Compliance:**
  - [ ] Kein User-Tracking (DSGVO)
  - [ ] Impressum/Datenschutz (optional, siehe Hinweis unten)
  - [ ] Google API Terms of Service eingehalten
- **Team Readiness:**
  - [ ] 1 Woche Post-Launch Support geplant
  - [ ] On-Call Rotation definiert (optional)
  - [ ] Runbooks für häufige Probleme vorhanden
- Go/No-Go Meeting:
  - Meeting-Protokoll mit Entscheidung
  - Launch-Datum festgelegt

**Hinweis Impressum/Datenschutz:**
Als Hochschulprojekt ggf. keine rechtliche Verpflichtung, aber empfohlen. Mit Hochschule/Betreuer klären.

**Test-Anforderungen:**
- Alle Checkboxen müssen checked sein für Go

**Doku-Anforderungen:**
- Launch-Checkliste vollständig
- Go/No-Go Meeting-Protokoll

---

### LAUNCH-402: Production Launch

**Kategorie:** Operations  
**Priorisierung:** Hoch  
**Schätzung:** 4 PH  
**Abhängigkeiten:** LAUNCH-401

**Beschreibung:**
Produktiver Launch der Anwendung (Hard Launch - sofort öffentlich).

**Akzeptanzkriterien:**
- **Launch-Day:**
  - Finaler Deployment-Check (alle Services grün)
  - DNS/Custom Domain aktivieren (falls vorhanden, sonst Azure-Domain)
  - Application Insights auf 100% Sampling (volle Sichtbarkeit)
  - Team bereit für Support (alle Teammitglieder online)
- **Launch-Kommunikation:**
  - Announcement vorbereiten:
    - Intern: Team, Hochschule, Betreuer
    - Optional: Social Media (falls gewünscht)
    - Optional: Hochschul-Newsletter
  - Link zur App: `https://[your-app].azurestaticapps.net`
  - Hinweis: MVP, Feedback willkommen
- **Health Monitoring:**
  - Live-Monitoring während Launch-Tag (alle 30 Min Check)
  - Dashboard permanent offen
  - Alerts aktiv überwachen
- **Smoke Tests nach Launch:**
  - Homepage lädt: ✓
  - API funktioniert: ✓
  - 5 Test-Transaktionen durchführen: ✓
  - Mobile funktioniert: ✓
- Launch-Protokoll:
  - Start-Zeit
  - Erste User (aus Logs)
  - Erste Fehler/Issues
  - Erste Feedback-E-Mails

**Test-Anforderungen:**
- Smoke Tests nach Launch erfolgreich

**Doku-Anforderungen:**
- Launch-Protokoll dokumentiert
- Launch-Announcement gespeichert

---

### OPS-402: Post-Launch Monitoring (1 Woche)

**Kategorie:** Operations  
**Priorisierung:** Hoch  
**Schätzung:** 8 PH (über 1 Woche verteilt)  
**Abhängigkeiten:** LAUNCH-402

**Beschreibung:**
Intensive Überwachung für 1 Woche nach Launch mit täglichen Check-ins.

**Akzeptanzkriterien:**
- **Tägliche Check-ins (7 Tage):**
  - Morgen-Check (10 Uhr):
    - Dashboard-Review
    - Alert-Review (neue Alerts?)
    - Error Rate Check
    - User Feedback E-Mails lesen
  - Abend-Check (18 Uhr):
    - Tages-Zusammenfassung
    - Neue Issues dokumentieren
    - Bugfixes priorisieren (Critical sofort, High innerhalb 24h)
- **Metriken tracken:**
  - Total Requests pro Tag
  - Unique Places angezeigt
  - Error Rate (täglich)
  - Response Times (p95)
  - Cosmos DB RU-Verbrauch
  - Azure Costs (täglich)
- **Issue-Tracking:**
  - Alle Issues in GitHub Issues oder Spreadsheet
  - Severity: Critical, High, Medium, Low
  - Status: Open, In Progress, Fixed, Closed
  - Owner assigned
- **Quick Fixes:**
  - Hotfix-Branch bei Critical Issues
  - Deployment via CI/CD (Fast-Track)
  - Verification nach Fix
- **Feedback-Handling:**
  - User-Feedback per E-Mail sammeln
  - Kategorisieren: Bug, Feature Request, Positive
  - Response innerhalb 24h (Danke + Status)
- Wöchentliches Post-Launch-Report:
  - Dokument: `docs/post-launch-week1-report.md`
  - Inhalte:
    - Metriken-Zusammenfassung
    - Issues gefunden und gefixt
    - User Feedback Summary
    - Lessons Learned
    - Empfehlungen für V2

**Test-Anforderungen:**
- Tägliche Check-ins durchgeführt
- Alle Critical Issues gefixt

**Doku-Anforderungen:**
- Tägliche Check-in Protokolle
- Wöchentlicher Post-Launch Report

---

### DOC-401: Known Issues & V2/V3 Roadmap

**Kategorie:** Documentation  
**Priorisierung:** Mittel  
**Schätzung:** 6 PH  
**Abhängigkeiten:** TEST-401, LAUNCH-401

**Beschreibung:**
Dokumentation bekannter Einschränkungen des MVP und Roadmap für V2/V3.

**Akzeptanzkriterien:**
- Dokument: `docs/known-issues-and-roadmap.md`
- **Known Issues / Limitations (MVP):**
  - Nur Stuttgart (keine anderen Städte)
  - Keine Admin-Funktionen (Bearbeiten/Ausblenden von Läden)
  - Keine Karte (Stadtplan mit Pins)
  - Kein Filter "Jetzt geöffnet"
  - Keine Preisklassen-Filter
  - Keine Vegan/Vegetarisch-Filter
  - Keine Textsuche
  - Keine Social Sharing Buttons
  - KI-Bewertungen nur auf Bildern basierend (keine Google-Text-Reviews)
  - Kein Rate Limiting (bei hoher Last potenziell problematisch)
  - Keine Custom Domain (nur Azure-Domain)
  - Keine SEO-Optimierung (Meta Tags)
- **Planned Features (V2 - Should-Have):**
  - F-09: Interaktive Karte mit Pins
  - F-10: Filter "Jetzt geöffnet"
  - F-12: Preisklassen-Filter
  - F-14: Freitextsuche mit Autocomplete
  - F-16: Admin-Dashboard (Bearbeiten/Ausblenden)
  - Custom Domain
  - Rate Limiting
  - Automatisierte E2E-Tests
- **Planned Features (V3 - Could-Have):**
  - F-11: Google-Textbewertungen in KI-Prompt
  - F-13: Vegan/Vegetarisch-Filter
  - F-15: Social Sharing (Share-Buttons)
  - F-17: Städte-Erweiterung (Mannheim, Heidelberg, etc.)
  - Multi-Language Support (EN)
  - PWA (Progressive Web App)
  - User Accounts & Favorites (optional)
- **Technical Debt:**
  - LLM-Provider-Wahl finalisieren (GPT-4o vs. Open Source)
  - Code-Refactoring in einigen Services
  - Test Coverage erhöhen (aktuell ~70%)
- Timeline:
  - V2: Q1 2026 (optional, falls Projekt weitergeführt)
  - V3: Q2 2026 (optional)

**Test-Anforderungen:**
- Review durch Team
- Roadmap realistisch

**Doku-Anforderungen:**
- Transparent und ehrlich
- Priorisierung klar (Should vs. Could)
- Timeline als Schätzung markiert

---

### DOC-402: Final Documentation Review & Cleanup

**Kategorie:** Documentation  
**Priorisierung:** Mittel  
**Schätzung:** 6 PH  
**Abhängigkeiten:** Alle DOC-Tasks

**Beschreibung:**
Finale Review und Cleanup aller Projekt-Dokumentation.

**Akzeptanzkriterien:**
- **Dokumentations-Review:**
  - Alle Dokumente auf Vollständigkeit prüfen
  - Veraltete Infos aktualisieren
  - Broken Links fixen
  - Rechtschreibung/Grammatik Check
  - Konsistente Formatierung
- **Dokumentations-Struktur:**
  ```
  /docs
  ├── README.md                   # Übersicht & Navigation
  ├── architecture.md             # System-Architektur
  ├── tech-stack.md              # Technologie-Stack
  ├── anforderungen.md           # Anforderungen (aus Projekt)
  ├── api-documentation.md       # API Docs
  ├── data-pipeline.md           # Pipeline-Architektur
  ├── monitoring-guide.md        # Monitoring & Alerts
  ├── disaster-recovery-plan.md  # DR Plan
  ├── known-issues-and-roadmap.md # Known Issues & V2/V3
  ├── e2e-test-checklist.md     # Testing
  ├── launch-checklist.md        # Launch
  ├── security-review.md         # Security
  ├── post-launch-week1-report.md # Post-Launch
  ├── diagrams/                  # Architektur-Diagramme
  ├── monitoring/                # Dashboard JSONs
  └── performance/               # Lighthouse Reports
  ```
- **Root README.md Update:**
  - Projekt-Beschreibung
  - Features (MVP)
  - Tech-Stack
  - Getting Started (für neue Entwickler)
  - Links zu wichtigen Docs
  - Hochschul-Projekt-Hinweis
  - Team-Credits
  - Lizenz (falls applicable)
- **Frontend README.md Update:**
  - Setup-Anleitung aktuell
  - Deployment-Prozess dokumentiert
- **Functions README.md Update:**
  - Alle 4 Function Apps dokumentiert
  - Shared Package erklärt
- Alle TODOs aus Code entfernen oder in Issues überführen
- Code-Kommentare Review (veraltete entfernen)

**Test-Anforderungen:**
- Review durch mindestens 2 Personen
- Alle Links funktionieren

**Doku-Anforderungen:**
- Dokumentation vollständig und aktuell
- Navigation klar
- Für externe Leser verständlich

---

### OPS-403: Cost Analysis & Budget Review

**Kategorie:** Operations  
**Priorisierung:** Niedrig  
**Schätzung:** 4 PH  
**Abhängigkeiten:** 1 Woche nach Launch (OPS-402)

**Beschreibung:**
Analyse der tatsächlichen Azure-Kosten und Vergleich mit Budget-Schätzung.

**Akzeptanzkriterien:**
- **Cost Analysis durchführen:**
  - Azure Cost Management & Billing öffnen
  - Kosten pro Service analysieren:
    - Cosmos DB (Serverless RU-Verbrauch)
    - Function Apps (Executions + Compute)
    - Storage Accounts (Blob + Table Storage)
    - Application Insights (Data Ingestion)
    - Static Web App
    - Azure AI Services (OpenAI oder Foundry)
    - Google Places API (extern)
  - Tägliche Kosten (Durchschnitt)
  - Monatliche Hochrechnung
- **Vergleich mit Budget:**
  - Ursprüngliche Schätzung: ~10€/Monat
  - Tatsächliche Kosten: ?
  - Delta analysieren
  - Kostenoptimierungs-Potenziale identifizieren:
    - Zu hohe Cosmos DB RUs? → Query-Optimierung
    - Zu viele Function Executions? → Timer anpassen
    - Zu viel App Insights Data? → Sampling reduzieren
    - LLM-Kosten zu hoch? → Modell wechseln (GPT-4o → Llama)
- **Cost Report:**
  - Dokument: `docs/cost-analysis-week1.md`
  - Inhalte:
    - Kostenaufschlüsselung pro Service
    - Kostenvergleich Budget vs. Actual
    - Empfehlungen für Kostenoptimierung
    - Prognose für Monat 2-3
- **Optimierungen umsetzen (optional):**
  - Falls Budget überschritten: Sofortmaßnahmen
  - Dokumentieren was geändert wurde

**Test-Anforderungen:**
- Cost Analysis vollständig
- Empfehlungen fundiert

**Doku-Anforderungen:**
- Cost Report mit Grafiken (Screenshots aus Azure Portal)
- Optimierungsempfehlungen priorisiert

---

### RETRO-401: Project Retrospective

**Kategorie:** Project Management  
**Priorisierung:** Mittel  
**Schätzung:** 4 PH  
**Abhängigkeiten:** OPS-402 (Post-Launch Woche abgeschlossen)

**Beschreibung:**
Team-Retrospektive nach Projektabschluss für Lessons Learned.

**Akzeptanzkriterien:**
- Retrospektive-Meeting (1.5-2 Stunden)
- **Agenda:**
  1. Was lief gut? (15 Min)
  2. Was lief schlecht? (15 Min)
  3. Was haben wir gelernt? (20 Min)
  4. Was würden wir beim nächsten Mal anders machen? (20 Min)
  5. Action Items für V2 (falls applicable) (20 Min)
- **Themen:**
  - Projektmanagement-Methodik (Kanban/Scrum-Hybrid)
  - Tech-Stack-Entscheidungen
  - Zusammenarbeit im Team
  - Zeitschätzungen (waren sie akkurat?)
  - CI/CD-Pipeline
  - Azure vs. andere Clouds
  - Herausforderungen und Lösungen
- Retrospektive-Protokoll:
  - Dokument: `docs/project-retrospective.md`
  - Inhalte:
    - Zusammenfassung der Diskussion
    - Lessons Learned (Top 5-10)
    - Action Items
    - Team-Feedback
- Optionale Umfrage:
  - Anonymous Team-Survey (Google Forms)
  - Fragen zu Zufriedenheit, Lernerfahrung, etc.

**Test-Anforderungen:**
- Alle Team-Mitglieder nehmen teil

**Doku-Anforderungen:**
- Retrospektive-Protokoll vollständig
- Lessons Learned klar formuliert

---

### DOC-403: Final Project Presentation (für Hochschule)

**Kategorie:** Documentation  
**Priorisierung:** Mittel  
**Schätzung:** 8 PH  
**Abhängigkeiten:** Projekt abgeschlossen

**Beschreibung:**
Präsentation für Hochschul-Abgabe erstellen (falls erforderlich für Masterkurs).

**Akzeptanzkriterien:**
- PowerPoint/Google Slides Präsentation
- **Inhalte:**
  1. Projekt-Übersicht (5 Folien)
     - Motivation & Ziel
     - Features (MVP)
     - Zielgruppe
  2. Architektur (5 Folien)
     - System-Diagramm
     - Data Pipeline
     - Frontend, API, Backend
     - Azure-Services
  3. Technologie-Stack (3 Folien)
     - Frontend: Next.js, Tailwind, DaisyUI
     - Backend: Azure Functions, Cosmos DB, Service Bus
     - AI: Azure OpenAI / Foundry, Azure AI Vision
     - DevOps: GitHub Actions, Terraform
  4. Herausforderungen & Lösungen (5 Folien)
     - Grid-basierte Suche
     - Bildklassifikation
     - LLM-Bewertungen
     - Separate Function Apps
  5. Demo (Live oder Screenshots) (5 Folien)
     - Homepage
     - Filter/Sortierung
     - Detailseite mit Carousel
     - Monitoring Dashboard
  6. Ergebnisse (3 Folien)
     - Anzahl Läden in DB
     - Performance-Metriken
     - User Feedback (falls vorhanden)
  7. Lessons Learned (3 Folien)
     - Was gut lief
     - Was schwierig war
     - Was würden wir anders machen
  8. Ausblick (2 Folien)
     - V2/V3 Roadmap
     - Potenzielle Erweiterungen
- **Zusätzlich:**
  - Live-Demo vorbereiten (falls möglich)
  - Handout (1-2 Seiten) mit Key Facts
  - Video-Demo (optional, 2-3 Min)
- Präsentation in `/docs/presentation/`

**Test-Anforderungen:**
- Probe-Präsentation im Team
- Zeitlimit einhalten (z.B. 20-30 Min)

**Doku-Anforderungen:**
- Präsentation vollständig
- Handout erstellt
- Demo funktioniert

---

## Milestone 4 - Zusammenfassung

**Gesamt-Tasks:** 14  
**Gesamt-Schätzung:** 78 Personenstunden  
**Prioritäten:**
- Hoch: 6 Tasks (54 PH)
- Mittel: 7 Tasks (28 PH)
- Niedrig: 1 Task (4 PH)

**Kritischer Pfad:**
1. TEST-401 (E2E Testing)
2. PERF-401 (Performance) parallel zu SEC-401 (Security)
3. MON-401 (Monitoring) parallel zu OPS-401 (Backup)
4. LAUNCH-401 (Pre-Launch Checklist)
5. LAUNCH-402 (Production Launch)
6. OPS-402 (Post-Launch Monitoring - 1 Woche)
7. Dokumentation, Retrospektive, Präsentation parallel nach Launch

**Parallelisierungs-Möglichkeiten:**
- PERF-401, SEC-401, MON-401, OPS-401 können parallel laufen (4 Personen)
- DOC-401, DOC-402, DOC-403 können parallel laufen nach Launch
- OPS-403, RETRO-401 nach Post-Launch Woche

**Launch-Strategie:**
- ✅ Hard Launch (sofort öffentlich)
- ✅ 1 Woche intensive Überwachung
- ✅ Tägliche Check-ins
- ✅ Schnelle Hotfixes bei Critical Issues
- ✅ User Feedback per E-Mail
- ✅ Kein Analytics/Tracking (DSGVO-konform)

**Ready for Production:**
- ✅ Alle Features getestet (E2E)
- ✅ Performance optimiert (Lighthouse >85)
- ✅ Security gehärtet (Review abgeschlossen)
- ✅ Monitoring aktiv (Dashboard + Alerts)
- ✅ Backups konfiguriert (Cosmos DB + Terraform State)
- ✅ Team bereit für Support (1 Woche)
- ✅ Dokumentation vollständig
- ✅ Known Issues + V2/V3 Roadmap transparent

**Post-Launch:**
- ✅ 1 Woche intensive Überwachung
- ✅ Cost Analysis nach 1 Woche
- ✅ Retrospektive durchführen
- ✅ Final Presentation für Hochschule

---

## 🎉 Projekt Backlog - Komplett!

**Gesamt-Übersicht aller Milestones:**

| Milestone | Tasks | Personenstunden | Dauer |
|-----------|-------|-----------------|-------|
| **M0: Setup & Foundations** | 17 | 71 PH | 2 Wochen |
| **M1: Data Pipeline** | 27 | 195 PH | 3-5 Wochen |
| **M2: API Layer** | 13 | 62 PH | 2 Wochen |
| **M3: Frontend MVP** | 24 | 128 PH | 3-4 Wochen |
| **M4: Integration & Launch** | 14 | 78 PH | 2 Wochen |
| **GESAMT** | **95** | **534 PH** | **12-15 Wochen** |

**Mit Team-Kapazität (35-70 PH/Woche):**
- **Optimistisch (70 PH/Woche):** ~8 Wochen
- **Realistisch (52 PH/Woche):** ~10 Wochen
- **Konservativ (35 PH/Woche):** ~15 Wochen

**Euer Ziel: 3 Monate (12-13 Wochen)** → ✅ **Realistisch erreichbar!**

**Verbesserungen durch Unterteilung:**
- ✅ Frontend-Tasks feingranularer (24 statt 18)
- ✅ Durchschnittliche Task-Größe: ~5-6 PH (optimal)
- ✅ Bessere Parallelisierung auf 7 Personen
- ✅ Klarere Verantwortlichkeiten

---

**Status:** ✅ Backlog vollständig  
**Bereit für:** Projektstart! 🚀
