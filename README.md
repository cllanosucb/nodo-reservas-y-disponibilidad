# Nodo · Reservas y disponibilidad

> **Curso:** Fundamentos de Domain-Driven Design · **Caso:** Nodo (coworking y eventos)
> **Área del grupo:** Reservas y disponibilidad (core)
> **Estado:** diseño estratégico, integración, modelo y componentes terminados — prototipo pendiente

---

## Integrantes y roles

| Integrante | Rol |
|---|---|
| Carlos Llanos Rodriguez | Subdominios, glosario y modelo |
| David Almaraz | EventStorming, domain stories y componentes |
| Jhoann Guerrero | Context map, integración, reglas e invariantes y decisiones |

---

## 1. Qué es Nodo y cuál es nuestra área

**Nodo** es un espacio de coworking en una ciudad mediana. Ocupa dos plantas: una sala diáfana con escritorios (flexibles y fijos), varias salas de reunión cerradas de distintos tamaños y un salón grande para charlas y eventos. Vive de tres fuentes de ingreso: las membresías, el alquiler puntual de salas por horas y el alquiler del salón para eventos.

**Nuestra área, Reservas y disponibilidad,** decide qué sala está libre y quién se queda con ella, y asigna el equipo portátil (proyector, televisores) que se pide junto con la sala. Es el **subdominio core**:

> Los clientes eligen Nodo porque **siempre encuentran un espacio disponible y reservarlo es rápido, sin solapes, sin sobrecupo y sin sorpresas**.

| Subdominio | Tipo |
|---|---|
| **Reservas y disponibilidad** | Core |
| Membresías y planes · Tarifas y precios · Agenda de eventos · Espacios y recursos | Supporting |
| Facturación y cobros · Control de acceso físico · Notificaciones (transversal) | Generic |

Detalle: [docs/01-subdominios.md](docs/01-subdominios.md) · Decisión: [decisiones/0002-clasificacion-del-area.md](decisiones/0002-clasificacion-del-area.md)

---

## 2. Bounded contexts identificados y por qué

El área se divide en **dos bounded contexts**, los dos core:

| Bounded context | Qué modela | Pregunta que responde |
|---|---|---|
| **Disponibilidad** | Franjas libres de cada sala y de cada unidad de equipo portátil, margen de 15 min en las salas y bloqueos por mantenimiento o limpieza | «¿Está libre la Sala B el jueves de 10:00 a 11:00, y hay un proyector libre?» |
| **Reserva** | El ciclo de vida de lo que pide una persona (miembro o visitante): solicitud, confirmación o rechazo, reserva asistida, cancelación, penalización, horas extra y lista de espera | «¿Puede Ana quedarse con esa franja, y qué pasa si cancela?» |

**Señales que justifican la frontera:**

1. **Polisemia:** *bloquear* (Nodo saca una franja, sin titular ni cobro) no es *cancelar* (el titular anula su reserva, con posible cobro).
2. **Reglas que cambian por su cuenta:** el margen y los bloqueos responden a la operación; la anticipación, la penalización y el pago del visitante, a la política comercial.
3. **Actores distintos:** bloquean recepción, administración o dirección; reservan el miembro, el visitante y recepción.
4. **Preguntas distintas:** «¿qué está libre?» frente a «¿esta persona puede quedarse con la franja?».

Detalle: [decisiones/0001-por-que-esta-frontera.md](decisiones/0001-por-que-esta-frontera.md)

---

## 3. Glosario del lenguaje ubicuo

| Contexto | Términos principales |
|---|---|
| **Disponibilidad** | Sala de reunión, Recurso reservable, Equipo portátil, Franja, Margen, Bloqueo, Ocupación, Disponibilidad |
| **Reserva** | Solicitud de reserva, Reserva, Confirmada, Reserva provisional, Plazo de pago, Equipo pedido, Anticipación, Reserva asistida, Miembro al día, Visitante, Invitado, Aforo, Cancelación, Cancelación tardía, Penalización, Horas extra, Salida anticipada, Reserva afectada, Lista de espera |

**Polisemias principales:**

| Palabra | En nuestra área | En otro contexto |
|---|---|---|
| **Reserva** | Franja de una sala de reunión asignada a una persona | Agenda de eventos: alquiler del salón · Facturación y cobros: cargo por cobrar |
| **Confirmar** | Franja disponible y miembro al día (o visitante con pago adelantado) | Agenda de eventos: depósito y aprobación de la dirección u operaciones |
| **Cliente** | Ocupante de la franja | Facturación y cobros: pagador · Control de acceso físico: persona identificada en la cerradura |

Detalle: [docs/02-glosario.md](docs/02-glosario.md)

---

## 4. Context map del área y sus vecinos

![Context map de Reservas y disponibilidad](docs/img/05-context-map.png)

| # | Relación | Qué cruza | Patrón | Mecanismo |
|---|---|---|---|---|
| ① | Disponibilidad ↔ Reserva | Ocupar o liberar la franja · Sala bloqueada | Shared kernel (Franja e identificador de sala) | Llamada en proceso · eventos de dominio en proceso |
| ② | Espacios y recursos → Disponibilidad | Salas de reunión reservables | Customer/Supplier + ACL | Mensajería async |
| ③ | Espacios y recursos → Reserva | Aforo de cada sala | Customer/Supplier + ACL | API REST |
| ④ | Tarifas y precios → Reserva | Precio vigente de la franja · reserva mensual cotizada | Customer/Supplier + ACL | API REST |
| ⑤ | Membresías y planes → Reserva | Miembro al día · horas incluidas · horas consumidas | Customer/Supplier + ACL | API REST |
| ⑥ | Facturación y cobros → Reserva | Pago por adelantado · penalización · horas extra · reembolso | ACL | API externa |
| ⑦ ⑧ | Notificaciones → Reserva | Confirmación, recordatorio y rechazo · aviso a la lista de espera | OHS + Conformist | Cola de mensajes |
| ⑨ | Control de acceso físico → Reserva | Quién actúa y su rol · reservas del día | OHS + ACL | Token con rol · API externa |
| ⑩ | Disponibilidad → Agenda de eventos | Equipo portátil compartido con el salón | Customer/Supplier | API REST |

La flecha va del contexto de aguas arriba (define el contrato) al de aguas abajo. Con **Agenda de eventos** no hay integración por los espacios (Separate Ways): el salón es un recurso aparte; solo se comparte el equipo portátil (⑩). Cuando una línea combina dos patrones o dos mecanismos, cada uno responde a un lado o a un sentido distinto de la relación ([justificación](docs/07-integracion.md#por-qué-algunas-líneas-combinan-dos-patrones-o-dos-mecanismos)).

Detalle: [docs/05-context-map.md](docs/05-context-map.md) · Integración (cuatro capas y detalle de cada ACL): [docs/07-integracion.md](docs/07-integracion.md) · Decisión: [decisiones/0003-patrones-de-integracion.md](decisiones/0003-patrones-de-integracion.md)

---

## 5. Reglas e invariantes

**Disponibilidad** (aggregate: *Agenda del recurso*, una por recurso y día)

- **INV-D1** · Nunca dos ocupaciones se solapan en el mismo recurso (una sala o una unidad de equipo portátil).
- **INV-D2** · Siempre quedan 15 minutos de margen, también frente a un bloqueo.
- **INV-D3** · Una franja bloqueada no admite nuevas ocupaciones.
- **INV-D4** · Solo se libera una franja ocupada, y una sola vez (en una salida anticipada, solo el tramo que falta).
- **INV-D5** · El exceso de tiempo nunca invade una reserva confirmada.

**Reserva** (aggregates: *Reserva* y *Lista de espera*)

- **INV-R1** · Toda reserva confirmada tiene ocupadas en Disponibilidad su sala y el equipo pedido (cruza contextos).
- **INV-R2** · Una reserva solo pasa por estados válidos.
- **INV-R3** · La confirmación automática exige miembro al día y 24 h o más de anticipación.
- **INV-R4** · Ninguna reserva se registra con menos de 3 h de anticipación.
- **INV-R5** · Los asistentes nunca superan el aforo.
- **INV-R6** · Ninguna reserva de visitante se confirma sin pago por adelantado.
- **INV-R7** · Toda cancelación tardía genera exactamente una penalización (la franja completa).
- **INV-R8** · Toda reserva asistida registra quién la hizo y a nombre de quién.
- **INV-R9** · Una persona no está dos veces en la lista de espera de la misma franja.

Las **políticas** P-01 a P-12 («cuando… entonces…») reaccionan a eventos y no son invariantes.

Detalle: [docs/06-reglas-invariantes.md](docs/06-reglas-invariantes.md) · Origen de las reglas: [docs/03-eventstorming.md](docs/03-eventstorming.md) y [docs/04-domain-stories.md](docs/04-domain-stories.md)

---

## 6. Modelo: aggregates y domain events

Cada invariante queda protegida por un aggregate. **Franja** (inicio, fin) es el único value object compartido por los dos contextos.

| Aggregate | Contexto | Raíz e identidad | Invariantes | Domain events principales |
|---|---|---|---|---|
| **Agenda del recurso** | Disponibilidad | `AgendaDelRecurso`: un recurso (sala o unidad de equipo) en un día (recursoId + fecha), con sus ocupaciones y bloqueos | INV-D1 a INV-D5 | Sala bloqueada · Franja liberada |
| **Reserva** | Reserva | `Reserva` (reservaId), con su estado, franja, titular, asistentes y precio conservado | INV-R2 a INV-R8 | Reserva solicitada · Reserva confirmada · Solicitud rechazada · Solicitud expirada · Reserva cancelada · Penalización aplicada |
| **Lista de espera** | Reserva | `ListaDeEspera`: una por sala y franja | INV-R9 | Añadido a la lista de espera · Lista de espera notificada |

**El último cupo:** la agenda tiene versión. Si dos solicitudes compiten por la misma sala el mismo día, una gana; la otra reintenta, encuentra la franja ocupada y se rechaza o entra en la lista de espera.

Cada domain event sale de una nota del EventStorming con el mismo nombre: la tabla de trazabilidad está en [docs/08-modelo.md](docs/08-modelo.md#7-trazabilidad-con-el-eventstorming-de-la-nota-naranja-al-aggregate).

Detalle: [docs/08-modelo.md](docs/08-modelo.md)

---

## 7. Componentes

| Pieza | Decisión |
|---|---|
| **Domain services** | Ninguno: cada regla tiene dueño (`AgendaDelRecurso.ocupar`, `Reserva.confirmar`, `PolíticaDeCancelación`). |
| **Repositories** | Uno por raíz: Reserva, Agenda del recurso y Lista de espera. En el prototipo, en memoria. |
| **Factories** | Sin clases factory: `Reserva.solicitar(...)`, `AgendaDelRecurso.abrir(...)` y `ListaDeEspera.abrir(...)`. |
| **Application services** | Coordinan, no deciden. Caso principal: **SolicitarReserva**. |

**Reservar una sala, en cinco pasos:** (1) la reserva se solicita y se valida la anticipación; (2) se consultan el aforo y el estado del miembro por sus ACL; (3) la agenda **ocupa la franja** de la sala y, si se pidió equipo, de una unidad libre de cada tipo; (4) si todo quedó ocupado, la reserva se confirma con el precio vigente (o queda provisional si es un visitante); (5) si no, la solicitud se rechaza y la persona entra en la lista de espera. Cada aggregate se guarda en su propia transacción.

Detalle (pseudocódigo y flujos): [docs/09-componentes.md](docs/09-componentes.md)

---

## 8. Estado del proyecto

**Diseño estratégico, integración, modelo y componentes terminados — prototipo pendiente.**

| Artefacto | Documento | Estado |
|---|---|---|
| Mapa de subdominios | [docs/01-subdominios.md](docs/01-subdominios.md) | Terminado |
| Glosario del lenguaje ubicuo | [docs/02-glosario.md](docs/02-glosario.md) | Terminado |
| EventStorming | [docs/03-eventstorming.md](docs/03-eventstorming.md) | Terminado |
| Domain stories (as-is / to-be) | [docs/04-domain-stories.md](docs/04-domain-stories.md) | Terminado |
| Context map | [docs/05-context-map.md](docs/05-context-map.md) | Terminado |
| Reglas e invariantes | [docs/06-reglas-invariantes.md](docs/06-reglas-invariantes.md) | Terminado |
| Integración (patrones, dirección, qué cruza y mecanismo) | [docs/07-integracion.md](docs/07-integracion.md) | Terminado |
| Modelo (entidades, value objects, aggregates y domain events) | [docs/08-modelo.md](docs/08-modelo.md) | Terminado |
| Componentes (domain services, repositorios, factories y application services) | [docs/09-componentes.md](docs/09-componentes.md) | Terminado |
| Decisiones (ADR 0001 a 0003) | [decisiones/](decisiones/) | Terminado |
| Prototipo por bounded context | `src/` | Pendiente |

Los artefactos visuales están en el tablero de Miro del proyecto; cada documento muestra la imagen exportada del frame correspondiente.

---

## Estructura del repositorio

```text
nodo-reservas-y-disponibilidad/
├── README.md
├── docs/
│   ├── 01-subdominios.md
│   ├── 02-glosario.md
│   ├── 03-eventstorming.md
│   ├── 04-domain-stories.md
│   ├── 05-context-map.md
│   ├── 06-reglas-invariantes.md
│   ├── 07-integracion.md
│   ├── 08-modelo.md
│   ├── 09-componentes.md
│   └── img/
├── decisiones/
│   ├── 0001-por-que-esta-frontera.md
│   ├── 0002-clasificacion-del-area.md
│   └── 0003-patrones-de-integracion.md
└── src/
```
