# MoneyPrinterTurbo → Coolify (VPS Hostinger) — Guía Resia

Despliegue tuneado para el VPS `srv1574889` (7.8 GB RAM compartidos con Coolify +
App Gastos + Resia). Objetivo: generar shorts de Resia **sin tumbar producción**.

---

## 0. Antes de empezar — 2 keys que solo tú puedes sacar

| Key | Dónde | Costo |
|-----|-------|-------|
| **Pexels** | https://www.pexels.com/api/ → registrarte → "Your API Key" | Gratis (200 req/h) |
| **Cerebras** | https://cloud.cerebras.ai/ → API Keys (la misma de App Gastos/Resia) | Gratis |

---

## 1. Crear la Application en Coolify

1. Coolify → **+ New** → **Application** → **Docker Compose**.
2. Source: **GitHub App `coolify-prestrepo`** → repo **`PRestrepo2312/MoneyPrinterTurbo`** → branch `main`.
3. **Compose file path:** `docker-compose.coolify.yml` (NO el `docker-compose.yml` original).

## 2. Fixes recurrentes Coolify v4 (revisar SIEMPRE — aprendido con Gastos/Resia)

- [ ] **Ports Exposes** → `8501` (no el default `3000`).
- [ ] **Domains** → con `https://` al inicio. Ej: `https://mpt.resia.cloud`.
- [ ] **Custom Docker Options** → quitar cualquier `--hostname=...`.
- [ ] **Container Labels** → Reset to defaults.
- [ ] **Pre-deployment commands** → vacío (Coolify mete `php artisan migrate` por default).
- [ ] **Watch Paths** → `docker-compose.coolify.yml` y `Dockerfile` (para no redeployar por cambios irrelevantes).
- [ ] **Healthcheck** → si falla, recordar que el UI OFF no desactiva el del Dockerfile.

## 3. Persistencia (Storages) — IMPORTANTE

Dos almacenamientos persistentes para que las keys y los videos sobrevivan a redeploys:

1. **File Mount**
   - Destination: `/MoneyPrinterTurbo/config.toml`
   - Contenido: copia/pega **todo** `config.resia.toml` del repo, y rellena:
     - `pexels_api_keys = ["TU_KEY_PEXELS"]`
     - `openai_api_key = "TU_KEY_CEREBRAS"`
2. **Volume**
   - Destination: `/MoneyPrinterTurbo/storage`
   - (videos generados + cache de clips de Pexels)

## 4. Límites de recursos — YA vienen en el compose

```yaml
limits: { cpus: "1.5", memory: 3g }
```
El render será más lento que en local, pero Resia/Gastos no se quedan sin CPU.
Si ves que igual sufre producción, bájalo a `cpus: "1.0"` y `memory: 2g`.

## 5. 🔒 Auth — EL WEBUI NO TIENE LOGIN

Si lo expones tal cual, cualquiera usa tu Cerebras/Pexels. Elegir UNA:

- **Cloudflare Access (recomendado)** — ya usas Cloudflare. Zero Trust gratis (≤50 users):
  1. Cloudflare → Zero Trust → Access → Applications → Add → Self-hosted.
  2. Subdominio `mpt.resia.cloud`, política: email OTP a tu correo (+ del equipo Resia).
  3. ⚠️ El record `mpt.resia.cloud` debe estar **proxied (naranja)**, no "DNS only".
     Tu wildcard sigue DNS-only; este subdominio puntual proxied no lo rompe (SSL Full).
- **No exponerlo** — sin Domain en Coolify; accedes por túnel SSH cuando lo necesites.
- **Basic Auth Traefik** — frágil en Coolify v4 (regenera labels). No recomendado.

## 6. Build — ojo con los mirrors chinos

El `Dockerfile` intenta mirrors de Aliyun/Tsinghua primero (lento/flaky desde Hostinger).
Hace fallback a Debian/PyPI default, pero el primer build puede tardar. Si molesta,
pídeme que parchee el Dockerfile del fork para ir directo a mirrors default.

## 7. Primer test (validar que todo corre)

En el WebUI → tema: `"Cómo administrar un conjunto residencial sin estrés"`.
- Idioma: Español. Voz: `es-CO-GonzaloNeural` o `es-CO-SalomeNeural` (Colombia).
- Proporción: 9:16. Duración clip: 3s. Que genere guion → revisa → "Generate Video".

---

## Kit de contenido Resia (faceless shorts → demo)

**Voces ES-CO:** `es-CO-GonzaloNeural` (m), `es-CO-SalomeNeural` (f).

**Términos Pexels que funcionan** (el LLM los genera, pero si quieres forzar):
`apartment building`, `residential community`, `modern condo`, `meeting`, `city skyline`, `keys handover`, `accountant desk`.

**Temas (alimentan las mismas keywords SEO del plan Astro):**
1. ¿Qué es la Ley 675 y por qué te afecta como administrador?
2. Cómo hacer una asamblea virtual válida en Colombia.
3. 3 errores al calcular la cuota de administración.
4. Coeficiente de copropiedad explicado en 60 segundos.
5. Comité de convivencia: para qué sirve realmente.
6. Cómo digitalizar la cartera de tu conjunto.

**CTA fijo (cierre + descripción):**
> "Administra tu copropiedad sin Excel ni dolores de cabeza → **resia.cloud**. Agenda un demo gratis."

Cada short es un mini-anuncio que apunta a la landing/demo. No dependes del algoritmo
para "descubrirte": tú distribuyes (grupos de administradores, LinkedIn, Reels).
