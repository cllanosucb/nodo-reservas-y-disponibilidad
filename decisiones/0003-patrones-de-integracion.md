# 0003 · Patrones de integración con los vecinos

- **Área:** Reservas y disponibilidad (core)

## Contexto
El [context map](../docs/05-context-map.md) dejó claro **qué cruza** cada una de las líneas ①–⑩, pero no **cómo** se relacionan los modelos, quién define el contrato ni con qué mecanismo se integran. Hay que decidirlo antes de modelar los aggregates y los servicios, porque de eso depende qué traduce nuestro código y qué usa tal cual.

Para cada línea se respondieron dos preguntas: **¿cooperan los equipos?** y **¿quién controla el modelo?**

## Decisión

| # | Relación | Patrón | Dirección (U → D) | Mecanismo |
|---|---|---|---|---|
| ① | Disponibilidad ↔ Reserva | Shared kernel (Franja e identificador de sala) | — | Llamada en proceso · eventos de dominio en proceso |
| ② | Espacios y recursos → Disponibilidad | Customer/Supplier + ACL | Espacios y recursos → Disponibilidad | Mensajería async + ACL |
| ③ | Espacios y recursos → Reserva | Customer/Supplier + ACL | Espacios y recursos → Reserva | API REST + ACL |
| ④ | Tarifas y precios → Reserva | Customer/Supplier + ACL | Tarifas y precios → Reserva | API REST + ACL |
| ⑤ | Membresías y planes → Reserva | Customer/Supplier + ACL | Membresías y planes → Reserva | API REST + ACL |
| ⑥ | Facturación y cobros → Reserva | ACL | Facturación y cobros → Reserva | API externa + ACL |
| ⑦ ⑧ | Notificaciones → Reserva | OHS + Conformist | Notificaciones → Reserva | Cola de mensajes (async) |
| ⑨ | Control de acceso físico → Reserva | OHS + ACL | Control de acceso físico → Reserva | Token con rol + ACL · API externa + ACL |
| ⑩ | Disponibilidad → Agenda de eventos (equipo portátil) | Customer/Supplier | Disponibilidad → Agenda de eventos | API REST |
| — | Reservas y disponibilidad ↔ Agenda de eventos (espacios) | Separate Ways | — | — |

El detalle de cada capa y de cada ACL está en [docs/07-integracion.md](../docs/07-integracion.md).

## Justificación
- **① Shared kernel acotado.** Disponibilidad y Reserva son del mismo equipo y de la misma área core, y las dos usan la franja y la sala exactamente igual. Compartir solo esas dos piezas evita traducir algo que no cambia de significado, y el resto de cada modelo queda separado.
- **① Llamada en proceso, un aggregate por transacción.** Las dos partes viven en el mismo monolito modular. Reserva confirma solo si Disponibilidad ocupó la franja, y la agenda de la sala garantiza aislamiento transaccional (HI-01). *Franja liberada* y *Sala bloqueada* viajan como eventos de dominio en proceso.
- **②–⑤ Customer/Supplier + ACL.** Son vecinos de Nodo con los que se puede negociar, pero sus modelos cambian por motivos propios: salas nuevas, planes nuevos, tarifas por temporada. El ACL evita que palabras como *escritorio fijo*, *plan_status* o *rate_id* entren a nuestras reglas.
- **⑥ y ⑨ ACL frente a un externo.** Facturación y cobros y Control de acceso físico son soluciones de terceros: mandan ellos. Si cambia el proveedor, solo se reemplaza el adaptador.
- **⑨ OHS + ACL.** Control de acceso físico publica un token igual para todas las áreas (OHS); nuestro ACL lo convierte en quién actúa y con qué rol, y convierte cada reserva confirmada en un permiso de ingreso.
- **⑦⑧ OHS + Conformist.** Notificaciones ofrece un contrato genérico (destinatario, plantilla, fecha) para todas las áreas, que no choca con nuestro lenguaje. Lo usamos tal cual por cola de mensajes: si Notificaciones no está disponible, la reserva igual se confirma.
- **Separate Ways con Agenda de eventos por los espacios.** El salón es un recurso aparte y «confirmar» significa otra cosa (depósito y aprobación de la dirección u operaciones).
- **⑩ Customer/Supplier por el equipo portátil.** El proyector y los televisores se usan en las salas y en el salón; una sola agenda evita prometer la misma unidad dos veces (R-08). Esa agenda es de Disponibilidad porque decidir si un recurso está libre en una franja es nuestro core. Agenda de eventos pide el equipo por API REST: necesita la respuesta en el momento para confirmar su evento.

## Alternativas descartadas
- **Partnership en ①.** Supone dos equipos que coordinan cada cambio; aquí es un solo equipo y lo que se comparte es un modelo pequeño.
- **Misma transacción para ocupar la franja y confirmar la reserva.** Modificaría dos aggregates a la vez. Se prefiere un aggregate por transacción, con la agenda de la sala como garante de que no haya solapes.
- **OHS de Disponibilidad.** Fuera del área solo la consume Agenda de eventos, y solo por el equipo (⑩); con un único consumidor basta un Customer/Supplier con una API acordada.
- **Separate Ways también para el equipo.** Cada área llevaría su propia lista del equipo y el mismo proyector podría prometerse dos veces (sesión 5).
- **Que Agenda de eventos sea dueña del equipo.** Las salas lo piden mucho más seguido que el salón, y la regla «sin solapes» ya vive en la agenda de Disponibilidad.
- **Customer/Supplier sin ACL en ④.** Las tarifas cambian por su cuenta y la reserva debe conservar su precio; sin ACL, Reserva tendría que entender la lista de precios.
- **Notificaciones escuchando nuestros eventos.** Obligaría a Notificaciones a conocer el lenguaje de cada área. Se prefiere que Reserva use su contrato y le pida programar y anular el recordatorio.
- **Rechazar la solicitud si Membresías y planes no responde.** Se pasa a recepción como reserva asistida (HI-13).

## Consecuencias
- Cada contexto tendrá en `src/` una carpeta de integración con un adaptador, un traductor y una fachada por vecino; el dominio solo ve la fachada.
- La franja y el identificador de sala se definen una sola vez y los usan los dos contextos.
- Las pruebas del prototipo pueden reemplazar cada fachada por una versión en memoria.
- En [docs/05-context-map.md](../docs/05-context-map.md), HI-18 queda así: Reserva pide anular el recordatorio y Notificaciones lo anula.

## Hotspots resueltos

| ID | Pregunta | Respuesta |
|---|---|---|
| HI-13 | ¿Qué pasa si Membresías y planes no responde? | La solicitud pasa a recepción, que verifica el estado y la registra como reserva asistida. |
| HI-14 | ¿Quién calcula el monto que se cobra? | Facturación y cobros, con el precio conservado que envía Reserva. |
| HI-17 / HI-18 | ¿Quién programa y anula el recordatorio? | Notificaciones, a pedido de Reserva por cola de mensajes. |
| HI-20 | ¿Cómo entra el visitante? | Con apertura manual o un código temporal que genera recepción. |
| HI-21 | ¿Quién lleva la agenda del equipo portátil compartido con el salón? | Disponibilidad; Agenda de eventos lo ocupa por ⑩. |
