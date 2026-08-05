# TU AGENTE — el cerebro no viene en el repo, y está bien

Este repositorio es el **cuerpo** (las once superficies) y usa Supabase como
**memoria**. El **cerebro** no está adentro, a propósito.

## Por qué no viene incluido

Un agente metido dentro del repositorio sería un modelo elegido por otro, con una
llave de API que pagas aparte, envejeciendo al ritmo al que alguien más actualice
este código. Y sobre todo: sería un cerebro peor conectado a un cuerpo que ya
tiene uno bueno.

Porque tú ya tienes un agente instalado, con acceso a tu disco, que lee y escribe
archivos, corre comandos y conoce tus proyectos. **Ese es el cerebro.** No hace
falta un segundo.

```
tu agente (Claude Code)  ──escribe──►  Supabase  ◄──lee──  esta aplicación
       el cerebro                     la memoria              el espejo
```

Nada en medio. Nada que mantener despierto. Nada que se caiga de noche.

## Las tres piezas que lo vuelven TU agente

Un agente recién instalado no sabe nada de tu negocio. Le puedes pedir que lea tu
base y lo va a hacer mal, porque no sabe qué tablas hay ni qué significan tus
números. Se arregla en este orden, porque cada pieza depende de la anterior.

### 1. Contexto — `CLAUDE.md` en la raíz de tu proyecto

Tu agente lo lee entero cada vez que arranca, sin que se lo pidas. Cuatro bloques:

- **Quién eres** y qué vendes.
- **Cómo está armado** tu sistema: piezas, rutas, qué corre dónde.
- **Errores pagados**: lo que ya salió mal y no se repite.
- **Reglas duras**: lo que no se hace nunca, aunque tú lo pidas distraído.

El cuarto es el que casi nadie escribe y el que más ahorra. Ejemplo de regla
dura: *"nunca inventes un número; si no tienes el dato, dilo"*.

**Regla de crecimiento:** cada vez que corriges a tu agente dos veces por lo
mismo, esa corrección deja de ser un mensaje y se vuelve una línea de este
archivo.

**Regla de tamaño:** si algo no cambia una decisión, no va ahí. Un `CLAUDE.md` de
tres mil líneas no es contexto, es ruido que pagas en cada conversación. Lo
correcto es un índice corto que apunta a documentos que se abren cuando el tema
aparece.

### 2. Memoria — una carpeta de archivos versionados

No una base mágica ni un servicio. Archivos de texto que puedes leer, editar y
borrar. Organizados por tipo, no por proyecto:

```
.claude/memory/
├── user/        quién es el dueño, sus decisiones
├── feedback/    correcciones, y POR QUÉ
├── project/     estado de lo que está en curso
└── reference/   dónde encontrar cosas en sistemas de fuera
```

Sirve para lo que no cabe en el contexto permanente pero tiene que durar. Ejemplo
real: *"los datos del embudo anteriores al 22 de febrero están incompletos porque
el rastreo estaba roto"*. Eso no es una regla ni una descripción: es una
corrección, y tiene que sobrevivir a la conversación donde nació.

**La trampa:** si tienes un índice de la memoria, el índice apunta pero **el
archivo manda**. Opinar leyendo solo el índice es operar con el resumen del
resumen. Si el tema de la conversación toca un archivo, ese archivo se abre
entero antes de opinar.

### 3. Skills — procedimientos que se cargan solos

Una skill es un procedimiento escrito que tu agente carga **solo cuando el tema
aparece**. Por eso puede ser largo y detallado sin costarte contexto: el agente
solo ve una descripción de un renglón hasta que la necesita.

La primera que deberías escribir es la más aburrida: la que repites siempre. Para
la mayoría, es **cómo consultar tus propios datos**: las consultas exactas, cuál
tabla es la fuente de verdad para el dinero, y las trampas que hacen que un
número salga mal.

Una skill así no hace a tu agente más listo. Lo hace **confiable**, que es
distinto y vale más: o te da el número correcto, o te dice que no lo tiene.

**El error que vas a cometer:** hacer sesenta skills. Cada una cobra atención en
todas las conversaciones, incluso en las que no la usan. Pocas por dominio, y lo
específico adentro como material que solo se abre cuando la madre lo pide. Un
árbol, no una lista.

## Cómo le das acceso a tu base

Tu agente no necesita un plugin. Necesita saber que la base existe y cómo
consultarla. Dos caminos, elige uno:

**A. Por credenciales, en una skill.** La ruta a tu `.env.local` y la instrucción
de leer de ahí la URL y la llave de servicio. Directo, sin dependencias.

**B. Por el MCP de Supabase.** Si ya lo usas, tu agente consulta con herramientas
en vez de con `curl`. Más limpio; una dependencia más.

En los dos casos, la regla es la misma: la llave de servicio **nunca** sale de
`.env.local` y **nunca** viaja en un mensaje.

## La prueba de que funcionó

Abre una sesión nueva y pregúntale algo de tu negocio que solo se pueda contestar
mirando tu base.

Aprobado es que te responda con **un dato real de una tabla tuya**. Si te da un
número que se ve bien pero no puedes rastrear hasta una fila, reprobó, y eso es
información útil: significa que le falta la skill que le dice de dónde sacar la
verdad.

---

## Y una advertencia que va con todo esto

Un agente con acceso a tu base puede escribir en ella. Antes de darle permiso de
escritura sobre datos que te importan:

- Ten una copia de seguridad que hayas **restaurado al menos una vez**. Un backup
  que nunca probaste no es un backup, es una esperanza.
- Empieza en solo lectura una o dos semanas. Lee lo que hace antes de dejarlo
  actuar.
- Las acciones irreversibles y de alcance masivo (mandar correos a tu lista,
  borrar filas, publicar algo) pasan por una confirmación tuya explícita, no por
  un botón de un clic.

Esa última la aprendí caro: un anuncio salió a cientos de personas desde un botón
que parecía de guardar. Lo único que separó el susto del daño fue que la
notificación por correo venía apagada por defecto.
