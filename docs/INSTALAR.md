# INSTALAR — el manual que ejecuta tu agente

> **Esto no es para que lo leas tú.** Es para tu agente. Abre Claude Code en esta
> carpeta y pégale:
>
> ```
> Lee docs/INSTALAR.md y móntame el Business OS.
> ```
>
> Y siéntate a mirar. Lo único que vas a hacer es pegar tres llaves cuando te
> las pida.

---

## Instrucciones para el agente

Sigue estas fases **en orden**. No saltes ninguna. Después de cada fase, dile al
humano en una línea qué quedó hecho.

**Tres reglas que mandan sobre todo lo demás:**

1. **Nunca inventes una credencial.** Si falta una llave, párate y pídela. Un
   `.env.local` con valores de relleno arranca la app y la rompe media hora
   después, que es el peor momento posible para descubrirlo.
2. **Nunca escribas secretos fuera de `.env.local`.** Ni en un comentario, ni en
   un commit, ni en un mensaje de vuelta al humano.
3. **Si una fase falla, detente y reporta el error completo.** No lo resumas y no
   sigas a la siguiente. Un instalador que continúa después de fallar produce un
   sistema roto que se ve instalado.

---

### Fase 1 — Terreno

```bash
node --version        # necesitas 22 o más
npm --version
git --version
```

Si Node es menor a 22, para aquí y dile al humano que instale Node 22 (con `nvm`
o desde nodejs.org). No intentes actualizarlo tú: tocar la versión de Node de la
máquina de alguien sin avisarle es de las cosas que no se hacen.

```bash
npm install
```

Tarda unos minutos. Si truena con errores de permisos, reporta y para.

---

### Fase 2 — Las seis llaves

El archivo `.env.example` tiene 31 variables. **Solo seis importan para arrancar.**
Las otras 25 encienden funciones opcionales y sin ellas la app corre igual, con
esa función apagada y sin romperse.

```bash
cp .env.example .env.local
```

Ahora pídele al humano estas seis, **de una en una y explicando para qué es cada
una**. No le pidas las seis de golpe en un bloque: se pierde y pega mal.

| Variable | Qué es | Dónde la saca |
|---|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | La dirección de su base | Supabase → Project Settings → API |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Llave pública. La ve cualquiera que abra la página, y está bien | misma pantalla |
| `SUPABASE_SERVICE_ROLE_KEY` | Llave de servicio. **Se salta todas las reglas de seguridad** | misma pantalla, sección `service_role` |
| `OWNER_EMAIL` | Su correo. Define quién es el dueño | lo escribe él |
| `NEXT_PUBLIC_OWNER_NAME` | Su nombre. Sale en la aplicación | lo escribe él |
| `NEXT_PUBLIC_AGENT_NAME` | Cómo se llama SU agente | lo escribe él |

**Antes de pedir las llaves de Supabase**, si el humano todavía no tiene proyecto,
dale estos pasos y espera:

1. Entra a supabase.com y crea una cuenta.
2. New project. El plan gratis alcanza.
3. Elige una contraseña de base de datos y **guárdala en tu gestor de contraseñas**,
   no en una nota. Supabase no te la vuelve a mostrar.
4. Espera a que el proyecto termine de aprovisionarse (uno o dos minutos).

**Advertencia que tienes que decirle en voz alta cuando le pidas la service role:**
esa llave da control total de su base. Vive solo en `.env.local`, ese archivo está
en `.gitignore`, y si algún día se le escapa a un repositorio o a una captura de
pantalla, se rota **ese mismo día** desde Supabase.

Verifica antes de seguir:

```bash
grep -c '^[A-Z].*=.\+' .env.local          # cuántas tienen valor
git check-ignore -v .env.local             # tiene que decir que está ignorado
```

Si `.env.local` **no** está ignorado, para todo y arréglalo antes de continuar.

---

### Fase 3 — El esquema

Aquí se crean las tablas: perfiles, tareas, chat, canvas, calendario, push.

```bash
npx supabase link --project-ref <project-ref>
npx supabase db push
```

El `project-ref` es el código que sale en la URL del proyecto
(`https://supabase.com/dashboard/project/ESTO`). Te va a pedir la contraseña de
la base: es la que el humano guardó en la fase 2.

**Esta fase es la más frágil del instalador y quiero que lo sepas.** Se probó
sobre bases existentes, y hasta hace poco no se había probado end-to-end contra
un proyecto recién creado. Si algo se rompe, es aquí. Errores conocidos:

| Error | Qué significa | Qué haces |
|---|---|---|
| `failed SASL auth` / `password authentication failed` | La contraseña de la base está mal | Pídesela otra vez. Si no la tiene, se resetea en Supabase → Settings → Database |
| `Cannot find project ref` | El `link` no se ejecutó o falló | Corre `npx supabase link` otra vez y verifica el ref |
| `relation "..." already exists` | Estás sobre una base que ya tenía tablas | **Para.** No fuerces. Reporta cuál tabla y espera instrucciones del humano |
| `permission denied for schema public` | Falta el rol o el proyecto todavía se está aprovisionando | Espera un minuto y reintenta una vez. Si vuelve a fallar, reporta |
| Cuelga sin salida | Suele ser la contraseña con caracteres que la terminal se come | Que la ponga entre comillas simples |

Cuando termine, **verifica que las tablas existen de verdad** en vez de confiar en
que el comando salió con cero:

```bash
npx supabase db diff --linked --schema public | head -20
```

Si eso devuelve un diff grande, las migraciones no se aplicaron. No sigas.

---

### Fase 4 — Su usuario

```bash
npm run dev
```

Dile al humano que abra `http://localhost:3000/signup` y se registre **con el
mismo correo** que puso en `OWNER_EMAIL`. Un correo distinto crea un usuario sin
permisos y luego cuesta entender por qué no ve nada.

---

### Fase 5 — La prueba de fuego

No des la instalación por buena porque la pantalla se ve bien. Verifica las tres:

1. La aplicación abre en `localhost:3000` y **el nombre del humano** aparece en
   la interfaz.
2. En Supabase → Table editor → `profiles` hay **exactamente una fila**, la suya,
   con `role = 'owner'`.
3. La consola del navegador no tiene errores en rojo.

El punto 2 es el que cuenta. Los otros dos los puede fingir una pantalla; una
fila en la base, no.

Reporta las tres con un sí o un no. Si alguna es no, dilo claro en vez de
cerrar con un "listo".

---

### Fase 6 — Lo que queda apagado, y decirlo

Cierra diciéndole al humano, en una lista corta, **qué quedó apagado** y qué
necesitaría cada cosa para encenderse:

- **Notificaciones al teléfono** → falta el par de llaves VAPID (`npx web-push generate-vapid-keys`).
- **Voz (dictado y HUD Live)** → falta `GROQ_API_KEY`.
- **Calendario** → falta el `gog` CLI autenticado y sus variables.
- **Chat con cerebro** → falta conectar su agente. Eso es el siguiente paso y
  está en [`docs/TU-AGENTE.md`](TU-AGENTE.md).

Un instalador que no dice qué dejó apagado entrega un sistema que el humano cree
completo. Eso es peor que dejarlo apagado.
