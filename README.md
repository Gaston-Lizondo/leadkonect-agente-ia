# Agente comercial de Lead Konect · Checkpoint 1

Curso AI Automation Avanzado · Gastón Lizondo

Primera versión del proyecto integrador: un agente de IA en n8n que atiende a dueños de pymes, emprendedores y profesionales interesados en Lead Konect, entiende su negocio, califica el lead (CALIENTE, TIBIO o FRÍO), lo registra en Google Sheets y envía un reporte de observabilidad por Gmail.

## Flujo

```
[Chat Trigger] → [AI Agent (Tools Agent, máx. 6 iteraciones)] → [Gmail: log de observabilidad] → [Respuesta al chat]
                        │ lateral          │ lateral          │ lateral
                 [Groq Chat Model]    [Simple Memory]  [Tool: Registrar lead LK (Google Sheets)]
```

| Nodo | Tipo | Función |
|---|---|---|
| Chat Trigger | Determinista | Captura el mensaje del usuario y genera el `sessionId`. |
| AI Agent | Probabilístico | Tools Agent con System Prompt modular (Rol → Ámbito → Objetivo → Reglas → Escalamiento). Guardrail de 6 iteraciones y pasos intermedios activados. |
| Groq Chat Model | Modelo | GPT-OSS 120B (openai/gpt-oss-120b) en Groq, temperatura 0,3. |
| Simple Memory | Memoria | Últimos 10 mensajes por sesión. En M3 se reemplaza por Airtable con Session_ID. |
| Registrar lead LK | Herramienta | Google Sheets (append). Descripción extensa que indica cuándo usarla y cuándo no. |
| Gmail - Log de observabilidad | Determinista | Envía mensaje, respuesta, puntaje y el Execution Log (herramienta usada, entrada y resultado). |
| Respuesta al chat | Determinista | Devuelve la respuesta del agente al chat. |

## Decisiones de diseño

- El agente no está detrás de ningún nodo IF ni Switch: decide solo si usa la herramienta.
- No da precios ni paquetes, no promete resultados, no inventa casos ni agenda reuniones. Deriva a Gastón los leads calientes.
- Español rioplatense con voseo y sin lenguaje inclusivo.
- La calificación es interna: el agente nunca le informa al usuario su puntaje ni que lo está evaluando.
- El agente solo confirma el registro si la herramienta devolvió éxito, para no afirmar algo que no ocurrió.
- Modelo: se usa el nodo nativo Groq Chat Model de n8n con GPT-OSS 120B, que soporta tool calling y tiene capa gratuita. Es el mismo proveedor que usa el proyecto MDX.

## Pruebas

| Prueba | Mensaje | Resultado esperado |
|---|---|---|
| Con herramienta | "Hola, soy Martín, tengo una plomería en Quilmes y casi no me entran consultas. Quiero arrancar este mes. Mi WhatsApp es 11 5555-0000." | Registra el lead como CALIENTE en la planilla y avisa que Gastón lo contacta, sin mencionarle la calificación. |
| Sin herramienta | "Hola, ¿cuánto cuesta un agente de WhatsApp?" | No da precio, explica que se define tras el diagnóstico y pide datos. No usa la herramienta. |
| Varios turnos | Datos repartidos en 2 o 3 mensajes | Gracias a la memoria, registra el lead cuando junta los datos mínimos. |

Los datos de las pruebas son ficticios.
