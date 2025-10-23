# **MVP-Anforderungen**

**Anforderungs-ID:** F-01
**Titel:** Dönerläden über Google Maps API finden
**User Story:** Als System möchte ich für einen gegebenen Ort (Landkreis Stuttgart) alle Dönerläden über die Google Maps API finden, um eine vollständige Liste potenzieller Läden zu haben.
**Priorität:** MVP
**Akzeptanzkriterien:**
*   Die Suche wird erfolgreich über die API ausgeführt.
*   Das Ergebnis ist eine Liste von Orten, die als Dönerladen identifiziert wurden.
*   Für jeden Ort werden mindestens Name, Adresse und Google Place ID gespeichert.

---
**Anforderungs-ID:** F-02
**Titel:** Bilder für einen Laden herunterladen & klassifizieren
**User Story:** Als System möchte ich für jeden gefundenen Dönerladen die zugehörigen Bilder herunterladen und in die Kategorien 'Essen' und 'Ambiente' klassifizieren, um die Grundlage für die KI-Analyse zu schaffen.
**Priorität:** MVP
**Akzeptanzkriterien:**
*   Die Bilder eines Ladens werden erfolgreich heruntergeladen.
*   Jedes Bild wird an den Azure AI Vision Service gesendet.
*   Basierend auf den zurückgegebenen Tags wird jedes Bild als 'Essen', 'Ambiente' oder 'Irrelevant' markiert.
*   Nur die URLs der 'Essen'- und 'Ambiente'-Bilder werden für den nächsten Schritt gespeichert.

---
**Anforderungs-ID:** F-03
**Titel:** KI-Bewertung generieren
**User Story:** Als System möchte ich die klassifizierten Bilder an ein LLM senden, um einen Bewertungstext sowie Scores für Geschmack, Belag, Verhältnis und Gesamt zu generieren.
**Priorität:** MVP
**Akzeptanzkriterien:**
*   Das LLM erhält eine Liste von Bild-URLs (getrennt nach Essen/Ambiente).
*   Die Antwort des LLM ist ein strukturiertes Objekt (z.B. JSON).
*   Das Objekt enthält einen Bewertungstext (String).
*   Das Objekt enthält vier separate Zahlenwerte: `score_geschmack`, `score_belag`, `score_verhaeltnis`, `score_gesamt` (jeweils 1-10).

---
**Anforderungs-ID:** F-04
**Titel:** Alle Ladendaten in Datenbank speichern
**User Story:** Als System möchte ich alle gesammelten und generierten Informationen (Ladenname, Adresse, Bilder-URLs, KI-Bewertungen/Scores) strukturiert in der CosmosDB speichern, um sie effizient für die Webseite abrufen zu können.
**Priorität:** MVP
**Akzeptanzkriterien:**
*   Für jeden Dönerladen wird ein eindeutiges Dokument in der CosmosDB erstellt.
*   Das Dokument enthält alle relevanten Informationen (Name, Adresse, Scores, Texte).
*   Der Lese- und Schreibzugriff auf die Datenbank funktioniert fehlerfrei.

---
**Anforderungs-ID:** F-05
**Titel:** Dönerläden in einer Liste anzeigen
**User Story:** Als Besucher der Webseite möchte ich eine Liste aller Dönerläden sehen, die den Namen, den Stadtbezirk und den KI-Gesamt-Score anzeigt, um mir einen schnellen Überblick zu verschaffen.
**Priorität:** MVP
**Akzeptanzkriterien:**
*   Die Startseite der Webseite zeigt die Liste an.
*   Jeder Listeneintrag ist klickbar.
*   Die Liste ist initial nach dem höchsten Gesamt-Score sortiert.

---
**Anforderungs-ID:** F-06
**Titel:** Detailseite für einen Laden anzeigen
**User Story:** Als Besucher möchte ich auf einen Laden in der Liste klicken können, um eine Detailseite mit dem kompletten KI-Bewertungstext und den Einzel-Scores zu sehen, um eine unterhaltsame "Einschätzung" zu erhalten.
**Priorität:** MVP
**Akzeptanzkriterien:**
*   Ein Klick auf einen Listeneintrag führt zu einer neuen Ansicht/Seite.
*   Diese Seite zeigt den Namen des Ladens, den generierten Bewertungstext und die vier Scores an.
*   Auch einige der 'Essen'- und 'Ambiente'-Bilder werden auf der Detailseite angezeigt.

---
**Anforderungs-ID:** F-07
**Titel:** Nach Gesamt-Score sortieren
**User Story:** Als Besucher möchte ich die Liste der Dönerläden nach dem KI-Gesamt-Score (auf- und absteigend) sortieren können, um schnell die besten oder schlechtesten Läden zu finden.
**Priorität:** MVP
**Akzeptanzkriterien:**
*   Es gibt ein sichtbares Steuerelement (z.B. Button, Dropdown) für die Sortierung.
*   Bei Auswahl "Absteigend" wird der Laden mit dem höchsten Score zuerst angezeigt.
*   Bei Auswahl "Aufsteigend" wird der Laden mit dem niedrigsten Score zuerst angezeigt.

---
**Anforderungs-ID:** F-08
**Titel:** Nach Stadtbezirk filtern
**User Story:** Als Besucher möchte ich die Liste nach einem oder mehreren Stuttgarter Stadtbezirken filtern können, um nur relevante Läden in meiner Nähe angezeigt zu bekommen.
**Priorität:** MVP
**Akzeptanzkriterien:**
*   Es gibt ein Filter-Steuerelement (z.B. Dropdown, Checkbox-Liste) mit allen Stadtbezirken.
*   Wenn ich einen Bezirk auswähle, zeigt die Liste nur noch Läden aus diesem Bezirk an.
*   Ich kann den Filter wieder zurücksetzen, um alle Läden zu sehen.

---

# **Zukünftige Anforderungen (Post-MVP)**

**Anforderungs-ID:** F-09
**Titel:** Interaktive Kartenansicht
**User Story:** Als Besucher möchte ich alle Dönerläden als Pins auf einer interaktiven Karte sehen, um eine geografische Suche durchführen zu können.
**Priorität:** V2-Should-Have
**Akzeptanzkriterien:**
*   Auf der Webseite gibt es eine Option, zur Kartenansicht zu wechseln.
*   Die Karte zeigt Stuttgart mit Pins für jeden Dönerladen.
*   Ein Klick auf einen Pin öffnet ein kleines Popup mit dem Laden-Namen, dem Gesamt-Score und einem Link zur Detailseite.

---
**Anforderungs-ID:** F-10
**Titel:** Filter "Jetzt geöffnet"
**User Story:** Als hungriger Besucher möchte ich einen Schalter haben, um nur die Läden anzuzeigen, die laut Google Maps Daten *jetzt gerade* geöffnet sind.
**Priorität:** V2-Should-Have
**Akzeptanzkriterien:**
*   Es gibt einen "Jetzt geöffnet"-Schalter in der Filtersektion.
*   Wenn aktiviert, werden Läden, die geschlossen sind oder keine Öffnungszeiten angeben, ausgeblendet.
*   Die Funktion berücksichtigt den aktuellen Wochentag und die Uhrzeit des Nutzers.

---
**Anforderungs-ID:** F-11
**Titel:** Einbeziehung von Google-Textbewertungen in KI-Analyse
**User Story:** Als System möchte ich Textbewertungen von Google Maps an das LLM übergeben, um eine nuanciertere KI-Bewertung zu generieren.
**Priorität:** V3-Could-Have
**Akzeptanzkriterien:**
*   Der Daten-Sammel-Prozess extrahiert zusätzlich die Top 3-5 Textbewertungen von Google.
*   Der Prompt für das LLM wird erweitert, um diese Texte als zusätzlichen Kontext zu nutzen.
*   Die generierte Bewertung reflektiert erkennbar Aspekte aus den Textbewertungen.

---
**Anforderungs-ID:** F-12
**Titel:** Filter "Preisklasse"
**User Story:** Als preisbewusster Besucher möchte ich die Läden nach der von Google Maps bereitgestellten Preisklasse (€, €€, €€€) filtern können, um Angebote passend zu meinem Budget zu finden.
**Priorität:** V2-Should-Have
**Akzeptanzkriterien:**
*   Es gibt ein Filter-Steuerelement (z.B. Checkboxen oder Buttons für €, €€, €€€).
*   Wenn ich eine Preisklasse auswähle, zeigt die Liste nur noch Läden an, denen Google diese Preisklasse zugeordnet hat.
*   Läden ohne Preisangabe werden bei einer aktiven Filterung ausgeblendet.
*   Ich kann meine Auswahl zurücksetzen, um wieder alle Läden zu sehen.

---
**Anforderungs-ID:** F-13
**Titel:** Filter "Vegan/Vegetarische Optionen"
**User Story:** Als Vegetarier/Veganer möchte ich gezielt nach Läden filtern können, die entsprechende Optionen anbieten, um sicherzugehen, dass ich dort etwas essen kann.
**Priorität:** V3-Could-Have
**Akzeptanzkriterien:**
*   Es gibt einen Filter-Schalter (z.B. eine Checkbox) mit der Beschriftung "Vegane/Vegetarische Optionen".
*   Wenn der Filter aktiviert ist, werden nur noch Läden angezeigt, die als "vegan/vegetarisch-freundlich" markiert sind.
*   **Wichtig:** Die Kennzeichnung basiert auf einer erweiterten KI-Analyse, die z.B. Google-Textbewertungen nach Schlüsselwörtern wie "Falafel", "Seitan", "vegetarisch" durchsucht, da diese Information selten direkt über die API verfügbar ist.
*   Der Filter kann jederzeit zurückgesetzt werden.

---
**Anforderungs-ID:** F-14
**Titel:** Freitextsuche nach Ladenname
**User Story:** Als Besucher, der einen bestimmten Dönerladen sucht, möchte ich ein Suchfeld haben, in das ich den Namen des Ladens eintippen kann, um ihn direkt zu finden.
**Priorität:** V2-Should-Have
**Akzeptanzkriterien:**
*   Auf der Webseite gibt es ein gut sichtbares Suchfeld.
*   Während ich tippe, werden mir passende Läden vorgeschlagen (Autocomplete).
*   Nach Absenden der Suche wird mir entweder die Detailseite des Ladens (bei einem Treffer) oder eine gefilterte Liste angezeigt.

---
**Anforderungs-ID:** F-15
**Titel:** KI-Bewertung teilen
**User Story:** Als Besucher, der eine besonders witzige oder treffende KI-Bewertung gefunden hat, möchte ich einen "Teilen"-Button haben, um einen Link zur Detailseite auf Social Media (z.B. Twitter, WhatsApp) zu posten.
**Priorität:** V3-Could-Have
**Akzeptanzkriterien:**
*   Auf jeder Detailseite gibt es einen "Teilen"-Button.
*   Bei Klick öffnet sich der native Teilen-Dialog des Betriebssystems/Browsers.
*   Der geteilte Link führt exakt zur richtigen Detailseite.

---
**Anforderungs-ID:** F-16
**Titel:** Admin-Dashboard zur Qualitätskontrolle
**User Story:** Als Projektteam möchte ich ein passwortgeschütztes Admin-Dashboard haben, in dem ich alle generierten KI-Bewertungen sehen und bei Bedarf manuell bearbeiten oder verbergen kann.
**Priorität:** V2-Should-Have
**Akzeptanzkriterien:**
*   Es gibt eine geheime URL (z.B. `/admin`), die nach einem Login fragt.
*   Das Dashboard zeigt eine Tabelle aller Läden und ihrer KI-Bewertungen.
*   Ich kann einen Bewertungstext bearbeiten und speichern.
*   Ich kann einen ganzen Laden aus der öffentlichen Ansicht ausblenden (Soft-Delete).

---
**Anforderungs-ID:** F-17
**Titel:** Erweiterung auf andere Städte
**User Story:** Als Systemarchitekt möchte ich die Anwendung so gestalten, dass das Hinzufügen einer neuen Stadt (z.B. "Berlin") möglich ist, ohne die gesamte Codebasis umschreiben zu müssen.
**Priorität:** V3-Could-Have (Architektur-Ziel)
**Akzeptanzkriterien:**
*   Die Stadt (z.B. "Stuttgart") ist eine Konfigurationsvariable und nicht fest im Code verankert.
*   Die Datenbankstruktur (CosmosDB) kann Daten für mehrere Städte aufnehmen, ohne dass es zu Konflikten kommt (z.B. durch ein `city`-Feld in jedem Dokument).

# Nicht-funktionale Anforderungen (Qualitätsmerkmale)

**Anforderungs-ID:** NFR-01
**Titel:** Mobile Optimierung (Responsive Design)
**User Story:** Als Besucher, der unterwegs mit seinem Smartphone nach einem Döner sucht, möchte ich, dass die Webseite auf meinem kleinen Bildschirm gut lesbar und einfach zu bedienen ist.
**Priorität:** MVP
**Akzeptanzkriterien:**
*   Die Webseite wird auf mobilen Browsern (Chrome auf Android, Safari auf iOS) ohne horizontales Scrollen korrekt dargestellt.
*   Schriftgrößen sind lesbar und Buttons sind groß genug, um sie mit dem Finger zu treffen.
*   Das Layout passt sich dynamisch an verschiedene Bildschirmgrößen an.

---
**Anforderungs-ID:** NFR-02
**Titel:** Transparenz der KI
**User Story:** Als Besucher der Webseite möchte ich klar erkennen können, dass die Bewertungen von einer KI stammen und nicht von echten Menschen, um die Natur des Projekts zu verstehen.
**Priorität:** MVP
**Akzeptanzkriterien:**
*   Auf der Webseite gibt es einen deutlichen und leicht verständlichen Hinweis (z.B. im Header oder bei jeder Bewertung), dass der Inhalt KI-generiert ist.
*   Es gibt eine "Über uns"-Seite, die das Projekt, das Team und die verwendete Technologie kurz erklärt.
