# Ley 21.719 - Skill para OpenCode

Skill de auditoría de cumplimiento de la **Ley 21.719 de Protección de Datos Personales de Chile** para [OpenCode](https://opencode.ai).

Audita páginas web y repositorios de código, genera un informe HTML con scoring ponderado, mapa de riesgo, checklist estandarizada y snippets de corrección listos para implementar.

## Checklist de 20 Requisitos

El skill evalúa contra una checklist fija y estandarizada (los mismos 20 requisitos en cada auditoría, para que sean comparables):

| # | Requisito | Peso |
|---|-----------|------|
| 1 | Política de privacidad visible y accesible | 8 |
| 2 | Consentimiento previo, informado e inequívoco | 10 |
| 3 | Informar finalidad del tratamiento | 8 |
| 4 | Informar derechos ARCO+ | 8 |
| 5 | Protección reforzada de datos sensibles | 10 |
| 6 | Gestor de cookies con consentimiento granular | 6 |
| 7 | Medidas de seguridad técnicas | 7 |
| 8 | No exponer PII en logs ni storage | 7 |
| 9 | Manejo seguro de autenticación | 7 |
| 10 | Credenciales fuera del código fuente | 8 |
| 11 | Cifrado en tránsito (HTTPS) | 6 |
| 12 | Cifrado o seudonimización de PII en reposo | 6 |
| 13 | Informar transferencias internacionales | 5 |
| 14 | Designar DPO / Delegado de Protección de Datos | 4 |
| 15 | Mecanismo para ejercicio de derechos ARCO+ | 8 |
| 16 | Registro de actividades de tratamiento | 5 |
| 17 | Notificación de brechas de seguridad | 6 |
| 18 | Política de retención y eliminación de datos | 5 |
| 19 | Verificación de integridad en webhooks/APIs | 5 |
| 20 | Rate limiting y protección contra fuerza bruta | 4 |

Cada requisito se evalúa con 3 estados: ✅ cumplido / 🟡 parcial / ❌ incumplido.

El scoring es **ponderado**: no todos los incumplimientos pesan igual. Un incumplimiento en consentimiento (peso 10) impacta más que uno en rate limiting (peso 4).

## Instalación

### Opción 1: Copia manual

Copia la carpeta `skills/ley21719/` a tu directorio de skills de OpenCode:

```bash
# Linux/macOS
cp -r skills/ley21719/ ~/.config/opencode/skills/ley21719/

# Windows (PowerShell)
Copy-Item -Recurse -Path "skills\ley21719" -Destination "$env:USERPROFILE\.config\opencode\skills\ley21719"
```

### Opción 2: Clonar el repo y linkear

```bash
git clone https://github.com/jago-gif/ley21719-opencode-skill.git
# Linux/macOS
ln -s $(pwd)/ley21719-opencode-skill/skills/ley21719 ~/.config/opencode/skills/ley21719
# Windows: crear junction
New-Item -ItemType Junction -Path "$env:USERPROFILE\.config\opencode\skills\ley21719" -Target "$PWD\skills\ley21719"
```

## Uso

Una vez instalado, el skill se activa automáticamente en OpenCode cuando pides una auditoría de la Ley 21.719.

### Auditar una página web

```
Audita https://midominio.cl contra la Ley 21.719
```

### Auditar un repositorio

```
Audita el repo en ./mi-proyecto contra la Ley 21.719
```

### Auditar ambos

```
Audita https://midominio.cl y el repo en ./mi-proyecto contra la Ley 21.719 genera informe HTML
```

## Informe HTML

El skill genera un archivo HTML autocontenido con:

- **Score ponderado** con barra de progreso visual (0-39% rojo, 40-69% amarillo, 70-100% verde)
- **Mapa de riesgo**: grilla visual de los 20 requisitos con colores
- **Checklist completa** con estados, artículos de la ley y pesos
- **Hallazgos** con severidad (crítica/alta/media/baja), artículo violado, descripción
- **Snippets de corrección**: código listo para copiar que soluciona cada hallazgo
- **Plan de acción**: pasos ordenados por impacto (severidad × peso)
- **Resumen ejecutivo**: 3-5 líneas para management
- **Evolución** (si hay auditorías previas): diff de score y hallazgos

Ejemplo de hallazgo con snippet:

```html
<div class="hallazgo severidad-critica">
  <h4>🔴 Credenciales en .env commiteado</h4>
  <div class="meta"><strong>Archivo:</strong> .env · <strong>Artículo:</strong> Art. 5 · <strong>Severidad:</strong> Crítica</div>
  <p>Se encontró archivo .env con credenciales de producción en el repositorio git.</p>
  <div class="recom"><strong>Recomendación:</strong> Rotar todas las credenciales inmediatamente y agregar .env a .gitignore</div>
  <div class="snippet"># Agregar a .gitignore
.env
.env.local
.env.production</div>
</div>
```

## Memoria persistente con Engram (opcional)

Si tenés [Engram](https://github.com/nicholasgriffintn/engram) configurado como MCP server en OpenCode:

- Las auditorías previas se buscan automáticamente para comparar evolución
- Los artículos de la Ley 21.719 se pre-cachean para auditorías más rápidas
- Los resultados se almacenan para追踪abilidad

Si no tenés Engram, el skill funciona igual, pero sin comparativa entre auditorías.

## Snippets de corrección incluidos

El skill genera código de corrección para los hallazgos más comunes:

| Hallazgo | Snippet |
|----------|---------|
| Tokens en localStorage | Interceptor Angular + middleware Express para cookies HttpOnly |
| console.log en producción | Logger condicional con flag de entorno |
| Sin CSP | Configuración de headers de seguridad |
| JWT secret débil | Generación con `crypto.randomBytes()` |
| Password por email | Flujo con token de un solo uso |
| Webhook sin firma | Verificación de firma MercadoPago |
| Sin rate limiting | Configuración de `express-rate-limit` |
| innerHTML sin sanitizar | Uso de `DomSanitizer` en Angular |
| Sin audit logging | Setup de winston/pino |

## Artículos clave de la Ley 21.719

- **Art. 5**: Principio de seguridad - medidas técnicas y organizativas
- **Art. 6**: Consentimiento previo, informado e inequívoco
- **Art. 7**: Formas de obtener el consentimiento
- **Art. 8**: Información que debe entregarse al titular
- **Art. 9**: Datos sensibles - tratamiento reforzado
- **Art. 9 ter**: Medidas de seguridad
- **Art. 9 quáter**: Registro de actividades de tratamiento
- **Art. 9 quinquies**: Notificación de brechas
- **Art. 12-17**: Derechos ARCO+
- **Art. 18**: Eliminación de datos
- **Art. 23-24**: Transferencias internacionales
- **Art. 30 quáter**: Delegado de protección de datos

## Licencia

MIT - ver [LICENSE](LICENSE)
