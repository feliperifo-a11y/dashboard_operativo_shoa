# DASHBOARD OPERATIVO SHOA

Autor: CF Felipe Rifo Espósito · feliperifo@gmail.com

## Qué hay en esta carpeta

| Archivo | Función |
|---|---|
| `index.html` | El dashboard. Abre en el **resumen** (1. Monitoreo estaciones, 2. Personal y unidades, 3. Noticias SHOA) y el botón **Ver mapa** (o la dirección `…/#mapa`) muestra estaciones, unidades y personal desplegado. Autocontenido, con una instantánea del último estado conocido. |
| `colector.py` | Consulta las fuentes y escribe `estado.json`. Solo biblioteca estándar de Python. |
| `.github/workflows/estado.yml` | Ejecuta el colector cada 10 minutos y publica el sitio en GitHub Pages. |
| `estado.json` | Estado inicial (2 de octubre de 2026). El flujo lo reemplaza en cada ejecución. |

## Fuentes del estado

- **Boyas de oleaje** (Iquique, Antofagasta, Concón, San Antonio, Talcahuano, Desertores, Punta Arenas): series horarias de 7 días del visor web de boyas. Una hora sin `idBoya` cuenta como hora sin dato.
- **Boyas DART** (Iquique, Mejillones, Caldera, Pichidangui, Constitución): archivos de tiempo real de NOAA/NDBC (estaciones 32401, 32403, 32402, 32404 y 34420).
- **Noticias**: las publicaciones de portada del sitio institucional.

El sitio de boyas y de noticias rechaza las consultas que vienen de servidores en la nube (GitHub responde con HTTP 405), pero sí responde a un navegador en Chile. Por eso el reparto es:

- **DART**: las consulta el colector en GitHub y quedan en `estado.json`.
- **Boyas de oleaje y noticias**: las consulta directamente el navegador de quien abre el dashboard, cada 5 minutos. Si esa consulta falla, se muestran los últimos datos disponibles y el pie del área 1 lo indica.

## Semáforo

| Estado | Oleaje | DART |
|---|---|---|
| Operativa | último dato ≤ 3 h | último dato ≤ 6 h |
| Con retraso | 3–24 h | 6–24 h |
| No operativa | > 24 h o sin datos en 7 días | > 24 h o sin archivo en NDBC |

Además se marca con ⚠ una DART en modo evento y una boya cuyo GPS la ubica fuera de su círculo de borneo.
El dashboard recalcula el semáforo cada minuto con la hora del equipo, de modo que una boya pasa a "con retraso" aunque el colector se detenga. Si `estado.json` tiene más de 40 minutos, aparece un aviso.

## Puesta en marcha en GitHub (≈10 minutos)

1. Crear un repositorio **público** nuevo (Actions y Pages son gratuitos en repositorios públicos).
2. Subir los cuatro elementos de esta carpeta, incluida la carpeta oculta `.github`.
3. En *Settings → Pages → Build and deployment → Source*, elegir **GitHub Actions**.
4. En la pestaña *Actions*, abrir "Estado de la red de boyas" y pulsar **Run workflow**.
5. El sitio queda en `https://<usuario>.github.io/<repositorio>/`.

Notas operativas:

- El cron de GitHub no es puntual: con carga, una ejecución de cada 10 minutos puede atrasarse o saltarse. Para mayor regularidad, un equipo externo puede llamar al evento `repository_dispatch` con tipo `actualizar`.
- GitHub **desactiva los flujos programados de un repositorio público tras 60 días sin actividad**. Un commit cualquiera, o reactivarlo desde la pestaña Actions, lo vuelve a encender.
- Las fuentes no son API documentadas. Si cambian, el colector registra el error en `estado.json` (campo `errores`) y la boya afectada aparece como no operativa.

## Buques en sondaje

El registro de buques, posiciones y dotaciones **se guarda solo en el navegador de cada equipo** y nunca se envía al sitio publicado: GitHub Pages es público aunque el repositorio sea privado. Para compartir el registro se usa *Exportar registro* (archivo `.json`) y *Cargar registro* en el otro equipo, por un canal autorizado.

La posición se puede escribir en grados y minutos (`36°42.50′S`, `073°07.20′W`), en decimal (`-36.7083`) o marcar con un clic en el mapa.

## Publicar información para todos

Lo que ven todos los que abren el link está en la carpeta `publicado/`:

| Archivo | Área | Contenido |
|---|---|---|
| `publicado/estaciones.json` | 1. Monitoreo estaciones | Material del SITREP (nivel del mar, DART, meteoceánicas, glider) y mensajes navales. |
| `publicado/personal.json` | 2. Personal y unidades | **Solo cifras, sin nombres**: total, oficiales por grado (CN, CF, CC, T1, T2, ST), gente de mar por grado (SO, S1, S2, C1, C2, M1), EC y PAC; despliegue por unidad, comisiones, embarcaciones y novedades. |
| `publicado/noticias.json` | 3. Noticias SHOA | Noticias propias o resumen de prensa, además de las de la portada. |
| `publicado/unidades.json` | Mapa | Buques y lanchas registrados y posiciones corregidas, con cantidad a bordo sin nombres. |

**Todos ven lo mismo.** Quien abre el link sin modo editor ve solo lo publicado: no tiene botones de carga y cualquier vista local antigua de su equipo se descarta.

**Cómo publicar (publicación directa):**

1. Abrir el dashboard en **modo editor**: agregar `?editor=1` a la dirección. Queda recordado en ese equipo.
2. La primera vez, conectar la publicación: al pie dice «Publicación directa: no conectada · conectar». Se pide un **token de GitHub** de grano fino con acceso solo a este repositorio y permiso *Contents: Read and write*. Se guarda únicamente en ese equipo.
3. En cada área, **Cargar información** → pegar el texto o abrir el Word → **Publicar para todos**. El cambio queda en el repositorio y el sitio se actualiza para todos en uno o dos minutos.
4. Lo que se registra o corrige en el mapa (buques, lanchas, posiciones) también se publica solo, sin nombres: solo la cantidad a bordo por categoría.

**Solo en este equipo** sirve para revisar un texto sin publicarlo. Sin la publicación conectada, sigue disponible el método manual (*Preparar publicación para todos* → copiar y pegar en GitHub).

El sitio es público. Por eso el área 2 y el mapa nunca publican nombres. El dashboard descargado como archivo también muestra lo publicado, si hay conexión a internet.

## Resumen de prensa diario (área 3)

En **Cargar información** del área 3, el botón **Abrir resumen de prensa (.docx)** lee directamente el Word que llega cada día. El documento debe seguir su formato habitual: FUENTE en negrita, TÍTULO en negrita, texto y enlace.

- En la página principal aparece cada noticia con su **fuente**, su **título** y el enlace **Ver noticia ↗** al medio.
- **Ver resumen completo →** abre el resumen entero, con textos e imágenes, en la dirección `…/#prensa`. Se puede compartir, imprimir o guardar como PDF.
- Cuando una noticia viene solo como imagen (sin título escrito), el título se toma del enlace.
- La firma del documento no se incorpora.
- Para publicarlo para todos: en modo editor, con la publicación conectada, **Publicar para todos**. El archivo publicado (`publicado/noticias.json`) pesa alrededor de 100 KB, porque las imágenes se comprimen.

## Probar sin red

```
SAMPLES_DIR=muestras python3 colector.py
```

lee respuestas guardadas (`wave_<id>.txt`, `dart_<id>.txt`, `news.json`) en lugar de consultar las fuentes.
