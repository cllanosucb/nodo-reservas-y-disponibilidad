# 0001 · Por qué esta frontera: Disponibilidad y Reserva

- **Estado:** Aceptada
- **Área:** Reservas y disponibilidad (core)

## Contexto
Al principio asumimos que el área era un solo contexto. El glosario ([docs/02-glosario.md](../docs/02-glosario.md)) y el EventStorming ([docs/03-eventstorming.md](../docs/03-eventstorming.md)) mostraron dos lenguajes y dos ritmos de cambio dentro de la misma área.

## Decisión
Dividimos el área en dos bounded contexts, los dos core:

| Contexto | Responsabilidad | Lenguaje propio |
|---|---|---|
| **Disponibilidad** | Qué franjas de qué recurso están libres (salas y unidades de equipo portátil): franjas, margen de 15 min en las salas y bloqueos por mantenimiento o limpieza. | Sala de reunión, Recurso reservable, Equipo portátil, Franja, Margen, Bloqueo, Ocupación, Disponibilidad |
| **Reserva** | El ciclo de vida de lo que pide una persona (miembro o visitante): solicitud, confirmación o rechazo, reserva asistida, cancelación, penalización, horas extra y lista de espera. | Solicitud de reserva, Reserva, Confirmada, Anticipación, Reserva asistida, Miembro al día, Visitante, Invitado, Aforo, Cancelación, Cancelación tardía, Penalización, Horas extra, Lista de espera |

## Señales que justifican la frontera
1. **Polisemia: "bloquear" vs. "cancelar".** Un bloqueo es de Nodo y actúa sobre una franja, sin titular ni cobro. Una cancelación es del titular o de recepción y actúa sobre una reserva, con posible cobro de la franja completa. Ambas quitan o liberan la franja, pero significan cosas distintas.
2. **Reglas que cambian por su cuenta.** El margen de 15 min y los bloqueos responden a la operación (limpieza, ventilación y mantenimiento). La anticipación, la penalización, el criterio de "al día" y el pago por adelantado del visitante son política comercial. Pueden cambiar sin tocar el otro lado.
3. **Actores distintos.** En Disponibilidad bloquean recepción, administración o dirección. En Reserva actúan el miembro, el visitante y recepción (reserva asistida, cancelación y resolución de una reserva afectada por un bloqueo).
4. **Preguntas distintas.** Disponibilidad responde «¿qué está libre?», que es el modelo de lectura *Salas disponibles*. Reserva responde «¿esta persona puede quedarse con esta franja y bajo qué condiciones?».

## Evidencia, herramienta por herramienta

La frontera se justifica con cinco elementos, no solo con el glosario. Los cinco señalan la misma frontera:

| Herramienta | Qué muestra | Dónde |
|---|---|---|
| **Glosario (polisemia)** | *Bloquear* no es *cancelar*; *liberar* le ocurre a la franja, *cancelar* a la reserva. Cada contexto agrupa sus términos: sala, franja, margen y bloqueo en Disponibilidad; solicitud, anticipación, penalización y lista de espera en Reserva. | [02-glosario.md](../docs/02-glosario.md) |
| **Reglas de negocio** | Las de Disponibilidad (R-01, R-06, R-07, R-08) cambian por la operación; las de Reserva (R-02 a R-05), por la política comercial. | [06-reglas-invariantes.md](../docs/06-reglas-invariantes.md) |
| **Actores y responsabilidades** | Bloquean recepción, administración o dirección; reservan el miembro, el visitante y recepción. | [03-eventstorming.md](../docs/03-eventstorming.md), sección 5 |
| **EventStorming** | La línea *Preparación* es de Disponibilidad y las demás de Reserva; *Franja liberada* y *Sala bloqueada* le ocurren a la agenda, no a la reserva. | [03-eventstorming.md](../docs/03-eventstorming.md), sección 6 |
| **Domain stories** | En el as-is, la misma recepcionista cuida dos cosas distintas: el calendario (solapes y margen) y la persona (si está al día, el cobro, el aviso). En el to-be, el «sistema de reservas» pregunta por la franja y decide por la persona. | [04-domain-stories.md](../docs/04-domain-stories.md) |

## Alternativas descartadas
- **Un solo contexto.** Mezcla reglas operativas y comerciales en el mismo modelo, y "liberar franja" queda ambiguo (¿bloqueo o cancelación?).
- **Lista de espera como contexto propio.** No tiene lenguaje ni reglas propias: solo reacciona a *Franja liberada* avisando a todos los que esperan. Vive dentro de Reserva.
- **Agenda de eventos dentro de Reserva.** "Confirmar" significa otra cosa (depósito y aprobación de la dirección u operaciones), y el salón es un recurso aparte. Agenda de eventos es otro subdominio (supporting).
- **Penalización y horas extra en Tarifas y precios.** Reserva decide *cuándo* aplican; Tarifas y precios entrega el precio vigente de la franja. La penalización es la franja completa.

## Consecuencias
- Cada contexto tiene su sección en el glosario y su módulo en el prototipo (`src/Disponibilidad` y `src/Reserva` del [repositorio del prototipo](../src/README.md)).
- Entre ambos cruza información en los dos sentidos (línea ① de [docs/05-context-map.md](../docs/05-context-map.md)):
  - Reserva le **pide** a Disponibilidad que ocupe la franja **antes de confirmar**, con una llamada en proceso: la reserva se confirma solo si la franja quedó ocupada (INV-R1; ver [0003-patrones-de-integracion.md](0003-patrones-de-integracion.md)).
  - Disponibilidad **escucha** *Reserva cancelada* y *Solicitud expirada* para liberar la franja (P-07, P-11).
  - Reserva **escucha** *Franja liberada* para avisar a la lista de espera (P-06) y *Sala bloqueada* para resolver una reserva afectada (P-09).
- Coherencia con el subdominio: los dos contextos pertenecen al core, y ninguno es genérico ni comprable ([0002-clasificacion-del-area.md](0002-clasificacion-del-area.md)).

## Hotspots resueltos

| ID | Pregunta | Respuesta |
|---|---|---|
| HI-01 | ¿Qué garantiza «una reserva por franja» cuando dos solicitudes piden la misma franja a la vez? | La agenda de la sala garantiza aislamiento transaccional: se confirma una sola solicitud y la otra se rechaza o entra en la lista de espera. |
| HI-14 | ¿Quién calcula el monto de la franja completa? | Facturación y cobros, con el precio conservado que envía Reserva. |
| — | ¿El escritorio flexible se reserva por franja o por día? | No se reserva: se usa según el plan de membresía. Disponibilidad solo modela salas de reunión. |
| — | ¿La penalización es un porcentaje o la franja completa? | La franja completa. |
