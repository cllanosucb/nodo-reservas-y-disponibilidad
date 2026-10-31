# Prototipo: Nodo Cowork

El código del prototipo está en un **repositorio aparte**:

**[github.com/davidcancioalmaraz/nodo-cowork](https://github.com/davidcancioalmaraz/nodo-cowork)**

Está separado porque es una aplicación completa con su propio stack: Laravel 13 + Jetstream (Inertia + Vue), PostgreSQL 17 y Docker.

| | |
|---|---|
| **Cómo ejecutarlo** | Las instrucciones completas están en el [README del prototipo](https://github.com/davidcancioalmaraz/nodo-cowork#readme). |

## Inicio rápido

Requisito: Docker y Docker Compose.

```bash
git clone https://github.com/davidcancioalmaraz/nodo-cowork.git
```

```bash
cd nodo-cowork
```

```bash
docker compose up -d
```

La aplicación queda en http://localhost:8000. Para ingresar, usa el usuario `miembro` o `recepcion`, con cualquier contraseña. El guion de la demo está en el README del prototipo.

---

## Qué parte del diseño implementa

Cada bounded context es un módulo en `src/` del prototipo, con su dominio, su aplicación y su infraestructura. El dominio no depende de Laravel.

| Diseño (este repositorio) | Prototipo |
|---|---|
| Shared kernel ①: Franja e identificador de sala ([07-integracion.md](../docs/07-integracion.md)) | `src/Compartido/` |
| Bounded context **Disponibilidad** · aggregate **Agenda del recurso** ([08-modelo.md](../docs/08-modelo.md)) | `src/Disponibilidad/` · `AgendaDelRecurso` |
| Bounded context **Reserva** · aggregate **Reserva** ([08-modelo.md](../docs/08-modelo.md)) | `src/Reserva/` · `Reserva` |
| Application services *SolicitarReserva* y *CancelarReserva* ([09-componentes.md](../docs/09-componentes.md)) | `src/Reserva/Aplicacion/` |
| Fachadas ACL de los vecinos ([07-integracion.md](../docs/07-integracion.md)) | `src/Reserva/Aplicacion/Puertos/` · vecinos simulados en `Infraestructura/` |

**Invariantes con tests:** INV-D1 a INV-D4, INV-R1 a INV-R5, INV-R7 e INV-R8. La tabla que relaciona cada invariante con el método que la protege y su test está en el README del prototipo.

**Fuera del alcance del prototipo:** lista de espera (INV-R9), visitante con pago por adelantado (INV-R6), equipo portátil, horas extra, salida anticipada y la resolución de reservas afectadas por un bloqueo (P-09). Están diseñados en [08-modelo.md](../docs/08-modelo.md) y [09-componentes.md](../docs/09-componentes.md).
