---
name: clima
description: Obtiene el clima actual y el pronóstico de 3 días de una ciudad (por defecto Querétaro, México). Úsala cuando el usuario pregunte por el clima, el tiempo, la temperatura o si va a llover, o cuando invoque /clima [ciudad].
argument-hint: "[ciudad]"
allowed-tools: Bash(curl:*)
---

# Clima

Consulta el clima con wttr.in (no necesita API key).

## Pasos

1. **Ciudad:** si se pasó un argumento, usa `$ARGUMENTS`. Si no, usa **Querétaro, México**.
2. **Consulta:** codifica la ciudad para URL (espacios → `+`, acentos en UTF-8 percent-encoding) y ejecuta:

   ```bash
   curl -s --max-time 15 "https://wttr.in/<ciudad>?format=j1&lang=es"
   ```

   Para la ciudad por defecto: `https://wttr.in/Quer%C3%A9taro,Mexico?format=j1&lang=es`

3. **Datos a leer del JSON:**
   - `current_condition[0]`: `temp_C`, `FeelsLikeC`, `humidity`, `windspeedKmph`, `winddir16Point`, `lang_es[0].value` (descripción), `localObsDateTime`.
   - `nearest_area[0]`: `areaName[0].value`, `region[0].value`, `country[0].value`, para confirmar que la ubicación es la correcta.
   - `weather[0..2]`: `date`, `maxtempC`, `mintempC` y el máximo de `hourly[].chanceofrain`.

4. **Respuesta (en español, °C y km/h):**
   - Ubicación detectada y hora de observación.
   - Condiciones actuales: descripción, temperatura, sensación térmica, humedad, viento.
   - Pronóstico de 3 días: fecha, mínima/máxima y probabilidad de lluvia.

5. **Errores:** si curl falla, la respuesta está vacía o no es JSON, o la región y el país de `nearest_area` no corresponden a la ciudad pedida (el `areaName` puede ser una localidad cercana, como "La Pada" para Querétaro; eso es normal), dilo claramente y no inventes datos.
