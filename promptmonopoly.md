Du bist ein senior Frontend-Game-Developer und sollst eine vollständig spielbare Monopoly-App als EINZIGES selbstständiges HTML-File erzeugen.

WICHTIG:
- Gib ausschließlich den vollständigen Code für eine einzige Datei namens index.html aus.
- Kein erläuternder Text vor oder nach dem Code.
- Keine TODOs, keine Platzhalter, keine unvollständigen Funktionen.
- Alles muss sofort lokal im Browser lauffähig sein.
- Nutze nur Vanilla HTML, CSS und JavaScript.
- Keine externen Frameworks, keine Build-Tools, keine Backend-Komponenten, keine externen APIs.
- Optional erlaubt: Google Fonts; ansonsten keine Abhängigkeiten.
- Sämtliches CSS im <style>-Tag und sämtliches JavaScript im <script>-Tag derselben Datei.
- Die App muss robust sein, auch bei längeren Partien, und automatisch in localStorage speichern.
- Bei Reload soll ein vorhandener Spielstand erkannt und angeboten werden: "Fortsetzen" oder "Neues Spiel".

ZIEL:
Baue eine modern gestaltete, vollständig spielbare Monopoly-Classic-Web-App für 2 bis 4 Gesamtspieler, davon 1 bis 4 menschliche Spieler und optional 1 bis 3 KI-Spieler, solange die Gesamtzahl maximal 4 ist.

================================
1. GRUNDLEGENDE PRODUKTANFORDERUNG
================================

Die App soll:
- auf Desktop sehr gut nutzbar sein;
- in einem einzigen HTML-File laufen;
- ein vollständiges Monopoly-Spiel ermöglichen;
- menschliche und KI-Spieler unterstützen;
- ein klar verständliches UI mit Brett, Spielerstatus, Aktionen, Modals und Log besitzen;
- ohne Nacharbeit direkt spielbar sein.

Nutze eine saubere Architektur mit:
- zentralem gameState-Objekt,
- klaren Datenstrukturen für Felder, Spieler, Karten, Auktionen, Trades und UI-Status,
- deterministischen Hilfsfunktionen für Regeln,
- Render-Funktionen getrennt von Spiellogik,
- Event-Delegation, wo sinnvoll.

================================
2. SETUP-SCREEN
================================

Implementiere vor Spielstart einen Setup-Screen mit:
- Auswahl der Gesamtspielerzahl 2 bis 4,
- Auswahl der Anzahl KI-Spieler 0 bis 3, wobei Gesamtspielerzahl nie überschritten wird,
- Eingabefelder für menschliche Spielernamen,
- automatische Namen für KI wie "KI 1", "KI 2", "KI 3",
- Auswahl einer Farbe / eines Tokens je Spieler,
- Validierung gegen doppelte Farben/Tokens,
- Button "Spiel starten".

Beim Start:
- Jeder Spieler erhält 1500 Startgeld.
- Alle Figuren starten auf LOS.
- Ereignis- und Gemeinschaftskarten werden gemischt.
- Ein zufälliger Startspieler wird bestimmt.
- Das Spiel speichert den Zustand sofort.

================================
3. SPIELBRETT
================================

Erstelle ein visuelles Monopoly-Brett mit 40 Feldern im Uhrzeigersinn als CSS-Grid im 11x11-Layout.
Die Mitte des Bretts kann für Logo, Logostatus, Kartenstapel und Aktionshinweise genutzt werden.

Pflichtfelder:
- LOS
- Badstraße
- Gemeinschaftsfeld
- Turmstraße
- Einkommenssteuer
- Südbahnhof
- Chausseestraße
- Ereignisfeld
- Elisenstraße
- Poststraße
- Gefängnis / Nur zu Besuch
- Seestraße
- Elektrizitätswerk
- Hafenstraße
- Neue Straße
- Westbahnhof
- Münchener Straße
- Gemeinschaftsfeld
- Wiener Straße
- Berliner Straße
- Frei Parken
- Theaterstraße
- Ereignisfeld
- Museumstraße
- Opernplatz
- Nordbahnhof
- Lessingstraße
- Schillerstraße
- Wasserwerk
- Goetheplatz
- Geh in das Gefängnis
- Rathausplatz
- Hauptstraße
- Gemeinschaftsfeld
- Bahnhofstraße
- Ostbahnhof
- Ereignisfeld
- Parkstraße
- Luxussteuer
- Schlossallee

Verwende die klassischen deutschen Straßennamen der Standardausgabe und gruppiere die Straßen in die üblichen 8 Farbgruppen. Die klassische Struktur mit 22 Straßen, 4 Bahnhöfen, 2 Werken, 3 Ereignisfeldern, 3 Gemeinschaftsfeldern, 2 Steuerfeldern und 4 Eckfeldern muss eingehalten werden. [web:22][web:25]

Jedes Feld zeigt:
- Name,
- Kaufpreis falls kaufbar,
- Farbstreifen bei Straßen,
- Besitzanzeige,
- Häuser/Hotel,
- Figuren der Spieler auf dem Feld.

================================
4. DATENMODELL FÜR FELDER
================================

Lege alle Felder als JavaScript-Array von 40 Objekten an.

Jedes Feldobjekt enthält je nach Typ:
- index
- type: start, street, railroad, utility, tax, chance, chest, jail, freeparking, gotojail
- name
- colorGroup bei Straßen
- price
- mortgage
- houseCost wenn bebaubar
- rents:
  - base
  - monopoly
  - house1
  - house2
  - house3
  - house4
  - hotel
- ownerId
- houses
- hotel
- mortgaged

Für Bahnhöfe:
- price 200
- mortgage 100
- rent abhängig von Anzahl besessener Bahnhöfe: 25 / 50 / 100 / 200. [web:21]

Für Werke:
- Elektrizitätswerk und Wasserwerk,
- Preis 150,
- Miete abhängig von Würfelsumme:
  - 4x Würfelsumme wenn 1 Werk im Besitz,
  - 10x Würfelsumme wenn beide Werke im Besitz.

Für Steuern:
- Einkommenssteuer 200
- Luxussteuer 100

================================
5. STRASSENDATEN
================================

Implementiere die 22 Straßen mit klassischen deutschen Namen, Kaufpreisen, Hypothekenwerten und Mieten. Die Preis- und Miettabelle soll den klassischen Monopoly-Werten entsprechen, z. B. Badstraße/Turmstraße 60, Chausseestraße/Elisenstraße 100, ... Parkstraße 350, Schlossallee 400. [web:25]

Nutze diese klassischen Kaufpreise pro Straße:
- Braun: 60, 60
- Hellblau: 100, 100, 120
- Lila/Rosa: 140, 140, 160
- Orange: 180, 180, 200
- Rot: 220, 220, 240
- Gelb: 260, 260, 280
- Grün: 300, 300, 320
- Dunkelblau: 350, 400 [web:25]

Nutze die klassischen Hauskosten:
- Braun/Hellblau: 50
- Lila/Orange: 100
- Rot/Gelb: 150
- Grün/Dunkelblau: 200

Nutze klassische Mieten pro Straße inklusive:
- Grundmiete unbebaut,
- verdoppelte Grundmiete bei vollständiger Farbgruppe und unbebauten, nicht beliehenen Straßen derselben Gruppe,
- Mieten für 1, 2, 3, 4 Häuser,
- Miete für 1 Hotel. [web:21][web:25]

================================
6. SPIELABLAUF
================================

Jeder Zug folgt dieser Logik:

A. Zugbeginn
- Prüfen, ob Spieler bankrott oder ausgeschieden ist.
- Prüfen, ob Spieler im Gefängnis ist.
- Falls im Gefängnis: UI mit Optionen:
  1. Pasch versuchen,
  2. 50 bezahlen,
  3. Gefängnis-Frei-Karte nutzen, falls vorhanden.
- Nach 3 erfolglosen Runden muss gezahlt oder eine Karte genutzt werden, bevor gezogen wird.

B. Würfeln
- Zwei Würfel mit sichtbarer Animation.
- Zeige die Ergebnisse deutlich an.
- Bei Pasch darf erneut gewürfelt werden.
- Bei 3 Paschen in demselben Zug: direkt ins Gefängnis.

C. Bewegung
- Figur animiert Feld für Feld verschieben.
- Beim Überschreiten oder Landen auf LOS erhält der Spieler 200.
- Danach Feldaktion ausführen.

D. Feldaktion
Je nach Feld:
- freie Straße/Bahnhof/Werk: Kaufdialog oder Auktion, wenn abgelehnt,
- fremdes Eigentum: Miete berechnen und zahlen,
- eigenes Eigentum: nichts,
- Steuer: an Bank zahlen,
- Chance/Community Chest: Karte ziehen und ausführen,
- Frei Parken: keine Sonderregel, nur nichts passiert,
- Geh ins Gefängnis: sofort ins Gefängnis ohne LOS-Geld,
- Gefängnis nur Besuch: keine Aktion.

E. Zugende
- Prüfen, ob gebaut werden darf,
- optional Handeln zulassen,
- Spielstand speichern,
- nächsten aktiven Spieler auswählen.

================================
7. KAUF, AUKTION, MIETE
================================

Kauf:
- Wenn Spieler auf einem freien kaufbaren Feld landet, erscheint Modal:
  - kaufen
  - nicht kaufen
- Wenn gekauft:
  - Geld abziehen
  - Besitzer setzen
  - Besitzurkunde im UI sichtbar machen

Auktion:
- Wenn Spieler nicht kauft, startet automatisch eine Auktion zwischen allen nicht-bankrotten Spielern.
- Einfaches Auktionssystem:
  - Menschliche Spieler können Gebot eingeben oder passen.
  - KI bietet in sinnvollen Schritten, solange Preis unter internem Limit liegt.
  - Gewinner zahlt, wenn finanzierbar.
  - Wenn niemand bietet, bleibt Feld frei.

Miete:
- Straßen: normale Miete oder erhöhte Miete durch Häuser/Hotel.
- Ist eine Straße unbebaut und Eigentümer besitzt die komplette Farbgruppe, verdoppelt sich die Grundmiete nur dann, wenn keine Straße dieser Gruppe hypothekarisch belastet ist. [web:21][web:27]
- Bahnhöfe: 25 / 50 / 100 / 200 abhängig von Anzahl im Besitz. [web:21]
- Werke: 4x oder 10x der gewürfelten Augensumme.
- Für beliehene Grundstücke wird keine Miete verlangt. [web:20]

Der Eigentümer muss nicht manuell die Miete einfordern; die App erledigt dies automatisch.

================================
8. HÄUSER UND HOTELS
================================

Implementiere vollständige Hausregeln:
- Häuser dürfen nur gebaut werden, wenn der Spieler alle Straßen einer Farbgruppe besitzt.
- Keine Straße der Gruppe darf belastet sein.
- Es muss gleichmäßig gebaut werden, also keine Straße darf mehr als ein Haus mehr haben als eine andere derselben Gruppe. [web:20]
- Maximal 4 Häuser pro Straße.
- Danach kann ein Hotel gekauft werden.
- Ein Hotel ersetzt 4 Häuser auf dieser Straße.
- Pro Straße maximal 1 Hotel.

UI:
- Ein "Bauen"-Modal oder Seitenpanel,
- Auswahl nur legaler Bauaktionen,
- Anzeige der Kosten,
- Prüfung ausreichender Liquidität.

Verkauf:
- Häuser/Hotels dürfen zum halben Kaufpreis an die Bank verkauft werden.
- Beim Verkauf ebenfalls gleichmäßige Rückabwicklung beachten.

================================
9. HYPOTHEK
================================

Implementiere Hypotheken vollständig:
- Nur unbebaute Grundstücke dürfen beliehen werden.
- Grundstücke einer Farbgruppe mit Gebäuden müssen zuerst vollständig und regelkonform rückgebaut werden, bevor ein einzelnes Grundstück der Gruppe beliehen werden kann.
- Beim Beleihen erhält der Spieler den Hypothekenwert.
- Zum Auslösen zahlt der Spieler Hypothekenwert plus 10 Prozent, kaufmännisch sinnvoll gerundet.
- Beliehene Grundstücke bringen keine Miete. [web:20]

Baue ein Verwaltungs-Modal:
- Besitztümer anzeigen,
- beleihen,
- Hypothek ablösen,
- Häuser/Hotels verkaufen.

================================
10. GEFÄNGNIS
================================

Regeln:
- Ins Gefängnis durch:
  - Landung auf "Geh in das Gefängnis",
  - 3 Pasche in einem Zug,
  - entsprechende Karte.
- Im Gefängnis:
  - maximal 3 Runden fest,
  - pro Runde Paschversuch möglich,
  - alternativ 50 zahlen,
  - oder "Du kommst aus dem Gefängnis frei"-Karte einsetzen.
- Bei Pasch: Spieler verlässt das Gefängnis und zieht um die Augenzahl.
- Nach der 3. erfolglosen Runde muss bezahlt oder Karte benutzt werden, dann normal ziehen.

Kennzeichne im UI, welche Spieler im Gefängnis sind.

================================
11. KARTEN
================================

Erstelle mindestens 10 Ereigniskarten und 10 Gemeinschaftskarten, jeweils als Datenarray.
Karten müssen gemischt und zyklisch oder per Nachmischen verwendet werden.

Beispiele für Effekte:
- Gehe vor bis LOS.
- Gehe ins Gefängnis.
- Rücke vor bis Schlossallee.
- Rücke vor bis Südbahnhof.
- Bank zahlt Dividende.
- Zahle Arztkosten.
- Erhalte Erbschaft.
- Straßenreparaturen: zahle pro Haus / pro Hotel.
- Gehe 3 Felder zurück.
- Du kommst aus dem Gefängnis frei.

Jede Karte braucht:
- id
- deckType
- text
- actionType
- parameter

Karten-UI:
- Modal mit Kartentext,
- Button "OK",
- Effekt wird danach ausgeführt.

================================
12. BANKROTT
================================

Wenn ein Spieler eine Zahlung nicht leisten kann:
- Prüfe automatisch, ob durch Beleihen / Gebäudeverkauf genug Geld beschafft werden könnte.
- KI soll dies automatisch versuchen.
- Menschliche Spieler erhalten ein Modal zur Sanierung:
  - Häuser verkaufen,
  - beleihen,
  - Hypothek ablösen nicht relevant in Notsituation,
  - erst wenn weiterhin nicht zahlungsfähig: Bankrott bestätigen.

Bankrottregeln:
- Gegenüber Bank: Besitztümer an Bank, Gebäude entfernt, Grundstücke werden frei oder optional direkt auktioniert.
- Gegenüber anderem Spieler: Geld/Grundstücke/Karten gehen an Gläubiger; belastete Grundstücke bleiben belastet.
- Bankrotter Spieler scheidet aus.
- Wenn nur noch ein Spieler aktiv ist, endet das Spiel.

================================
13. HANDEL
================================

Implementiere einfachen Handel nur zwischen menschlichen Spielern:
- Ein Spieler kann in seinem Zug einen Trade starten.
- Auswahl eines anderen aktiven menschlichen Spielers.
- Auswahl angebotener unbebauter Grundstücke und Geldsumme.
- Auswahl geforderter unbebauter Grundstücke und Geldsumme.
- Beide Seiten müssen bestätigen.
- KI handelt nicht.
- Handel nur mit unbebauten Grundstücken.
- Hypothekenstatus muss sichtbar sein; Trades mit hypothekarisch belasteten Grundstücken sind erlaubt, aber klar gekennzeichnet.

================================
14. KI-LOGIK
================================

Die KI muss spielbar und nachvollziehbar agieren:
- würfelt automatisch nach kurzer Pause,
- kauft freie Grundstücke, wenn nach Kauf mindestens 200 Bargeld Reserve bleiben;
- bewertet Bahnhöfe hoch,
- bewertet Werke moderat,
- lehnt teure Käufe bei sehr niedrigem Cash ab;
- bietet in Auktionen mit einfachem Heuristiksystem;
- baut Häuser automatisch am Zugende, wenn sie eine vollständige Farbgruppe hat und danach mindestens 200 Bargeld übrig bleiben;
- baut gleichmäßig;
- verkauft Häuser und beleiht Grundstücke automatisch, wenn Geld für Pflichtzahlungen fehlt;
- nutzt Gefängnis-Frei-Karten sinnvoll;
- protokolliert alle Entscheidungen im Log.

Nutze kein perfektes Minimax; einfache Heuristik reicht, aber die KI muss regelkonform sein.

================================
15. UI / UX
================================

Designvorgaben:
- Monopoly-artige Farbwelt mit grünem Filztisch-Look,
- hochwertige Karten/Panel-Optik,
- gute Lesbarkeit,
- moderne Buttons und Modals,
- responsive Desktop-Ansicht, sinnvoll skalierend.

Pflichtbereiche:
- zentrales Spielbrett,
- rechte oder linke Seitenleiste mit Spielerstatus,
- Bereich für Aktionsbuttons,
- scrollbares Log,
- Bereich für aktuelle Würfel,
- Statushinweis, welcher Spieler am Zug ist.

Spielerpanel pro Spieler:
- Name
- Mensch oder KI
- Farbe/Token
- Bargeld
- Nettovermögen
- Position
- Gefängnisstatus
- Anzahl Bahnhöfe/Werke
- Besitzliste
- Karten "Gefängnis frei"

Aktionsbuttons für menschliche Spieler:
- Würfeln
- Zug beenden
- Bauen
- Verwalten
- Handeln
- Spiel speichern
- Neues Spiel

Modals:
- Kaufentscheidung
- Auktion
- Kartenanzeige
- Gefängnisoptionen
- Bauen/Verkaufen
- Hypothekenverwaltung
- Handel
- Bankrott / Sanierung
- Siegeranzeige

Log:
- Zeigt alle Aktionen chronologisch,
- z. B. "Anna würfelt 3 + 4 und zieht auf Opernplatz",
- "KI 1 kauft Südbahnhof für 200",
- "Ben zahlt 28 Miete an Clara",
- "David baut ein Haus auf Berliner Straße".

================================
16. REGELFUNKTIONEN
================================

Implementiere robuste Hilfsfunktionen, unter anderem:
- canBuyField(player, field)
- canBuildOnStreet(player, field)
- getMonopolyStatus(player, colorGroup)
- getStreetRent(field, owner)
- getRailroadRent(owner)
- getUtilityRent(owner, diceTotal)
- mortgageField(player, field)
- unmortgageField(player, field)
- sellHouse(player, field)
- buildHouse(player, field)
- sendToJail(player)
- releaseFromJail(player)
- processLanding(player, field, diceTotal)
- triggerAuction(field)
- resolveCard(card, player)
- computeNetWorth(player)
- getNextActivePlayer()
- saveGame()
- loadGame()

Achte auf sauberes Error-Handling und verhindere illegale Aktionen über die UI.

================================
17. LOKALER SPEICHER
================================

Speichere automatisch:
- nach jedem Kauf,
- nach jedem Zug,
- nach jedem Bau/Verkauf,
- nach jeder Karte,
- nach Gefängnisstatusänderungen,
- nach Trades,
- nach Auktionen.

Nutze localStorage-Schlüssel wie:
- monopoly-singlefile-save

Beim Laden:
- Vollständigen Zustand rekonstruieren,
- Decks, Positionen, Besitz, Häuser, Hotels, Hypotheken, Log und aktuellen Spieler wiederherstellen.

================================
18. TECHNISCHE QUALITÄT
================================

Der Code muss:
- modular innerhalb einer Datei organisiert sein,
- gut benannte Funktionen haben,
- keine globalen Konflikte erzeugen,
- DOM performant aktualisieren,
- keine Syntaxfehler enthalten,
- in aktuellen Chromium- und Firefox-Browsern lokal funktionieren.

Nutze bei Bedarf:
- requestAnimationFrame für Bewegungen,
- kleine Timeouts für KI und Würfelanimationen,
- Utility-Funktionen für Modal-Rendering,
- CSS-Klassen für Besitzfarben und Gebäudestatus.

================================
19. ABSCHLUSSANFORDERUNG
================================

Liefere jetzt direkt den vollständigen Inhalt von index.html.
Wirklich vollständig.
Kein erklärender Text.
Kein Markdown außerhalb des Codes.
Keine Auslassungen.