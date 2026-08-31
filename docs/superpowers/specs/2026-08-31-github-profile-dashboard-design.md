# Diseño del dashboard para el perfil de GitHub

Fecha: 2026-08-31

## Objetivo

Rediseñar el `README.md` del perfil `kevinsrb` para acercarlo a la referencia visual proporcionada: una interfaz oscura, tecnológica y profesional, con acentos naranjas, una tarjeta personal lateral y un área principal de contenido. El resultado debe conservar enlaces y contenido accesibles en GitHub, evitando convertir todo el perfil en una sola imagen.

## Punto de retorno

El estado anterior al rediseño está guardado y publicado en el commit `0cd0827` (`fix: migrate GitHub stats cards`). El rediseño no modificará ni eliminará `assets/banner-final.svg`, que seguirá disponible como respaldo visual adicional.

## Enfoque seleccionado

Se usará una composición híbrida modular:

- SVG locales para los elementos visuales complejos: tarjeta personal y banner principal.
- HTML y Markdown compatibles con GitHub para enlaces, botones, textos, tecnologías, proyectos y estadísticas.
- Una composición de escritorio de aproximadamente 28 % para la columna lateral y 72 % para el contenido principal.

La división principal se construirá con una tabla HTML admitida por GitHub. El contenido interior usará HTML explícito cuando Markdown no se procese dentro de sus celdas.

Este enfoque ofrece mayor fidelidad que un README completamente nativo y más accesibilidad y facilidad de mantenimiento que un único SVG de página completa.

## Restricciones

- GitHub no permite CSS libre dentro de un README; la composición debe usar HTML admitido, tablas, imágenes y atributos simples.
- El diseño será optimizado principalmente para escritorio. En anchos pequeños debe conservar legibilidad, aunque las dos columnas se compriman.
- No se mostrarán métricas, estrellas, seguidores ni fechas inventadas.
- No se incluirá una hora local estática porque quedaría desactualizada.
- Los datos privados no formarán parte de los recursos generados.

## Arquitectura visual

### Columna lateral

Una nueva tarjeta `assets/profile-sidebar.svg` contendrá:

- Avatar actual de GitHub guardado como recurso local.
- Fotografía circular con aro naranja y un indicador de estado verde decorativo.
- Nombre `Kevin Rodriguez` y usuario `kevinsrb`.
- Rol `Full Stack Developer · Backend Focused`.
- Biografía pública resumida.
- Ubicación `San Marcos - Sucre, Colombia`.
- Correo `kevinsrb.1999@gmail.com`.
- LinkedIn `linkedin.com/in/kevinsamirrodriguez`.
- Usuario `@kevinsrb`.
- Fecha real de creación de la cuenta: `Joined Apr 2021`.
- Un bloque visual de conexión que acompañará enlaces nativos ubicados en el README.

La tarjeta no incluirá cantidades manuales de repositorios, seguidores, estrellas o contribuciones. Las métricas visibles se obtendrán de tarjetas dinámicas en la sección de actividad.

### Contenido principal

Un nuevo banner, sin reemplazar el banner actual, contendrá:

- Monograma `KR` grande y claramente reconocible.
- Nombre `KEVIN RODRIGUEZ` con separación correcta.
- Subtítulo `FULL STACK DEVELOPER · BACKEND FOCUSED`.
- Chips de especialidad: backend, arquitectura limpia, APIs escalables y cloud.
- Iconos oficiales y a color para Node.js, TypeScript, NestJS, AWS, GCP, Docker y Kubernetes.
- Detalles geométricos y circuitos naranjas sobre un fondo grafito.

Debajo del banner aparecerán botones nativos y clicables para GitHub, LinkedIn, correo y contacto. `Email` abrirá el correo directamente y `Contacto` usará el mismo correo con el asunto `Contacto profesional desde GitHub`.

## Secciones de contenido

### Sobre mí

El texto actual se reorganizará en dos grupos visuales para acercarse a la referencia. Mantendrá los datos verificables del README existente:

- Más de seis años de experiencia.
- Enfoque en backend, APIs y arquitectura de software.
- Node.js, TypeScript, NestJS y .NET/C#.
- Clean Architecture, DDD y arquitectura hexagonal cuando sean apropiadas.
- PostgreSQL, MySQL, MongoDB, Firestore y Redis.
- AWS, GCP, Docker y Kubernetes.
- Interés en IA, automatización y DevOps.

### Tecnologías

Los iconos actuales se reorganizarán en categorías con etiquetas legibles:

- Backend: Node.js, TypeScript, NestJS, Express, Fastify, .NET y C#.
- Frontend: Angular, React y Vue.js.
- Bases de datos: PostgreSQL, MySQL, MongoDB, Redis y Firestore.
- DevOps y cloud: AWS, GCP, Docker, Kubernetes, GitHub Actions y Linux.

Los badges tendrán fondo grafito, borde discreto e iconos oficiales a color. Deben conservar texto alternativo.

Los nombres permanecerán como texto HTML y los iconos se cargarán desde archivos locales en `assets/icons/`. No se dependerá de un servicio externo para representar esta sección.

### Proyectos destacados

Se mostrarán tres repositorios públicos reales en tarjetas equivalentes:

1. `inventory-management-api`.
2. `create-flex-stack`.
3. `yugioh-app`.

Cada tarjeta tendrá enlace al repositorio, descripción breve basada en su contenido real, tecnologías principales y lenguaje predominante. No se mostrarán cantidades estáticas de estrellas o forks.

### Actividad en GitHub

Se mantendrán las dos tarjetas dinámicas actuales de `github-stats-extended.vercel.app`:

- Estadísticas generales.
- Lenguajes más usados.

La sección tendrá texto alternativo útil y un enlace al perfil para que el fallo temporal del servicio no bloquee el resto del README.

## Datos y dependencias

- Los datos públicos de cuenta y repositorios se verificarán mediante GitHub antes de generar los recursos.
- El avatar se descargará como `assets/avatar-kevin.png` y también se incorporará en la tarjeta lateral para evitar que el SVG dependa de una referencia externa interna.
- Los iconos esenciales del banner estarán incorporados o almacenados localmente.
- Las estadísticas serán la única dependencia visual dinámica externa imprescindible.
- Los enlaces externos se mantendrán fuera de los SVG cuando necesiten interacción individual.

## Accesibilidad y degradación

- Todas las imágenes tendrán texto alternativo descriptivo.
- El contraste entre texto y fondo será suficiente para lectura sobre grafito.
- Los textos informativos importantes permanecerán como HTML o Markdown cuando sea posible.
- Si una tarjeta dinámica no carga, el resto del perfil y sus enlaces seguirán funcionando.
- Los SVG usarán dimensiones y `viewBox` coherentes para escalar sin deformación.

## Archivos previstos

- Modificar `README.md` para la composición, contenido y enlaces.
- Crear `assets/profile-sidebar.svg`.
- Crear `assets/banner-dashboard.svg` para el nuevo banner principal.
- Crear `assets/avatar-kevin.png`.
- Crear en `assets/icons/` únicamente los iconos necesarios para las tecnologías mostradas.
- Conservar todos los SVG existentes, incluido `assets/banner-final.svg`.

## Verificación

Antes de publicar el rediseño se comprobará:

1. Que los SVG sean XML válido y conserven su proporción al renderizarse.
2. Que el avatar y los iconos locales aparezcan correctamente.
3. Que GitHub, LinkedIn, correo, contacto y los tres proyectos apunten a destinos válidos.
4. Que las tarjetas dinámicas respondan con imágenes SVG válidas.
5. Que no existan recursos faltantes ni rutas locales de Windows en `README.md`.
6. Que la composición siga siendo legible en un ancho reducido.
7. Que `git diff` solo contenga archivos pertenecientes al rediseño.

## Criterios de aceptación

- El perfil se reconoce visualmente como una versión cercana a la referencia oscura y naranja.
- La tarjeta lateral usa el avatar y los datos reales aprobados.
- El banner conserva el monograma `KR`, el nombre correctamente separado y los iconos oficiales.
- Los botones y proyectos son clicables.
- Se muestran tres proyectos públicos reales.
- No aparecen cifras o repositorios inventados.
- El estado anterior sigue recuperable desde `0cd0827` y los recursos visuales anteriores permanecen en el historial y en el árbol de trabajo.

## Fuera de alcance

- Replicar exactamente la interfaz completa de GitHub mostrada en la referencia.
- Añadir CSS, JavaScript o interacciones que GitHub elimina de los README.
- Crear un sitio web independiente.
- Automatizar la actualización periódica de datos estáticos del perfil.
