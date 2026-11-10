# Modelo de datos de Livify

## 1. Funcionalidades
- (propietario) Registrar una o varias propiedades.
- (propietario) Registrar uno o varios inquilinos en una propiedad.
- (propietario) Registrar facturas.
- Gestionar los cobros automáticamente: los inquilinos pagan desde la app y al propietario se le actualiza el estado de los cobros.
- (inquilino) Registrar una o varias incidencias en su propiedad.
- (inquilino) Hacer pagos.

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
| province | lista de valores | sí | una de las 52 provincias de España |
| bedrooms | número entero | sí | informativo; 0 o más (un estudio puede tener 0) |
| bathrooms | número entero | sí | 1 o más |
| square_meters | número decimal | sí | mayor que 0; 2 decimales |
| rental_mode | lista de valores | sí | FULL (completo), ROOMS (por habitaciones) |
| monthly_rent | dinero | solo si FULL | renta de la vivienda completa; vacío si ROOMS |
| created_at | fecha y hora | sí | se rellena sola al crear |
| updated_at | fecha y hora | sí | se actualiza sola al modificar |

**Reglas**
- owner_id debe ser un usuario con rol OWNER.
- Si rental_mode = FULL, monthly_rent es obligatorio y mayor que 0. Si es ROOMS, va vacío.
- rental_mode solo se puede cambiar si la propiedad no tiene contratos activos.
- En contratos completos, al actualizar monthly_rent se actualiza también la renta del contrato activo.
- Los dos primeros dígitos de postal_code deben corresponder a la provincia.


## 5. Relaciones

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
