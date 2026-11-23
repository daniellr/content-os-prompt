# Content OS (el prompt)

Un prompt para que Claude Code te arme tu propio sistema de contenido para Instagram. Todos los días lee tus métricas por la API oficial, las guarda en tu repo y te muestra un dashboard privado con cómo vas. Si quieres, también te propone qué publicar cada lunes y a la semana revisa si funcionó.

Acá no hay app que instalar ni cuenta que crear. Lo armas tú, con Claude Code, en tu propio GitHub, y queda tuyo.

## Qué hace

- Lee todos los días tus reels, posts, carruseles e historias, más una foto diaria de la cuenta: seguidores, alcance, views e interacciones.
- Guarda todo en archivos CSV dentro de tu repo, con historia.
- Arma un dashboard con estas pestañas:
  - **Resumen**: cómo vas esta semana contra la anterior, la cuenta, lo mejor de la semana y lo que tienes pendiente.
  - **Piezas**: todo lo que has publicado, ordenable por cualquier métrica, con link a cada pieza.
  - **Ficha** de cada pieza: sus números y cómo le fue contra tus diez piezas anteriores del mismo tipo.
  - **Qué funciona**: tus temas y formatos comparados entre sí.
  - **Stories**: cuánta gente llega hasta la última historia del día y dónde se van.
  - **Calendario**: hacia atrás, lo publicado con sus números; hacia adelante, lo que tienes planeado.
- Opcional: los lunes te propone qué publicar, con el dato detrás de cada propuesta, y a los 7 días revisa si se cumplió.

## ¿Y por qué no usar los Insights de Instagram?

Los Insights sirven, y este sistema lee lo mismo que ellos. Lo que agrega:

- **Comparación con tus piezas anteriores.** Instagram te muestra cada pieza por separado. Acá cada reel se compara con la mediana de tus diez reels anteriores, así sabes si de verdad le fue bien o si fue un día normal.
- **Métricas cada mil views.** Saves, shares y seguidores por cada mil views, para comparar un reel de 10 mil con uno de 50 mil.
- **Tus propias etiquetas.** Tema, formato, trial reel o contenido para una marca. Con eso puedes ver qué tema se guarda más o qué formato trae más seguidores.
- **Historia completa.** Instagram limita cuánto puedes mirar hacia atrás en las métricas de cuenta. Acá se guardan día a día desde que lo prendes.

## Lo que necesitas

- Una cuenta de Instagram profesional (creador o empresa) conectada a una página de Facebook.
- Una cuenta de desarrollador en Meta. Es gratis.
- Claude Code.
- Una cuenta de GitHub.
- Un par de horas la primera vez. Lo más lento es la configuración en Meta.

Opcional: una cuenta gratis de Cloudflare, si quieres abrir el dashboard desde el teléfono con login.

## Cómo se usa

1. Crea un repo **privado** y vacío en GitHub, y clónalo en tu computador.
2. Abre Claude Code en esa carpeta.
3. Copia el contenido completo de [PROMPT.md](PROMPT.md) y pégalo.
4. Claude Code avanza por fases. Al terminar cada una se detiene, te muestra lo que hizo y espera que le digas "sigue". Los pasos en Meta los haces tú en su web, y Claude Code te guía uno por uno.

Antes de pegarlo puedes cambiar lo que quieras: las pestañas, los tipos de pieza, los horarios.

## Límites, para que no te pillen

- La API no entrega cuántos seguidores trajo cada reel ni el porcentaje de no seguidores que lo vio. Eso solo aparece en los insights de la app. El prompt incluye una forma opcional de anotarlo a mano; nunca automatiza la web ni la app de Instagram.
- Tampoco entrega la curva de retención ni las fuentes de tráfico de cada reel.
- No publica por ti. Solo lee.
- Hacia atrás llega hasta donde Meta deja. Las métricas diarias de cuenta empiezan a acumularse el día que lo prendes.
- Los seguidores ganados en un día terminan de cerrarse uno o dos días después, así que el último día siempre se ve más bajo.
- Meta cambia la API seguido. Si algo deja de funcionar, pégale el error a Claude Code.

## Privacidad

- El token de Meta vive en tu computador y en los Secrets de GitHub. Nunca en el repo.
- Tu repo tiene que ser privado, porque guarda tus métricas.
- El dashboard no se publica en GitHub Pages. O lo abres en tu computador, o lo subes a Cloudflare detrás de un login que solo tú puedes pasar.
- Este repo no tiene datos de nadie. Es solo el prompt.

## Créditos

Lo armé con Claude para mi propia cuenta. Si te armas el tuyo, cuéntame cómo te fue.

Licencia MIT: úsalo, cámbialo y compártelo.
