---
name: clima
description: Consulta el clima actual y el pronóstico de la ubicación local del usuario (detectada por IP) o de una ciudad indicada. Úsala cuando el usuario pregunte por el clima, el tiempo, la temperatura, si va a llover, o invoque /clima.
argument-hint: "[ciudad opcional]"
allowed-tools: Bash(curl:*)
---

# Clima local

Obtén el clima usando el servicio público **wttr.in** (no requiere API key). Responde siempre en español.

## Ubicación

- Si el usuario pasó una ciudad como argumento (`$ARGUMENTS`), úsala. Reemplaza los espacios por `+` (ej. `San+José`).
- Si no hay argumento, deja la ubicación vacía: wttr.in detecta la ubicación local por la IP.

## Pasos

1. **Clima actual** (una sola línea):

   ```bash
   curl -s --max-time 10 "https://wttr.in/<CIUDAD>?format=%l:+%c+%C,+%t+(sensación+%f),+humedad+%h,+viento+%w,+lluvia+%p&lang=es"
   ```

2. **Pronóstico de hoy y los próximos 2 días** (solo si se pide pronóstico o si pregunta si va a llover):

   ```bash
   curl -s --max-time 10 "https://wttr.in/<CIUDAD>?format=j1&lang=es"
   ```

   Del JSON usa `weather[i].date`, `maxtempC`, `mintempC`, y de `weather[i].hourly[]` los campos `chanceofrain` y `lang_es[0].value`. Resume por día: fecha, mínima/máxima y probabilidad máxima de lluvia.

## Formato de respuesta

```
📍 San José, CR
🌧️ Lluvia ligera · 19 °C (sensación 21 °C)
💧 Humedad 100% · 🌬️ Viento 4 km/h
```

Agrega el pronóstico como lista corta solo si se pidió.

## Errores

- Si `curl` falla, devuelve vacío o `Unknown location`, informa que no se pudo obtener el clima y sugiere indicar la ciudad (ej. `/clima Heredia`).
- No inventes datos del clima si el servicio no respondió.
