# Gobierno del agente

Aquí quedan las evidencias de control. Los archivos los crea el cuaderno `agente/agente_quejas.ipynb` al ejecutarse.

| Archivo | Qué contiene |
|---|---|
| `registro_decisiones.csv` | Una fila por queja: qué propuso el modelo y por qué, con qué modelo y versión de instrucciones, qué decidieron las reglas (ruta, área, plazo, fecha límite) y el nivel de revisión. Las columnas `decision_humana`, `revisado_por`, `fecha_revision` y `correccion` son para la persona que aprueba o corrige |
| `resultados_ataques.csv` | Las 20 quejas con órdenes escondidas: qué pedía cada ataque, qué propuso el modelo, si obedeció, si lo detectó y si alguna pasó sin revisión completa |
| `resultados_ataques.jsonl` | Respuestas completas del modelo a los ataques, para auditoría |

Los riesgos y sus controles están en la sección 6 del [README del proyecto](../README.md).
