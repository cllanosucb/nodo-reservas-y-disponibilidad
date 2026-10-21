# 02 · Glosario del lenguaje ubicuo — Reservas y disponibilidad

> Rebanada: **Reservar una sala**. Significados acordados con el experto. Separado por bounded context.

## Contexto: Disponibilidad

| Término | Significado acordado | Qué NO es | Regla / invariante | Evento |
|---|---|---|---|---|
| **Sala** | Espacio de reunión reservable por franja. | No es un escritorio. | Una sola reserva por franja. | — |
| **Recurso reservable** | Espacio que se reserva por franja: sala de reunión o escritorio flexible. | No incluye el escritorio fijo (se asigna a un miembro) ni el salón de eventos. | Una sola reserva por franja y recurso. | Reserva confirmada |
| **Escritorio flexible** | Escritorio de la sala abierta; el miembro se sienta donde haya lugar libre. | No es un escritorio fijo asignado a una persona. | ⚠️ Hotspot: ¿se reserva por franja o por día? | Reserva solicitada |
| **Franja** | Intervalo horario reservable en una sala. | No es el día completo. | Dos franjas reservadas no se pisan. | Reserva confirmada / Franja liberada |
| **Margen (buffer)** | 15 min obligatorios entre reservas para limpiar y ventilar. | No es tiempo reservable. | Se aplica siempre, de forma automática, entre dos reservas de la misma sala. | — |
| **Bloqueo** | Franja que Nodo saca de disponibilidad (p. ej., mantenimiento). | No es una reserva: no tiene miembro ni cobro. | Solo recepción o administración pueden bloquear. | Sala bloqueada |
| **Disponibilidad** | Estado de una sala en una franja: libre, sin reserva, sin bloqueo y fuera del margen. | No equivale a que la sala exista. | Solo se reserva una franja disponible, con 15 min de margen. | — (modelo de lectura "Salas disponibles") |

## Contexto: Reserva

| Término | Significado acordado | Qué NO es | Regla / invariante | Evento |
|---|---|---|---|---|
| **Solicitud de reserva** | Pedido de un miembro para una franja, aún sin validar. | No es una reserva: todavía no ocupa la franja. | Se valida contra disponibilidad, plan y anticipación. | Reserva solicitada |
| **Reserva** | Franja de un recurso asignada a un miembro. | No es un bloqueo: la reserva es de un miembro; el bloqueo, de Nodo. | Requiere franja libre con margen. | Reserva confirmada |
| **Confirmada** | Estado de una reserva: hay disponibilidad y el miembro está al día; no requiere pago previo. | No es la confirmación de un evento (depósito + aprobación de dirección). | Automática con ≥ 24 h de anticipación. | Reserva confirmada |
| **Anticipación** | Tiempo entre el momento de reservar y el inicio de la franja. | No es la duración de la reserva. | ≥ 24 h: automática; entre 3 y 24 h: reserva asistida. ⚠️ Máxima por definir. | — |
| **Reserva asistida** | Reserva coordinada con recepción (poca anticipación o a nombre de otra persona). | No es automática: la confirma recepción. | Mínimo unas 3 h de anticipación (a validar). | Reserva confirmada |
| **Miembro al día** | Miembro sin facturas vencidas. | No equivale a *activo*: un miembro con membresía vigente puede deber la última factura. | Solo un miembro al día confirma reservas. | — |
| **Visitante** | Persona sin membresía que reserva una sala. | No es miembro: no tiene plan ni horas incluidas. | ⚠️ Hotspot: ¿qué se le exige para confirmar? | Reserva solicitada |
| **Cancelación** | Anulación de una reserva por el miembro o por recepción. | No es un bloqueo (el bloqueo actúa sobre una franja, no sobre una reserva). | Sin cobro hasta 4 h antes del inicio. | Reserva cancelada / Franja liberada |
| **Cancelación tardía** | Cancelación con menos de 4 h de anticipación. | No es una cancelación sin costo. | Genera penalización. | Penalización aplicada |
| **Penalización** | Cobro por cancelar fuera de plazo. | No es una hora extra ni un depósito. | Solo aplica a cancelaciones tardías. ⚠️ Monto: ¿10–20 % o franja completa? | Penalización aplicada |
| **Horas extra** | Horas de sala que exceden las incluidas en el plan. | No son una penalización. | Se cobran a fin de mes. | Horas extra registradas |
| **Lista de espera** | Miembros interesados en una franja ocupada. | No es una reserva: no garantiza la sala. | Se avisa cuando la franja se libera. | Añadido a la lista de espera / Lista de espera notificada |

## Polisemias detectadas
| Palabra | En Reserva / Disponibilidad | En otro contexto |
|---|---|---|
| **Confirmar** | Hay franja libre y el miembro está al día; sin pago | **Eventos:** depósito + aprobación de dirección |
| **Al día vs. activo** | *Al día* = sin facturas vencidas | **Membresías:** *activo* = membresía vigente |
| **Bloquear** | Nodo saca una franja de disponibilidad | No es cancelar una reserva |
| **Escritorio** | *Flexible*: recurso reservable | *Fijo*: asignación a un miembro, fuera de Reservas |

## Hotspots abiertos
1. Escritorio flexible: ¿se reserva por franja o por día?
2. Visitante: ¿qué se exige para confirmar (pago previo, datos)?