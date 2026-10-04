# Tintico Lab — Intelligent Post-Sales Automation

Proyecto de automatización de postventa para un e-commerce ficticio de café de especialidad, desarrollado de forma incremental con **n8n**, IA y lógica determinística.

El objetivo del proyecto es construir una solución de postventa mantenible, modular y preparada para evolucionar hacia integraciones reales con e-commerce, CRM, mensajería y sistemas internos.

---

## Problema

En un proceso manual de postventa, una persona debe:

- interpretar el mensaje del cliente;
- identificar el tipo de problema;
- localizar el pedido;
- revisar datos del producto;
- consultar políticas;
- decidir qué corresponde;
- solicitar información faltante;
- escalar excepciones.

Este proceso puede ser lento, inconsistente y difícil de auditar.

Tintico Lab busca automatizar progresivamente ese flujo sin delegar reglas críticas de negocio a un modelo generativo.

---

## Enfoque

El proyecto utiliza una arquitectura híbrida:

**IA para interpretar lenguaje natural.**  
**Tools para obtener hechos.**  
**Reglas determinísticas para tomar decisiones de negocio.**

Principio central:

```text
Interpretar → Obtener datos → Validar → Aplicar política → Decidir → Estructurar resultado
```

---

## Roadmap

### M1 — Classification + Tool + Policies

**Estado: completado**

Primera versión funcional del sistema de postventa.

Incluye:

- clasificación de consultas y reclamos con IA;
- extracción estructurada de datos;
- Tool `buscar_pedido`;
- dataset de pedidos simulado;
- políticas editables en Google Sheets;
- selección de políticas por categoría y tipo de producto;
- motor determinístico de decisiones;
- manejo de datos faltantes;
- escalamiento humano;
- prioridades;
- contrato de salida unificado;
- pruebas oficiales: **9/9 PASS**.

Arquitectura simplificada:

```text
Input
  ↓
AI Agent
  ├── OpenAI Chat Model
  ├── Tool buscar_pedido
  └── Structured Output Parser
  ↓
Evaluar estado inicial
  ↓
¿Necesita política?
  ├── No → Resolver sin política
  │
  └── Sí → Buscar política
             ↓
          Seleccionar política
             ↓
          Aplicar reglas
             ↓
         Salida final M1
```

---

### M2 — Manager + Workers

**Estado: próximo**

M2 evolucionará la arquitectura hacia una estructura modular con un workflow coordinador y workflows especializados.

Componentes previstos:

```text
Manager
├── Worker — Logística
├── Worker — Cambios y Devoluciones
└── Worker — Garantías
```

El objetivo es separar responsabilidades, mejorar mantenibilidad y preparar el sistema para crecer sin convertir el workflow principal en un monolito.

---

### Próximas etapas

El roadmap contempla progresivamente:

- memoria por cliente o reclamo;
- integraciones reales;
- RAG para políticas, procedimientos y documentación;
- procesamiento de voz;
- Human-in-the-Loop;
- evaluación automática de respuestas;
- métricas, logs y trazabilidad;
- modularidad avanzada, asincronía e idempotencia.

---

## Categorías soportadas en M1

```text
PRODUCTO_DANADO
PRODUCTO_INCORRECTO
PEDIDO_DEMORADO
CAMBIO_DEVOLUCION
GARANTIA
FUERA_DE_ALCANCE
```

Estados de decisión:

```text
RESUELTO
FALTA_UN_DATO
ESCALAR
FUERA_DE_POLITICA
```

---

## Ejemplo de salida

```json
{
  "category": "PEDIDO_DEMORADO",
  "orderId": "TL-1003",
  "decision": "ESCALAR",
  "priority": "HIGH",
  "requiresHuman": true,
  "missingData": [],
  "reason": "El pedido superó la fecha estimada de entrega."
}
```

---

## Stack

- n8n
- OpenAI
- Google Sheets
- JavaScript
- JSON
- Git / GitHub

---

## Estructura del repositorio

```text
tintico-lab-postventa/
│
├── README.md
├── CHANGELOG.md
│
├── docs/
│   └── m1/
│       └── documento-final-m1.md
│
├── workflows/
│   └── m1/
│       ├── tintico-lab-m1-postventa.json
│       └── buscar-pedido.json
│
└── data/
    ├── pedidos-simulados.json
    └── politicas-postventa.csv
```

Las carpetas de nuevos milestones se agregan a medida que se desarrollan.

Por ejemplo:

```text
docs/m2/
workflows/m2/
```

---

## Documentación

La documentación técnica de cada milestone se mantiene separada del README principal.

M1 incluye documentación sobre:

- problema de negocio;
- alcance;
- arquitectura;
- responsabilidades de cada componente;
- contratos de entrada y salida;
- políticas;
- dataset simulado;
- decisiones técnicas;
- pruebas;
- limitaciones;
- preparación para la siguiente etapa.

---

## Datos y seguridad

Este repositorio utiliza datos ficticios y simulados.

No deben incluirse:

- API keys;
- tokens;
- passwords;
- secretos de webhooks;
- credenciales;
- datos personales reales de clientes.

Los workflows exportados de n8n deben revisarse antes de cada commit para evitar publicar información sensible escrita manualmente en nodos, headers o código.

---

## Estado actual

```text
M1 → completado
Pruebas oficiales → 9/9 PASS
M2 → próximo
```

---

## Objetivo del proyecto

Tintico Lab funciona como proyecto de aprendizaje y portfolio orientado a demostrar progresión desde workflows simples hacia automatizaciones profesionales con:

- integración de APIs;
- diseño modular;
- reglas de negocio;
- IA aplicada con control;
- manejo de errores;
- trazabilidad;
- buenas prácticas de automatización.
