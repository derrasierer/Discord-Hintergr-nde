# Borderlands Background Switcher

Beschreibung
------------
Dieses Repository enthält CSS-Anpassungen für das Translucence BetterDiscord‑Theme, mit denen du ein komplettes Borderlands‑Bild als Discord‑Hintergrund verwenden und zwischen mehreren Bildern wechseln kannst. Zusätzlich gibt es Variablen für Popout-/Profil‑Rundungen, Blur/Overlay und Card‑Transparenzen.

Wichtig: Dieses Projekt ändert nur CSS (BetterDiscord theme). Du benötigst BetterDiscord oder eine vergleichbare Client‑Modifikation, um das Theme zu verwenden.

Inhalt
------
- `theme.css` (oder deine Theme-Datei): enthält die Bild‑Variablen, UI‑Variablen und die Hintergrund‑Rules.
- Bilder liegen im Ordner `Pictures/` als JPG-Dateien (Raw‑URLs werden verwendet).

Installation (schnell)
---------------------
1. Öffne dein BetterDiscord Themes‑Verzeichnis (%appdata%/BetterDiscord/Main.css).
2. Lege die angepasste `theme.css` in das Verzeichnis oder füge die Änderungen in dein Betterdiscord ein.
3. In Discord: Settings → BetterDiscord → Themes → Theme neu laden / Theme aktivieren. Oder Client neu starten (Ctrl+R/Strg+R).

Wo die Bild‑Variablen stehen
---------------------------
Die Bild‑Variablen befinden sich im `:root`-Block deiner CSS. Beispiel:

```css
:root{
  --img_designed: url("https://raw.githubusercontent.com/derrasierer/Discord-Hintergr-nde/main/Pictures/Borderlands%203%20Designed%20Head.jpg");
  --img_holding:  url("https://raw.githubusercontent.com/derrasierer/Discord-Hintergr-nde/main/Pictures/Borderlands%203%20Holding%20Head.jpg");
  --img_load:     url("https://raw.githubusercontent.com/derrasierer/Discord-Hintergr-nde/main/Pictures/Borderlands%203%20Loadingscreen.jpg");
  --img_mond:     url("https://raw.githubusercontent.com/derrasierer/Discord-Hintergr-nde/main/Pictures/Borderlands%204%20Mond.jpg");
  --img_sirene:   url("https://raw.githubusercontent.com/derrasierer/Discord-Hintergr-nde/main/Pictures/Borderlands%204%20Sirene.jpg");
  --img_cave:     url("https://raw.githubusercontent.com/derrasierer/Discord-Hintergr-nde/main/Pictures/Borderlands%20idk%20Cave.jpg");

  /* Wähle Standardbild */
  --selected-bg: var(--img_holding);
}
Hintergrund dauerhaft ändern (per CSS)
Öffne theme.css und setze --selected-bg auf eine der --img_* Variablen, z. B.:
CSS
--selected-bg: var(--img_mond);
Speichern → Theme neu laden (BetterDiscord) oder Discord neu starten.
Bild temporär (nur bis Reload) per Browserkonsole ändern
Öffne Discord Desktop → Rechtsklick → Untersuchen (DevTools) → Console.
Führe z. B. aus:
js
// Auf eine vorhandene Variable setzen:
document.documentElement.style.setProperty('--selected-bg', 'var(--img_sirene)');

// Oder direkt mit URL:
document.documentElement.style.setProperty('--selected-bg', 'url("https://raw.githubusercontent.com/derrasierer/Discord-Hintergr-nde/main/Pictures/Borderlands%204%20Mond.jpg")');
Hinweis: Konsolenänderungen verfallen nach Reload.
Automatisches Wechseln (Rotation) alle N Minuten
Zwei Möglichkeiten:

A) Kurztest (Console, temporär)

js
(() => {
  const files = [
    "https://raw.githubusercontent.com/derrasierer/Discord-Hintergr-nde/main/Pictures/Borderlands%203%20Designed%20Head.jpg",
    "https://raw.githubusercontent.com/derrasierer/Discord-Hintergr-nde/main/Pictures/Borderlands%203%20Holding%20Head.jpg",
    "https://raw.githubusercontent.com/derrasierer/Discord-Hintergr-nde/main/Pictures/Borderlands%203%20Loadingscreen.jpg",
    "https://raw.githubusercontent.com/derrasierer/Discord-Hintergr-nde/main/Pictures/Borderlands%204%20Mond.jpg",
    "https://raw.githubusercontent.com/derrasierer/Discord-Hintergr-nde/main/Pictures/Borderlands%204%20Sirene.jpg",
    "https://raw.githubusercontent.com/derrasierer/Discord-Hintergr-nde/main/Pictures/Borderlands%20idk%20Cave.jpg"
  ];
  const minutes = 60; // Intervall in Minuten
  let idx = 0;
  document.documentElement.style.setProperty('--selected-bg', `url("${files[idx]}")`);
  const id = setInterval(() => {
    idx = (idx + 1) % files.length;
    document.documentElement.style.setProperty('--selected-bg', `url("${files[idx]}")`);
    console.log("Hintergrund geändert zu:", files[idx]);
  }, minutes * 60 * 1000);
  window.__bgRotatorId = id; // zum Stoppen: clearInterval(window.__bgRotatorId)
})();
B) Dauerhaft (empfohlen): BetterDiscord Plugin

Speicher die Datei BackgroundRotator.plugin.js in dein BetterDiscord plugins-Ordner.
Beispiel-Plugin (einfach, konfigurierbar):
js
/**
 * @name BackgroundRotator
 * @version 1.0.2
 * @author derrasierer
 * @description Rotates the --selected-bg CSS variable through a list of URLs every N minutes.
 */

module.exports = class BackgroundRotator {
  constructor() {
    this.intervalId = null;
    this.index = 0;
    this.previous = null;
    this._bgObserver = null;

    // Konfiguration: Minuten und URLs anpassen
    this.settings = {
      minutes: 60,
      urls: [
        "https://raw.githubusercontent.com/derrasierer/Discord-Hintergr-nde/main/Pictures/Borderlands%203%20Designed%20Head.jpg",
        "https://raw.githubusercontent.com/derrasierer/Discord-Hintergr-nde/main/Pictures/Borderlands%203%20Holding%20Head.jpg",
        "https://raw.githubusercontent.com/derrasierer/Discord-Hintergr-nde/main/Pictures/Borderlands%203%20Loadingscreen.jpg",
        "https://raw.githubusercontent.com/derrasierer/Discord-Hintergr-nde/main/Pictures/Borderlands%204%20Mond.jpg",
        "https://raw.githubusercontent.com/derrasierer/Discord-Hintergr-nde/main/Pictures/Borderlands%204%20Sirene.jpg",
        "https://raw.githubusercontent.com/derrasierer/Discord-Hintergr-nde/main/Pictures/Borderlands%20idk%20Cave.jpg"
      ],
      startRandom: false
    };
  }

  getName() { return "BackgroundRotator"; }
  getVersion() { return "1.0.2"; }
  getAuthor() { return "derrasierer"; }
  getDescription() { return "Rotates the --selected-bg CSS variable through a list of URLs every N minutes."; }

  start() {
    try {
      // vorherigen Wert merken (inline style oder computed fallback)
      this.previous = document.documentElement.style.getPropertyValue('--selected-bg') ||
                      getComputedStyle(document.documentElement).getPropertyValue('--selected-bg') ||
                      "";

      // Startindex
      if (this.settings.startRandom) this.index = Math.floor(Math.random() * this.settings.urls.length);
      else this.index = 0;

      // sofort setzen (wenn gültige URL vorhanden)
      this.applyBackground(this.index);

      // Interval starten
      const ms = Math.max(0.1, Number(this.settings.minutes) || 60) * 60 * 1000;
      this.intervalId = setInterval(() => {
        this.index = (this.index + 1) % this.settings.urls.length;
        this.applyBackground(this.index);
        if (window.BdApi && BdApi.showToast) BdApi.showToast(`Hintergrund: Bild ${this.index+1}`, {type: "info"});
      }, ms);

      // MutationObserver: wenn body.style.backgroundImage verändert wird und nicht unserem URL entspricht, neu setzen
      if (!this._bgObserver) {
        this._bgObserver = new MutationObserver(() => {
          try {
            const current = getComputedStyle(document.body).backgroundImage || '';
            const desired = `url("${this.settings.urls[this.index]}")`;
            if (!current.includes(this.settings.urls[this.index])) {
              console.log('[BackgroundRotator] detected external change, reapplying background');
              this.applyBackground(this.index);
            }
          } catch (e) { /* ignore */ }
        });
        this._bgObserver.observe(document.body, { attributes: true, attributeFilter: ['style'] });
      }

      window.BackgroundRotator = { instance: this };
      console.log(`[BackgroundRotator] started, interval ${this.settings.minutes} min`);
    } catch (e) {
      console.error("[BackgroundRotator] start error:", e);
    }
  }

  stop() {
    try {
      if (this.intervalId) {
        clearInterval(this.intervalId);
        this.intervalId = null;
      }
      if (this._bgObserver) {
        this._bgObserver.disconnect();
        this._bgObserver = null;
      }

      // vorherigen Wert wiederherstellen oder entfernen
      if (this.previous && this.previous.trim() !== "") {
        // vorherigen Wert wieder als inline setzen
        document.documentElement.style.setProperty('--selected-bg', this.previous, 'important');
        document.body.style.removeProperty('background-image'); // optional
        console.log("[BackgroundRotator] restored previous --selected-bg");
      } else {
        // nichts vorher gesetzt -> entferne unsere Inline-Overrides
        document.documentElement.style.removeProperty('--selected-bg');
        document.body.style.removeProperty('background-image');
        document.body.style.removeProperty('background-size');
        document.body.style.removeProperty('background-position');
        document.body.style.removeProperty('background-repeat');
        console.log("[BackgroundRotator] removed --selected-bg and body inline styles");
      }

      delete window.BackgroundRotator;
      console.log("[BackgroundRotator] stopped");
    } catch (e) {
      console.error("[BackgroundRotator] stop error:", e);
    }
  }

  applyBackground(i) {
    const url = this.settings.urls[i];
    if (!url || typeof url !== "string") {
      console.warn(`[BackgroundRotator] skipped invalid url at index ${i}`);
      return;
    }

    // Setze CSS-Variable mit important
    document.documentElement.style.setProperty('--selected-bg', `url("${url}")`, 'important');

    // Setze zusätzlich das body background-image direkt (mit important) — überschreibt viele andere Regeln
    document.body.style.setProperty('background-image', `url("${url}")`, 'important');

    // Optional: setze auch background-size/position falls dein theme das nicht macht
    document.body.style.setProperty('background-size', 'cover', 'important');
    document.body.style.setProperty('background-position', 'center center', 'important');
    document.body.style.setProperty('background-repeat', 'no-repeat', 'important');

    console.log(`[BackgroundRotator] set background to index ${i}: ${url}`);
  }
};
Aktiviere das Plugin in BetterDiscord → Plugins.
Wichtige CSS‑Variablen (Übersicht)
--img_* : einzelne Bild‑URLs (als url("...")).
--selected-bg : wird vom body als Hintergrund benutzt (setze auf var(--img_...) oder eine url(...)).
--popout-radius : Popout / Profil-Boxen Rundung (z. B. 36px).
--app-blur : globaler Blur‑Wert, wird je nach Theme verwendet.
--card-color / --card-color-select : werden für Karten/Blöcke benutzt (kannst du anpassen).
--activeblock-alpha : (wenn verwendet) Alpha für "Jetzt aktiv" Blöcke.
Tipps & Troubleshooting
Tippfehler killen Regeln: Achte z. B. auf rgba(0,0,0,0.06) — rgba(0,0,0,06) ist ungültig und macht die Deklaration nutzlos.
Reihenfolge: Setze deinen :root-Block direkt NACH dem @import-Statement, damit deine Variablen die importierten Werte überschreiben.
Leerzeichen in Dateinamen: Raw‑URLs enthalten %20 für Leerzeichen. Wenn du JavaScript benutzt, verwende encodeURIComponent(filename).
Hotlinking / CORS: GitHub Raw‑URLs funktionieren in der Regel; falls nicht, lade die Bilder auf einen anderen Host (GitHub Pages, Imgur).
Caching: BetterDiscord/Discord kann cachen — Theme neu laden oder Client neu starten (Strg+R).
Wenn eine Regel nicht greift: Discord verwendet dynamische Klassennamen. Nutze robuste Selektoren wie [class*="activity"] oder prüfe in DevTools die tatsächlichen Klassennamen.

Tests: nach Änderungen Theme neu laden (BetterDiscord) und prüfen, ob Hintergrund & Popouts korrekt gerendert werden.

Kontakt / Hinweise
Wenn etwas nicht funktioniert, poste die Fehlermeldung aus der Browser‑DevTools Console oder screenshots der DevTools (Element‑Inspektor), dann helfe ich beim Debuggen.

Viel Spaß beim Anpassen deines Discord‑Hintergrunds!

Code
