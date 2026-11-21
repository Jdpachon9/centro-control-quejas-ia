# Cómo armar el tablero en Power BI

El tablero responde en una sola página qué genera más quejas, dónde se atasca el proceso, qué casos están por vencerse y qué tan bien clasifica el agente. El archivo terminado es `centro_control_quejas.pbix`; esta guía sirve para rehacerlo o cambiarlo.

Los nombres de menús van en español y, entre paréntesis, como aparecen si Power BI está en inglés.

## Lo que necesitas

- **Power BI Desktop.** Es gratis y funciona en Windows. Se instala desde Microsoft Store. Si al abrirlo pide iniciar sesión, puedes cerrar esa ventana.
- **Tres archivos de esta carpeta**, que el cuaderno del agente crea al ejecutarse completo:
  - `quejas_tablero.csv`: una fila por queja, con su clasificación, fecha límite y estado.
  - `cola_diaria.csv`: cuántas quejas tenía abiertas cada área al final de cada día.
  - `evaluacion_agente.csv`: si el agente acertó o no en cada queja.

## 1. Cargar los datos

1. *Inicio > Obtener datos > Texto o CSV* (*Home > Get data > Text/CSV*) y elige `quejas_tablero.csv`.
2. En la vista previa revisa que diga *65001: Unicode (UTF-8)* y *Delimitador: Coma*, y dale **Cargar** (**Load**).
3. Repite con `cola_diaria.csv` y `evaluacion_agente.csv`.
4. En la *Vista de tabla* (*Table view*) revisa que `fecha_recibo`, `semana`, `fecha_limite`, `fecha_respuesta` y `fecha` sean de tipo *Fecha* (*Date*).
5. En la *Vista de modelo* (*Model view*) revisa que `quejas_tablero` y `evaluacion_agente` estén unidas por `id`. Si no, arrastra `id` de una tabla a la otra.

## 2. Crear las medidas

Una medida es un cálculo que se actualiza con los filtros. Para cada una: clic derecho en `quejas_tablero` > *Nueva medida* (*New measure*), pega la fórmula y presiona Enter.

Crea cada medida con *Nueva medida*. Si pegas una fórmula en la barra de otra medida, la reemplazas.

```
Quejas = COUNTROWS(quejas_tablero)
Abiertas = SUM(quejas_tablero[abierta])
Vencidas abiertas = CALCULATE([Quejas], quejas_tablero[estado] = "Abierta y vencida") + 0
Por vencer = SUM(quejas_tablero[por_vencer])
% a tiempo = DIVIDE(SUM(quejas_tablero[a_tiempo]), SUM(quejas_tablero[respondida]))
Días promedio de respuesta = AVERAGE(quejas_tablero[dias_habiles_respuesta])
% revisión completa = DIVIDE(CALCULATE([Quejas], quejas_tablero[nivel_revision] = "Revisión completa") + 0, [Quejas])
% acierto del agente = AVERAGE(evaluacion_agente[producto_y_motivo_correctos])
```

- El `+ 0` hace que la tarjeta muestre 0 en vez de quedar en blanco cuando no hay casos.
- Las tres medidas que empiezan con `%` van con formato de porcentaje y un decimal (*Herramientas de medición > %*, *Measure tools > %*). `Días promedio de respuesta` va con un decimal.
- Si Power BI marca error en las comas, cambia cada `,` dentro de los paréntesis por `;`.

## 3. Armar la página

Tamaño de página: *Personalizado* (*Custom*), 1280 × 960. Título: "Centro de control de quejas · Banco Ejemplo (datos simulados)".

| Lugar | Visual | Cómo se arma |
|---|---|---|
| Arriba a la derecha | Dos segmentaciones (*Slicer*) | `canal` y `producto`, con estilo de lista desplegable (*Dropdown*) |
| Franja bajo el título | Una tarjeta (*Card*) con 7 cifras | `Quejas`, `% a tiempo`, `Abiertas`, `Vencidas abiertas`, `Por vencer`, `% acierto del agente` y `% revisión completa` en la misma casilla |
| Centro, izquierda | Barras agrupadas (*Clustered bar chart*) | Eje Y: `motivo`. Eje X: `Quejas`. Orden descendente y etiquetas de datos |
| Centro, derecha | Líneas (*Line chart*) | De `cola_diaria`: eje X `fecha` (la fecha, no la jerarquía), eje Y `abiertas_al_cierre`, leyenda `area`. Fraude en rojo |
| Abajo, izquierda | Matriz (*Matrix*) | Filas: `area`. Valores: `Quejas`, `% a tiempo`, `Días promedio de respuesta` y `Abiertas` |
| Abajo, derecha | Tabla (*Table*) | `id`, `area`, `motivo`, `fecha_limite`, `dias_habiles_restantes` y `estado`, filtrada a "Abierta y vencida" y "Abierta por vencer" y ordenada por días restantes |

En la tabla:

- `fecha_limite` va como fecha y no como jerarquía.
- `dias_habiles_restantes` va sin resumir (*Don't summarize*).
- La fecha lleva el formato `dd/mm/yyyy`, para que no se lea al estilo de Estados Unidos.

La guía original traía un gráfico de acierto según la confianza del agente. Se quitó porque el modelo dijo «alta» en las 300 quejas, así que mostraba una sola barra (hallazgo 2 del README).

## 4. Guardar y exportar

1. *Archivo > Guardar como* (*File > Save as*): `tablero/centro_control_quejas.pbix`.
2. *Archivo > Exportar > Exportar a PDF* (*File > Export > Export to PDF*). Power BI no pregunta dónde guardarlo: lo abre y lo deja en `%TEMP%\Power BI Desktop\print-job-…`. Cópialo a `tablero/centro_control_quejas.pdf`.
3. La imagen `tablero.png` del README sale de ese PDF.

Para publicar el tablero en la web de Power BI se necesita una licencia paga. Para el portafolio basta con el `.pbix`, el PDF y la imagen.
