# ¿Algo no funciona?

*[English version: TROUBLESHOOTING.md](TROUBLESHOOTING.md)*

Este proyecto no tiene canal de soporte — la idea es que resuelvas los
problemas tú mismo, idealmente con Claude Code o claude.ai abierto en la
misma conversación que construyó esto contigo. Pega cualquiera de las
revisiones de abajo (o una captura de pantalla) y pídele que aplique la
solución. Todo lo que hay en esta página ya existe en algún lugar del
repositorio; esto solo lo junta según lo que en realidad estás VIENDO, no
según de qué archivo o sesión vino.

Busca tu síntoma abajo.

---

## "Funcionaba antes — ahora todo da error"

**Causa probable: tu proyecto de Supabase se pausó solo.** Los proyectos
gratuitos de Supabase se pausan después de una semana más o menos sin
actividad real en su API. Un proyecto pausado se ve exactamente como uno
roto desde afuera — la app no puede alcanzar la base de datos en absoluto.

**Confírmalo:** corre esto con tu propia URL de proyecto y tu llave anon
(ambas están en `config.js`) —

```
curl -s "https://TU-PROYECTO.supabase.co/rest/v1/thoughts?select=id&limit=1" \
  -H "apikey: TU_LLAVE_ANON"
```

Es la misma petición que el workflow de keep-alive hace en su horario
(`.github/workflows/keep-alive.yml`) y la misma forma que se usa para
detectar un proyecto vivo en el Paso 0 de [UPGRADE.md](UPGRADE.md). Un
proyecto vivo responde en menos de un segundo — con filas reales, una lista
vacía, o hasta un error de permisos, no importa cuál, una respuesta es una
respuesta. Uno pausado se cuelga o simplemente no conecta.

(Ya que estás ahí: correr esa misma petición SIN ninguna llave — quita el
encabezado `apikey` por completo — también debería fallar. Si en cambio
regresa filas reales, la seguridad de tu base de datos está abierta para
cualquiera que tenga tu llave pública. Ese es un problema distinto y más
serio — revisa el Paso 1, chequeo 4, de UPGRADE.md para saber qué significa
y cómo cerrarlo.)

**Arréglalo:**
1. Ve a [supabase.com/dashboard](https://supabase.com/dashboard) e inicia
   sesión
2. Abre tu organización, busca el proyecto — va a decir "Paused"
3. Haz clic en **Resume project**, confirma

Regresa en pocos minutos. No se pierde nada — misma base de datos, mismos
datos, misma configuración, misma URL. Esto funciona hasta un año después de
que el proyecto se pausó — es el límite propio de Supabase, no de este
proyecto. (Si estás leyendo esto más de un año después de haber tocado el
proyecto por última vez, el botón automático ya no va a estar ahí y
necesitarías el proceso manual de recuperación de respaldos de Supabase en
su lugar — vale la pena saberlo, prácticamente nunca te va a pasar a ti.)

**Para que pase menos seguido:** este proyecto ya trae dos avisos
independientes pensados para mantener el proyecto activo — un trabajo
pg_cron dentro de tu propia base de datos (`keep-brain-awake`, configurado
en la Sesión 2, Paso 6, o en el Paso 6e de UPGRADE.md si actualizaste) y un
workflow de GitHub Actions que hace ping desde afuera. Ninguno de los dos
está comprobado que en verdad detenga una pausa — solo que sale una petición
real dos veces por semana. Si de todas formas pasa, esta página, no esos
pings, es la solución real.

---

## Guardar no hace nada, o muestra un error que no entiendes

Intenta guardar otra vez y lee el texto rojo que aparezca — la app sí
muestra un mensaje de error, no falla completamente en silencio. Después
revisa, más o menos en el orden de qué tan seguido es la causa real:

1. **`config.js` todavía tiene sus valores de ejemplo.** Ábrelo y busca
   `PASTE_YOUR_PROJECT_URL_HERE` o `PASTE_YOUR_PUBLIC_KEY_HERE` — si
   cualquiera de los dos sigue ahí, la app nunca se conectó de verdad a una
   base de datos. Es exactamente la misma revisión que hace el workflow de
   keep-alive sobre sí mismo antes de hacer ping a cualquier cosa.
2. **No tienes sesión iniciada, o se venció.** Busca el punto de estado
   cerca de arriba en la app — debe decir "conectado", no "error de base de
   datos" ni una pantalla de configuración. Cierra sesión y vuelve a
   entrar.
3. **El proyecto está pausado.** Ve el síntoma de arriba — un proyecto
   pausado hace que guardar falle de la misma forma que hace fallar todo lo
   demás.

---

## La búsqueda no encuentra nada, o encuentra lo que no es

Casi siempre significa que el pensamiento se guardó pero nunca recibió su
huella de significado (su embedding) — la búsqueda por palabra exacta sigue
funcionando sin ella, pero "buscar por significado" no.

**Confírmalo** en el Editor SQL de Supabase:

```sql
select count(*) from thoughts where embedding is null;
```

Un número mayor que cero significa que a algunos pensamientos les falta su
huella. Para una captura **recién hecha**, dale primero 15–30 segundos — el
enriquecimiento corre justo después de guardar, no al instante (si nunca
llega, ese es el siguiente síntoma de abajo, no este). Para pensamientos
**viejos** que llevan tiempo sin su huella, corre el mismo backfill que este
proyecto ya trae para exactamente esto — Paso 7 de [UPGRADE.md](UPGRADE.md),
primero en modo de prueba:

```
curl -s -X POST "https://TU-PROYECTO.supabase.co/functions/v1/backfill-brain" \
  -H "Authorization: Bearer TU_SERVICE_ROLE_KEY" \
  -H "Content-Type: application/json" \
  -d '{"dry_run": true}'
```

Te dice exactamente cuántos pensamientos necesitan una huella y cuánto va a
costar (una cantidad pequeña de dinero real contra tu llave de OpenRouter)
antes de gastar nada. Quita `dry_run` para correrlo de verdad.

---

## Las capturas nunca reciben etiquetas, categoría o resumen

Esto es que el enriquecimiento no se está disparando — un problema distinto
al de la búsqueda de arriba, y tiene su propia revisión en dos partes, las
dos ya usadas durante la construcción misma (Sesión 2, Paso 6, y Paso 6d de
UPGRADE.md):

**1. ¿Existe el trigger?**

```sql
select tgname from pg_trigger
 where tgrelid = 'thoughts'::regclass and not tgisinternal;
```

Deberías ver `on_thought_created`. Si falta, nada llama a `enrich-thought`
cuando guardas — vuelve a correr el bloque del trigger de la Sesión 2, Paso 6
(`webhook.sql`).

**2. Si el trigger sí está, revisa los logs de la función:**
Panel de Supabase → Edge Functions → `enrich-thought` → Logs.

- **Los logs están vacíos** → la función nunca corrió. Eso apunta de vuelta
  al chequeo 1, o a que `pg_net` no está habilitado.
- **Los logs muestran errores** → casi siempre es que `OPENROUTER_API_KEY`
  falta o está mal escrita, o que no queda crédito en la cuenta de
  OpenRouter.

---

## El bot de Telegram se quedó callado

Revisa qué cree Telegram que está pasando — él mismo lo guarda, sin
necesidad de iniciar sesión en Supabase:

```
https://api.telegram.org/bot<TU_TOKEN_DE_BOT>/getWebhookInfo
```

Lee `last_error_message` en la respuesta — normalmente dice exactamente qué
está mal (es la misma revisión que el Paso 5 de la Sesión 2b te hace correr
si no llega nada la primera vez). El hallazgo más común: un `401`, que
significa que la función `telegram-bot` se desplegó sin la bandera
`--no-verify-jwt`. Vuelve a desplegarla con la bandera:

```
npx supabase functions deploy telegram-bot --no-verify-jwt
```

Si el webhook se ve bien pero los mensajes de una persona en particular se
ignoran mientras los tuyos sí funcionan — eso es `TELEGRAM_CHAT_ID` haciendo
exactamente lo que debe (el Paso 4 de la Sesión 2b bloquea el bot a un solo
teléfono a propósito).

Si ha estado callado para todos, tú incluido, revisa también que el
proyecto no esté pausado — ve el primer síntoma de esta página.

---

## Ninguno de estos es lo que estás viendo

Pega una captura de pantalla o el texto del error en Claude Code o
claude.ai — el que sea que construyó esto contigo — y pídele que lo
diagnostique en vivo. Tiene acceso a tus logs y tu base de datos de una
forma que esta página no puede.
