# 0001 · Por qué esta frontera: Disponibilidad y Reserva

- **Estado:** Aceptada (provisional hasta la Sesión 4)
- **Área:** Reservas y disponibilidad (Core)

## Contexto
Al principio asumimos que el área era un solo contexto. El glosario y el EventStorming mostraron dos lenguajes y dos ritmos de cambio dentro de la misma área.

## Decisión
Dividimos el área en dos bounded contexts:

| Contexto | Responsabilidad | Lenguaje propio |
|---|---|---|
| **Disponibilidad** | Qué franjas de qué recurso están libres: franjas, margen de 15 min y bloqueos. | Sala, Recurso reservable, Escritorio flexible, Franja, Margen, Bloqueo, Disponibilidad |
| **Reserva** | El ciclo de vida de lo que pide un miembro: solicitud, confirmación, cancelación, penalización y lista de espera. | Solicitud, Reserva, Confirmada, Anticipación, Reserva asistida, Miembro al día, Visitante, Cancelación, Penalización, Horas extra, Lista de espera |

## Señales que justifican la frontera
1. **Polisemia: "bloquear" vs. "cancelar".** Un bloqueo es de Nodo y actúa sobre una franja, sin miembro ni cobro. Una cancelación es de un miembro y actúa sobre una reserva, con posible penalización. Ambas liberan o quitan la franja, pero significan cosas distintas.
2. **Reglas que cambian por su cuenta.** El margen de 15 min y los bloqueos dependen de operaciones (limpieza, mantenimiento). La anticipación, la penalización y el criterio de "al día" son política comercial. Pueden cambiar sin tocar el otro lado.
3. **Actores distintos.** En Disponibilidad decide recepción u operaciones (quien bloquea). En Reserva actúan el miembro, el visitante y recepción en la reserva asistida.
4. **Preguntas distintas.** Disponibilidad responde "¿qué está libre?", que es el modelo de lectura *Salas disponibles*. Reserva responde "¿esta persona puede quedarse con esta franja y bajo qué condiciones?".

## Alternativas descartadas
- **Un solo contexto.** Mezcla reglas operativas y comerciales en el mismo modelo, y "liberar franja" queda ambiguo (¿bloqueo o cancelación?).
- **Lista de espera como contexto propio.** Se descartó porque no tiene lenguaje ni reglas propias. Solo reacciona a *Franja liberada*, así que vive dentro de Reserva.
- **Eventos dentro de Reserva.** Se descartó porque "confirmar" significa otra cosa (depósito + aprobación de dirección). Eventos pertenece a otro subdominio (supporting).
- **Penalización y horas extra en Tarifas.** Reserva decide *cuándo* aplican; Tarifas solo dice *cuánto*. El monto queda como hotspot de integración.

## Consecuencias
- Cada contexto tiene su carpeta en `src/` y su sección en el glosario.
- Entre ambos cruza información en los dos sentidos:
  - Reserva le **pide** a Disponibilidad si una franja está libre.
  - Disponibilidad **escucha** *Reserva confirmada* y *Reserva cancelada* para ocupar o liberar la franja.
  - Reserva **escucha** *Franja liberada* para avisar a la lista de espera.
- Coherencia con el subdominio: los dos contextos pertenecen al core, y ninguno es genérico ni comprable.

## Hotspots (para la Sesión 4)
1. **¿Quién garantiza el invariante "una reserva por franja"?** Si dos solicitudes piden la misma franja a la vez, la consistencia cruza la frontera. Opciones: Disponibilidad ocupa la franja de forma atómica, o se reconsidera la frontera.
2. ¿Escritorio flexible por franja o por día? Si es por día, ¿Disponibilidad lo modela igual que una sala?
3. ¿Penalización y horas extra se calculan en Reserva o se delegan a Tarifas?