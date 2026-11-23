# El prompt

Copia todo lo que está bajo la línea y pégalo en Claude Code, abierto dentro de un repo privado y vacío.

---

Quiero que me ayudes a construir mi propio sistema de contenido para Instagram en este repo. Se va a llamar Content OS. Lee todo este mensaje antes de partir.

Trabaja por fases. Al terminar cada una, muéstrame qué hiciste, qué viene y qué necesitas de mí, y espera a que te diga "sigue". No avances de fase sin mi OK. Háblame en español.

## Reglas que no se negocian

1. **Solo la API oficial.** Usa la Instagram Graph API de Meta y nada más. Nunca scrapees Instagram, nunca automatices su web ni su app, nunca uses librerías no oficiales y nunca me pidas mi contraseña de Instagram.
2. **El token es secreto.** Vive en `~/.config/content-os/env` (permisos 600) y en GitHub Secrets. Nunca en el repo, nunca en un log, nunca en el chat y nunca como argumento de un comando, porque queda en el historial. Los scripts lo leen desde el archivo o desde la variable de entorno. Agrega `.gitignore` para cualquier `.env`, y un chequeo antes de cada commit que falle si algo parece un token.
3. **No inventes datos.** Si una métrica no viene, se guarda vacía y en el dashboard se muestra "—" con una nota de por qué. Nunca rellenes con estimaciones.
4. **El repo es privado.** Si detectas que es público, detente y avísame.
5. **Revisa la documentación vigente de Meta** antes de cada fase. Los nombres de las métricas cambian y algunas se deprecan. Si una métrica de este prompt ya no existe, dime cuál la reemplaza en vez de adivinar.
6. **Código simple.** Python 3 con lo mínimo afuera de la librería estándar, CSV como base de datos y HTML estático para el dashboard. Sin frameworks, sin bases de datos, sin servidores.
7. **Documenta mientras avanzas.** Un `README.md` del repo y un `motor/README.md` con qué hace cada script, de dónde sale cada métrica y qué limita la API.

## Fase 0: conocerme

Hazme pocas preguntas a la vez, no un formulario:

- Cuál es mi cuenta y de qué hablo.
- Mis temas (de 3 a 5) y los formatos que uso: reel, carrusel, foto, trial reel, contenido para marcas.
- Qué quiero lograr este año y cada cuánto publico.
- Mi zona horaria.

Guarda las respuestas en `CONTEXT.md`. El dashboard y las propuestas se apoyan en ese archivo.

## Fase 1: conexión con Meta

Esta fase la hago yo en la web de Meta. Guíame de a un paso por mensaje y espera mi confirmación antes de seguir. Si un menú cambió de nombre, ayúdame a encontrarlo.

1. Mi cuenta de Instagram tiene que ser profesional (creador o empresa) y estar conectada a una página de Facebook.
2. En developers.facebook.com, crear una app de tipo Empresa y agregarle el producto Instagram con la opción de API con inicio de sesión de Facebook (Instagram API with Facebook Login).
3. En la configuración del negocio de Meta (business.facebook.com): agregar la página y la cuenta de Instagram al portafolio, crear un usuario del sistema con rol de administrador, y asignarle la app, la página y la cuenta de Instagram.
4. Generar un token para ese usuario del sistema, sin vencimiento, con estos permisos: `instagram_basic`, `instagram_manage_insights`, `pages_show_list`, `pages_read_engagement` y `business_management`.
5. Tú creas `~/.config/content-os/env` vacío con permisos 600 y me dices cómo abrirlo. Yo pego ahí `IG_TOKEN=...`. Tú no lo ves ni lo imprimes.
6. Verifica sin mostrar el token: con `debug_token`, que no vence y que tiene los permisos; con `/me/accounts`, que aparece la página; y desde la página, el `instagram_business_account`. Muéstrame solo el nombre de la página, el usuario de Instagram y los ids.
7. Guarda los ids, sin el token, en `motor/config.json`.

## Fase 2: la ingesta

Escribe `motor/ingest_instagram.py`. Tiene que ser idempotente: correrlo dos veces el mismo día no duplica nada. Deja la versión de la Graph API en una constante y usa la última estable.

**Piezas.** Desde `/{ig-user-id}/media`, paginado, con todo el historial:

- Datos de cada pieza: id, fecha, `media_type`, `media_product_type`, permalink, caption y miniatura.
- Insights de reels: `views`, `reach`, `likes`, `comments`, `shares`, `saved`, `total_interactions`, `ig_reels_avg_watch_time` e `ig_reels_video_view_total_time`.
- Insights de posts y carruseles: `views`, `reach`, `likes`, `comments`, `shares`, `saved`, `total_interactions`, y `profile_visits` y `follows` cuando vengan.
- Una fila por pieza y por día de lectura en `data/piezas_snapshots.csv`, para ver cómo crece cada pieza, y la última lectura en `data/piezas.csv`.
- Mis etiquetas en `data/etiquetas.csv`: id de la pieza, tema, formato (reel, carrusel, foto, trial, marca) y notas. Las piezas nuevas entran sin etiqueta y el dashboard me las muestra como pendientes. Los trial reels los marco yo, porque la API no siempre los distingue.

**Historias.** Desde `/{ig-user-id}/stories`. Duran 24 horas, así que se leen cada 3 horas:

- Por historia: `views`, `reach`, `replies`, `shares`, `total_interactions`, `follows`, `profile_visits` y `navigation` con el desglose `story_navigation_action_type` (adelante, atrás, salir, siguiente cuenta).
- Guarda la miniatura en `data/stories/` antes de que expire.
- Todo en `data/stories.csv`, actualizando la misma fila en cada lectura.

**Cuenta.** Una vez al día:

- Desde `/{ig-user-id}`: `followers_count` y `media_count`.
- Insights diarios: `reach`, `views`, `accounts_engaged`, `total_interactions` y `follows_and_unfollows` (desglose `follow_type`), con `metric_type=total_value` donde corresponda.
- Una fila por día en `data/cuenta_diaria.csv`.

Trampas conocidas de la API, manéjalas desde el principio:

- El día de Meta corta a medianoche de Los Ángeles (`America/Los_Angeles`), no en mi zona. Guarda las fechas con ese corte y convierte solo al mostrar.
- `follows_and_unfollows` tarda uno o dos días en cerrar. Relee los últimos 3 días en cada corrida y sobrescribe.
- Las consultas de alcance de cuenta con ventanas de más de 30 días fallan (error #100). Pide ventanas cortas.
- `follower_count` solo cubre los últimos 30 días. Por eso la foto diaria de `followers_count` es la fuente del histórico.
- Ante límites de uso (códigos 4, 17, 32, 613), reintenta con espera creciente. Si el token deja de servir (código 190), falla con un mensaje claro y sin imprimir el token.

Al cerrar la fase, corre la ingesta una vez y muéstrame un resumen: cuántas piezas, desde qué fecha, cuántas historias y la foto de la cuenta.

## Fase 3: que corra solo

- `.github/workflows/ingest.yml` con dos horarios: la ingesta completa una vez al día, en un minuto que no sea en punto (por ejemplo 10:17 UTC, después del corte de Los Ángeles), y las historias cada 3 horas.
- El token va en el secret `IG_TOKEN`. Súbelo con `gh secret set` leyendo el valor por la entrada estándar desde el archivo, nunca escrito en el comando.
- Commit de los datos con el mensaje `data: ingest AAAA-MM-DD`, solo si algo cambió.
- Permisos mínimos en el workflow: `contents: write`, y `issues: write` para el vigilante.
- Vigilante: si la última ingesta completa exitosa tiene más de 36 horas, abre un issue en el repo (uno solo, sin duplicar) y ciérralo cuando vuelva a funcionar.
- Antes de cerrar la fase, muéstrame la primera corrida exitosa en Actions.

## Fase 4: el dashboard

`motor/build_dashboard.py` genera un solo archivo `dashboard/index.html`, estático y con todo adentro, que se lee bien en el computador y en el teléfono. Se regenera en cada ingesta.

Antes de programarlo, muéstrame un mockup en HTML de Resumen y Piezas con mis datos reales y espera mi OK.

Pestañas:

1. **Resumen.** Arriba, una frase corta de cómo voy, escrita con los números de la semana: cuánto publiqué, a cuánta gente llegué contra la semana anterior y cuántos seguidores gané. Después, la cuenta (seguidores, alcance, views e interacciones contra la semana anterior), lo mejor de la semana, y una lista corta de pendientes accionables: piezas sin etiqueta, lecturas manuales pendientes y propuestas sin responder.
2. **Piezas.** Tabla con todo lo publicado. Ordenable por cualquier columna, con filtros por tipo y tema en una sola fila, y link a cada pieza en Instagram. Columnas: fecha, miniatura, tipo, tema, views, alcance, saves, shares, comentarios, saves cada mil views, shares cada mil views, seguidores cada mil views (si los anoté) y cómo le fue contra la mediana.
3. **Ficha.** Se abre al hacer clic en una pieza. Todos sus números, cómo creció en sus primeros días (desde los snapshots) y la comparación con la mediana de mis 10 piezas anteriores del mismo tipo, escrita como "1,8× tu mediana de views". Si hay menos de 5 anteriores, dilo en vez de comparar.
4. **Qué funciona.** Mis temas y formatos comparados por mediana de views, alcance, saves cada mil y shares cada mil, con cuántas piezas hay detrás de cada número. Con menos de 3 piezas no saques conclusiones. Los trial reels van aparte.
5. **Stories.** Por día: cuántas historias subí, cuánta gente vio la primera, cuánta llegó a la última y en cuál se van más. Tendencia de las últimas 4 semanas.
6. **Calendario.** Vista mensual. Hacia atrás, cada pieza en su día con un número clave y link a su ficha. Hacia adelante, lo que tengo planeado en `pipeline/plan.md`, que edito yo.

Diseño:

- Sobrio y legible. Una familia tipográfica, números grandes donde importan, un color por tipo de pieza y el resto en neutros.
- Sin degradados en las tarjetas de números, sin emoji como íconos, sin flechas de colores en todo. Un número sin con qué compararse va sin porcentaje.
- Sin dependencias externas: CSS y JavaScript dentro del mismo archivo. En el teléfono, una sola columna.

## Fase 5: verlo

Pregúntame cuál de estas dos quiero:

- **Local.** Abro `dashboard/index.html` en mi computador. Se actualiza con `git pull`.
- **En línea y privado.** Cloudflare (Workers con assets estáticos, o Pages) detrás de Cloudflare Access, con login por código al correo y solo para mi correo. Access se configura antes del primer deploy, para que el dashboard no quede público ni un minuto. El deploy corre en el mismo workflow, después de cada ingesta.

Nunca uses GitHub Pages: en cuentas normales el sitio queda público aunque el repo sea privado.

## Fase 6 (opcional): lo que la API no da

A los 7 días de cada reel, el Resumen me recuerda anotar lo que solo se ve en los insights de la app: seguidores que trajo y porcentaje de no seguidores. Lo anoto en `data/manual.csv`, con el formato que tú me des, o te lo dicto y tú lo escribes. Esa lectura nunca se automatiza y nunca abres Instagram en un navegador para sacarla.

## Fase 7 (opcional): propuestas de los lunes

Arma un procedimiento en `tareas/propuestas-lunes.md` que pueda correr cada lunes, como tarea programada o a mano:

1. Lee los datos de las últimas 8 semanas y `CONTEXT.md`.
2. Propone de 3 a 5 piezas para la semana. Cada una con qué es, por qué (el dato detrás, con número) y qué métrica se va a mirar a los 7 días para saber si funcionó.
3. Las escribe en `pipeline/propuestas/AAAA-SS.md` con estado `pendiente`.
4. Yo respondo en una conversación: sí, no o aplazar, con el motivo. Tú actualizas el estado.
5. A los 7 días de publicada cada propuesta aprobada, revisa si se cumplió lo esperado y lo anota en el mismo archivo.

Las propuestas salen de mis datos, no de tendencias genéricas. Si no hay datos suficientes para afirmar algo, dilo.

## Al terminar

- El `README.md` del repo explica qué hace, cómo correrlo a mano, de dónde sale cada métrica, los límites de la API y qué hacer si Meta cambia algo.
- Dame una lista de lo que quedó pendiente o no se pudo hacer, y por qué.
