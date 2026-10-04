# Documento final M1 — Tintico Lab

## Sistema inteligente de gestión de postventa para e-commerce

**Versión:** M1 final  
**Estado:** Entregable funcional  
**Fecha de referencia para pruebas:** 2026-09-18

---

## 1. Resumen ejecutivo

Tintico Lab M1 es la primera versión funcional de un sistema de gestión de postventa para un e-commerce especializado en café de especialidad, máquinas, molinos, métodos y accesorios.

El objetivo de M1 es demostrar una arquitectura híbrida en n8n que combine:

- interpretación con IA;
- consulta de datos operativos;
- políticas editables por negocio;
- decisiones determinísticas.

El sistema recibe un mensaje de cliente, lo clasifica, extrae los datos relevantes, decide si necesita consultar un pedido, recupera una política aplicable y produce una decisión estructurada.

M1 no intenta automatizar toda la operación de postventa. Su alcance es resolver correctamente la interpretación y decisión inicial de un caso. Las acciones posteriores —por ejemplo, realizar un reembolso, enviar un reemplazo, crear una orden de reparación o enviar una respuesta real por WhatsApp— quedan fuera de esta versión.

El resultado final de M1 es un contrato JSON estable que puede ser consumido posteriormente por otros workflows, un CRM, un canal de mensajería, un dashboard o una base de datos.

---

## 2. Problema de negocio

En un proceso manual de postventa, una persona debe:

1. leer el mensaje del cliente;
2. interpretar el problema;
3. localizar el pedido;
4. revisar datos del producto;
5. consultar políticas;
6. decidir qué corresponde;
7. solicitar información faltante;
8. registrar el caso;
9. escalarlo si requiere intervención humana.

Este proceso tiene varios puntos de fricción:

- depende de interpretación manual;
- puede aplicar políticas de forma inconsistente;
- obliga a consultar información en distintas fuentes;
- dificulta mantener trazabilidad;
- no produce necesariamente un resultado estructurado reutilizable;
- puede escalar innecesariamente casos simples o resolver mal excepciones.

M1 automatiza la etapa de interpretación y decisión inicial sin eliminar la intervención humana cuando corresponde.

---

## 3. Objetivo de M1

M1 debe ser capaz de:

- recibir un mensaje de postventa simulado;
- clasificarlo dentro de un conjunto cerrado de categorías;
- extraer datos reportados por el cliente;
- identificar si necesita consultar un pedido;
- utilizar una Tool para buscar el pedido cuando corresponda;
- detectar datos faltantes;
- consultar políticas activas en Google Sheets;
- seleccionar una política específica por tipo de producto o utilizar un fallback genérico;
- aplicar reglas de negocio mediante lógica determinística;
- calcular `decision`, `priority` y `requiresHuman`;
- producir una salida final consistente;
- validar el comportamiento mediante nueve casos oficiales de prueba.

---

## 4. Alcance y fuera de alcance

### Incluido en M1

- entrada simulada mediante Manual Trigger + Edit Fields;
- interpretación del lenguaje del cliente con AI Agent;
- categorías de postventa definidas;
- uso de una Tool real de Agent para consultar pedidos;
- dataset de pedidos simulado;
- políticas editables en Google Sheets;
- routing previo a políticas;
- selección de política por `productType`;
- motor determinístico de reglas;
- manejo de datos faltantes;
- manejo de pedido no encontrado;
- prioridad B2B para escalamiento de máquinas;
- contrato final de salida;
- pruebas oficiales 9/9.

### Fuera de alcance en M1

- WhatsApp, email o formularios reales;
- WooCommerce o Tiendanube reales;
- CRM real;
- persistencia de casos;
- memoria conversacional;
- RAG;
- procesamiento de documentos;
- ejecución real de reembolsos, reemplazos o reparaciones;
- envío automático de mensajes al cliente;
- Human-in-the-Loop productivo;
- monitoreo y observabilidad productiva;
- manejo completo de errores técnicos de APIs externas;
- múltiples productos relevantes dentro de un mismo pedido;
- autenticación y permisos de una tienda real.

---

## 5. Arquitectura final

```text
Manual Trigger
      ↓
Input de prueba
      ↓
Agente de postventa
  ├── OpenAI Chat Model
  ├── Tool buscar_pedido
  └── Structured Output Parser
      ↓
Evaluar estado inicial
      ↓
¿Necesita política?
   ├── false
   │      ↓
   │ Resolver sin política
   │      ↓
   │ Salida final M1
   │
   └── true
          ├────────────────────────→ Merge caso + política [Input 1]
          │
          └→ Buscar políticas activas
                    ↓
             Seleccionar política
                    ↓
          Merge caso + política [Input 2]
                    ↓
             Consolidar contexto
                    ↓
              Aplicar políticas
                    ↓
               Salida final M1
```

La arquitectura separa interpretación, obtención de hechos, configuración de negocio y toma de decisiones.

---

## 6. Principio de diseño

M1 aplica una regla central:

> **IA donde existe ambigüedad lingüística. Lógica determinística donde existen reglas claras.**

El AI Agent interpreta lenguaje natural y decide cuándo utilizar una Tool.

No decide:

- `priority`;
- `requiresHuman`;
- la decisión comercial final.

Las reglas de negocio se aplican mediante nodos determinísticos y parámetros almacenados en Google Sheets.

Esto reduce el riesgo de que un modelo genere decisiones plausibles pero no autorizadas por el negocio.

---

## 7. Categorías y decisiones

### Categorías admitidas

```text
PRODUCTO_DANADO
PRODUCTO_INCORRECTO
PEDIDO_DEMORADO
CAMBIO_DEVOLUCION
GARANTIA
FUERA_DE_ALCANCE
```

### Estados de decisión

```text
RESUELTO
FALTA_UN_DATO
ESCALAR
FUERA_DE_POLITICA
```

Categoría y decisión son conceptos distintos.

Ejemplo:

```text
category = CAMBIO_DEVOLUCION
decision = FUERA_DE_POLITICA
```

---

## 8. Modelo de entrada

M1 utiliza una entrada simulada.

Ejemplo:

```json
{
  "channel": "whatsapp",
  "customerIdentifier": "+54XXXXXXXXXX",
  "identifierType": "phone",
  "message": "El pedido TL-1003 todavía no llegó.",
  "receivedAt": "2026-09-18T10:30:00",
  "currentDate": "2026-09-18"
}
```

`Input de prueba` no interpreta el mensaje. Su función es simular datos que en una versión posterior podrían llegar desde WhatsApp, email, webhook o formulario.

Se utiliza una `currentDate` fija en las pruebas para que los resultados sean reproducibles.

---

## 9. AI Agent

El Agent recibe como User Prompt:

```text
{{ $json.message }}
```

### Responsabilidades

- interpretar el mensaje del cliente;
- clasificarlo;
- extraer `orderId` cuando esté presente;
- extraer producto mencionado;
- extraer atributos reportados;
- extraer condiciones declaradas por el cliente;
- decidir si corresponde utilizar `buscar_pedido`;
- respetar el contrato del Structured Output Parser.

### El Agent no debe

- inventar datos;
- inventar requisitos de negocio;
- pedir fotos o documentación no definida;
- copiar datos de la Tool dentro de `reportedAttributes`;
- calcular `decision`;
- calcular `priority`;
- calcular `requiresHuman`.

Para `PRODUCTO_INCORRECTO`, `reportedAttributes` describe lo que el cliente afirma haber recibido. Los atributos esperados permanecen dentro del pedido recuperado por la Tool.

---

## 10. Normalización de atributos de café

Se definió un vocabulario estable para evitar que expresiones diferentes representen de forma inconsistente el mismo concepto.

### format

```text
whole_beans
ground
```

### grind

```text
espresso
v60
french_press
null
```

Reglas:

- café en grano → `format = whole_beans` y `grind = null`;
- café molido → `format = ground`;
- “molido para espresso” → `format = ground`, `grind = espresso`.

Esta normalización permite que el motor determinístico compare valores estables en lugar de lenguaje libre.

---

## 11. Tool `buscar_pedido`

Subworkflow:

```text
Tintico Lab — Tool — buscar_pedido
```

Input:

```text
orderId: string
```

Contrato cuando encuentra el pedido:

```json
{
  "found": true,
  "order": {}
}
```

Contrato cuando no lo encuentra:

```json
{
  "found": false,
  "order": null
}
```

`found = false` no representa un error técnico. Es un resultado válido de negocio/datos.

El Agent tiene configurado `orderId` como parámetro definido por el modelo, de forma que puede extraer el identificador del mensaje y pasarlo a la Tool.

---

## 12. Dataset simulado

El dataset operativo de M1 vive dentro del subworkflow `buscar_pedido`.

Pedidos principales:

- **TL-1001**: B2C, café Huila 500 g, `whole_beans`, entrega 2026-09-14.
- **TL-1002**: B2C, máquina Tintico Home Mini, compra 2026-05-10.
- **TL-1003**: B2B, máquina Barista One, `in_transit`, entrega estimada 2026-09-17.
- **TL-1004**: B2C, café Nariño 500 g, entrega 2026-09-13.
- **TL-1005**: B2C, máquina Tintico Home Plus, entrega 2026-09-14.

TL-1005 fue agregado para probar devoluciones de máquinas dentro de la ventana permitida.

El dataset es deliberadamente simulado. En una versión posterior podrá reemplazarse por WooCommerce, Tiendanube u otra fuente operativa manteniendo el contrato lógico de `buscar_pedido`.

---

## 13. Políticas en Google Sheets

Las políticas se almacenan en:

```text
Tintico Lab — Políticas de Postventa
```

Campos principales:

```text
policyId
category
productType
referenceDateField
windowDays
delayGraceDays
warrantyMonths
requiresPackageClosed
requiresUnused
active
notes
```

Políticas principales:

- P001 — `PRODUCTO_DANADO` — all — 7 días desde `deliveryDate`.
- P002 — `PRODUCTO_INCORRECTO` — all — comparar atributos.
- P003 — `PEDIDO_DEMORADO` — all — evaluar `estimatedDeliveryDate`.
- P004 — `CAMBIO_DEVOLUCION` — coffee — 30 días + paquete cerrado.
- P005 — `CAMBIO_DEVOLUCION` — machine — 30 días + sin uso.
- P006 — `CAMBIO_DEVOLUCION` — accessory — 30 días + sin uso.
- P007 — `GARANTIA` — machine — 12 meses desde `orderDate`.
- P008 — `GARANTIA` — grinder — 12 meses desde `orderDate`.

Google Sheets contiene configuración de negocio. La semántica del sistema y las reglas de routing permanecen en n8n.

---

## 14. Selección de política

`Buscar políticas activas` recupera todas las filas activas de la categoría.

`Seleccionar política` aplica:

1. buscar política específica para el `productType` real del pedido;
2. si no existe, utilizar `productType = all`;
3. si no existe ninguna, generar un error de configuración.

Ejemplos:

```text
GARANTIA + machine
→ P007
```

```text
GARANTIA + grinder
→ P008
```

```text
PEDIDO_DEMORADO + machine
→ fallback all
→ P003
```

No se utiliza el orden de las filas de Google Sheets como mecanismo implícito de selección.

---

## 15. Routing previo a políticas

`Evaluar estado inicial` resuelve situaciones transversales antes de consultar Google Sheets.

### FUERA_DE_ALCANCE

```text
decision = RESUELTO
needsPolicy = false
```

### Categoría que requiere pedido pero no tiene orderId

```text
decision = FALTA_UN_DATO
needsPolicy = false
```

### orderId presente pero buscar_pedido devuelve found = false

```text
decision = ESCALAR
needsPolicy = false
```

### Caso con pedido válido

```text
needsPolicy = true
```

Este routing evita consultas innecesarias y accesos a estructuras nulas como `toolResult.order`.

---

## 16. Motor determinístico de decisiones

### PEDIDO_DEMORADO

Compara:

```text
currentDate
vs
estimatedDeliveryDate + delayGraceDays
```

Si `currentDate` supera la fecha límite:

```text
decision = ESCALAR
```

Si todavía está dentro:

```text
decision = RESUELTO
```

### PRODUCTO_DANADO

Compara:

```text
currentDate
vs
deliveryDate + windowDays
```

Dentro de 7 días:

```text
decision = RESUELTO
```

Fuera del plazo:

```text
decision = ESCALAR
```

### GARANTIA

Calcula:

```text
orderDate + warrantyMonths
```

Dentro del período:

```text
decision = ESCALAR
```

Fuera del período:

```text
decision = FUERA_DE_POLITICA
```

La garantía vigente se escala porque la revisión técnica, reparación o reemplazo continúa siendo humana en M1.

### CAMBIO_DEVOLUCION — coffee

Requiere:

- estar dentro de `windowDays`;
- paquete cerrado.

```text
packageOpened = true
→ FUERA_DE_POLITICA
```

```text
packageOpened = false
→ RESUELTO
```

```text
packageOpened desconocido
→ FALTA_UN_DATO
→ missingData incluye packageOpened
```

### CAMBIO_DEVOLUCION — machine/accessory

Requiere:

- estar dentro de `windowDays`;
- producto sin uso.

```text
productUsed = true
→ FUERA_DE_POLITICA
```

```text
productUsed = false
→ RESUELTO
```

```text
productUsed desconocido
→ FALTA_UN_DATO
→ missingData incluye productUsed
```

### PRODUCTO_INCORRECTO

Compara atributos reportados contra atributos del pedido.

Atributos iniciales comparables:

```text
origin
size
format
grind
```

Solo se comparan atributos que el cliente realmente reportó.

Los strings se normalizan con `trim()` y `toLowerCase()` antes de compararse.

Si se confirma alguna diferencia:

```text
decision = RESUELTO
```

Si no existe información suficiente para validar:

```text
decision = ESCALAR
```

---

## 17. requiresHuman y priority

`requiresHuman` se deriva exclusivamente de `decision`.

```text
decision = ESCALAR
→ requiresHuman = true
```

Cualquier otra decisión:

```text
requiresHuman = false
```

Regla de prioridad:

```text
decision = ESCALAR
AND customerType = B2B
AND productType = machine
→ priority = HIGH
```

En cualquier otro caso:

```text
priority = NORMAL
```

Si un pedido no existe, `priority` permanece `NORMAL` porque no existe evidencia verificable para aplicar la prioridad B2B/machine.

---

## 18. Contrato de salida final

M1 expone un contrato externo unificado:

```json
{
  "category": "...",
  "orderId": "...",
  "decision": "...",
  "priority": "...",
  "requiresHuman": true,
  "missingData": [],
  "reason": "..."
}
```

Ejemplo:

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

Los datos internos como `policyId`, `windowDays`, `warrantyMonths`, `customerType` u `orderItem` permanecen disponibles para debugging y auditoría, pero no forman parte del contrato externo.

---

## 19. Casos oficiales de prueba

| # | Mensaje | Resultado esperado | Estado |
|---|---|---|---|
| 1 | Pedido TL-1001. Pedí el Huila en grano pero me llegó molido para espresso. | PRODUCTO_INCORRECTO → RESUELTO | PASS |
| 2 | Mi cafetera llegó rota. | PRODUCTO_DANADO → FALTA_UN_DATO | PASS |
| 3 | El pedido TL-1003 todavía no llegó. | PEDIDO_DEMORADO → ESCALAR → HIGH | PASS |
| 4 | Quiero devolver un café que abrí hace cinco días. | CAMBIO_DEVOLUCION → FALTA_UN_DATO | PASS |
| 5 | Quiero devolver el café del pedido TL-1004. Abrí el paquete y probé una taza. | CAMBIO_DEVOLUCION → FUERA_DE_POLITICA | PASS |
| 6 | La máquina del pedido TL-1002 dejó de calentar. | GARANTIA → ESCALAR | PASS |
| 7 | Mi máquina dejó de calentar. | GARANTIA → FALTA_UN_DATO | PASS |
| 8 | ¿Venden café descafeinado? | FUERA_DE_ALCANCE → RESUELTO | PASS |
| 9 | El pedido TL-9999 no llegó. | PEDIDO_DEMORADO → ESCALAR | PASS |

**Resultado global: 9 / 9 casos oficiales aprobados.**

---

## 20. Decisiones técnicas y buenas prácticas

M1 incorpora las siguientes decisiones:

- IA para interpretar lenguaje, no para ejecutar reglas claras;
- Tools para obtener hechos externos;
- Structured Output para evitar respuestas libres;
- contratos estables entre componentes;
- políticas editables separadas de la lógica;
- routing antes de consultar políticas;
- fallback explícito de política específica a `all`;
- comparación determinística de fechas y atributos;
- normalización de lenguaje a valores estructurados;
- separación entre datos internos y contrato externo;
- fallo visible ante configuración de política inexistente;
- `currentDate` fija durante pruebas para reproducibilidad;
- subworkflow reutilizable para `buscar_pedido`.

---

## 21. Limitaciones actuales

M1 posee limitaciones conocidas y deliberadas:

- los pedidos son simulados;
- no existe una tienda real conectada;
- no existe canal real de entrada;
- no existe persistencia de casos;
- no existe memoria multi-turn;
- no ejecuta acciones comerciales reales;
- no solicita ni valida evidencia física;
- confía en condiciones declaradas por el cliente, como `packageOpened` o `productUsed`;
- el pedido se simplifica a un item relevante en la lógica actual;
- los valores numéricos vacíos de Google Sheets pueden normalizarse como `0`;
- no existe manejo productivo completo de timeout, API 500, rate limits o retries;
- no existe monitoreo ni alertas;
- el output final informa una decisión, pero no redacta ni envía la respuesta al cliente.

Estas limitaciones no impiden cumplir el objetivo de M1.

---

## 22. Riesgos y consideraciones profesionales

En una implementación real deberán considerarse:

- credenciales y permisos de mínimo privilegio;
- protección de datos personales;
- logs y trazabilidad;
- tratamiento diferenciado de errores técnicos y resultados de negocio;
- retries controlados;
- idempotencia;
- rate limits de servicios externos;
- múltiples items por pedido;
- validación de evidencia cuando el riesgo comercial lo requiera;
- manejo de estados del caso;
- separación desarrollo/producción;
- versionado de políticas;
- monitoreo de ejecuciones fallidas;
- procesos de escalamiento humano reales.

Una automatización de postventa puede generar consecuencias económicas. Por eso las acciones irreversibles o de mayor riesgo no deben ejecutarse sin controles adecuados.

---

## 23. Estado final de M1

```text
Estado: ENTREGABLE FUNCIONAL

Construcción: COMPLETADA
Pruebas oficiales: 9 / 9 PASS
Ajustes y regresión: COMPLETADOS
Contrato final: ESTABLE
Integraciones reales: PENDIENTES PARA MÓDULOS POSTERIORES
```

M1 demuestra correctamente la arquitectura central de Tintico Lab:

```text
interpretar
→ consultar
→ validar
→ aplicar política
→ decidir
→ estructurar resultado
```

---

## 24. Preparación para M2

M1 deja una base modular que permite evolucionar sin reconstruir el sistema desde cero.

M2 puede incorporar una arquitectura Manager + Workers, separando responsabilidades de postventa en componentes especializados:

- Logística;
- Cambios y Devoluciones;
- Garantías.

La evolución debe reutilizar los contratos, políticas y principios establecidos en M1.

El objetivo de M2 no será reemplazar lo construido, sino aumentar modularidad y especialización manteniendo una arquitectura controlable, mantenible y auditable.

---

## 25. Conclusión

Tintico Lab M1 demuestra que un proceso de postventa puede combinar IA y automatización determinística sin delegar reglas comerciales críticas a un modelo generativo.

```text
El Agent interpreta el lenguaje humano.
La Tool obtiene hechos verificables.
Google Sheets aporta configuración editable.
n8n aplica reglas determinísticas.
El sistema produce un contrato estable y escalable.
```

La versión M1 queda cerrada como una base funcional para continuar la evolución del proyecto hacia integraciones reales, mayor modularidad, memoria, RAG, observabilidad y automatización operativa.
