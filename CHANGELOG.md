# Changelog

Todos los cambios relevantes del proyecto Tintico Lab se documentan en este archivo.

El proyecto se desarrolla por milestones funcionales. Cada milestone representa una evolución de la misma solución, no un proyecto independiente.

---

## [M1] — Classification + Tool + Policies

**Estado:** completado

### Objetivo

Construir una primera versión funcional del sistema de postventa capaz de interpretar mensajes de clientes, consultar pedidos simulados, aplicar políticas de negocio y devolver una decisión estructurada.

### Incluye

- clasificación de consultas y reclamos con IA;
- extracción estructurada de datos;
- AI Agent con Structured Output Parser;
- Tool `buscar_pedido`;
- dataset de pedidos simulado;
- políticas editables en Google Sheets;
- selección de política por categoría y `productType`;
- fallback a políticas `productType = all`;
- routing previo a políticas;
- motor determinístico de reglas de negocio;
- manejo de datos faltantes;
- manejo de pedidos no encontrados;
- prioridad especial para casos B2B de máquinas;
- contrato final de salida unificado;
- pruebas funcionales oficiales.

### Categorías

- `PRODUCTO_DANADO`
- `PRODUCTO_INCORRECTO`
- `PEDIDO_DEMORADO`
- `CAMBIO_DEVOLUCION`
- `GARANTIA`
- `FUERA_DE_ALCANCE`

### Decisiones

- `RESUELTO`
- `FALTA_UN_DATO`
- `ESCALAR`
- `FUERA_DE_POLITICA`

### Validación

- Casos oficiales ejecutados: 9
- Casos aprobados: 9
- Resultado: **9/9 PASS**

### Ajustes finales

- se agregó un contrato de salida único mediante `Salida final M1`;
- se corrigió el mapeo dinámico de `requiresHuman`;
- se mantuvo visible el error cuando no existe una política aplicable;
- se documentó la normalización de celdas numéricas vacías de Google Sheets como `0`;
- se realizó una regresión final sobre distintas ramas del workflow.

### Limitaciones conocidas

- pedidos simulados;
- sin tienda real conectada;
- sin canal real de entrada;
- sin persistencia;
- sin memoria conversacional;
- sin acciones comerciales reales;
- sin monitoreo productivo;
- manejo técnico de errores externos todavía limitado.

---

## [M2] — Manager + Workers

**Estado:** próximo

### Objetivo previsto

Evolucionar M1 hacia una arquitectura modular con un workflow coordinador y workers especializados.

Componentes previstos:

- Manager;
- Worker de Logística;
- Worker de Cambios y Devoluciones;
- Worker de Garantías.

M2 reutilizará los principios, contratos y aprendizajes establecidos en M1.
