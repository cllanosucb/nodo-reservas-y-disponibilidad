# 01 · Mapa de subdominios — Nodo

> Caso Nodo (coworking y eventos). Área del grupo: **Reservas y disponibilidad**.

## ¿Por qué nos eligen?
Los clientes eligen Nodo porque **siempre encuentran un espacio disponible y reservarlo es rápido, sin solapes ni sorpresas**. Esa promesa define el core.

## Clasificación

| Subdominio | Tipo | Justificación | Construir / integrar |
|---|---|---|---|
| **Reservas y disponibilidad** | Core | Es la promesa de valor: disponibilidad fiable y reserva sin fricción. Si falla, Nodo pierde frente a otro coworking. | Construir (propio, máxima inversión) |
| Membresías y planes | Supporting | Define quién puede reservar (miembro al día, horas incluidas). Todo coworking tiene planes: es necesario, pero no diferencia. | Construir simple |
| Tarifas y precios | Supporting | Valora horas extra y penalizaciones. Son reglas propias de Nodo, pero no son la razón de elegirlo. | Construir simple |
| Eventos | Supporting | Usa los mismos espacios, pero con otro ciclo (depósito + aprobación de dirección) y menor frecuencia. | Construir simple |
| Espacios y recursos | Supporting | Catálogo de salas, escritorios y salón. Es el dato maestro que Reservas consume. | Construir simple (catálogo) |
| Notificaciones | Generic | Es transversal: confirmación, recordatorio y aviso de sala liberada. Es un problema ya resuelto. | Integrar un proveedor de email/WhatsApp |
| Facturación y cobros | Generic | Facturar y cobrar es igual en cualquier negocio. | Integrar un sistema de facturación/pasarela |
| Acceso físico | Generic | Puertas y credenciales; depende del hardware. | Integrar una solución de control de acceso |

## Nuestra área: Reservas y disponibilidad (Core)
Al analizarla aparecen **dos bounded contexts**: **Disponibilidad** (franjas, margen, bloqueos) y **Reserva** (solicitud, confirmación, cancelación, lista de espera). La justificación está en `decisiones/0001-por-que-esta-frontera.md`.

## Casos límite
- **¿El core es Reservas o Membresías?** Es Reservas. La membresía es un modelo comercial que cualquiera replica; lo que el cliente percibe es encontrar sala y reservar sin solapes. Membresías solo aporta un dato a Reservas: si el miembro está al día y cuántas horas incluidas tiene.
- **¿Eventos es core?** No. Comparte salas, pero su "confirmar" exige depósito y aprobación de dirección. Tiene otro proceso y otro lenguaje, así que es supporting y va en un contexto aparte.
- **¿Qué recursos entran en Reservas?** La sala de reunión y el escritorio flexible. El escritorio fijo queda fuera porque es una asignación, no una reserva. El salón de eventos también queda fuera porque pertenece a Eventos.

## Expertos del dominio
| Experto | Qué conoce |
|---|---|
| Recepción | Reservas asistidas, bloqueos, cancelaciones |
| Responsable de operaciones | Margen entre reservas, mantenimiento |
| Dirección | Aprobación de eventos, política comercial |