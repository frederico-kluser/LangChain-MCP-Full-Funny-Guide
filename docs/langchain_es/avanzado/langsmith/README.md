# LangSmith

LangSmith es una plataforma para monitorear, evaluar y probar aplicaciones LLM construidas con LangChain.

## Características Principales

- Seguimiento y visualización de ejecuciones de chains
- Evaluación de salidas de LLM según métricas definidas
- Pruebas comparativas de diferentes modelos y chains
- Depuración y análisis de errores
- Monitoreo de rendimiento en desarrollo y producción

## Integración con LangChain

LangSmith puede integrarse con aplicaciones LangChain con cambios mínimos de código:

```python
import os
from langchain.callbacks.tracers import LangChainTracer

os.environ["LANGCHAIN_TRACING"] = "true"
tracer = LangChainTracer(project_name="Mi Proyecto")

# Tu código LangChain
chain = LLMChain(llm=llm, prompt=prompt, callbacks=[tracer])
```

## Uso de la Plataforma LangSmith

La plataforma LangSmith proporciona:

- Interfaz visual para examinar trazas
- Paneles de evaluación
- Herramientas de revisión colaborativa
- Gestión de conjuntos de datos para pruebas