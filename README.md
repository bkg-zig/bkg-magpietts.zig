# BKG MagpieTTS Alpha — native Zig + Magpie

Eigenständiger Projekt-Root: /srv/bkg/services/bkg-magpietts.zig/alpha

Die bisherigen Arbeiten aus bkg-magpietts.zig (HTTP-/Realtime-Server und React/WASM-UI) sowie bkg-tts.zig (native CLI, WAV-Ausgabe und Batch-Synthese) sind in einem Build zusammengeführt. CLI und HTTP-Server verwenden dieselbe native Magpie-C-ABI aus magpie-tts.cpp.

## Laufzeitprinzip

- Zig 0.16 für HTTP-Server, CLI, REST, WebSocket, Request-Logs und Aussprachewörterbuch.
- Lokale CPU-Inferenz über libmagpie-tts.so und GGUF.
- Kein Python, NeMo, Riva-Relay oder Shell-Fallback für Synthese.
- React/Vite als Browser-UI; ein freestanding WASM-Modul prüft den API-Vertrag.
- Node ist ausschließlich ein Build-Werkzeug, kein Runtime-Server.
- C++/ggml wird als native CPU-Library gebaut; CUDA-/Metal-/Vulkan-/HIP-Backends sind ausgeschaltet.

## Bauen und testen

- zig build baut zig-out/bin/bkg-magpietts und zig-out/bin/bkg-tts.
- zig build test führt Server-/API-/WASM- und CLI-Unit-Tests aus.
- zig build wasm erzeugt zig-out/bin/bkg-magpietts-wasm.wasm.
- scripts/build-frontend.sh baut WASM und das React-Frontend.
- scripts/test.sh führt Zig-Tests, WASM-Validierung und den Frontend-Produktionsbuild aus.
- scripts/build-image.sh erstellt aus diesem Alpha-Root, dem BKG-Framework und dem lokalen C++-Quellbaum ein CPU-only Ubuntu-22.04-Image.
- Docker/Traefik-Konfigurationen liegen in docker/.

## CLI

zig-out/bin/bkg-tts say "Hallo Welt" --voice Aria --lang de -o /tmp/hallo.wav

zig-out/bin/bkg-tts read-batch texte.txt --voice Aria --lang de -o /tmp/sprecher

BKG_MAGPIE_MODEL ist erforderlich; BKG_MAGPIE_LIB_PATH ist optional, wenn libmagpie-tts.so unter /usr/local/lib/libmagpie-tts.so liegt. Unkonfigurierte Engines melden einen Fehler statt einen Erfolg zu erfinden.

## Konfiguration des Servers

- MAGPIE_HOST, Standard 0.0.0.0
- MAGPIE_PORT, Standard 8092
- BKG_MAGPIE_MODEL, Pfad zum GGUF
- BKG_MAGPIE_LIB_PATH, Pfad zur nativen C-ABI-Library
- MAGPIE_WEB_DIST, Standard /srv/magpie/web/dist
- MAGPIE_PRONUNCIATION_PATH, persistente JSON-Datei

Das Live-Modell wird aus dem Modell-Volume eingebunden und ist nicht Teil des Source-Verzeichnisses.

## Referenzen

- TASKS.md: aktueller Umsetzungsstand, Abhängigkeiten und Abnahmekriterien.
- docs/SOURCE-INVENTORY.md: Analyse und Herkunft der übernommenen Teile.
- docs/API-COMPAT.md: REST-/WebSocket-Vertrag und offene Paritätsgrenzen.
- docs/DEPLOYMENT.md: Image-Build, Paralleltest und Rollout.
# bkg-magpietts.zig
