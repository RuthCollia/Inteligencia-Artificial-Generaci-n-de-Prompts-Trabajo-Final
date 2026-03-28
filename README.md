# 🌸 Sistema de Atención al Cliente con Prompt Engineering
### Coberturas Florales — Proyecto Final · Prompt Engineer · CoderHouse

> **Autora:** L. Ruth Collia  
> **Curso:** Prompt Engineer — CoderHouse  
> **Año:** 2026

---

## 📋 Índice

- [Resumen](#-resumen)
- [Problema y Contexto](#-problema-y-contexto)
- [Propuesta de Solución](#-propuesta-de-solución)
- [Arquitectura del Sistema](#-arquitectura-del-sistema)
- [System Prompt Base](#-system-prompt-base)
- [Prompts Implementados](#-prompts-implementados)
- [Ejemplos de Ejecución](#-ejemplos-de-ejecución)
- [Prompts Texto-Imagen](#-prompts-texto-imagen)
- [Comparación Claude vs Gemini](#-comparación-claude-vs-gemini)
- [Resultados](#-resultados)
- [Conclusiones](#-conclusiones)
- [Estructura del Repositorio](#-estructura-del-repositorio)

---

## 🗂 Resumen

**Coberturas Florales** recibe diariamente consultas repetitivas por WhatsApp sobre envíos, pagos, suscripciones y reclamos. Responder manualmente consume tiempo, genera inconsistencias de tono y crea riesgo de prometer condiciones que no están en las políticas del negocio.

Este proyecto diseña e implementa un **sistema de prompts** que:

- ✅ Clasifica la intención del cliente (tag)
- ✅ Genera respuestas de 2–3 líneas con CTA listos para WhatsApp
- ✅ Audita que ninguna respuesta prometa algo fuera de las políticas
- ✅ Compara resultados entre **Claude** y **Gemini**
- ✅ Genera piezas visuales para Instagram y WhatsApp (texto-imagen)

---

## 🔍 Problema y Contexto

### El negocio

**Coberturas Florales** ofrece dos servicios principales:

| Servicio | Descripción |
|---|---|
| **Suscripción a domicilio** | Ramos/arreglos de 24 varas. Frecuencia semanal, quincenal o mensual. Solo Mercado Pago. CABA sin costo, Zona Norte GBA hasta Pilar. |
| **Suscripción a cementerios** | Ofrendas florales con foto de comprobación. CABA y PBA hasta tercer cordón. Múltiples medios de pago. |

### El problema

```
Consulta WhatsApp (x30/día)
        ↓
Respuesta manual inconsistente
        ↓
Riesgo de prometer zonas/horarios fuera de política
        ↓
Pérdida de tiempo + clientes confundidos
```

**Tres consecuencias concretas:**
1. **Tiempo operativo**: 3–8 minutos por respuesta manual
2. **Inconsistencias**: distintas personas responden distinto al mismo cliente
3. **Promesas no respaldadas**: sin guardrails, se confirman condiciones que no existen

---

## 💡 Propuesta de Solución

Un sistema de **Prompt Chaining en 3 etapas** + módulo de texto-imagen:

```
Mensaje cliente
      │
      ▼
┌─────────────────┐
│  PROMPT I       │  ← Clasificador de intención
│  Clasifica tag  │     Output: JSON { tag, dato_faltante, nivel }
│  Detecta dato   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  PROMPT B/C/F/H │  ← Generador de respuesta
│  Genera WhatsApp│     Output: JSON { respuesta, cta, dato_a_pedir }
│  2–3 líneas+CTA │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  PROMPT J       │  ← Auditor QA
│  Verifica vs    │     Output: JSON { ok, frases_en_riesgo, respuesta_corregida }
│  políticas      │
└─────────────────┘
```

---

## 🏗 Arquitectura del Sistema

| Módulo | Prompt | Función |
|---|---|---|
| System Prompt Base | — | Contexto y políticas compartidas por todos los prompts |
| Clasificador | Prompt I | Tag + dato faltante + nivel |
| Generador principal | Prompt B | Respuesta WhatsApp genérica |
| Urgencia | Prompt C | "¿Llega hoy?" |
| Sin stock | Prompt D | Alternativas florales |
| Objeción precio | Prompt F | "Está caro" |
| Confianza | Prompt G | "¿Tenés fotos?" |
| Reclamo | Prompt H | "No era como la foto" |
| Auditor QA | Prompt J | Verificación de cumplimiento |
| Texto-imagen | Prompts K1/K2/K3 | Piezas para IG y WhatsApp |

---

## 🔧 System Prompt Base

> **Mejora clave:** Un único system prompt con el contexto del negocio, compartido por todos los prompts. Evita repetir reglas, mantiene tono uniforme y facilita mantenimiento.

Ver archivo completo: [`prompts/00_system_prompt_base.md`](prompts/00_system_prompt_base.md)

```
Sos el sistema de atención al cliente de Coberturas Florales.
TONO: cercano, amable, directo.

POLÍTICAS (reglas atómicas):
• Suscripción domicilio — pago: SOLO Mercado Pago, mensual adelantado.
• Zona domicilio: CABA sin costo. Norte GBA con costo hasta Pilar.
• Horarios: Dom 10–16 hs; Jue y Vie 8–12 y 16–20 hs.
[... ver archivo completo]

GUARDRAILS:
• Solo afirmar lo que esté en políticas.
• Si no está: "Te lo confirmo por WhatsApp" + 1 pregunta.
• No inventar zonas, horarios, precios ni disponibilidad.
```

---

## 📝 Prompts Implementados

### Estructura uniforme de todos los prompts

```
ROL        → qué hace este prompt
INPUT      → variables {{entre_dobles_llaves}}
REGLAS     → instrucciones específicas numeradas
OUTPUT     → schema JSON estricto en una sola línea
```

| Archivo | Prompt | Descripción |
|---|---|---|
| [`prompts/01_clasificador.md`](prompts/01_clasificador.md) | Prompt I | Clasifica intención y detecta dato faltante |
| [`prompts/02_respuesta_whatsapp.md`](prompts/02_respuesta_whatsapp.md) | Prompt B | Genera respuesta principal |
| [`prompts/03_urgencia.md`](prompts/03_urgencia.md) | Prompt C | Caso "¿llega hoy?" |
| [`prompts/04_sin_stock.md`](prompts/04_sin_stock.md) | Prompt D | Alternativas florales |
| [`prompts/05_objecion_precio.md`](prompts/05_objecion_precio.md) | Prompt F | Manejo de "está caro" |
| [`prompts/06_confianza.md`](prompts/06_confianza.md) | Prompt G | "¿Tenés fotos?" |
| [`prompts/07_reclamo.md`](prompts/07_reclamo.md) | Prompt H | Reclamo por diferencia |
| [`prompts/08_auditor_qa.md`](prompts/08_auditor_qa.md) | Prompt J | Auditor de cumplimiento |

---

## 📸 Ejemplos de Ejecución

Ver carpeta completa: [`ejemplos/`](ejemplos/)

### Ejemplo rápido — Cadena completa

**Input del cliente:**
```
"Hola, quiero saber si hacen entregas en Palermo y cuánto sale"
```

**Prompt I — Clasificador:**
```json
{"tag":"envíos","dato_faltante":"N/A","nivel":"simple","razon_corta":"Palermo es CABA, zona cubierta sin costo"}
```

**Prompt B — Respuesta:**
```json
{
  "respuesta": "¡Hola! Sí, hacemos entregas en Palermo sin costo adicional ya que está en CABA. 🌸 Podés elegir suscripción semanal (4 ramos/mes), quincenal o mensual.",
  "cta": "¿Te cuento cómo funciona cada frecuencia?",
  "dato_a_pedir": "N/A"
}
```

**Prompt J — Auditor:**
```json
{"ok":true,"frases_en_riesgo":"","motivo":"","respuesta_corregida":""}
```

---

## 🖼 Prompts Texto-Imagen

Ver archivos: [`prompts/09_imagen_ig_ramo.md`](prompts/09_imagen_ig_ramo.md) | [`prompts/10_imagen_cementerios.md`](prompts/10_imagen_cementerios.md) | [`prompts/11_imagen_story_ws.md`](prompts/11_imagen_story_ws.md)

| Pieza | Herramienta | Output |
|---|---|---|
| Ramo semanal IG | DALL·E 3 / Ideogram | Foto editorial + caption |
| Ofrenda cementerios | DALL·E 3 | Ilustración + texto respetuoso |
| Story WhatsApp | Canva AI / DALL·E | Story 9:16 + overlay de texto |

---

## ⚖️ Comparación Claude vs Gemini

Ver resultados completos: [`resultados/comparacion_llms.md`](resultados/comparacion_llms.md)

| Caso | Mensaje | Claude | Gemini | Ganador |
|---|---|---|---|---|
| C1 | "¿Hacen envíos a Palermo?" | ⭐ 5/5 | ⭐ 4.3/5 | Claude |
| C3 | "¿Llega hoy?" | ⭐ 4/5 | ⭐ 2/5 | Claude |
| C8 | "No era como la foto" | ⭐ 5/5 | ⭐ 3/5 | Claude |
| C10 | "¿Llegan a Rosario?" | ⭐ 5/5 | ⭐ 2/5 | Claude |

**Promedio total: Claude 4.82 / Gemini 4.08**

---

## 📊 Resultados

### Impacto v1 (baseline) → v2 (con system prompt + separación)

| Métrica | v1 (baseline) | v2 (mejorado) | Mejora |
|---|---|---|---|
| Fidelidad a políticas | 3.5/5 | 4.8/5 | +37% |
| Respuestas con CTA | 60% | 95% | +35pp |
| Alucinaciones (en 10 casos) | 4/10 | 1/10 | -75% |
| Respuestas en 2–3 líneas | 55% | 100% | +45pp |

---

## 🏁 Conclusiones

1. **La separación de responsabilidades** es la técnica de mayor impacto: un prompt que hace UNA sola cosa produce respuestas más predecibles.
2. **El System Prompt base** es la base de cualquier sistema de prompts escalable: evita repetición y garantiza uniformidad.
3. **Los guardrails específicos** son más efectivos que genéricos: "no informar zonas fuera de CABA y Norte GBA hasta Pilar" funciona mejor que "no inventar información".
4. **Claude supera a Gemini** en fidelidad a políticas, especialmente en edge cases (zonas no cubiertas, urgencias).

---

## 📁 Estructura del Repositorio

```
📦 coberturas-florales-prompt-engineering/
├── 📄 README.md                          ← Este archivo
├── 📁 prompts/
│   ├── 00_system_prompt_base.md
│   ├── 01_clasificador.md
│   ├── 02_respuesta_whatsapp.md
│   ├── 03_urgencia.md
│   ├── 04_sin_stock.md
│   ├── 05_objecion_precio.md
│   ├── 06_confianza.md
│   ├── 07_reclamo.md
│   ├── 08_auditor_qa.md
│   ├── 09_imagen_ig_ramo.md
│   ├── 10_imagen_cementerios.md
│   └── 11_imagen_story_ws.md
├── 📁 ejemplos/
│   ├── 01_envios_palermo.md
│   ├── 02_pago_tarjeta.md
│   ├── 03_urgencia_hoy.md
│   ├── 04_cambio_ofrenda.md
│   ├── 05_no_estaba_en_casa.md
│   ├── 06_zona_pilar.md
│   ├── 07_objecion_precio.md
│   ├── 08_reclamo_foto.md
│   ├── 09_frecuencia_semanal.md
│   └── 10_zona_no_cubierta.md
└── 📁 resultados/
    ├── rubrica_evaluacion.md
    └── comparacion_llms.md
```

---

*Proyecto Final — Prompt Engineer · CoderHouse · L. Ruth Collia*
