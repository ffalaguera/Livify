# Modelo de datos de Livify

## 1. Funcionalidades
- (propietario) Registrar una o varias propiedades.
- (propietario) Registrar uno o varios inquilinos en una propiedad.
- (propietario) Registrar facturas.
- Gestionar los cobros automáticamente: los inquilinos pagan desde la app y al propietario se le actualiza el estado de los cobros.
- (inquilino) Registrar una o varias incidencias en su propiedad.
- (inquilino) Hacer pagos.
- (propietario) Ver gráficas de cada propiedad: evolución de la renta por años y gasto mensual de cada suministro.

## 2. Entidades
- **users**: personas que utilizan la app; pueden ser propietarios o inquilinos.
- **properties**: viviendas que un propietario pone en alquiler.
- **rooms**: son habitaciones que el propietario pone en alquiler cuando quiere que el contrato sea por habitaciones. pertenecen a una propiedad.
- **leases**: contrato de alquiler, puede ser completo o por habitaciones. los completos pueden tener varios inquilinos, los por habitacion solo un inquilino.
- **charges**: la deuda que los inquilinos deben pagar.
- **payments**: el dinero que pagan los inquilinos al propietario.
- **invoices**: facturas que registra el propietario. pueden ser de luz, agua, gas, etc...
- **issues**: incidencias que un inquilino registra y notifica al propietario
- **notifications**: notificaciones que le pueden llegar al propietario o al inquilino.
- **companies**: Listado de compañias de suministros que operan en España, algunas son colaboradoras de Livify. El propietario elige de esta lista al registrar un suministro.
- **utilities**: Contratos de suministro de cada propiedad, que servicio es y con que compañia esta contratado.
- **rent_changes**: historial de rentas: guarda la renta inicial de cada contrato y cada actualización anual. Sirve para la grafica
- **rent_indexes**: Valores del indice oficial que limita cuanto se puede subir la renta cada año.
- **company_services**: servicios que ofrece cada compañía (luz, agua, gas, internet). Sirve para filtrar la lista de compañías según el suministro.
- **provinces**: las 52 provincias de España (50 provincias más Ceuta y Melilla) con su código oficial, que coincide con los dos primeros dígitos del código postal.

## 3. Reglas de negocio

### Contratos
- Un contrato puede ser completo o por habitaciones.
- Una propiedad no puede tener a la vez un contrato completo y contratos por habitaciones.
- En un contrato completo, el propietario asigna a uno de los inquilinos como encargado de los pagos, y puede cambiarlo cuando quiera.
- En un contrato por habitaciones, cada inquilino paga lo suyo por separado.
- Solo el encargado tiene pasarela de pago; si un inquilino no es el encargado, no la tiene.
- Si se cambia al encargado, el nuevo encargado ve y paga también los cobros pendientes.

### Facturas
- Las facturas se reparten entre los inquilinos activos al registrarla, de forma proporcional a los dias que han estado dentro del periodo.
- Si un inquilino se va a mitad de mes, la factura la asumen los inquilinos que se quedan. La parte del que se va la arreglan entre ellos en persona; la aplicación no tiene función para esto.
- Si un inquilino entra a mitad de mes, paga la parte proporcional de los días que lleva en ese mes. El día de entrada cuenta como día completo.
- Se cuentan los días reales de cada periodo (febrero incluido).
- En el dashboard, todos los inquilinos (contrato completo o por habitaciones) ven la factura total y la cuantía que les corresponde.
- En el contrato completo, la factura la paga el encargado. En el de habitaciones, cada inquilino paga su parte.
- Los céntimos que sobren o falten al redondear los asume el encargado (contrato completo) o el inquilino más antiguo (contrato por habitaciones).

### Inquilinos
- Un inquilino no puede registrarse por su cuenta; necesita que un propietario lo registre.
- Al borrar un inquilino se borran sus datos personales y su contrato. Se conservan, sin vincularlos a ninguna persona, sus cobros, sus pagos y el historial de rentas, porque el propietario los necesita
para declarar sus ingresos.
- Se conservan sus incidencias, sus notificaciones se borran.

### Rentas
- Todos los contratos tienen su renta. En los completos es siempre la de la propiedad, y se actualizan juntas.
- La renta se puede actualizar una vez al año, contando desde la fecha de inicio del contrato.
- Se guarda el historial de rentas de cada contrato, empezando por la renta inicial.
- El historial apunta a la propiedad(y a la habitación, si la hay), para que se conserve aunque se borre el contrato y para poder calcular la evolución del precio de la vivienda.
- En los contratos completos, la nueva renta no puede superar el límite del índice oficial. Los de habitaciones no tienen límite, pero también se actualizan una vez al año.
- Se guardan el IRAV y el IPC. Los contratos completos firmados a partir del 26 de mayo de 2023 usan el IRAV, y los anteriores el IPC. La fecha de inicio del contrato decide cuál.
- Se usa el último valor publicado en el momento de actualizar la renta.
- Cuando un contrato cumple año, la app muestra el índice aplicable y sugiere la nueva renta. El propietario decide la renta final.
- Si el índice es negativo, la app sugiere la bajada correspondiente, y el propietario decide si la aplica.
- Se avisa al inquilino de la nueva renta con al menos un mes de antelación.

### Propiedad
- tipos: piso, casa, estudio y ático.
- La dirección se pide desglosada (calle, numero, piso, puerta, codigo postal, ciudad y provincia) y se muestra junta.
- El modo de alquiler (completo o por habitaciones) lo fija la propiedad, y solo se puede cambiar si no tiene contratos activos.
- El numero de habitaciones de la propiedad es informativo, las habitaciones que se alquilan se registran en rooms.
- Los estudios solo se alquilan completos.
- Una propiedad no se borra: se archiva (venta, uso propio o dejar de alquilarla) y se puede desarchivar.

### Suministros
- El propietario elige la compañia de una lista filtrada por el tipo de suministro. Si no esta, elige "Otra" y el suministro queda sin compañia asignada.
## 4. Campos de cada entidad

### users
| Campo | Tipo | Obligatorio | Notas |
|---|---|---|---|
| id | UUID | sí | se genera solo |
| full_name | texto | sí | máximo 100 caracteres |
| email | texto | sí | único; máximo 255 caracteres; se usa para iniciar sesión |
| password_hash | texto | sí | hash BCrypt, nunca la contraseña en claro |
| phone | texto | no | máximo 20 caracteres; admite +34 y espacios |
| role | lista de valores | sí | OWNER, TENANT |
| lease_id | UUID | solo inquilinos | contrato al que pertenece; vacío en propietarios |
| is_payer | sí/no | sí | encargado de pagos de su contrato; en propietarios siempre "no" |
| must_change_password | sí/no | sí | por defecto "no"; "sí" cuando el propietario crea al inquilino |
| created_at | fecha y hora | sí | se rellena sola al crear |
| updated_at | fecha y hora | sí | se actualiza sola al modificar |

**Reglas**
- Un OWNER no puede tener lease_id ni is_payer = sí.
- Un TENANT tiene que tener lease_id.
- Solo puede haber un encargado por contrato.

### companies
| Campo | Tipo | Obligatorio | Notas |
|---|---|---|---|
| id | UUID | sí | se genera solo |
| name | texto | sí | único; máximo 100 caracteres |
| is_partner | sí/no | sí | colaboradora de Livify; por defecto "no" |
| created_at | fecha y hora | sí | se rellena sola al crear |
| updated_at | fecha y hora | sí | se actualiza sola al modificar |

### company_services
| Campo | Tipo | Obligatorio | Notas |
|---|---|---|---|
| company_id | UUID | sí | apunta a companies |
| service_type | lista de valores | sí | ELECTRICITY, WATER, GAS, INTERNET |

**Reglas**
- No se puede repetir la misma pareja compañía + servicio.
- Si se borra una compañía, se borran sus servicios.

### rent_indexes
| Campo | Tipo | Obligatorio | Notas |
|---|---|---|---|
| id | UUID | sí | se genera solo |
| index_type | lista de valores | sí | IRAV, IPC |
| period | fecha | sí | mes al que corresponde (se guarda el día 1 del mes) |
| rate | número decimal | sí | porcentaje con 2 decimales (ej: 2,47) |
| published_at | fecha | sí | día en que el INE lo publicó |
| created_at | fecha y hora | sí | se rellena sola al crear |

**Reglas**
- No puede haber dos valores del mismo índice para el mismo mes.
- Los valores se descargan del INE automáticamente cada mes.

### properties
| Campo | Tipo | Obligatorio | Notas |
|---|---|---|---|
| id | UUID | sí | se genera solo |
| owner_id | UUID | sí | apunta a users; propietario de la vivienda |
| property_type | lista de valores | sí | FLAT (piso), HOUSE (casa), STUDIO (estudio), PENTHOUSE (ático) |
| street | texto | sí | nombre de la vía, ej: "Calle Colón"; máximo 150 caracteres |
| street_number | texto | sí | texto porque puede ser "42B" o "s/n"; máximo 10 caracteres |
| floor | texto | no | ej: "3", "Bajo", "Entresuelo"; vacío en casas |
| door | texto | no | ej: "A", "12"; vacío en casas |
| postal_code | texto | sí | texto de 5 caracteres para no perder el 0 inicial (ej: 08001) |
| city | texto | sí | máximo 100 caracteres |
| province_code | texto (2) | sí | apunta a provinces; debe coincidir con los 2 primeros dígitos de postal_code |
| bedrooms | número entero | sí | informativo; 0 o más (un estudio puede tener 0) |
| bathrooms | número entero | sí | 1 o más |
| square_meters | número decimal | sí | mayor que 0; 2 decimales |
| rental_mode | lista de valores | sí | FULL (completo), ROOMS (por habitaciones) |
| monthly_rent | dinero | solo si FULL | renta de la vivienda completa; vacío si ROOMS |
| status | lista de valores | sí | ACTIVE (activa), ARCHIVED (archivada); por defecto ACTIVE |
| archived_at | fecha y hora | no | cuándo se archivó; solo si ARCHIVED |
| created_at | fecha y hora | sí | se rellena sola al crear |
| updated_at | fecha y hora | sí | se actualiza sola al modificar |

**Reglas**
- owner_id debe ser un usuario con rol OWNER.
- Si rental_mode = FULL, monthly_rent es obligatorio y mayor que 0. Si es ROOMS, va vacío.
- rental_mode solo se puede cambiar si la propiedad no tiene contratos activos.
- En contratos completos, al actualizar monthly_rent se actualiza también la renta del contrato activo.
- Los dos primeros dígitos de postal_code deben corresponder a la provincia.
- Si property_type = STUDIO, rental_mode solo puede ser FULL.
- Solo se puede archivar si no tiene contratos activos (primero hay que finalizarlos).
- Al archivarla, sus suministros ACTIVE y PENDING pasan a ENDED.
- Una propiedad archivada no admite contratos nuevos ni facturas nuevas, pero su historial se puede consultar.
- Se puede desarchivar en cualquier momento; la app propone reactivar los suministros que tenía.

### provinces
| Campo | Tipo | Obligatorio | Notas |
|---|---|---|---|
| code | texto (2) | si | clave primaria; codigo oficial (ej: 46 = Valencia) |
| name | texto | si | único; maximo 50 caracteres |

### rooms
| Campo | Tipo | Obligatorio | Notas |
|---|---|---|---|
| id | UUID | si | se genera solo |
| property_id | UUID | si | apunta a properties |
| room_number | numero entero | si | 1 o mas; unico dentro de la propiedad |
| created_at | fecha y hora | si | se rellena sola al crear |
| updated_at | fecha y hora | si | se actualiza sola al modificar |

**Reglas**
- Solo se puede crear habitaciones en propiedades con rental_mode = ROOMS.
- No se puede repetir el mismo número de habitación en una propiedad.
- No hay límite de habitaciones respecto a bedrooms; el propietario puede crear las que quiera y borrar las que no tengan contrato activo.
- Una habitación solo se puede borrar si no tiene un contrato activo.
- Al borrar una habitación, su historial de rentas se conserva ligado a la propiedad, sin indicar la habitación.

### utilities
| Campo | Tipo | Obligatorio | Notas |
|---|---|---|---|
| id | UUID | sí | se genera solo |
| property_id | UUID | sí | apunta a properties |
| service_type | lista de valores | sí | ELECTRICITY, WATER, GAS, INTERNET |
| company_id | UUID | no | apunta a companies; vacío = "Otra" |
| status | lista de valores | si | PENDING (registrado, aun no en uso), ACTIVE (en uso), ENDED (finalizado) |
| ended_at | fecha | no | fecha en que se finalizo; solo si status = ENDED |
| created_at | fecha y hora | sí | se rellena sola al crear |
| updated_at | fecha y hora | sí | se actualiza sola al modificar |

**Reglas**
- Solo puede haber un suministro ACTIVE de cada tipo por propiedad y como mucho uno PENDING.
- Al cambiar de compañía se crea un suministro nuevo. Si ya llegó la última factura de la compañía actual, la actual pasa a ENDED y la nueva a ACTIVE. Si no, la nueva queda PENDING hasta que llegue la última factura. Las facturas antiguas no se modifican.
- La compañía elegida debe ofrecer ese tipo de servicio (según company_services).
- Las facturas solo se registran en suministros ACTIVE.
- Al registrar una factura marcada como “última factura”, el suministro activo pasa a ENDED y el pendiente (si lo hay) pasa a ACTIVE.
- Un suministro solo se puede borrar si no tiene facturas; si las tiene, se finaliza (ENDED).
- Si se borra su compañía, el suministro pasa a "Otra" (company_id vacío).

### leases
| Campo | Tipo | Obligatorio | Notas |
|---|---|---|---|
| id | UUID | sí | se genera solo |
| property_id | UUID | sí | apunta a properties |
| room_id | UUID | solo si ROOM | apunta a rooms; vacío si FULL |
| lease_type | lista de valores | sí | FULL, ROOM |
| start_date | fecha | sí | inicio; decide el aniversario y si se aplica IRAV o IPC |
| end_date | fecha | sí | fin pactado; se amplía si se prorroga |
| monthly_rent | dinero | sí | renta actual; en FULL, igual a la de la propiedad |
| deposit_amount | dinero | sí | importe de la fianza |
| rent_update_agreed | sí/no | sí | si el contrato permite actualizar la renta; si "no", la app no sugiere actualizaciones |
| created_at | fecha y hora | sí | se rellena sola al crear |
| updated_at | fecha y hora | sí | se actualiza sola al modificar |

**Reglas**
- lease_type debe coincidir con el rental_mode de la propiedad.
- Si lease_type = ROOM, room_id es obligatorio y el contrato tiene un único inquilino.
- Los cobros de renta se generan el día 1 de cada mes y vencen el día 5 (igual para todos los contratos).
- Se avisa al propietario X meses antes de end_date (X es un parámetro configurable según la ley).
- Al llegar end_date, la app pregunta al propietario si el contrato ha finalizado o se ha prorrogado. Si se prorroga, end_date se amplía un año.
- Mientras el propietario no conteste, el contrato sigue activo y la app le recuerda la pregunta. La app nunca finaliza un contrato por su cuenta.
- El contrato finaliza cuando se van todos sus inquilinos, o cuando el propietario confirma que ha finalizado.
- Al finalizar, la app muestra la fianza a devolver.

### rent_changes
| Campo | Tipo | Obligatorio | Notas |
|---|---|---|---|
| id | UUID | sí | se genera solo |
| property_id | UUID | sí | apunta a properties |
| room_id | UUID | no | apunta a rooms; vacío si el contrato es completo o si la habitación se borró |
| lease_id | UUID | no | apunta a leases; se vacía si el contrato se borra |
| change_type | lista de valores | sí | INITIAL (renta inicial), UPDATE (actualización anual) |
| effective_date | fecha | sí | desde cuándo se aplica la renta nueva |
| previous_rent | dinero | no | vacío si change_type = INITIAL |
| new_rent | dinero | sí | renta aplicada |
| index_type | lista de valores | no | IRAV, IPC; vacío si INITIAL o si no hay límite (habitaciones) |
| index_rate | número decimal | no | valor del índice usado (ej: 2,47) |
| suggested_rent | dinero | no | renta máxima que sugirió la app |
| created_at | fecha y hora | sí | se rellena sola al crear |

**Reglas**
- Al crear un contrato se guarda una fila INITIAL con su renta.
- Cada actualización anual guarda una fila UPDATE.
- Si se borra el contrato o la habitación, la fila se conserva (lease_id o room_id se vacían) y sigue ligada a la propiedad.
- Las filas no se modifican una vez creadas (es un historial).

### invoices
| Campo | Tipo | Obligatorio | Notas |
|---|---|---|---|
| id | UUID | sí | se genera solo |
| utility_id | UUID | sí | apunta a utilities; debe estar ACTIVE al registrarla |
| amount | dinero | sí | importe total; mayor que 0 |
| period_start | fecha | sí | inicio del periodo facturado |
| period_end | fecha | sí | fin del periodo; igual o posterior a period_start |
| due_date | fecha | sí | orientativa: 15 días desde el registro; sin recargo |
| is_final | sí/no | sí | última factura de esa compañía; por defecto "no" |
| notes | texto | no | notas del propietario; máximo 500 caracteres |
| created_at | fecha y hora | sí | se rellena sola al crear |
| updated_at | fecha y hora | sí | se actualiza sola al modificar |

**Reglas**
- Al registrarla se generan los cobros según las reglas de reparto.
- Si is_final = sí, el suministro pasa a ENDED y el pendiente (si lo hay) a ACTIVE.
- Al pasar due_date sin pagar, el cobro queda retrasado (se calcula), se avisa al propietario y se envían recordatorios al inquilino.
- El propietario puede editarla o borrarla:
  - Los cobros no pagados se recalculan, o pasan a VOID si se borra la factura.
  - Si el importe baja y alguien ya pagó, se le devuelve la diferencia (REFUND en payments).
  - Si el importe sube y alguien ya pagó, se le genera un cobro ADJUSTMENT por la diferencia.

### charges
| Campo | Tipo | Obligatorio | Notas |
|---|---|---|---|
| id | UUID | sí | se genera solo |
| property_id | UUID | sí | apunta a properties; se conserva siempre |
| room_id | UUID | no | apunta a rooms; vacío si el contrato es completo o si la habitación se borró |
| lease_id | UUID | no | apunta a leases; se vacía si el contrato se borra |
| charge_type | lista de valores | sí | RENT (renta), INVOICE (parte de una factura), ADJUSTMENT (diferencia al subir una factura ya pagada) |
| invoice_id | UUID | no | apunta a invoices; solo en INVOICE y ADJUSTMENT; se vacía si la factura se borra |
| description | texto | sí | concepto, ej: "Renta noviembre 2026" o "Luz – Iberdrola, octubre 2026"; se guarda al crear; máximo 200 caracteres |
| amount | dinero | sí | importe; mayor que 0 |
| due_date | fecha | sí | renta: día 5 del mes; factura y ajuste: 15 días desde que se crea |
| status | lista de valores | sí | OPEN (pendiente), PAID (pagado), VOID (anulado), UNCOLLECTIBLE (dado por perdido) |
| paid_at | fecha y hora | no | cuándo se pagó; solo si status = PAID |
| created_at | fecha y hora | sí | se rellena sola al crear |
| updated_at | fecha y hora | sí | se actualiza sola al modificar |

**Reglas**
- Los cobros de renta se generan el día 1 de cada mes y vencen el día 5.
- Solo el encargado del contrato puede pagar, y siempre el importe íntegro.
- "Retrasado" no es un estado: es un cobro OPEN cuya due_date ya pasó (se calcula).
- Si se borra una factura, sus cobros OPEN pasan a VOID. Los PAID siguen PAID, y la devolución se registra en payments.
- Si una factura sube y ya estaba pagada, se crea un cobro ADJUSTMENT por la diferencia.
- UNCOLLECTIBLE solo lo puede marcar el propietario.
- Al borrar un inquilino, sus cobros se conservan sin datos personales (lease_id se vacía).
- La parte informativa de cada inquilino en un contrato completo no se guarda: se calcula al mostrar el dashboard.

### payments
| Campo | Tipo | Obligatorio | Notas |
|---|---|---|---|
| id | UUID | sí | se genera solo |
| charge_id | UUID | sí | apunta a charges; cobro que salda o del que se devuelve dinero |
| payment_type | lista de valores | sí | PAYMENT (pago), REFUND (devolución) |
| amount | dinero | sí | mayor que 0; en PAYMENT, igual al importe del cobro |
| method | lista de valores | sí | STRIPE, TRANSFER, CASH, BIZUM, OTHER |
| paid_at | fecha y hora | sí | cuándo se pagó o se devolvió |
| notes | texto | no | notas del propietario; máximo 500 caracteres |
| created_at | fecha y hora | sí | se rellena sola al crear |

**Reglas**
- Un cobro solo puede tener un PAYMENT (el pago es íntegro).
- La suma de los REFUND de un cobro no puede superar su importe.
- Hasta que haya Stripe, los pagos y las devoluciones los registra el propietario a mano.
- Los pagos no se modifican una vez creados; si hay un error, se corrige con una devolución.
- Al borrar un inquilino, sus pagos se conservan sin datos personales (siguen ligados al cobro, y este a la propiedad).
- El identificador de Stripe se añadirá con la épica de Stripe.

### issues
| Campo | Tipo | Obligatorio | Notas |
|---|---|---|---|
| id | UUID | sí | se genera solo |
| property_id | UUID | sí | apunta a properties; se conserva siempre |
| room_id | UUID | no | apunta a rooms; vacío si es contrato completo o si la habitación se borró |
| lease_id | UUID | no | contrato del inquilino que la abrió; se vacía si se borra |
| issue_type | lista de valores | sí | PLUMBING (fontanería), ELECTRICITY (electricidad), APPLIANCES (electrodomésticos), HVAC (calefacción y climatización), LOCKSMITH (cerrajería), DAMP (humedades), OTHER (otros) |
| title | texto | sí | máximo 150 caracteres |
| description | texto | sí | máximo 2.000 caracteres |
| priority | lista de valores | sí | LOW, MEDIUM, HIGH, URGENT |
| status | lista de valores | sí | OPEN (abierta), IN_PROGRESS (en curso), RESOLVED (resuelta) |
| planned_resolution_date | fecha | no | fecha prevista; sin límite futuro; no anterior a la apertura |
| resolved_at | fecha | no | fecha real; solo si RESOLVED; entre la apertura y hoy |
| created_at | fecha y hora | sí | se rellena sola al crear |
| updated_at | fecha y hora | sí | se actualiza sola al modificar |

**Reglas**
- La abre el inquilino; el propietario cambia el estado y las fechas.
- Las incidencias resueltas se conservan como historial de la propiedad.
- Al borrar al inquilino, sus incidencias se conservan sin datos personales (lease_id se vacía).

### notifications
| Campo | Tipo | Obligatorio | Notas |
|---|---|---|---|
| id | UUID | sí | se genera solo |
| recipient_id | UUID | sí | apunta a users; si se borra el usuario, se borran sus notificaciones |
| notification_type | lista de valores | sí | ver lista abajo |
| title | texto | sí | máximo 150 caracteres |
| message | texto | sí | máximo 500 caracteres |
| related_type | lista de valores | no | ISSUE, CHARGE, INVOICE, LEASE, PAYMENT |
| related_id | UUID | no | id del elemento, para abrirlo al pulsar |
| read_at | fecha y hora | no | vacío = no leída |
| email_sent_at | fecha y hora | no | cuándo se envió por email; vacío si no se envió |
| created_at | fecha y hora | sí | se rellena sola al crear |

**Tipos** (con ✉ los que también se envían por email)
- Propietario: NEW_ISSUE, PAYMENT_RECEIVED, CHARGE_OVERDUE ✉, LEASE_ENDING_SOON ✉, LEASE_END_REACHED ✉, RENT_REVIEW_DUE ✉
- Inquilino: NEW_INVOICE, CHARGE_REMINDER ✉, ISSUE_STATUS_CHANGED, RENT_UPDATED ✉, PAYMENT_FAILED ✉

**Reglas**
- Al pulsarla, se marca como leída (read_at) y se abre el elemento relacionado.
- Las leídas se borran automáticamente a los 2 días de leerse. Las no leídas no se borran nunca.
- Al borrar un usuario, se borran sus notificaciones.

## 5. Relaciones

| Relación | Tipo | Campo | Al borrar el "padre" |
|---|---|---|---|
| Propietario → propiedades | 1:N | properties.owner_id | solo al borrar la cuenta: todo en cascada |
| Propiedad → habitaciones | 1:N | rooms.property_id | la propiedad no se borra, se archiva |
| Propiedad → suministros | 1:N | utilities.property_id | la propiedad no se borra, se archiva |
| Propiedad → contratos | 1:N | leases.property_id | la propiedad no se borra, se archiva |
| Propiedad → cobros, historial de rentas, incidencias | 1:N | property_id | la propiedad no se borra, se archiva |
| Habitación → contratos | 1:N | leases.room_id | no se puede borrar con contrato activo |
| Habitación → cobros, historial de rentas, incidencias | 1:N | room_id | se vacía |
| Contrato → inquilinos | 1:N | users.lease_id | al finalizar el contrato se borran sus inquilinos |
| Contrato → cobros, historial de rentas, incidencias | 1:N | lease_id | se vacía |
| Compañía → servicios | 1:N | company_services.company_id | se borran |
| Compañía → suministros | 1:N | utilities.company_id | se vacía (pasa a "Otra") |
| Suministro → facturas | 1:N | invoices.utility_id | solo se puede borrar sin facturas |
| Factura → cobros | 1:N | charges.invoice_id | se vacía; los OPEN pasan a VOID |
| Cobro → pagos | 1:N | payments.charge_id | los cobros no se borran (solo VOID) |
| Usuario → notificaciones | 1:N | notifications.recipient_id | se borran |
| Provincia → propiedades | 1:N | properties.province_code | no se borran |

**Borrado de la cuenta de un propietario**
- Primero tiene que finalizar todos sus contratos activos.
- Después se borra todo lo suyo en cascada, con un aviso previo y la opción de descargar su historial.

## 6. Diagrama

## 7. Decisiones descartadas y mejoras futuras
- No se comparte el historial de morosidad entre propietarios: es un dato personal y lo impide el RGPD.
- El Stripe se implementara despues del nucleo. primero funcionara con "marcar como pagado" para no bloquear el resto del desarrollo.
- El prorrateo de quien se va se resuelve fuera de la app. asi se simplifica el calculo del reparto.
- No hay tabla de roles: el rol es un campo de users, porque solo hay dos roles fijos.
- Cobro automatico en tarjeta o domiciliacion: queda como mejora futura: de momento, el encargado paga cuando quiere.
- Borrado de inquilinos: eligió anonimizar(borrar los datos personales y conservar los economicos), en lugar del borrado fisico completo,
para que el propietario pueda declarar sus ingresos.
- Locales y garajes descartados por ahora. porque no tienen habitaciones ni admiten alquiler por habitaciones.
- Estado de la cuenta (activa, suspendida, impago…): descartado por ahora. Los inquilinos se borran al dejar la propiedad, y para los propietarios no hay funcionalidad que lo necesite. Se valorará si Livify cobra suscripciones.
- Avatar o foto de perfil: descartado, no esta en las funcionalidades.
- stripe_account_id (propietarios): se añadirá con la épica de Stripe. 
- Ids: UUID en todas las tablas para que no se puedan adivinar en las URLs.
- Galería de fotos de entrada y salida (inventario): mejora futura. Las fotos irían ligadas al contrato, con momento CHECK_IN o CHECK_OUT, y se guardarían en S3.
- Stripe: revisar el flujo de pago y la experiencia del inquilino (redireccion o formulario incrustado) antes de implementarlo
- Archivos (fotos de incidencias, documentos de facturas, galería de inventario): aplazados hasta tener S3, por coste. Se guardarán en S3 privado con las medidas de seguridad acordadas, y en la base de datos solo la referencia (file_key).
- Parámetros configurables: meses de preaviso del fin de contrato (según la LAU, 4 o 6) y días hasta borrar las notificaciones leídas (2).
- Ids: UUID en todas las tablas excepto provinces, que usa su codigo oficial.