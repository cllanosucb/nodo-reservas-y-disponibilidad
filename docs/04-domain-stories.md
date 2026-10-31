# 04 · Domain stories: Reservas y disponibilidad

> **Proyecto:** Nodo (coworking y eventos) · **Área:** Reservas y disponibilidad (core)

---

## Vista general

![Domain stories de Reservas y disponibilidad](img/04-domain-stories.png)

---

## 1. Propósito

Las domain stories cuentan cómo se reserva hoy en Nodo (**as-is**) y cómo se reservaría con el sistema (**to-be**). Sirven para dos cosas:

- **Ver dónde duele el proceso actual:** lo manual, lo lento y lo que se olvida (fricciones).
- **Mostrar cómo lo resuelve el sistema:** cada paso del to-be indica la nota del EventStorming ([03-eventstorming.md](03-eventstorming.md)) con la que concuerda.

---

## 2. Cómo se construyó

**Notación (Domain Storytelling).** Cada historia se dibuja con:

| Elemento | Símbolo en Miro | Qué es |
|---|---|---|
| Actor | 👤 persona · 💻 sistema | Quién hace algo |
| Objeto de trabajo | 📄 documento · 💬 mensaje · 💵 pago · 📊 modelo de lectura | Con qué trabaja el actor |
| Actividad | Flecha con un verbo | Qué hace el actor con el objeto |
| Secuencia | ①, ②, ③… sobre la flecha | El orden en que se lee la historia |
| Fricción | Nota roja (solo en el as-is) | Algo manual, lento o que se olvida |

Cada paso se lee como una frase: *actor — actividad — objeto de trabajo — a / en quién*.

**Alcance:** detallado (*fine-grained*), as-is y to-be, digitalizado. Cada historia muestra su camino feliz; las variantes se indican debajo del to-be.

**Las cuatro historias**

| Historia | Tipo | Por qué está |
|---|---|---|
| 1. Reservar una sala | Caso feliz | Es el flujo principal del área |
| 2. Cancelar con menos de 4 h y avisar a la lista de espera | Excepción | Es el caso del curso (cancelar 3 h antes) y concentra los avisos manuales |
| 3. Reserva asistida | Alternativa | Recepción registra la reserva cuando la anticipación es de 3 a 24 h o es a nombre de un invitado |
| 4. Visitante reserva y paga por adelantado | Alternativa | La persona sin membresía confirma con un pago previo |

---

## 3. Actores y objetos de trabajo

| Tipo | Elementos |
|---|---|
| **Actores (personas)** | Miembro, Visitante, Recepción, Titular o recepción, Personas en lista de espera |
| **Actores (sistemas)** | Sistema de reservas, Membresías y planes, Facturación y cobros, Notificaciones |
| **Objetos de trabajo hoy** | Solicitud de sala, hoja de cálculo, calendario compartido, estado de la membresía, reserva, mensajes de WhatsApp, cobro anotado, pago por adelantado |
| **Objetos de trabajo con el sistema** | Salas disponibles, solicitud de reserva, reserva asistida, reserva confirmada, cancelación, penalización, franja liberada, confirmación, recordatorio, aviso de franja liberada, pago por adelantado |

Los términos son los de [02-glosario.md](02-glosario.md).

---

## 4. Las historias

### 4.1 Historia 1: Reservar una sala

**Hoy (as-is)**

| N.º | Paso | Fricción |
|---|---|---|
| 1 | Miembro pide una sala a Recepción, en persona o por WhatsApp | Depende de que recepción atienda |
| 2 | Recepción consulta la hoja de cálculo y el calendario compartido | Los solapes y el margen de 15 min se cuidan a mano |
| 3 | Recepción verifica que la membresía esté activa y al día | |
| 4 | Recepción anota la reserva en la hoja de cálculo | |
| 5 | Recepción envía la confirmación por WhatsApp al Miembro | Recepción tiene que estar pendiente |
| 6 | Recepción envía el recordatorio por WhatsApp, un día antes, al Miembro | Se puede olvidar |

Al miembro no se le exige pago por adelantado: el uso se descuenta de las horas incluidas en su plan o se cobra como hora extra a fin de mes.

**Con el sistema (to-be)**

| N.º | Paso | Nota del muro |
|---|---|---|
| 1 | Miembro consulta las salas disponibles en el Sistema de reservas | Modelo de lectura |
| 2 | Miembro envía la solicitud de reserva (franja y asistentes) al Sistema de reservas | Reserva solicitada |
| 3 | Sistema de reservas consulta a Membresías y planes si el miembro está al día | R-02 |
| 4 | Sistema de reservas confirma la reserva: anticipación de 24 h o más, franja libre con margen y asistentes dentro del aforo | R-01, R-02, R-04, P-01, Reserva confirmada |
| 5 | Sistema de reservas pide el aviso a Notificaciones | |
| 6 | Notificaciones envía la confirmación al Miembro | Confirmación enviada |
| 7 | Notificaciones envía el recordatorio, un día antes, al Miembro | P-03, Recordatorio enviado |

*Variante:* si la solicitud no cumple las reglas o pierde la última franja, el Sistema de reservas la rechaza y Notificaciones avisa al Miembro (P-08, Solicitud rechazada). Si la franja está ocupada, la solicitud entra en la lista de espera (P-02).

*Variante con equipo:* en el paso 2 el miembro puede pedir equipo portátil (proyector o televisor). En el paso 4 el sistema asigna una unidad libre de cada equipo en la misma franja; si no hay, rechaza la solicitud con ese motivo (R-08).

*Variante de uso:* si el miembro termina antes, Recepción registra la salida anticipada y el sistema libera el tramo que falta y avisa a la lista de espera (P-12, Franja liberada).

### 4.2 Historia 2: Cancelar con menos de 4 h y avisar a la lista de espera

**Hoy (as-is)**

| N.º | Paso | Fricción |
|---|---|---|
| 1 | Miembro avisa la cancelación a Recepción | |
| 2 | Recepción comprueba si faltan menos de 4 h para el inicio de la reserva | |
| 3 | Recepción anota el cobro en la hoja de cálculo | El cobro se anota a mano y depende de su criterio: hoy aplica del 10 al 20 % según si había otras solicitudes (experto) |
| 4 | Facturación y cobros suma el cobro a la factura mensual | |
| 5 | Recepción libera la franja en la hoja de cálculo y el calendario compartido | La franja se libera a mano |
| 6 | Recepción envía el aviso de sala liberada por WhatsApp a las personas en lista de espera | Recepción tiene que estar pendiente de quién espera |

**Con el sistema (to-be)**

| N.º | Paso | Nota del muro |
|---|---|---|
| 1 | Titular o recepción cancela la reserva en el Sistema de reservas | Reserva cancelada |
| 2 | Sistema de reservas registra la penalización (franja completa) y la entrega a Facturación y cobros | P-05, Penalización aplicada |
| 3 | Sistema de reservas libera la franja | P-07, Franja liberada |
| 4 | Sistema de reservas pide el aviso a Notificaciones | |
| 5 | Notificaciones envía el aviso de franja liberada a todas las personas en lista de espera | P-06, Lista de espera notificada |

*Variante:* con 4 h o más de anticipación no hay cobro; se cumplen los pasos 1, 3, 4 y 5.

*Del as-is al to-be:* el sistema aplica una sola regla, la del enunciado y la entrevista escrita: con menos de 4 h se cobra la franja completa (H-08). El porcentaje variable de hoy era una decisión manual de recepción.

### 4.3 Historia 3: Reserva asistida

**Hoy (as-is)**

| N.º | Paso | Fricción |
|---|---|---|
| 1 | Miembro pide a Recepción una reserva con poca anticipación o a nombre de un invitado | |
| 2 | Recepción consulta la hoja de cálculo y el calendario compartido | Los solapes y el margen de 15 min se cuidan a mano |
| 3 | Recepción anota la reserva, con los datos del invitado, en la hoja de cálculo | |
| 4 | Recepción envía la confirmación por WhatsApp al Miembro | Recepción tiene que estar pendiente |

**Con el sistema (to-be)**

| N.º | Paso | Nota del muro |
|---|---|---|
| 1 | Miembro pide la reserva a Recepción (anticipación de 3 a 24 h o a nombre de un invitado) | |
| 2 | Recepción registra la reserva asistida en el Sistema de reservas | Registrar reserva asistida, R-03 |
| 3 | Sistema de reservas verifica la franja, el margen y el aforo, y confirma la reserva | R-01, R-04, Reserva confirmada |
| 4 | Sistema de reservas pide el aviso a Notificaciones | |
| 5 | Notificaciones envía la confirmación al Miembro | Confirmación enviada |

*Variante:* con menos de 3 h de anticipación no se puede reservar (R-03). Si es a nombre de un invitado, la reserva queda bajo la responsabilidad del miembro.

### 4.4 Historia 4: Visitante reserva y paga por adelantado

**Hoy (as-is)**

| N.º | Paso | Fricción |
|---|---|---|
| 1 | Visitante pide una sala a Recepción | |
| 2 | Recepción consulta la hoja de cálculo y el calendario compartido | Los solapes y el margen de 15 min se cuidan a mano |
| 3 | Visitante paga por adelantado en Recepción (tarjeta, transferencia, código QR o efectivo) | El pago y la confirmación dependen de recepción |
| 4 | Recepción anota la reserva en la hoja de cálculo | |
| 5 | Recepción confirma la reserva al Visitante | |

**Con el sistema (to-be)**

| N.º | Paso | Nota del muro |
|---|---|---|
| 1 | Visitante consulta las salas disponibles en el Sistema de reservas | Modelo de lectura |
| 2 | Visitante envía la solicitud de reserva al Sistema de reservas | Reserva solicitada |
| 3 | Visitante paga por adelantado en Facturación y cobros, dentro del plazo límite; mientras tanto, la franja queda apartada de forma provisional | R-05 |
| 4 | Facturación y cobros registra el pago en el Sistema de reservas | Pago por adelantado registrado |
| 5 | Sistema de reservas confirma la reserva | P-10, Reserva confirmada |
| 6 | Sistema de reservas pide el aviso de confirmación a Notificaciones | Confirmación enviada |

*Variante:* si el visitante no paga dentro del plazo, la solicitud expira y la franja se libera (P-11, Solicitud expirada).

---

## 5. Contraste: qué conviene automatizar

| Qué conviene automatizar | Fricción de hoy | Cómo lo resuelve el to-be | Muro |
|---|---|---|---|
| Consultar qué sala está libre | Depende de que recepción atienda | El miembro o el visitante consulta las salas disponibles por su cuenta | Modelo de lectura |
| Solapes, margen de 15 min y aforo | Se cuidan a mano en la hoja de cálculo y el calendario | El sistema garantiza las reglas | R-01, R-04 |
| Verificar que el miembro está al día | Recepción lo verifica a mano | Se consulta a Membresías y planes al momento de confirmar | R-02 |
| Confirmación, recordatorio y aviso a la lista de espera | Los escribe recepción por WhatsApp y puede olvidarlos | Salen solos al producirse el evento | P-03, P-06 |
| Cobro de la cancelación tardía | Se anota a mano en la hoja de cálculo | La penalización se registra y se entrega a Facturación y cobros | P-05 |
| Liberar la franja | Se libera a mano | Se libera al cancelar | P-07 |
| Reserva asistida | Recepción anota todo a mano | Recepción la registra en el sistema, que aplica las mismas reglas | R-03 |
| Equipo portátil | Se presta a pedido y puede estar ya prometido en otra sala o en el salón | El sistema lo asigna por franja y nunca dos veces a la vez | R-08 |
| Sala que se desocupa antes | Recepción la ofrece de palabra a quien esté cerca | Se libera el tramo que falta y se avisa a la lista de espera | P-12 |
| Pago del visitante | El pago y la confirmación dependen de recepción | El pago registrado dispara la confirmación | R-05, P-10 |
