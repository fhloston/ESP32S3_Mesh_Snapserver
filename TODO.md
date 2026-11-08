# TODO

Stand 2026-11-08. Nur was zu tun ist. Hintergründe, Messungen und Erledigtes
stehen in [docs/notizen.md](docs/notizen.md).

## Beheben

- **Knackser beim Speichern.** Jede gespeicherte Live-Einstellung hält den
  Server gut 40 ms an, der I2S-Ausgang läuft leer, alle Lautsprecher knacksen
  einmal (gesehen 2026-10-10). Ursache: Beim Schreiben ins Flash schaltet
  ESP-IDF den Cache ab, Code aus dem Flash und das PSRAM stehen so lange.
  Lösung: `CONFIG_SPIRAM_XIP_FROM_PSRAM=y`. Code und Konstanten laufen dann
  aus dem PSRAM, und der Cache bleibt beim Flash-Schreiben an
  (`SPI_FLASH_CACHE_NO_DISABLE` in `spi_flash_os_func_app.c`, IDF 5.4.3).
  Kostet rund 1,5 MB PSRAM (Größe des Images). Einbauen und auf Hardware
  prüfen: Knackser weg, Start, Rechenzeit des Encoders.
- **Einstellungen gehen beim Ausschalten verloren.** Der Server speichert
  Lautstärke, Delay und Namen der Clients erst 3 s nach der letzten Änderung
  (`client_store`). Wer in diesen 3 s am Poti-Schalter ausschaltet, verliert
  die Änderung. Nach dem Fix oben die Wartezeit verkürzen.
- **TinyUSB belegt 10,5 KB internes RAM auf jedem Gerät**, auch ohne
  USB-Audio. FIFO verkleinern oder ins PSRAM legen.
- **Aussetzer, wenn ein Client beitritt.** Nur `server_task` (accept) unter
  die Sender legen, `client_task` gleichauf lassen. Alles pauschal absenken
  war deutlich schlechter (2026-09-19).
- **SnapAnnounce sendet leise.** Mikrofonquellen „Standard-Mikrofon“ und
  „Spracherkennung“ gegen „Telefonat“ vergleichen, dann den Default für
  „Max. Verstärkung“ in `SettingsStore.kt` anpassen.

## Auf Hardware prüfen

- **Strobo:** Ein Tipp, WLED blitzt 2,5 s und spielt danach sofort seinen
  Effekt weiter. WLEDs an mehreren Geräten.
- **Durchsage:** neuen Empfang (Sequenznummer) auf 14E4 und 7788 flashen
  und dort messen.
- **WLED:** mehr als ein WLED; ein WLED hinter einem Gerät der dritten Ebene.
- **USB-Soundkarte unter Windows:** Flashen nach „Neustart zum Flashen“;
  Dauerlauf (FIFO-Füllstand, Ausgleichs-Frames).
- **Root-Wächter:** Server ausschalten, während mehrere Clients laufen, und
  deren Logs 1–2 min mitschneiden.
- **Mesh:** zehn Clients gleichzeitig (`SNAPSERVER_MAX_CLIENTS = 10`); ein
  Relay mit mehreren Kindern (Kette erzwingen, `heap … children=` des Relays
  mitschneiden); ein hängender Zweig am Server (greift das Schließen nach
  3 s aus 44fc601?).
- **Pin-Seite:** Update ändert keine Pins; Pinwechsel samt Neustart;
  Rückfall auf die Standardbelegung nach drei missglückten Starts; Client
  auf Ebene 3.
- **App:** Geräteliste, Einstellungen eines Clients in der App, mit
  eingeschalteten mobilen Daten.
- **Einrichtung:** Passwort unter 8 Zeichen wird abgelehnt; SSID mit 32
  Zeichen; der Einrichtungs-AP schaltet nach 3 Minuten ab.
- **Fehlstarts:** Client mit falschem Passwort öffnet beim achten Start den
  Einrichtungs-AP, auch wenn der Root-Wächter ihn neu startet; Server ohne
  Clients bleibt nach acht Starts im Mesh.
- **Board mit 4 oder 8 MB Flash.**

## Messen

- **Versatz Server ↔ Clients am Lautsprecher.** Ein bekanntes Signal
  abspielen und Server- und Client-Lautsprecher gleichzeitig aufnehmen. Das
  Ergebnis kommt in `SERVER_LEAD_MS` (`audio_i2s.c`).
- **Rechenzeit auf Kern 0 bei Durchsagen:** `tiT` und `wifi` steigen von 8 und
  6 % auf 29 und 25 %. Pakete pro Sekunde und Retransmits messen. Dafür mit
  Laufzeitstatistik bauen (siehe `sdkconfig.defaults`).
- **WLED an BDFC:** Signalstärke des WLED ins Log, um die Ping-Lücken von
  1,5–3 s zuzuordnen.

## Nächstes Release

Release-Notizen schreiben (`docs/releases/<tag>.md`). Seit v0.5.3:

- Opus rechnet mit Festkomma statt Gleitkomma.
- USB-Soundkarte am Server (Schalter in der Web-UI, Neustart zum Flashen).
- WLED: wird erkannt und bekommt die Musik, Link auf der Server-Seite,
  Strobo-Knopf.
- Durchsagen verdecken Funkaussetzer, statt still zu werden.
- Client startet neu, statt Root zu werden (Wächter).
- Ein/Aus-Taster schaltet erst nach 1 s Halten ein.
- Gerätename überall mit zwei MAC-Bytes.
- Der Server zählt keine Fehlstarts mehr und fällt ohne Clients nicht mehr
  in den Einrichtungsmodus. Clients zählen weiter.
- Slave-Takt 80 MHz und Glitch-Filter aus v0.5.2 sind wieder ausgebaut.

## Ideen

- OTA über die Web-Seite. Die Partitionen sind da, es fehlt der Teil in
  der Firmware, siehe Notizen.
- USB-Audio am Client als Mikrofon für einen PC.
- Erkennen, ob das Handy bei Durchsagen direkt am Server hängt.
- Spotify direkt auf den Server, siehe Notizen.
- Strobo im Gleichtakt über alle Geräte (nach der Serveruhr).
