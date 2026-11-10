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
- **utilities**: Contratos de suministro de cada propiedad, que servio es y con que compañia esta contratado.
- **rent_changes**: historial de rentas: guarda la renta inicial de cada contrato y cada actualzación anual. Sirve para la grafica
- **rent_indexes**: Valores del indice oficial que limita cuanto se puede subir la renta cada año.

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
- Todos los contratos tienen sus renta. En los completos es siempre la de la propiedad, y se actualizan juntas.
- La renta se puede actualizar una vez al año, contando desde la fecha de inicio del contrato, y como maximo segun el indice oficial.
- Se guarda el historial de rentas de cada contrato, empezando por la renta inicial.
- El historial apunta a la propiedad(y a la habitación, si la hay), para que se conserve aunque se borre el contrato y para poder calcular la evolución del precio de la vivienda.

### Propiedad
- tipos: piso, casa, estudio y ático.
- La dirección se pide desglosada (calle, numero, piso, puerta, codigo postal, ciudad y provincia) y se muestra junta.
- El modo de alquiler (completo o por habitaciones) lo fija la propiedad, y solo se puede cambiar si no tiene contratos activos.
- El numero de habitaciones de la propiedad es informativo, las habitaciones que se alquilan se registran en rooms.

### Suministros
- El propietario elige la compañia de una lista filtrada por el tipo de suministro. Si no esta, elige "Otra".
## 4. Campos de cada entidad

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
