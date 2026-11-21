# Claude Code: Tokens sparen und das Wochenlimit seltener erreichen

Claude-Wochenlimit erreicht – was tun? Meist liegt es nicht an zu viel Arbeit, sondern daran, wie gearbeitet wird:
lange Chats, viele Bilder im Hauptchat und Opus für jede Kleinigkeit. Dieses Repository enthält zwei Anleitungen,
die du deinem eigenen Claude gibst. Claude geht sie dann Schritt für Schritt mit dir durch: erst messen, dann
gezielt ändern, nach einer Woche wieder messen.

Kostenlos, für Privatnutzer und Solo-Selbstständige mit Claude Pro oder Max, die Claude Code im Terminal, in der IDE
oder im Code-Tab von Claude Desktop nutzen (Windows oder Mac).

Von René Hillers, [rhse.de](https://rhse.de/) – Software-Entwicklung & KI-Automatisierung.

## So benutzt du die Anleitungen

1. Lade das ganze Repository herunter: oben auf **Code** → **Download ZIP**, dann entpacken.
2. Gib deinem Claude den Pfad zu `anleitungen/01-rollen-und-modelle.md` (oder zieh die Datei in den Chat) und
   schreib: „Lies sie ganz und geh sie Schritt für Schritt mit mir durch.“
3. Später dasselbe mit `anleitungen/02-wissen-in-dateien.md`.

Die Vorlagen im Ordner `vorlagen/` musst du nicht selbst anfassen. Die Anleitungen verweisen darauf, und Claude
nimmt sie an der passenden Stelle. Deshalb das ganze Repository herunterladen, nicht nur eine Datei.

Claude ändert nichts, bevor du zu der jeweiligen Änderung Ja gesagt hast. Das steht in beiden Anleitungen ganz
oben. Programmierkenntnisse brauchst du nicht, aber du solltest Claude Code schon benutzen.

Die Anleitungen sind auf Englisch, weil Claude sie liest. Claude spricht mit dir in deiner Sprache.

## Warum ist mein Claude-Limit so schnell erreicht?

Claude Code schickt bei jeder Anfrage den ganzen bisherigen Chat erneut mit, und jeder Werkzeugaufruf ist eine
weitere Anfrage. Laut Doku zieht deshalb selbst eine einzeilige Frage in einer Sitzung, die schon den ganzen Tag
läuft, Verbrauch für das ganze Gespräch. Dazu kommen Bilder: ein Screenshot kostet bis zu 4.784 Tokens und reist bei
jeder weiteren Anfrage mit. Und Subagents ohne eigenes Modell laufen auf dem Hauptmodell, auf Pro und Max also
standardmäßig auf Opus.
→ [Anleitung 1, Abschnitt 1](anleitungen/01-rollen-und-modelle.md#1-why-claude-code-hits-the-limit-too-fast---the-mechanics)

## Wie spare ich Tokens in Claude Code?

Erst messen: `/usage` zeigt auf Pro und Max, welcher Anteil auf Subagents, Skills, Plugins und einzelne
MCP-Server entfällt, und `/context` zeigt, was den Kontext füllt. Danach wirken meist drei Dinge am stärksten:
kürzere Chats mit Übergabe über Dateien, Bilder in einen eigenen Subagent statt in den Hauptchat, und ein passendes
Modell pro Aufgabe. Nach einer Woche misst du mit `/usage` erneut und vergleichst.
→ [Anleitung 1, Abschnitt 2](anleitungen/01-rollen-und-modelle.md#2-step-1---measure-token-usage-first-with-the-user-about-15-minutes-change-nothing)

## Welches Modell: Opus, Sonnet oder Haiku?

Opus für offene Probleme, Planung und Entscheidungen, bei denen ein Fehler teuer ist. Sonnet für klar beschriebene
Arbeit: Code nach Vorgabe, Skripte, Konfiguration, Texte aus einer Gliederung. Haiku für Suchen, Nachschlagen und
mechanische Arbeit mit eindeutiger Antwort. Die offizielle Doku empfiehlt ebenfalls Sonnet für die meisten
Coding-Aufgaben und Haiku für einfache Subagent-Aufgaben. Mit `/model opusplan` plant Opus und Sonnet führt aus.
→ [Anleitung 1, Abschnitt 3](anleitungen/01-rollen-und-modelle.md#3-step-2---opus-vs-sonnet-vs-haiku-which-model-for-which-work)

## Wie stelle ich das Modell für Claude Code Subagents ein?

Jede Rollendatei in `~/.claude/agents/` oder `.claude/agents/` bekommt ein eigenes Feld `model:` (z. B. `haiku`)
und am besten auch `effort:`. Fehlt `model`, erbt der Subagent das Modell der Hauptsitzung. Wichtig: Ein Modell,
das die Hauptsitzung beim Start des Subagents mitgibt, hat Vorrang vor der Rollendatei. Deshalb gehört in die
CLAUDE.md die Regel, beim Start eines Subagents kein Modell zu setzen. Vorlagen für drei Rollen liegen in
`vorlagen/agents/`.
→ [Anleitung 1, Abschnitt 4](anleitungen/01-rollen-und-modelle.md#4-step-3---subagents-with-an-explicit-model-check-the-existing-ones-first-then-add-new-ones)

## Was gehört in die CLAUDE.md?

Was das Projekt ist, wo Dinge liegen, die Arbeitsregeln und Verweise auf weitere Dateien. Kein Verlauf, keine
Zwischenstände: Die CLAUDE.md lädt in jede Sitzung, jede Zeile kostet also bei jeder Anfrage. Die Doku empfiehlt
unter 200 Zeilen pro Datei. Den aktuellen Stand, Entscheidungen und Erkenntnisse hältst du in eigenen Dateien
fest (`STAND.md`, `decisions.md`, `memory.md`), die nur bei Bedarf gelesen werden.
→ [Anleitung 2, Abschnitt 3](anleitungen/02-wissen-in-dateien.md#3-where-knowledge-lives-claudemd-standmd-decisionsmd-memorymd)

## /compact oder /clear?

`/clear` beginnt ein neues Gespräch und kostet nichts. `/compact` fasst das Gespräch zusammen und liest es dafür
selbst noch einmal, bei großem Kontext ist das eine große Anfrage. Faustregel: Ist eine Aufgabe fertig, Stand in
`STAND.md` schreiben und `/clear`. Mitten in einer Aufgabe, wenn der Kontext voll wird, `/compact` mit einem Hinweis,
was erhalten bleiben soll.
→ [Anleitung 2, Abschnitt 4](anleitungen/02-wissen-in-dateien.md#4-the-rhythm)

## Kontextfenster voll: Was passiert dann?

Claude Code fasst den Chat automatisch zusammen, bevor das Fenster voll ist. Auf Pro und Max läuft das
Standardmodell mit einem Fenster von 1 Million Tokens und verdichtet erst bei etwa 967.000 Tokens. Bis dahin wird
bei jeder Anfrage der ganze Verlauf mitgeschickt. Mit `/autocompact 200k` setzt du diese Grenze früher, als
Sicherheitsnetz. Besser ist eine eigene Übergabe vorher.
→ [Anleitung 2, Abschnitt 5](anleitungen/02-wissen-in-dateien.md#5-guard-rails-in-the-setup-with-the-users-yes)

## Wo sehe ich mein Wochenlimit, und kann ich es erhöhen?

`/usage` zeigt im Terminal deine Plan-Limits und den Verbrauch der letzten 24 Stunden oder 7 Tage. In Claude
Desktop zeigt der Verbrauchsring unter dem Eingabefeld die Plan-Nutzung. Ist das Limit erreicht, nennt die Meldung
den Zeitpunkt, zu dem es zurückgesetzt wird. Mehr Volumen gibt es nur über einen größeren Plan oder kostenpflichtige
Zusatznutzung (`/usage-credits`). Dieses Repository hilft, mit weniger Tokens auszukommen, nicht, Limits zu umgehen.

## Was ist drin?

```
anleitungen/
  01-rollen-und-modelle.md   Messen, Modell pro Aufgabe, Subagents mit festem Modell, Bilder, Sitzungen
  02-wissen-in-dateien.md    Wissen in Dateien statt im Chat, Übergabe, Rhythmus, Leitplanken
vorlagen/
  agents/scout.md            Rolle für Suchen und Nachschlagen (haiku)
  agents/visual-check.md     Rolle für Bildprüfungen, damit Bilder aus dem Hauptchat bleiben (sonnet)
  agents/prompt-writer.md    Rolle für Prompts und Aufgabenbeschreibungen (sonnet)
  STAND.md                   Übergabedatei für den nächsten Chat
  decisions.md               Entscheidungen, datiert, mit deinen Worten
  memory.md                  Fakten und Erkenntnisse
  CLAUDE-md-lines.md         Zeilen für die persönliche ~/.claude/CLAUDE.md, inkl. Sprach-Einstellung
```

## Hinweise

- Inoffiziell. Nicht mit Anthropic verbunden. „Claude“ ist eine Marke von Anthropic.
- Alle technischen Angaben wurden am 21.11.2026 gegen die offizielle Dokumentation geprüft
  (code.claude.com/docs und platform.claude.com/docs). Die Quellen stehen am Ende jeder Anleitung. Die Doku ändert
  sich, Befehle und Felder können sich ändern. Im Zweifel gilt die aktuelle Doku.
- Wie viel du sparst, hängt von deiner Arbeitsweise ab. Eine bestimmte Ersparnis wird nicht versprochen.

## Lizenz

[CC BY 4.0](LICENSE). Du darfst die Texte und Vorlagen teilen und anpassen, auch kommerziell, mit Namensnennung.

## Über den Autor

René Hillers – [rhse.de](https://rhse.de/), Software-Entwicklung & KI-Automatisierung. Über 25 Jahre
Softwareentwicklung; heute baut er mit Coding-Agenten und einem eigenen Multi-Agent-System.
[Referenzen](https://rhse.de/referenzen)

Für Teams in Unternehmen und Selbstständige gibt es dazu eine Claude-Schulung, online und auf Deutsch: [rhse.de/claude-schulung](https://rhse.de/claude-schulung)
