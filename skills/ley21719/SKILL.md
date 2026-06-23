---
name: ley21719
description: Auditar compliance con Ley 21.719 de Protección de Datos Personales en páginas web y repositorios. Usar cuando el usuario pida auditar, revisar o verificar cumplimiento de la ley de datos personales chilena. Incluye scoring ponderado, checklist estandarizada, comparativa con auditorías previas, y generación de snippets de corrección.
license: MIT
compatibility: opencode
metadata:
  audience: legal, compliance, development
  workflow: audit
---

## Qué hace

- Comprende la Ley 21.719 y su checklist estandarizada de 20 requisitos
- Audita páginas web y repositorios contra cada requisito
- Scoring ponderado: no todos los incumplimientos pesan igual
- Estados: ✅ cumplido / 🟡 parcial / ❌ incumplido
- Compara contra auditorías previas (Engram)
- Genera snippets de código de corrección
- Genera informe HTML completo con mapa de riesgo visual

## Checklist Estándar Ley 21.719

Usar esta checklist fija en TODA auditoría. Cada requisito tiene un peso para el score:

| # | Requisito | Artículo | Peso | Tipo auditoría |
|---|-----------|----------|------|----------------|
| 1 | Política de privacidad visible y accesible | Art. 8 letra d | 8 | web+repo |
| 2 | Consentimiento previo, informado e inequívoco | Art. 6, 7 | 10 | web+repo |
| 3 | Informar finalidad del tratamiento | Art. 8 letra b | 8 | web+repo |
| 4 | Informar derechos ARCO+ (acceso, rectificación, cancelación, oposición, portabilidad, bloqueo) | Art. 8 letra e, Art. 12-17 | 8 | web+repo |
| 5 | Protección reforzada de datos sensibles (RUT, salud, biometría, orientación sexual) | Art. 4, 9 | 10 | web+repo |
| 6 | Gestor de cookies con consentimiento previo granular | Art. 6 | 6 | web |
| 7 | Medidas de seguridad técnicas (XSS, CSP, HSTS, inyección SQL) | Art. 5, 9 ter | 7 | web+repo |
| 8 | No exponer PII en logs ni storage sin protección | Art. 5, 9 ter | 7 | web+repo |
| 9 | Manejo seguro de autenticación (tokens HttpOnly, MFA) | Art. 5, 9 ter | 7 | web+repo |
| 10 | Credenciales y secretos fuera del código fuente | Art. 5 | 8 | repo |
| 11 | Cifrado de datos en tránsito (HTTPS) | Art. 5, 9 ter | 6 | web+repo |
| 12 | Cifrado o seudonimización de PII en reposo | Art. 5, 9 ter | 6 | repo |
| 13 | Informar transferencias internacionales de datos | Art. 8 letra f, Art. 23-24 | 5 | web+repo |
| 14 | Designar DPO / Delegado de Protección de Datos | Art. 30 quáter | 4 | web+repo |
| 15 | Mecanismo para ejercicio de derechos ARCO+ | Art. 12-17 | 8 | web+repo |
| 16 | Registro de actividades de tratamiento (audit log) | Art. 9 quáter | 5 | repo |
| 17 | Notificación de brechas de seguridad | Art. 9 quinquies | 6 | repo |
| 18 | Política de retención y eliminación de datos | Art. 8 letra g, Art. 18 | 5 | repo |
| 19 | Verificación de integridad en webhooks/APIs entrantes | Art. 5 | 5 | repo |
| 20 | Rate limiting y protección contra fuerza bruta | Art. 5, 9 ter | 4 | repo |

### Cálculo de Score

```
score = sum(peso_en_requisitos_cumplidos) / sum(peso_total_requisitos_aplicables) * 100
```

- ✅ cumple → peso completo
- 🟡 parcial → peso * 0.5
- ❌ incumple → 0

Niveles: 0-39% bajo (rojo) | 40-69% medio (amarillo) | 70-100% alto (verde)

## Workflow

### 0. Buscar auditorías previas

Siempre ANTES de auditar:

```
engram search "auditoria ley21719 [nombre]" --top 5
```

- Si hay auditoría previa, comparar progreso (qué se ha arreglado, qué sigue pendiente)
- Incluir sección "Evolución" en el informe con diff vs auditoría anterior

### 1. Cargar conocimiento de la ley

- Usar `webfetch` para obtener texto actualizado desde leychile.cl
- Si Engram ya tiene artículos precargados, usar esos primero:
  ```
  engram recall "ley 21719 articulo" --raw
  ```
- Si no existen, extraer artículos clave y guardarlos:
  ```
  engram store "Ley 21.719 Art. [n]: [resumen]" --type semantic --importance 1.0
  ```
- Artículos a precargar: 1, 3, 4, 5, 6, 7, 8, 9, 9 ter, 9 quáter, 9 quinquies, 12-17, 18, 23-24, 30 quáter

### 2. Auditar página web

Para cada URL:

1. Fetch del sitio con `webfetch`
2. Buscar y extraer: política de privacidad, aviso de consentimiento, formularios de datos, cookies, third-party scripts
3. Verificar contra cada requisito de la checklist que aplique (tipo "web" o "web+repo")
4. Asignar estado: ✅ / 🟡 / ❌
5. Para cada ❌ o 🟡, crear hallazgo con severidad, artículo, y snippet de corrección si aplica
6. Almacenar resultado en Engram

### 3. Auditar repositorio

Para cada repo:

1. Inspeccionar estructura, dependencias, config
2. Buscar patrones de riesgo con grep/glob:
   - `.env` commiteado, `config.js` con credenciales hardcodeadas
   - `console.log` / `logger.info` con PII
   - `localStorage.setItem`, `innerHTML`, `document.cookie`
   - Tokens en storage no seguro
   - Ausencia de HTTPS, CSP, HSTS, rate limiting
   - Cifrado ausente en DB queries con PII
   - Webhook sin verificación de firma
   - Email con passwords en texto plano
3. Verificar contra cada requisito de la checklist que aplique (tipo "repo" o "web+repo")
4. Asignar estado: ✅ / 🟡 / ❌
5. Para cada ❌ o 🟡, crear hallazgo con snippet de corrección
6. Almacenar resultado en Engram

### 4. Generar snippets de corrección

Para cada hallazgo, incluir un snippet de código que solucione el problema:

- **Tokens en localStorage → cookies HttpOnly**: snippet de interceptor Angular + middleware Express
- **console.log en producción**: snippet de logger condicional
- **Sin CSP**: snippet de vercel.json con headers
- **JWT secret débil**: snippet de generación con crypto
- **Password por email**: snippet de flujo con token de un solo uso
- **Webhook sin firma**: snippet de verificación MercadoPago
- **Sin rate limiting**: snippet de express-rate-limit
- **innerHTML sin sanitizar**: snippet con DomSanitizer Angular
- **Sin audit logging**: snippet de winston/pino setup

### 5. Generar informe HTML

Generar archivo HTML con:

- **Header**: nombre, fecha, tipo, tecnología
- **Si hay auditoría previa**: sección "Evolución" con diff (evolución de score, hallazgos resueltos, nuevos hallazgos)
- **Score ponderado** con barra de progreso
- **Mapa de riesgo**: grilla visual con los 20 requisitos × severidad
- **Checklist** con estados ✅/🟡/❌, artículo, peso
- **Puntos a reparar**: hallazgos con severidad, artículo, descripción, recomendación, snippet de corrección
- **Plan de acción**: pasos ordenados por severidad × peso (mayor impacto primero), con estimación de tiempo
- **Resumen ejecutivo**: 3-5 líneas para management

Estructura del HTML:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Auditoría Ley 21.719 - [nombre]</title>
  <style>
    body { font-family: system-ui, sans-serif; max-width: 960px; margin: 2rem auto; padding: 0 1rem; color: #1a1a1a; }
    h1 { border-bottom: 3px solid #2563eb; padding-bottom: .5rem; }
    h2 { margin-top: 2.5rem; color: #1e40af; }
    .score { font-size: 3.5rem; font-weight: bold; text-align: center; margin: 2rem 0 .5rem; }
    .score.bajo { color: #dc2626; } .score.medio { color: #f59e0b; } .score.alto { color: #16a34a; }
    .bar-bg { background: #e5e7eb; height: 24px; border-radius: 12px; overflow: hidden; margin: 1rem 0 2rem; }
    .bar-fill { height: 100%; border-radius: 12px; background: linear-gradient(90deg, #dc2626, #f59e0b, #16a34a); }
    .resumen { text-align: center; font-size: 1.1rem; color: #555; margin-bottom: 2rem; }
    .ejecutivo { background: #f0f9ff; border: 1px solid #bfdbfe; border-radius: 8px; padding: 1.25rem; margin: 2rem 0; }
    .evolucion { background: #f0fdf4; border: 1px solid #bbf7d0; border-radius: 8px; padding: 1rem; margin: 1.5rem 0; }
    .evolucion .mejora { color: #16a34a; } .evolucion .empeora { color: #dc2626; }
    .mapa-riesgo { display: grid; grid-template-columns: repeat(auto-fill, minmax(140px, 1fr)); gap: .5rem; margin: 1.5rem 0; }
    .mapa-item { padding: .5rem; border-radius: 6px; text-align: center; font-size: .8rem; }
    .mapa-ok { background: #dcfce7; color: #166534; }
    .mapa-parcial { background: #fef9c3; color: #854d0e; }
    .mapa-fail { background: #fecaca; color: #991b1b; }
    table { width: 100%; border-collapse: collapse; margin: 1.5rem 0; }
    th, td { text-align: left; padding: .65rem; border-bottom: 1px solid #e5e7eb; }
    th { background: #f3f4f6; font-weight: 600; }
    .ok { color: #16a34a; } .parcial { color: #ca8a04; } .fail { color: #dc2626; }
    .hallazgo { margin: .75rem 0; padding: 1rem; border-radius: 8px; }
    .hallazgo h4 { margin: 0 0 .35rem; }
    .hallazgo .meta { font-size: .85rem; color: #666; }
    .hallazgo .recom { padding: .5rem .75rem; background: rgba(37,99,235,.08); border-radius: 6px; margin-top: .5rem; }
    .hallazgo .snippet { background: #1e293b; color: #e2e8f0; padding: .75rem; border-radius: 6px; margin-top: .5rem; font-family: monospace; font-size: .8rem; overflow-x: auto; white-space: pre; }
    .severidad-critica { background: #fef2f2; border-left: 4px solid #dc2626; }
    .severidad-alta { background: #fffbeb; border-left: 4px solid #f59e0b; }
    .severidad-media { background: #f0f9ff; border-left: 4px solid #2563eb; }
    .severidad-baja { background: #f9fafb; border-left: 4px solid #9ca3af; }
    .tags { display: flex; gap: .5rem; flex-wrap: wrap; margin: 1rem 0; }
    .tag { padding: .25rem .75rem; border-radius: 999px; font-size: .8rem; font-weight: 500; }
    .tag-critica { background: #fef2f2; color: #dc2626; } .tag-alta { background: #fffbeb; color: #f59e0b; }
    .tag-media { background: #f0f9ff; color: #2563eb; } .tag-baja { background: #f3f4f6; color: #6b7280; }
    .plan-item { padding: .5rem 0; border-bottom: 1px solid #f1f5f9; }
    footer { margin-top: 3rem; padding-top: 1rem; border-top: 1px solid #e5e7eb; font-size: .85rem; color: #999; text-align: center; }
  </style>
</head>
<body>
  <h1>Auditoría Ley 21.719</h1>
  <!-- meta info, advertencia si credenciales expuestas -->

  <!-- Si auditoría previa existe -->
  <div class="evolucion">
    <h3>Evolución vs auditoría anterior</h3>
    <p>Score anterior: [X]% → Actual: [Y]% (<span class="[mejora|empeora]">[+/-N]</span>)</p>
    <p>Hallazgos resueltos: [n] | Nuevos hallazgos: [n] | Pendientes: [n]</p>
  </div>

  <div class="score [bajo|medio|alto]">[score]%</div>
  <div class="bar-bg"><div class="bar-fill" style="width:[score]%"></div></div>
  <div class="resumen">[cumplidos] de [total] requisitos cumplidos | [hallazgos] hallazgos</div>

  <div class="ejecutivo">
    <h3>Resumen Ejecutivo</h3>
    <p>[3-5 líneas para management: riesgos principales, prioridades, estimación de esfuerzo total]</p>
  </div>

  <h2>Mapa de Riesgo</h2>
  <div class="mapa-riesgo">
    <!-- Cada requisito como cajita colorizada -->
    <div class="mapa-item mapa-[ok|parcial|fail]">[#] [requisito corto]</div>
  </div>

  <h2>Checklist</h2>
  <table>
    <tr><th>#</th><th>Requisito</th><th>Art.</th><th>Peso</th><th>Estado</th></tr>
    <tr><td>[n]</td><td>[desc]</td><td>[art]</td><td>[peso]</td><td class="[ok|parcial|fail]">[✅|🟡|❌]</td></tr>
  </table>

  <h2>Puntos a Reparar</h2>
  <div class="tags"><!-- tags por severidad --></div>
  <!-- por cada hallazgo -->
  <div class="hallazgo severidad-[critica|alta|media|baja]">
    <h4>[emoji] [título]</h4>
    <div class="meta"><strong>Archivo:</strong> [path] · <strong>Artículo:</strong> [art] · <strong>Severidad:</strong> [sev]</div>
    <p>[descripción]</p>
    <div class="recom"><strong>Recomendación:</strong> [texto]</div>
    <div class="snippet">[código de corrección si aplica]</div>
  </div>

  <h2>Plan de Acción</h2>
  <ol>
    <li class="plan-item"><strong>[severidad emoji] [acción]</strong> — [detalle] (~[horas] horas)</li>
  </ol>

  <footer>Generado por Ley 21.719 Skill · OpenCode</footer>
</body>
</html>
```

Guardar como `auditoria-ley21719-[nombre].html`.

## Engram

Usar Engram como memoria persistente:

```
# Precargar artículos de la ley
engram store "Ley 21.719 Art. [n]: [resumen]" --type semantic --importance 1.0

# Guardar hallazgo
engram store "Auditoría [sitio/repo]: [hallazgo] - [artículo] - [severidad]" --type semantic --importance 0.8

# Guardar resultado de auditoría (para comparativa futura)
engram store "Resultado auditoría [nombre]: score=[X]% hallazgos=[n] fecha=[fecha]" --type semantic --importance 0.9

# Buscar auditorías previas
engram search "resultado auditoria [nombre]" --top 5

# Recordar artículos clave
engram recall "ley 21719 articulo [n]" --raw
```
