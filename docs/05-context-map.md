# 05 · Context map: Reservas y disponibilidad

> **Proyecto:** Nodo (coworking y eventos) · **Área:** Reservas y disponibilidad (core)
> **Alcance:** nuestra área al centro y sus relaciones con las áreas vecinas. Las líneas son **simples, sin tipo ni dirección**: cada una indica qué dato o evento cruza.

---

## Vista general del mapa

![Context map de Reservas y disponibilidad](img/05-context-map.png)

**Cómo leer el mapa**

| Forma en el mapa | Tipo | Qué significa para nosotros |
|---|---|---|
| Recuadro naranja claro con borde naranja oscuro | **Área core** | Reservas y disponibilidad, nuestra área. |
| Cajas blancas con borde naranja, dentro del área | **Propio** | Los dos bounded contexts del área: Disponibilidad y Reserva. Los modelamos nosotros (Sesiones 5 y 6). |
| Caja blanca con borde gris continuo | **Vecino (supporting)** | Otra área del negocio de Nodo, con su propio modelo: Espacios y recursos, Membresías y planes, Tarifas y precios. Hay que acordar qué cruza la frontera. |
| Caja gris con borde punteado | **Externo (generic)** | No diferencia a Nodo; se integra: Facturación y cobros, Control de acceso físico. |
| Franja gris con borde punteado, al pie del mapa | **Transversal (generic)** | Lo usan todas las áreas, no solo la nuestra: Notificaciones. |
| Línea simple con número ①–⑨ | **Relación** | Qué dato o evento cruza. Sin tipo ni dirección. |

Cada número ①–⑨ tiene su detalle en [Qué cruza cada relación](#3-qué-cruza-cada-relación). Los nombres son los de [01-subdominios.md](01-subdominios.md) y los términos, los de [02-glosario.md](02-glosario.md).

---

## 1. Propósito

Este mapa sirve para:

- **Ubicar nuestra frontera:** qué decide nuestra área y qué solo consulta o dispara hacia otras.
- **Listar qué cruza cada línea** (qué pedimos y qué entregamos).
- **Dejar a la vista dónde habrá que protegerse o negociar**: los hotspots de integración, marcados como **HI-xx**.

---

## 2. Nuestra área por dentro: dos contextos

La justificación de la frontera está en [decisiones/0001-por-que-esta-frontera.md](../decisiones/0001-por-que-esta-frontera.md). Aquí se resume lo que importa para el mapa:

| Contexto | Qué decide | Pregunta que responde | Lenguaje propio |
|---|---|---|---|
| **Disponibilidad** | Si una franja de una sala de reunión se puede ocupar | «¿Está libre la Sala B el jueves de 10:00 a 11:00?» | sala de reunión, recurso reservable, franja, margen, bloqueo, disponibilidad, franja liberada, salas disponibles |
| **Reserva** | Si una solicitud de una persona se convierte en reserva y qué pasa con ella después | «¿Puede Ana reservarla, y qué pasa si cancela?» | solicitud de reserva, reserva, confirmada, anticipación, reserva asistida, miembro al día, visitante, invitado, aforo, cancelación, penalización, horas extra, lista de espera |

**Por qué la línea ① es la más importante del mapa:** la regla «nunca dos reservas en la misma franja» (R-01) vive en **Disponibilidad**, pero la reserva nace en **Reserva**. Que las dos se mantengan siempre de acuerdo, incluso cuando dos personas piden la última franja a la vez, es el principal desafío de integración del área (HI-01).

---

## 3. Qué cruza cada relación

Para cada vecino: **qué le pedimos**, **qué le entregamos**, el **dato o evento** concreto que viaja y los **hotspots de integración**. Los nombres de eventos y los códigos R/P/H son los de [03-eventstorming.md](03-eventstorming.md).

### ① Disponibilidad ↔ Reserva (interna, propia)

| | |
|---|---|
| **Reserva le pide a Disponibilidad** | «¿Está libre esta franja de esta sala, respetando el margen de 15 min y los bloqueos?» y, si lo está, **ocuparla** para la reserva. |
| **Disponibilidad le entrega a Reserva** | La respuesta (libre / ocupada), **Franja liberada** para que Reserva avise a la lista de espera, y **Sala bloqueada** cuando la franja bloqueada tenía una reserva confirmada. |
| **Datos que cruzan** | sala, fecha, hora de inicio, hora de fin, identificador de la reserva que la ocupa. |
| **Eventos que cruzan** | Reserva confirmada → ocupa la franja · Reserva cancelada → {P-07} → **Franja liberada** · Solicitud expirada (visitante sin pago, P-11) → **Franja liberada** · **Sala bloqueada** → {P-09} → Resolver reserva afectada → **Reserva reasignada** / **Reserva reprogramada** / **Reembolso solicitado**. |
| **Hotspots resueltos** | **HI-01:** si dos personas piden la última franja a la vez, la agenda de la sala garantiza aislamiento transaccional: se confirma una sola solicitud y la otra se rechaza o entra en la lista de espera (H-03). |

### ② Espacios y recursos ↔ Disponibilidad

| | |
|---|---|
| **Le pedimos** | El catálogo de **salas de reunión** reservables. Disponibilidad no crea salas: las recibe. |
| **Le entregamos** | Nada. |
| **Datos que cruzan** | identificador de la sala y tipo de recurso. Los escritorios (flexibles y fijos) y el salón están en el catálogo, pero no se reservan en nuestra área. |
| **Hotspots resueltos** | **HI-04:** si Espacios y recursos da de baja una sala con reservas futuras, se trata igual que un bloqueo: se reasigna a una sala equivalente, se reprograma o se reembolsa. |

### ③ Espacios y recursos ↔ Reserva

| | |
|---|---|
| **Le pedimos** | El **aforo** de cada sala de reunión, para verificar que los asistentes de una reserva no lo superen (R-04). |
| **Le entregamos** | Nada. |
| **Datos que cruzan** | identificador de la sala y aforo. |
| **Hotspots resueltos** | Ninguno. |

### ④ Tarifas y precios ↔ Reserva

| | |
|---|---|
| **Le pedimos** | El **precio vigente de la franja** al confirmar la reserva: la reserva lo conserva aunque la dirección cambie la tarifa después. También la **cotización de una reserva mensual** cuando el cliente la acepta. |
| **Le entregamos** | Nada. |
| **Datos que cruzan** | sala, franja, precio vigente; para la reserva mensual: cliente, sala, días y horario, vigencia de la cotización. |
| **Reglas que cruzan** | La cotización vence a los 4 días si no se confirma con pago o depósito, y **no aparta la franja** mientras está vigente. |
| **Hotspots resueltos** | **HI-09:** cada fecha de la reserva mensual cumple las mismas reglas que una reserva normal (margen de 15 min, aforo y anticipación). · **HI-10:** si una fecha de la serie ya está ocupada, se confirman las fechas libres y recepción coordina otra sala u horario para la ocupada. |

### ⑤ Membresías y planes ↔ Reserva

| | |
|---|---|
| **Le pedimos** | Si el miembro **está al día** al momento de confirmar (R-02) y cuántas **horas incluidas** tiene su plan (P-04). |
| **Le entregamos** | Las horas **consumidas**: cada **Reserva confirmada** y cada **Reserva cancelada** cambia lo que el miembro lleva usado. |
| **Datos que cruzan** | identificador del miembro, estado (activo, al día), plan, horas incluidas, horas usadas. |
| **Hotspots resueltos** | **HI-12:** la cuenta de horas usadas la lleva Membresías y planes; Reserva le informa cada reserva confirmada o cancelada. · **HI-13:** si Membresías y planes no responde, la solicitud pasa a recepción, que verifica el estado y la registra como reserva asistida. |

### ⑥ Reserva ↔ Facturación y cobros

| | |
|---|---|
| **Le pedimos** | Que registre el **pago por adelantado** del visitante: ese hecho dispara la confirmación de su reserva (P-10). |
| **Le entregamos** | **Penalización aplicada** (cobro de la franja completa, P-05), **Horas extra registradas** (sobre las incluidas o por exceso de tiempo, P-04 y R-06) y **Reembolso solicitado** (reserva afectada por un bloqueo, P-09). |
| **Datos que cruzan** | persona (miembro o visitante), reserva, franja, horas extra, motivo del cobro o del reembolso. |
| **Hotspots resueltos** | **HI-14:** el monto de la franja completa lo calcula Facturación y cobros con el precio conservado que envía Reserva. |

### ⑦ y ⑧ Reserva ↔ Notificaciones

| | |
|---|---|
| **Le pedimos** | Que envíe el aviso por WhatsApp o correo. Nosotros no redactamos ni enviamos mensajes. |
| **Le entregamos** | ⑦ **Reserva confirmada** (→ Confirmación enviada), la fecha para el **recordatorio del día antes** (P-03) y **Solicitud rechazada** (P-08). ⑧ Cuando Disponibilidad avisa **Franja liberada**, Reserva entrega la lista de espera de esa franja: se avisa a **todos** (P-06). Disponibilidad no habla con Notificaciones: no sabe quién espera. |
| **Datos que cruzan** | destinatario, tipo de aviso, sala, fecha y hora. |
| **Hotspots resueltos** | **HI-17:** el recordatorio lo programa Notificaciones con la fecha que envía Reserva al confirmar. · **HI-18:** si la reserva se cancela, Notificaciones escucha *Reserva cancelada* y anula el recordatorio pendiente. |

### ⑨ Control de acceso físico ↔ Reserva

| | |
|---|---|
| **Le pedimos** | Quién actúa y con qué **rol** (miembro, visitante, recepción, administración, dirección u operaciones), para saber si puede registrar una reserva asistida, bloquear una sala o resolver una reserva afectada. |
| **Le entregamos** | Las **reservas del día**: para Control de acceso físico, una reserva es una autorización temporal de entrada. El visitante entra con apertura manual o un código temporal que genera recepción. El día de la reserva, Control de acceso físico valida si el miembro está al día; si no lo está, la puerta no abre y recepción revisa el caso. |
| **Datos que cruzan** | identificador de la persona, rol, a nombre de quién actúa, sala y horario. |
| **Hotspots resueltos** | Ninguno. |

---

## 4. Resumen: qué pedimos y qué entregamos

| # | Vecino | Tipo | Contexto nuestro que toca | Qué le pedimos | Qué le entregamos |
|---|---|---|---|---|---|
| ① | Disponibilidad ↔ Reserva | Propio | ambos | ocupar / liberar una franja | Franja liberada, Sala bloqueada |
| ② | Espacios y recursos | Supporting | Disponibilidad | salas de reunión reservables | — |
| ③ | Espacios y recursos | Supporting | Reserva | aforo de cada sala | — |
| ④ | Tarifas y precios | Supporting | Reserva | precio vigente de la franja, cotización de reserva mensual | — |
| ⑤ | Membresías y planes | Supporting | Reserva | miembro al día, horas incluidas | horas consumidas |
| ⑥ | Facturación y cobros | Generic | Reserva | pago por adelantado del visitante | penalización, horas extra, reembolso |
| ⑦ | Notificaciones | Transversal | Reserva | enviar el aviso | confirmación, recordatorio, rechazo |
| ⑧ | Notificaciones | Transversal | Reserva (tras Franja liberada) | enviar el aviso | lista de espera de la franja |
| ⑨ | Control de acceso físico | Generic | Reserva | quién actúa y su rol | reservas del día |

**Por qué estas clasificaciones** (ver [01-subdominios.md](01-subdominios.md)):

- **Espacios y recursos, Tarifas y precios y Membresías y planes** son **supporting**: tienen reglas propias de Nodo (catálogo, precios, planes), pero no son la razón por la que eligen a Nodo.
- **Facturación y cobros** y **Control de acceso físico** son **generic**: se integran con un software de facturación, una pasarela de pago y la solución de la cerradura electrónica.
- **Notificaciones** es **generic y transversal**: casi todas las áreas la disparan, y el envío por correo o mensajería se integra.

---

## 5. Lo que no tiene línea, y por qué

| Relación descartada | Por qué no se dibuja |
|---|---|
| Nuestra área ↔ Agenda de eventos | El salón es un recurso aparte, se alquila por horas o jornadas y pertenece a Agenda de eventos. Nuestra área solo reserva salas de reunión. |
| Disponibilidad ↔ Notificaciones | Lo que se avisa es a **quienes esperaban**, y la lista de espera es de Reserva. |
| Disponibilidad ↔ Membresías y planes | Si una franja está libre no depende de quién la pida. El estado del miembro solo le importa a Reserva. |
| Disponibilidad ↔ Facturación y cobros | Una franja no se cobra; se cobra lo que hace una persona con una reserva. |
| Reserva ↔ WhatsApp directo | Enviar mensajes no es nuestra responsabilidad. Si cada área escribiera por WhatsApp por su cuenta, se repetiría el problema de hoy (avisos a mano y olvidados). |

---

## 6. Polisemias en las fronteras

Una misma palabra con dos sentidos señala la frontera entre dos contextos:

| Palabra | En nuestra área | En el vecino | Frontera |
|---|---|---|---|
| **Reserva** | Una franja de una sala de reunión asignada a una persona | Agenda de eventos: alquiler del salón. Facturación y cobros: cargo por cobrar. Control de acceso físico: autorización temporal de entrada | ⑥, ⑨ |
| **Miembro** | Persona con membresía que reserva (también puede reservar un visitante) | Membresías y planes: titular de un plan | ⑤ |
| **Penalización** | Hecho: «canceló con menos de 4 h» | Facturación y cobros: cargo de la franja completa | ⑥ |
| **Horas** | Duración de la franja reservada | Membresías y planes: horas incluidas en el plan. Facturación y cobros: horas extra facturables | ⑤, ⑥ |
| **Aforo** | Límite de asistentes de una reserva | Tarifas y precios: variable que influye en el precio | ③, ④ |
| **Precio** | Lo que conserva una reserva confirmada | Tarifas y precios: *tarifa* es la lista de precios; *precio*, lo que paga esa persona | ④ |
| **Evento** | (En DDD) algo que ocurrió: «Reserva confirmada» | Agenda de eventos: un acto en el salón de Nodo | — |
| **Cancelación** | De una reserva | Membresías y planes: baja de un plan al final del ciclo | ⑤ |

---

## 7. Hotspots de integración resueltos

| ID | Pregunta | Respuesta | Relación |
|---|---|---|---|
| HI-01 | ¿Qué garantiza que la reserva y la ocupación de la franja nunca queden en desacuerdo, incluso si dos personas piden la última franja a la vez? | La agenda de la sala garantiza aislamiento transaccional: se confirma una sola solicitud y la otra se rechaza o entra en la lista de espera. | ① |
| HI-02 | ¿Un bloqueo puede pisar una reserva ya hecha? | Sí, pero la reserva no se elimina sin solución: se reasigna, se reprograma o se reembolsa (P-09). | ① |
| HI-03 | ¿Quién bloquea una sala? | Recepción, administración o dirección, por mantenimiento o limpieza (R-07). | ① |
| HI-04 | ¿Qué pasa con las reservas futuras de una sala dada de baja? | Igual que con un bloqueo: se reasignan, se reprograman o se reembolsan. | ② |
| HI-05 | ¿Los escritorios se reservan por franja? | No: el flexible se usa según el plan y el fijo se asigna. Disponibilidad solo recibe salas de reunión. | ② |
| HI-06 / HI-07 | ¿Eventos y salas comparten espacio? | El salón es un recurso aparte de Agenda de eventos; nuestra área solo reserva salas de reunión. La línea se quitó. | — |
| HI-08 | ¿La cotización aparta la franja? | No. Vence a los 4 días si no se confirma. | ④ |
| HI-09 | ¿La reserva mensual cumple las mismas reglas? | Sí: margen de 15 min, aforo y anticipación en cada fecha. | ④ |
| HI-10 | ¿Qué pasa si una fecha de la serie ya está ocupada? | Se confirman las libres y recepción coordina otra sala u horario para la ocupada. | ④ |
| HI-11 | ¿Qué estado de membresía vale y cuándo? | El estado al momento de confirmar (R-02). | ⑤ |
| HI-12 | ¿Quién lleva la cuenta de horas usadas? | Membresías y planes; Reserva le informa cada reserva confirmada o cancelada. | ⑤ |
| HI-13 | ¿Qué pasa si Membresías y planes no responde? | La solicitud pasa a recepción, que verifica el estado y la registra como reserva asistida. | ⑤ |
| HI-14 | ¿Quién calcula el monto de la franja completa? | Facturación y cobros, con el precio conservado que envía Reserva. | ④, ⑥ |
| HI-15 | Si se reserva a nombre de un invitado, ¿quién responde? | El miembro. | ⑥ |
| HI-16 | ¿Se avisa a toda la lista de espera o por orden? | A todos (P-06). | ⑧ |
| HI-17 | ¿Quién programa el recordatorio del día antes? | Notificaciones, con la fecha que envía Reserva al confirmar. | ⑦ |
| HI-18 | ¿Cómo se anula el recordatorio si la reserva se cancela? | Notificaciones escucha *Reserva cancelada* y lo anula. | ⑦ |
| HI-19 | ¿Cómo confirma el visitante? | Pagando por adelantado (R-05, P-10). | ⑥ |
| HI-20 | ¿La reserva da acceso al visitante? | Entra con apertura manual o un código temporal de recepción. | ⑨ |

**Dónde protegerse:** en ⑤ (Membresías y planes), ②–③ (Espacios y recursos) y ④ (Tarifas y precios). Sus modelos cambian por motivos propios (planes nuevos, salas nuevas, tarifas por temporada) y no queremos que eso nos obligue a cambiar nuestras reglas.

**Qué se acordó con los vecinos:** Facturación y cobros calcula los montos (⑥); la reserva mensual sigue las mismas reglas y recepción coordina las fechas ocupadas (④); Membresías y planes lleva las horas usadas (⑤); Notificaciones programa y anula los recordatorios (⑦/⑧).
