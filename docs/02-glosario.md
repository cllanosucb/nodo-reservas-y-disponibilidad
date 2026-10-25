# 02 · Glosario del lenguaje ubicuo: Reservas y disponibilidad

> **Proyecto:** Nodo (coworking y eventos) · **Área:** Reservas y disponibilidad (core)
> **Rebanada:** Reservar una sala. Significados acordados con el experto, separados por bounded context.

---

## 1. Contexto: Disponibilidad

| Término | Significado acordado | Qué NO es | Regla / invariante | Evento |
|---|---|---|---|---|
| **Sala de reunión** | Espacio cerrado que se reserva por franja. Hay salas de distintos tamaños, cada una con su aforo. | No es un escritorio ni el salón de eventos. | Una sola reserva por franja, con 15 min de margen entre reservas. | — |
| **Recurso reservable** | Espacio que se reserva por franja, con hora de inicio y de fin. En nuestra área, la sala de reunión. | No incluye los escritorios (flexibles o fijos) ni el salón (pertenece a Agenda de eventos). | Una sola reserva por franja y recurso. | Reserva confirmada |
| **Escritorio flexible** | Escritorio de la sala diáfana que se usa según el plan de membresía: el miembro se sienta donde haya lugar libre. | No es un recurso reservable: no tiene agenda ni se reserva por franja. No es un escritorio fijo. | Su uso lo define el plan (Membresías y planes). No tiene margen: se limpia una vez al día. | — |
| **Franja** | Intervalo horario reservable en un recurso reservable. | No es el día completo. | Dos franjas reservadas del mismo recurso no se pisan. | Reserva confirmada / Franja liberada |
| **Margen** | 15 min obligatorios después de cada reserva de una sala de reunión, para limpiar y ventilar. También se le dice *buffer*; usamos *margen*. | No es tiempo reservable. | Se aplica siempre, de forma automática: entre dos reservas de la misma sala de reunión y entre una reserva y un bloqueo. | — |
| **Bloqueo** | Franja que Nodo saca de disponibilidad por mantenimiento o limpieza. | No es una reserva: no tiene titular ni cobro. No es una cancelación. | Solo recepción, administración o dirección pueden bloquear. Si la franja ya tiene una reserva confirmada, la reserva no se elimina sin solución: recepción u operaciones reasigna al titular a una sala equivalente o coordina la reprogramación o el reembolso, según lo que prefiera el titular. | Sala bloqueada / Reserva reasignada / Reserva reprogramada / Reembolso solicitado |
| **Disponibilidad** | Estado de un recurso reservable en una franja: libre, sin reserva, sin bloqueo y fuera del margen. | No equivale a que el recurso exista: una franja reservada, bloqueada o dentro del margen no está disponible. | Solo se reserva una franja disponible. | — (modelo de lectura «Salas disponibles») |

---

## 2. Contexto: Reserva

| Término | Significado acordado | Qué NO es | Regla / invariante | Evento |
|---|---|---|---|---|
| **Solicitud de reserva** | Pedido de un miembro o de un visitante para una franja de un recurso reservable, aún sin validar. | No es una reserva: todavía no ocupa la franja. | Se valida contra disponibilidad, anticipación, aforo y la condición de confirmación de quien la pide. Si no cumple, se rechaza. | Reserva solicitada / Solicitud rechazada |
| **Reserva** | Franja de un recurso reservable asignada a una persona (miembro o visitante). | No es un bloqueo: la reserva tiene titular; el bloqueo es de Nodo. | Requiere una franja disponible y no superar el aforo. | Reserva confirmada |
| **Confirmada** | Estado de una reserva que cumple las reglas. Miembro: hay disponibilidad y está al día; no paga por adelantado. Visitante: hay disponibilidad y pagó por adelantado. | No es la confirmación de un evento, que exige depósito y aprobación de la dirección u operaciones. | Automática solo con 24 h o más de anticipación. | Reserva confirmada |
| **Anticipación** | Tiempo entre el momento de reservar y el inicio de la franja. | No es la duración de la reserva. | 24 h o más: confirmación automática. Entre 3 y 24 h: solo como reserva asistida. Menos de 3 h: no se puede reservar. No hay anticipación máxima. | — |
| **Reserva asistida** | Reserva que registra recepción: cuando la anticipación está entre 3 y 24 h, o cuando un miembro reserva a nombre de un invitado. | No es una reserva automática: la registra y confirma recepción. | Nunca con menos de 3 h de anticipación. | Reserva confirmada |
| **Miembro al día** | Miembro sin facturas vencidas. | No equivale a *activo*: un miembro con membresía vigente puede deber la última factura. | Solo un miembro al día confirma reservas sin pago previo. Vale el estado al momento de confirmar: si el plan vence después, la reserva se mantiene. | — |
| **Visitante** | Persona sin membresía que reserva una sala de reunión por su cuenta. | No es un miembro: no tiene plan ni horas incluidas. No es un invitado. | Paga por adelantado dentro de un plazo límite; mientras tanto, la franja queda apartada de forma provisional. Si no paga a tiempo, la solicitud expira y la franja se libera. | Pago por adelantado registrado / Solicitud expirada |
| **Invitado** | Tercero a nombre del cual un miembro reserva un espacio. | No es un visitante: no reserva ni paga por su cuenta. | Siempre se registra por recepción (reserva asistida). La reserva queda bajo la responsabilidad del miembro e indica el nombre y los datos del invitado. | Reserva confirmada |
| **Aforo** | Cantidad máxima de personas que admite un recurso reservable. El dato lo entrega Espacios y recursos. | No es la cantidad de personas que asisten a una reserva. | La cantidad de asistentes de una reserva nunca supera el aforo del recurso: no hay sobrecupo. | — |
| **Cancelación** | Anulación de una reserva, hecha por su titular o por recepción. | No es un bloqueo: el bloqueo actúa sobre una franja, no sobre una reserva. | Sin costo si se cancela con 4 h o más de anticipación; la franja se libera. No se cancela una reserva ya empezada. | Reserva cancelada / Franja liberada |
| **Cancelación tardía** | Cancelación hecha con menos de 4 h de anticipación. | No es una cancelación sin costo. | Genera una penalización. | Penalización aplicada |
| **Penalización** | Cobro de la franja completa por una cancelación tardía o por inasistencia. | No es una hora extra ni un depósito. | Se cobra al titular, aunque cancele recepción. La calcula Facturación y cobros. | Penalización aplicada |
| **Horas extra** | Horas de sala que exceden las incluidas en el plan del miembro, o el tiempo que una persona se queda en la sala después de su franja. | No son una penalización. | Se cobran a fin de mes. Si quien se excede tiene una reserva confirmada después, debe desalojar la sala; si la franja siguiente está libre, se registran como horas extra. | Horas extra registradas |
| **Lista de espera** | Personas interesadas en una franja ocupada. | No es una reserva: no garantiza la sala. | Una solicitud sobre una franja ocupada entra automáticamente en la lista. Cuando la franja se libera, se avisa a todas las personas de la lista por WhatsApp o correo. No hay asignación automática por orden de llegada: la franja la toma quien la reserve. | Añadido a la lista de espera / Lista de espera notificada |

---

## 3. Polisemias

Una misma palabra cambia de significado al cruzar la frontera de nuestra área.

| Palabra | En Reservas y disponibilidad | En otro contexto |
|---|---|---|
| **Reserva** | Franja de un recurso reservable asignada a una persona. | **Agenda de eventos:** alquiler del salón para un evento. **Facturación y cobros:** cargo por cobrar. **Control de acceso físico:** autorización temporal de entrada en un horario. |
| **Confirmar** | Hay franja disponible y quien reserva cumple su condición (miembro al día o visitante con pago adelantado). | **Agenda de eventos:** depósito pagado y aprobación de la dirección u operaciones. **Facturación y cobros:** registrar el pago o descontar las horas del plan. |
| **Cliente** | Ocupante: la persona que usa el espacio durante la franja. | **Facturación y cobros:** titular de la cuenta y pagador de la factura. **Control de acceso físico:** persona identificada en la cerradura (PIN, QR o huella). |
| **Aforo** | Límite de personas de un recurso reservable; evita el sobrecupo. | **Tarifas y precios:** variable que influye en el precio. **Control de acceso físico:** máximo de personas con permiso de ingreso simultáneo. |
| **Margen** | 15 min después de cada reserva de una sala de reunión. | **Agenda de eventos:** 1 h de reacondicionamiento del salón. |
| **Al día vs. activo** | *Al día:* sin facturas vencidas. Es lo que exige la reserva. | **Membresías y planes:** *activo* es tener la membresía vigente. A los 5 días de impago se suspende el acceso físico. |
| **Bloquear** | Nodo saca una franja de disponibilidad por mantenimiento o limpieza. | No es cancelar una reserva. |
| **Escritorio** | Ningún escritorio es un recurso reservable. | **Membresías y planes:** el *flexible* se usa según el plan (te sientas donde haya libre); el *fijo* se asigna a un miembro (escritorio dedicado). |
| **Visitante vs. invitado** | *Visitante:* reserva y paga por su cuenta. *Invitado:* un miembro reserva a su nombre por recepción. | **Control de acceso físico:** ambos entran con apertura manual o un código temporal de recepción. |

---

## 4. Pruebas «¿qué pasa si…?»

| ¿Qué pasa si…? | Respuesta | Términos |
|---|---|---|
| Un miembro al día pide una sala con 2 días de anticipación | Se confirma automáticamente. | Anticipación, Confirmada |
| Pide la sala con 10 h de anticipación | Solo puede reservarla por recepción. | Anticipación, Reserva asistida |
| Pide la sala con 2 h de anticipación | No se puede reservar. | Anticipación |
| Pide la sala desde las 11:10 y hay otra reserva que termina a las 11:00 | No se puede: falta el margen de 15 min. Desde las 11:15, sí. | Margen, Disponibilidad |
| Dos personas piden la última franja al mismo tiempo | Se confirma una sola. La otra se rechaza o entra en la lista de espera. | Franja, Lista de espera |
| Un visitante pide una sala | La reserva se confirma cuando paga por adelantado. | Visitante, Confirmada |
| Un miembro quiere reservar para un cliente suyo | Lo pide a recepción, que registra la reserva a nombre del invitado; la responsabilidad es del miembro. | Invitado, Reserva asistida |
| Una reserva indica más asistentes que el aforo de la sala | No se acepta: no hay sobrecupo. | Aforo |
| Cancela 3 h antes del inicio | Se cobra la franja completa y se avisa a la lista de espera. | Cancelación tardía, Penalización, Lista de espera |
| Cancela 4 h antes del inicio | Sin costo; la franja se libera. | Cancelación |
| Se queda 30 min más y la franja siguiente está reservada | Debe desalojar la sala. | Horas extra |
| Se queda 30 min más y la franja siguiente está libre | Se registran horas extra y se cobran a fin de mes. | Horas extra |
| Recepción bloquea una sala por limpieza | La franja deja de estar disponible; no tiene titular ni cobro. | Bloqueo, Disponibilidad |
| Se bloquea una sala que tiene una reserva confirmada en esa franja | La reserva no se elimina sin solución: recepción u operaciones reasigna al titular a una sala equivalente o coordina la reprogramación o el reembolso. | Bloqueo, Reserva |
| Se libera una franja y tres personas la esperan | Se avisa a las tres; la franja la toma quien la reserve. | Lista de espera |
| Un miembro quiere reservar un escritorio flexible para mañana | No se reserva: se sienta donde haya lugar libre, según su plan. | Escritorio flexible |
| Un miembro quiere reservar una sala para dentro de dos meses | Se puede: no hay anticipación máxima. | Anticipación |

---

## 5. Hotspots resueltos

| # | Hotspot (pregunta) | Respuesta del experto | Cómo queda en el glosario |
|---|---|---|---|
| 1 | ¿El escritorio flexible se reserva por franja o por día? | No se reserva por franja: se gestiona a través de los planes de membresía (pase por día, mensual flexible) y «te sientas en cualquiera que esté libre». La reserva por franja aplica a las salas de reunión. | *Escritorio flexible* deja de ser un recurso reservable; *Recurso reservable* es la sala de reunión. |
| 2 | ¿Cuál es la anticipación máxima para reservar? | No hay tope máximo. La anticipación ideal es de 24 h y la mínima de 3 h. Las cotizaciones (eventos o reservas mensuales) vencen a los 4 días si no se confirman con pago o depósito. | *Anticipación*: no hay anticipación máxima. |
| 3 | Cuando se libera una franja, ¿se avisa a todos o por orden de llegada? | Se avisa a todos los de la lista de espera de esa franja. No hay asignación automática por orden de llegada: la franja la toma quien la reserve. | *Lista de espera*: aviso a todos, sin asignación automática. |
| 4 | ¿Qué pasa si se bloquea una franja que ya tiene una reserva confirmada? | La reserva no se elimina sin solución: recepción u operaciones reasigna al titular a una sala equivalente o coordina la reprogramación o el reembolso, según su preferencia. | *Bloqueo*: se suma la regla de reasignación. |
