# WebGPU-Migration

Umstellung des Viewers von WebGL2-only auf **WebGPU mit WebGL2-Fallback**.

## Was geändert wurde

1. **Engine-Version** (`index.html`, `<script src>`): `playcanvas@2.17.0` → `playcanvas@2.19.7`
   (das WebGPU-Mobile-Update steckt in der neueren Engine).
2. **App-Init** umgestellt von `new pc.Application(canvas, {...})` (WebGL2-only, synchron)
   auf den asynchronen Pfad:
   - `pc.createGraphicsDevice(canvas, { deviceTypes: [WEBGPU, WEBGL2], ... })`
     → versucht zuerst WebGPU, fällt automatisch auf WebGL2 zurück.
   - `pc.AppBase` + `pc.AppOptions` mit manuell registrierten Component-Systemen
     (`Render`, `Camera`, `Light`, `GSplat`), Resource-Handler `pc.GSplatHandler`
     und `xr: pc.XrManager` (Letzteres ist **zwingend** für AR).
   - Kein `glslang`/`twgsl` nötig, da nur Engine-eigene Shader verwendet werden.
3. **Renderer-Anzeige**: `showRendererBadge()` zeigt oben links kurz `Renderer: WebGPU`
   bzw. `Renderer: WebGL2` an (mobil ohne DevTools sichtbar). Zusätzlich `console.log`.

### WebGPU ↔ AR: gegenseitiger Ausschluss (wichtig!)

WebXR **immersive-AR braucht einen WebGL-Context**. WebGPU + WebXR ist noch
experimentell (nur Chrome Canary 135+ mit Flags „WebXR/WebGPU Bindings"). Auf
normalem Handy-Chrome lässt sich mit einem WebGPU-Device **keine AR-Session starten**.

Deshalb wählt der Viewer den Renderer **automatisch**:
- Gerät kann `immersive-ar` (`navigator.xr.isSessionSupported('immersive-ar')`)
  → **WebGL2** (AR funktioniert).
- Sonst → **WebGPU** (bessere Performance, z. B. Desktop ohne AR).

Folge: AR-fähige Handys laufen auf WebGL2 (Badge zeigt `WebGL2`), nicht auf WebGPU.
Das ist aktuell unvermeidbar. Wer WebGPU auf dem Handy erzwingen will, müsste AR
aufgeben (dann `deviceTypes` fest auf `[WEBGPU, WEBGL2]` setzen).

### Wichtige Engine-2.19-Stolpersteine (per Testlauf gefunden)

- **`pc.TextureHandler` muss registriert sein** — `.sog`-Splats laden intern mehrere
  `.webp`-Sub-Texturen. Ohne den Handler bleiben sie ohne Ressource → kein Splat.
- **`gsplat`-Component mit `unified: false`** — die neue Engine rendert Splats per Default
  über das „unified gsplat"-System, das die klassische `instance.meshInstance.aabb` nicht
  mehr füllt. Unser Code braucht diese AABB (Kamera-Zentrierung & AR-Platzierung), darum
  erzwingen wir den klassischen Per-Entity-Pfad.

Der restliche Code (Orbit, Splat-Laden, AR-Platzierung, Rotation) ist **unverändert** —
`AppBase` hat dieselbe Laufzeit-API wie `Application`.

## Verifizieren

- Seite laden → Badge oben links zeigt den aktiven Renderer.
- Auf modernen Handys (Android Chrome, iOS 26+ Safari) sollte `WebGPU` erscheinen,
  sonst `WebGL2`.

## Revert (falls Probleme)

Der stabile WebGL2-Stand ist als Git-Tag **`pre-webgpu`** gesichert.

Nur `index.html` zurücksetzen:
```bash
git checkout pre-webgpu -- index.html
git commit -m "revert: back to WebGL2-only build"
git push
```

Oder den kompletten Stand ansehen/auschecken:
```bash
git show pre-webgpu:index.html      # Inhalt ansehen
git checkout pre-webgpu             # kompletter Stand (detached HEAD)
```

Tag-Commit: `6577806` ("Stable WebGL2 build before WebGPU migration").
