# Cómo armar el tablero en Power BI

El tablero responde cuatro preguntas en una sola página: qué genera más quejas, dónde se atasca el proceso, qué casos están por vencerse y qué tan bien clasifica el agente.

## Lo que necesitas

- **Power BI Desktop.** Es gratis y funciona en Windows. Se instala desde Microsoft Store buscando "Power BI Desktop". Si al abrirlo pide iniciar sesión, puedes cerrar esa ventana.
- **Tres archivos de esta carpeta**, que crea el cuaderno del agente al ejecutarlo completo:
  - `quejas_tablero.csv`: una fila por queja, con su clasificación, fecha límite y estado.
  - `cola_diaria.csv`: cuántas quejas tenía abiertas cada área al final de cada día.
  - `evaluacion_agente.csv`: si el agente acertó o no en cada queja.

## 1. Cargar los datos

1. *Inicio > Obtener datos > Texto o CSV* y elige `quejas_tablero.csv`.
2. En la vista previa revisa que diga *65001: Unicode (UTF-8)* y *Delimitador: Coma*. Haz clic en **Cargar**.
3. Repite con `cola_diaria.csv` y `evaluacion_agente.csv`.
4. Abre la *Vista de tabla* (ícono de cuadrícula, a la izquierda) y revisa que las columnas `fecha_recibo`, `semana`, `fecha_limite`, `fecha_respuesta` y `fecha` tengan el ícono de calendario. Si alguna no lo tiene, selecciónala y cambia *Herramientas de columnas > Tipo de datos > Fecha*.
5. En la *Vista de modelo* (tercer ícono) revisa que exista una línea entre `quejas_tablero[id]` y `evaluacion_agente[id]`. Si no aparece, arrastra `id` de una tabla a la otra.

## 2. Crear las medidas

Una medida es un cálculo que se actualiza con los filtros. Selecciona la tabla `quejas_tablero`, haz clic en *Nueva medida* y pega cada línea por separado:

```
Quejas = COUNTROWS(quejas_tablero)
Abiertas = SUM(quejas_tablero[abierta])
Vencidas abiertas = CALCULATE([Quejas], quejas_tablero[estado] = "Abierta y vencida")
Por vencer = SUM(quejas_tablero[por_vencer])
% a tiempo = DIVIDE(SUM(quejas_tablero[a_tiempo]), SUM(quejas_tablero[respondida]))
Días promedio de respuesta = AVERAGE(quejas_tablero[dias_habiles_respuesta])
% revisión completa = DIVIDE(CALCULATE([Quejas], quejas_tablero[nivel_revision] = "Revisión completa"), [Quejas])
% acierto del agente = AVERAGE(evaluacion_agente[producto_y_motivo_correctos])
```

- A las tres medidas que empiezan con `%`, dales formato de porcentaje: *Herramientas de medición > Formato > Porcentaje*.
- Si Power BI marca error en las comas, tu equipo usa punto y coma: cambia cada `,` que separa datos dentro de los paréntesis por `;`.

## 3. Armar la página

Ponle de título "Centro de control de quejas · Banco Ejemplo (datos simulados)".

**Arriba: cinco tarjetas.** Visual *Tarjeta*, una por medida: `Quejas`, `% a tiempo`, `Abiertas`, `Vencidas abiertas` y `Por vencer`.

**¿Qué genera más quejas?** Visual *Gráfico de barras agrupadas*.
- Eje Y: `motivo`. Eje X: `Quejas`.
- En los tres puntos del visual, *Ordenar eje > Quejas* y orden descendente.

**¿Dónde se atasca el proceso?** Dos visuales.
- *Gráfico de líneas* con datos de `cola_diaria`. Eje X: `fecha`. Eje Y: `abiertas_al_cierre` (suma). Leyenda: `area`. En *Formato > Líneas*, pon Fraude en un color fuerte y las demás áreas en grises.
- *Matriz*. Filas: `area`. Valores: `Quejas`, `% a tiempo`, `Días promedio de respuesta` y `Abiertas`.

**¿Qué está por vencerse?** Visual *Tabla*.
- Columnas: `id`, `area`, `motivo`, `fecha_limite`, `dias_habiles_restantes` y `estado`.
- En el panel *Filtros*, en *Filtros en este objeto visual*, arrastra `estado` y marca solo "Abierta y vencida" y "Abierta por vencer".
- Ordena por `dias_habiles_restantes`, de menor a mayor: arriba quedan las más atrasadas.

**¿Qué tan bien clasifica el agente?**
- *Tarjeta* con `% acierto del agente` y otra con `% revisión completa`.
- *Gráfico de barras agrupadas*. Eje Y: `confianza` de la tabla `evaluacion_agente`. Eje X: la medida `% acierto del agente`. Muestra si la confianza que declara el agente sirve para decidir qué revisar: las quejas de confianza alta deberían tener más acierto que las de confianza baja.

**Segmentadores:** visual *Segmentación de datos* con `mes`, `canal` y `producto`. Filtran todo menos el gráfico de líneas, que viene de otra tabla.

## 4. Guardar y exportar

1. *Archivo > Guardar como*: `tablero/centro_control_quejas.pbix`.
2. *Archivo > Exportar > Exportar a PDF*: `tablero/centro_control_quejas.pdf`.
3. Toma una captura de la página con `Windows + Shift + S` y guárdala como `tablero/tablero.png`. Esa imagen va en el README del repositorio.

Para publicar el tablero en la web de Power BI se necesita una licencia paga. Para el portafolio basta con el archivo `.pbix`, el PDF y la captura.
