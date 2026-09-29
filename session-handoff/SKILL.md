---
name: session-handoff
description: Usar cuando el usuario diga "traspaso de sesión", "cierra la sesión", "resume antes de limpiar", "session handoff", "wrap up session", "hand off" o "handoff summary", o quiera un resumen estructurado de fin de sesión antes de limpiar el contexto. Produce un traspaso solo en el chat con decisiones, cambios entregados, ficheros clave, estado en marcha, pasos de verificación, aplazamientos y preguntas abiertas, para que un agente nuevo continúe sin perder el hilo.
---

# Traspaso de sesión

Produce un resumen de fin de sesión repetible para que el usuario pueda hacer `/clear` y arrancar un agente nuevo sin perder continuidad. El siguiente agente tiene que poder retomar el trabajo leyendo solo este resumen.

Es un **artefacto de traspaso de contexto**, no un informe de estado. El destinatario es una instancia futura de ti, no una persona interesada en el proyecto.

## Cuándo invocarla

El usuario dice: "traspaso de sesión", "cierra la sesión", "vamos cerrando", "resume antes de limpiar", "session handoff", "wrap up session", "hand off", "handoff summary" o algo equivalente. Invócala también por iniciativa propia si el usuario dice que va a hacer `/clear` y todavía no la ha ejecutado.

## Cómo producir el resumen

1. **Repasa la conversación entera**, no solo los últimos turnos. Los traspasos pierden cosas cuando solo resumen el contexto reciente.
2. **Saca el estado de estas fuentes (en este orden):**
   - Ficheros de plan citados en la sesión (mira `~/.claude/plans/` si se mencionó un plan).
   - Estado de TodoWrite: tareas en curso o pendientes.
   - Procesos en segundo plano que lanzaste con `run_in_background`: sus IDs de shell son imprescindibles para el siguiente agente.
   - Ficheros creados o modificados en esta sesión: sabes lo que tocaste; no hagas grep para redescubrirlo.
   - Ficheros de memoria escritos o actualizados (`~/.claude/projects/<proyecto>/memory/`).
   - Preguntas sin resolver: lo que preguntaste al usuario y nunca tuvo respuesta clara, o lo que el usuario preguntó y quedó sin contestar.
3. **NO audites el sistema de ficheros.** Es una síntesis de lo que pasó en ESTA sesión. Nada de `git log` ni barridos amplios con `Glob`. Si no lo tocaste en esta sesión, no va aquí.
4. **Escribe el resultado en el chat.** No escribas ningún fichero. No actualices la memoria. Solo chat.

## Plantilla de salida: usa exactamente esta estructura, siempre

```
# Traspaso de sesión — <título de una línea sobre de qué iba esta sesión>

## Punto de partida
<2-3 frases: qué pidió el usuario y qué encuadre o restricciones clave surgieron>

## Decisiones cerradas y lo entregado
- <decisión o cambio> — <por qué, y dónde vive (ruta absoluta si es un fichero)>
- ...

## Ficheros clave para la siguiente sesión
- `<ruta absoluta>` — <por qué debe leerlo primero el siguiente agente>
- Fichero de plan: `<ruta>` (si un plan guio la sesión)
- Ficheros de memoria tocados: `<rutas>` (si los hay)

## Estado en marcha
- Procesos en segundo plano: <IDs de shell + qué son + cómo pararlos> — o "ninguno"
- Servidores de desarrollo / puertos: <url + puerto> — o "ninguno"
- Worktrees / ramas abiertas: <rutas> — o "ninguno"

## Verificación: cómo comprobar que todo sigue funcionando
- `<comando>` — <resultado esperado>
- ...

## Aplazado y preguntas abiertas
- Aplazado: <asunto> — <por qué se dejó para después>
- Abierta: <pregunta que necesita respuesta del usuario> — <contexto>

## Retomar aquí
<1-2 frases: la siguiente acción más probable para un agente nuevo>
```

## Reglas estrictas

1. **Solo salida en el chat.** Nunca escribas el traspaso en un fichero. Nunca actualices la memoria desde esta skill.
2. **Nunca inventes estado.** Si una sección no tiene nada que contar, escribe "ninguno": no la omitas. La estabilidad de la estructura es lo que da sentido a la skill.
3. **Rutas absolutas siempre.** El siguiente agente puede tener otro directorio de trabajo.
4. **Si un plan guio la sesión, nómbralo el primero** en "Ficheros clave", para que el siguiente agente lo lea antes que nada.
5. **Sin emojis, sin bombo, sin resúmenes de "buen trabajo".** Seco y concreto: rutas, comandos, IDs de shell, decisiones. El tono de un ingeniero con experiencia que entrega el turno.
6. **Los IDs de procesos en segundo plano son críticos.** Si lanzaste shells con `run_in_background`, sus IDs tienen que aparecer en "Estado en marcha" con el comando para pararlos: el siguiente agente no tiene otra forma de encontrarlos.

## Antipatrones: no hagas esto

- Resumir los últimos 3 turnos y llamarlo traspaso.
- Listar ficheros con rutas relativas.
- Saltarte "Estado en marcha" porque "no hay nada en marcha": escribe "ninguno".
- Escribir el resumen en `~/.claude/handoffs/` o en cualquier fichero. Es solo chat, a propósito.
- Añadir una retrospectiva de "qué fue bien / qué fue mal". Esto no es una retro.
- Recomendar pasos más allá de la línea única de "Retomar aquí". El siguiente agente decide; tú solo entregas el turno.
