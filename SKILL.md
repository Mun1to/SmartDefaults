---
name: smart-defaults
description: >-
  Hace que una web arranque en el tema (claro/oscuro) y el idioma correctos según las preferencias
  del sistema/navegador del visitante, sin parpadeo y sin forzar redirecciones. Úsalo al crear el
  arranque (boot) de cualquier landing o app web, al añadir un selector de tema o de idioma, o
  cuando el usuario pida "que la web detecte el idioma/tema automáticamente" o se queje de que
  tiene que pulsar un toggle manualmente cada vez.
---

# SmartDefaults, arranque inteligente de tema e idioma

## Qué logra

La web arranca ya en el tema y el idioma que el visitante espera, sin que tenga que tocar nada.

## Política de tema por defecto

**Por defecto, SIN icono ni botón de tema.** El tema sigue **siempre** al sistema o al navegador,
en vivo: si el visitante cambia su sistema de oscuro a claro con la pestaña abierta, la web cambia
sola.

- **Tier A, el default, usar salvo razón de peso:** puro CSS, **cero JavaScript**, cero parpadeo
  posible (no hay JS que pueda llegar tarde): variables en `:root` para claro más un override
  completo dentro de `@media (prefers-color-scheme: dark)`. El navegador re-evalúa la media query
  solo, así que sigue al sistema en vivo gratis. Ver `plantillas/vanilla/theme-css-only.html`.
- **Tier B, excepción, solo con justificación real de producto** (por ejemplo una app donde el
  usuario necesita FIJAR un tema distinto al de su sistema por una razón funcional concreta): el
  patrón JS más `localStorage` con selector visible, descrito abajo. **No lo apliques por
  defecto**: si dudas entre A y B, quédate en A y pregunta antes de añadir un toggle.
- **Si un proyecto YA tiene un botón de tema manual:** propón quitarlo y pasar a Tier A, salvo que
  haya una razón real para conservarlo. Ante la duda, señálala en vez de decidir solo.

### Tier B en detalle, solo si de verdad hace falta un override manual

El tema se decide en un `<script>` **inline**, colocado **antes** de cualquier
`<link rel="stylesheet">` o CSS-in-JS que pinte colores. Si se decide después (por ejemplo en
`DOMContentLoaded`, o en un framework que hidrata tarde), el navegador ya pintó con el tema por
defecto y el usuario VE el cambio: eso es el parpadeo (FOUC), inaceptable en una web premium.

**Prioridad de resolución, en este orden:**

1. `localStorage.getItem('<clave>-theme')`, el usuario ya eligió antes, se respeta SIEMPRE.
2. Si no hay nada guardado, `matchMedia('(prefers-color-scheme: dark)').matches`, tema del sistema.
3. Fallback si `matchMedia` no existe (rarísimo hoy), `'light'`.

**Comportamiento en vivo:** si el usuario NO tiene una elección explícita guardada, escuchar
`matchMedia(...).addEventListener('change', ...)` para que la web siga al sistema con la pestaña
abierta. En cuanto pulsa el toggle manual, esa escucha deja de aplicar: ya hay una elección
explícita que manda para siempre.

**Bonus barato, aplica a AMBOS tiers:** poner `color-scheme: light dark` en el CSS (o
`<meta name="color-scheme">`) para que los controles NATIVOS del navegador (barras de scroll,
inputs, selects) también usen la versión correcta.

## Idioma: auto-seleccionar, NUNCA forzar

Verificado en la guía de Google Search Central: **redirigir automáticamente** según el idioma del
navegador rompe el botón "atrás", los marcadores, y confunde a los rastreadores. La regla:

**Prioridad de resolución, en este orden:**

1. `localStorage.getItem('<clave>-lang')`, elección explícita previa, se respeta SIEMPRE.
2. Si NO hay nada guardado (primera visita), mirar `navigator.languages` (más fiable que el viejo
   `navigator.language`; con backend, la cabecera HTTP `Accept-Language` es aún mejor, pero en
   sitios estáticos no hay acceso a ella) y hacer *match* contra los idiomas soportados de la web.
3. Si ninguno coincide, el idioma nativo de la web.

**Nunca más de una vez:** el auto-detect solo actúa en la primera visita. En cuanto el usuario
elige, o simplemente ya hay un valor guardado, esa elección se respeta para siempre: el sitio NO
vuelve a "adivinar" en cada carga.

**Siempre visible:** un selector de idioma manual, alcanzable, para quien quiera cambiarlo.

**Si el sitio tiene URLs separadas por idioma** (`/en/`, `/es/`) en vez de intercambiar textos en
la misma URL, añadir `<link rel="alternate" hreflang="es|en">`. Con el patrón de "misma URL,
textos intercambiados" no aplica.

⚠️ **Ahí termina esta skill y empieza WebIndex.** SmartDefaults resuelve el arranque en el
navegador: qué tema y qué idioma ve el visitante al entrar. Lo que decide si esas versiones existen
para el buscador es otra cosa: qué URLs hay, el `hreflang` con enlaces de ida y vuelta, y que el
`canonical` de cada idioma apunte a sí mismo y no a otro. Eso es la fase 6 de **WebIndex**. Y ojo,
porque los dos lados dicen lo mismo por motivos distintos: aquí no se redirige porque rompe el
botón "atrás" y los marcadores; allí, porque Googlebot rastrea sin cabecera `Accept-Language` y
sobre todo desde IPs de Estados Unidos, así que una redirección automática lo deja siempre en la
misma versión y las demás no se indexan nunca.

## Flujo de trabajo

1. Confirmar que la web es la candidata correcta: ¿tiene o va a tener tema claro/oscuro y/o más de
   un idioma? Si solo tiene uno de los dos, implementar solo esa mitad.
2. **Tema:** por defecto, copiar `plantillas/vanilla/theme-css-only.html` (Tier A, sin JS, sin
   icono) y traducir las variables de marca del proyecto a los dos bloques `:root`. Solo si hay
   una razón de producto real para permitir override manual, usar `plantillas/vanilla/boot-prefs.html`
   (Tier B) en su lugar.
3. **Idioma**, si el proyecto tiene más de uno: copiar la mitad de idioma de `boot-prefs.html` y
   adaptar el diccionario i18n y los idiomas soportados.
4. Verificar en el navegador: cambiar el tema del SISTEMA (o el color-scheme emulado en las
   herramientas de desarrollo) y recargar. La web debe seguirlo sin parpadeo, y en vivo si es
   Tier A. Si hay idioma: sin nada en `localStorage` debe autoseleccionar; con un valor guardado
   debe ignorar el idioma del navegador.
5. Pasar el checklist.

## Checklist

- [ ] Tema: ¿NO hay icono ni botón manual, salvo justificación de producto explícita?
- [ ] ¿El tema sigue al sistema EN VIVO? (Tier A: automático por CSS. Tier B: `matchMedia` `change`
      solo si no hay override guardado.)
- [ ] ¿Cero parpadeo? (Tier A: gratis, es CSS. Tier B: script inline ANTES del primer
      `<link rel="stylesheet">`.)
- [ ] ¿`color-scheme: light dark` presente, para que los controles nativos sean coherentes?
- [ ] Si hay idioma: ¿se auto-selecciona SOLO si no hay elección previa guardada?
- [ ] Si hay idioma: ¿hay selector visible y alcanzable, y la elección explícita se respeta para
      siempre?
- [ ] ¿Funciona sin JS razonablemente? (Tier A ya funciona al 100% sin JS; para idioma, fallback al
      idioma nativo de la web.)

## El patrón mínimo, para reconocerlo en producción

```html
<script>
  document.documentElement.dataset.theme =
    localStorage.getItem("app-theme") ||
    (matchMedia("(prefers-color-scheme: dark)").matches ? "dark" : "light");
</script>
<link rel="stylesheet" href="styles.css">
```

Eso es Tier B en su versión más corta: decisión inline, antes de la hoja de estilos. Tier A hace lo
mismo sin una sola línea de JavaScript.

## Índice de recursos

- `plantillas/vanilla/theme-css-only.html`, **Tier A, el default**: tema automático en vivo, cero
  JS, cero icono. Punto de partida para el 90% de los casos.
- `plantillas/vanilla/boot-prefs.html`, Tier B (tema con override manual) más idioma completo,
  comentada para entender el porqué de cada línea. Usar solo la mitad de idioma si el tema ya se
  resolvió con Tier A.
- Skill **WebIndex**, fase 6, para la otra mitad del idioma: estructura de URL, `hreflang` y por
  qué la redirección automática rompe la indexación.
