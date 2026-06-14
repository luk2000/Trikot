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
