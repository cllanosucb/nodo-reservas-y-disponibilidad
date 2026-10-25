# 0002 · Clasificación del área: Reservas y disponibilidad es el core

- **Estado:** Aceptada
- **Área:** Reservas y disponibilidad (core)

## Contexto
Nodo tiene ocho subdominios: Reservas y disponibilidad, Membresías y planes, Tarifas y precios, Agenda de eventos, Espacios y recursos, Facturación y cobros, Control de acceso físico y Notificaciones. Hay que decidir cuál es el core, es decir, dónde invertir más diseño y desarrollo propio, y cuáles se construyen de forma simple o se integran.

Las dudas principales eran dos: si el core es Reservas y disponibilidad o Membresías y planes, que es la primera fuente de ingreso, y si Agenda de eventos también es core, porque comparte los espacios físicos.

## Decisión
**Reservas y disponibilidad es el subdominio core.** Membresías y planes, Tarifas y precios, Agenda de eventos y Espacios y recursos son supporting. Facturación y cobros, Control de acceso físico y Notificaciones son generic; Notificaciones es además transversal.

## Justificación
- **Ventaja competitiva:** los clientes eligen Nodo porque siempre encuentran un espacio disponible y reservarlo es rápido, sin solapes, sin sobrecupo y sin sorpresas. Esa promesa vive en Reservas y disponibilidad.
- **Reglas del área:** dos reservas no pueden solaparse sobre la misma sala, entre una y otra debe quedar un margen de 15 minutos, el sistema debe evitar el sobrecupo y, si dos personas piden la última franja a la vez, solo una debe quedarse con ella. Son las reglas más exigentes del negocio, y todas son de esta área.
- **Lo que hoy duele:** el equipo gestiona todo con una hoja de cálculo, un calendario compartido y grupos de WhatsApp, y la recepcionista envía los avisos a mano. Es la operación que más gana con un sistema propio.

## Alternativas descartadas
- **Membresías y planes como core.** Es una fuente de ingreso importante, pero el modelo de planes (pase por día, mensual flexible, escritorio dedicado, plan de equipo) existe en cualquier coworking. Para nuestra área solo aporta dos datos: si el miembro está al día y cuántas horas incluidas tiene.
- **Agenda de eventos como core.** Comparte el espacio, pero tiene otro ciclo (depósito, aforo máximo, servicios extra, aprobación de la dirección u operaciones). «Confirmar» significa otra cosa.
- **Construir los genéricos.** Facturar, cobrar, abrir una puerta con PIN, código QR o huella y enviar correos o mensajes no diferencia a Nodo; se integran.

## Consecuencias
- La mayor inversión de diseño y desarrollo va a Reservas y disponibilidad, que se divide en los bounded contexts Disponibilidad y Reserva (ver [0001-por-que-esta-frontera.md](0001-por-que-esta-frontera.md)).
- Los subdominios supporting se construyen de forma simple y entregan a nuestra área solo los datos que necesita.
- Los genéricos se integran con servicios externos.
- El área cubre las salas de reunión, que se reservan por franja. Los escritorios quedan fuera porque no se reservan: el flexible se usa según el plan de membresía y el fijo se asigna a un miembro. El salón también queda fuera: pertenece a Agenda de eventos.

## Hotspots resueltos

| Pregunta | Respuesta |
|---|---|
| **Escritorio fijo:** se nombra como «recurso reservable» y, a la vez, como «asignado a un miembro». ¿Se asigna o también se reserva por franja? | Se asigna a una persona o equipo y no se reserva. Queda fuera del área. |
| **Segunda sede:** si Nodo abre otra sede, ¿la disponibilidad se consulta por sede o para todas a la vez? | Cada sala pertenece a una sede; quien reserva elige la sede o ve las salas de todas. |
