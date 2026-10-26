# 09 · Componentes: del modelo al código

> **Proyecto:** Nodo (coworking y eventos) · **Área:** Reservas y disponibilidad (core)
> **Alcance:** qué piezas necesita el modelo de [08-modelo.md](08-modelo.md) para funcionar: domain services, repositories, factories, application services y dependencias. Todo en **pseudocódigo neutro**: el lenguaje se define en el prototipo. Caso principal: **reservar una sala**; cancelar reutiliza las mismas piezas.

---

## Vista general

![Componentes: reservar una sala](img/09-componentes.png)

**Cómo leer el frame**

| Forma en el frame | Qué es |
|---|---|
| Recuadros 1 a 7 | Una pregunta de diseño por recuadro (P01–P05) y su respuesta |
| Recuadros de código | Pseudocódigo: interfaces de repositorios y application services |
| Caja azul (flujos) | Application service o método de un aggregate |
| Caja naranja (flujos) | Domain event |
| Caja gris (flujos) | Entrada, ACL o vecino |

---

## 1. Aggregates e invariantes (P01: ¿qué regla se protege y dónde vive?)

| Aggregate | Contexto | Invariantes | Detalle |
|---|---|---|---|
| **Reserva** | Reserva | INV-R2 a INV-R8 | [08-modelo.md](08-modelo.md#3-aggregate-reserva) |
| **Agenda del recurso** | Disponibilidad | INV-D1 a INV-D5 | [08-modelo.md](08-modelo.md#2-aggregate-agenda-del-recurso-disponibilidad) |
| **Lista de espera** | Reserva | INV-R9 | [08-modelo.md](08-modelo.md#4-aggregate-lista-de-espera-reserva) |

INV-R1 (toda reserva confirmada tiene su franja ocupada) cruza los dos contextos: la cumple el application service con la secuencia «ocupar la franja → confirmar la reserva».

---

## 2. Domain services (P02: ¿método de entidad o servicio?)

**Ninguno.** Cada regla tiene un dueño dentro de un aggregate:

| Regla | Dónde vive | Por qué no es un domain service |
|---|---|---|
| No solapar, con 15 min de margen | `AgendaDelRecurso.ocupar()` | Necesita el estado de la agenda y su versión; un servicio sin estado no protege el último cupo. |
| Penalización por cancelación tardía | `PolíticaDeCancelación` (value object) | Solo depende de la reserva y de su política. |
| Confirmar | `Reserva.confirmar()` | Usa sus propios datos y los que recibe (al día, precio vigente). |
| Reasignar a una sala equivalente | Recepción (P-09) | Es una decisión de una persona, no una regla automática. |

Crear un servicio para estas reglas vaciaría los aggregates (modelo anémico).

---

## 3. Repositories (P03: ¿uno por aggregate raíz?)

Uno por raíz. **Ocupación** y **Bloqueo** viajan con su agenda: no tienen repositorio propio. El miembro no es nuestro: se consulta con la fachada `EstadoDelMiembro` del ACL de Membresías y planes ([07-integracion.md](07-integracion.md), sección 5). En el prototipo, los repositorios son **en memoria**.

```text
ReservaRepository
  porId(reservaId): Reserva?
  provisionalesVencidas(ahora): Reserva[]
  guardar(reserva)

AgendaDelRecursoRepository
  de(salaId, fecha): AgendaDelRecurso?
  guardar(agenda)            // falla si cambió la versión (bloqueo optimista)

ListaDeEsperaRepository
  de(salaId, franja): ListaDeEspera?
  guardar(lista)
```

---

## 4. Factories (P04: ¿resuelve complejidad real?)

**Sin clases factory:** métodos de creación en la raíz.

| Método | Qué hace |
|---|---|
| `Reserva.solicitar(titular, salaId, franja, asistentes, registradaPor?, ahora)` | Fija el estado Solicitada, calcula la Anticipación (INV-R4), distingue miembro, visitante y reserva asistida, y asigna `PolíticaDeCancelación.estándar()`. |
| `AgendaDelRecurso.abrir(salaId, fecha)` | Crea la agenda vacía la primera vez que se pide esa sala ese día. |
| `ListaDeEspera.abrir(salaId, franja)` | Crea la lista vacía la primera vez que alguien espera esa franja. |

Ninguno coordina otros aggregates: una clase factory solo envolvería un constructor.

---

## 5. Application services (P05: coordinan, no deciden)

### Reservar una sala

```text
SolicitarReserva.ejecutar(cmd)
  ahora   = reloj.ahora()
  franja  = Franja(cmd.inicio, cmd.fin)
  reserva = Reserva.solicitar(cmd.titular, cmd.salaId, franja,
              cmd.asistentes, cmd.registradaPor, ahora)   // INV-R4
  aforo   = aforoDeSala.consultar(cmd.salaId)             // ACL ③
  estado  = estadoDelMiembro.consultar(cmd.titular)       // ACL ⑤
  si estado = sin respuesta: derivar a recepción (reserva asistida)   // HI-13
  si no cumple aforo o al día: reserva.rechazar(motivo)   // INV-R3, INV-R5

  agenda = agendas.de(cmd.salaId, franja.dia)
           ?? AgendaDelRecurso.abrir(cmd.salaId, franja.dia)
  r = agenda.ocupar(franja, reserva.id, provisional = reserva.esVisitante)
  agendas.guardar(agenda)       // versión cambiada: reintenta una vez

  si r = no disponible:
    reserva.rechazar("franja ocupada")                    // Solicitud rechazada
    lista = listas.de(cmd.salaId, franja) ?? ListaDeEspera.abrir(cmd.salaId, franja)
    lista.añadir(cmd.titular)                             // P-02
    listas.guardar(lista)
  si no, si reserva.esVisitante:
    reserva.dejarProvisional(plazoDePago)                 // INV-R6
  si no:
    precio = precioDeFranja.vigente(cmd.salaId, franja)   // ACL ④
    reserva.confirmar(estado.alDia, precio, ahora)        // Reserva confirmada

  si falla reservas.guardar(reserva):
    agenda.liberar(reserva.id); agendas.guardar(agenda)   // compensación: INV-R1
  eventos.publicar(reserva.eventos)
  devolver reserva.id
```

**Por qué así:**
- El application service **coordina** (pide datos, llama a los aggregates, guarda y publica); las reglas las deciden `Reserva` y `AgendaDelRecurso`.
- **Ocupar antes de confirmar** cumple INV-R1 sin modificar dos aggregates en la misma transacción. Si guardar la reserva falla, se libera la franja.
- Si la franja está ocupada, la solicitud queda **rechazada** y la persona se **añade a la lista de espera** (D-57): quien espera no tiene reserva.

### Cancelar una reserva

```text
CancelarReserva.ejecutar(cmd)
  reserva = reservas.porId(cmd.reservaId)
  reserva.cancelar(cmd.quien, reloj.ahora())   // INV-R7: menos de 4 h → Penalización aplicada
  reservas.guardar(reserva)
  eventos.publicar(reserva.eventos)            // Reserva cancelada

// Eventos en proceso
LiberarFranja.al(Reserva cancelada | Solicitud expirada)    // Disponibilidad, P-07 y P-11
  agenda.liberar(e.reservaId)                               // Franja liberada
AvisarListaDeEspera.al(Franja liberada)                     // Reserva, P-06
  lista.notificarATodos()                                   // Lista de espera notificada → Notificaciones (cola)
```

### Otros application services

| Application service | Qué coordina | Origen |
|---|---|---|
| Registrar reserva asistida | Igual que `SolicitarReserva`, con `registradaPor` y, si corresponde, los datos del invitado | INV-R8, R-03 |
| Confirmar pago del visitante | Al recibir *Pago por adelantado registrado*: `agenda.confirmarOcupación` y `reserva.confirmar` | P-10 |
| Expirar solicitudes vencidas | Periódico: `reloj` y `provisionalesVencidas(ahora)` → *Solicitud expirada* → *Franja liberada* | P-11, D-60 |
| Bloquear sala · Retirar bloqueo | `agenda.bloquear` → *Sala bloqueada* · `agenda.retirarBloqueo` → *Franja liberada* | R-07, D-53 |
| Resolver reserva afectada | Recepción u operaciones elige reasignar, reprogramar o reembolsar | P-09 |
| Registrar exceso de tiempo | `agenda.verificarExceso` → *Horas extra registradas* o desalojar | R-06, INV-D5 |

---

## 6. Dependencias

- El **dominio** no depende de nada externo.
- Los **application services** dependen de interfaces: repositorios, `EstadoDelMiembro`, `AforoDeSala`, `PrecioDeFranja`, `Reloj` y el publicador de eventos. La **infraestructura** las implementa (ACL y cola hacia Notificaciones, [07-integracion.md](07-integracion.md)).
- Los aggregates **no se conocen**: ocupar es una llamada en proceso desde el application service; liberar y avisar a la lista de espera van por eventos en proceso. Cada aggregate se guarda en su propia transacción.
- Cada bounded context es un **módulo del monolito**, con su dominio, su aplicación y su infraestructura; el dominio no depende de las otras capas.

---

## 7. Flujos

| Flujo | Secuencia |
|---|---|
| **1 · Reservar una sala** | Controller → **SolicitarReserva** → ACL (aforo ③, al día ⑤) → **AgendaDelRecurso.ocupar** (Disponibilidad). Ocupada → **Reserva.confirmar** (+ precio vigente ④) → *Reserva confirmada* → Notificaciones, Membresías y planes, Control de acceso físico. No disponible → *Solicitud rechazada* (aviso por Notificaciones) → **ListaDeEspera.añadir** → *Añadido a la lista de espera*. |
| **2 · Cancelar una reserva** | Controller → **CancelarReserva** → *Reserva cancelada* (+ *Penalización aplicada* si faltan menos de 4 h) → **LiberarFranja** (Disponibilidad) → *Franja liberada* → **AvisarListaDeEspera** (Reserva) → *Lista de espera notificada* → Notificaciones (cola). |

---

## 8. Decisiones de esta etapa

| ID | Decisión |
|---|---|
| D-57 | Franja ocupada al solicitar: la solicitud se rechaza y la persona se añade a la lista de espera. |
| D-58 | Componentes en pseudocódigo neutro; lenguaje y estructura de carpetas pendientes hasta el prototipo; repositorios en memoria. |
| D-59 | Ningún domain service: cada regla tiene dueño. |
| D-60 | Expirar solicitudes vencidas: application service periódico, solo en la tabla. |
