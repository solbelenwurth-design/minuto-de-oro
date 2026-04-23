# Minuto de Oro — Resumen para Desarrollo

## 1. Qué es y para qué sirve

Minuto de Oro es un módulo nuevo a integrar en la aplicación Warrior (tablet del vendedor) que le exige al vendedor preparar una estrategia comercial **antes** de iniciar la visita a un cliente. Le propone productos a ofrecer según el historial del cliente, lo que nunca compró, lanzamientos y promociones vigentes. El vendedor debe preseleccionar un mínimo de productos y recién ahí se habilita el botón de Iniciar Visita.

El concepto central es "**dinero nuevo**": vender productos que el cliente le compra a la competencia y que Würth todavía no le vende. El objetivo medible (benchmark Brasil) es pasar de 6,4 a 8 líneas promedio por pedido.

Todo el módulo funciona **offline**. La tablet sincroniza al inicio del día y durante la jornada trabaja con datos locales. Al recuperar conexión, sincroniza automáticamente.

---

## 2. Flujo del vendedor, paso a paso

### Paso 1 — Selección del cliente
El vendedor entra al dashboard del cliente desde la hoja de ruta del día. Aplica tanto a clientes activos como inactivos de la cartera del vendedor.

### Paso 2 — Abrir Minuto de Oro
El vendedor presiona el botón "Minuto de Oro". Se abre una pantalla con cuatro bloques de productos:

- **Reposición**: productos que el cliente ya compró en los últimos 12 meses. Se muestra el total mes a mes, la media 12M y la sugestão (cantidad sugerida).
- **Dinero nuevo (Magníficos e Imprescindibles)**: productos estratégicos de la división (magníficos) y del rubro (imprescindibles) que el cliente **nunca** compró.
- **Lanzamientos y Promociones**: productos lanzados en los últimos 6 meses + promociones vigentes del mes, del rubro del cliente. Agrupados por familia. Se excluyen los que ya figuran en Magníficos/Imprescindibles.
- **Selección propia**: navegador del catálogo completo para agregar productos libres.

### Paso 3 — Preseleccionar productos (mínimos)
El vendedor tilda productos. Hay mínimos obligatorios por bloque:

- Reposición: mínimo **4**
- Dinero nuevo: mínimo **3**
- Lanzamientos y Promociones: mínimo **3**
- Selección propia: opcional

Un contador muestra el estado (X/4, X/3, X/3). El botón "Iniciar Visita" permanece **deshabilitado** hasta cumplir los tres mínimos. Al llegar a los tres, se habilita automáticamente.

### Paso 4 — Iniciar visita
Al presionar Iniciar Visita, el sistema:

1. Registra fecha/hora local de inicio.
2. Genera un **precarrito** con los productos preseleccionados, con cantidades sugeridas.
3. Marca los productos del Minuto de Oro con una **barra amarilla vertical** para diferenciarlos del resto.
4. Navega a la pantalla de pedido en edición.

### Paso 5 — Durante la visita
El vendedor puede:

- Modificar cantidades.
- Eliminar productos (si el producto viene del Minuto de Oro, el sistema **obliga** a elegir un motivo).
- Agregar productos del catálogo (estos no llevan barra amarilla).
- Marcar un producto como "Para próxima visita".
- Volver al Minuto de Oro y regresar sin perder el carrito.
- Consultar información del producto, videos, promociones asociadas, stock.
- Compartir ficha de producto por WhatsApp.

### Paso 6 — Cierre de visita
Hay dos escenarios principales:

**Visita no efectiva** (no hubo contacto comercial). Motivos de cierre:
- "No me atendió / no estaba" → abre WhatsApp con el chat directo al cliente (si tiene número) o WhatsApp convencional.
- "Cerrado" → mismo comportamiento que el anterior.
- "Otro" → habilita campo de texto libre.
- "Cargar horarios" → abre modal para registrar días/horarios de atención del cliente.

En todos los casos, el sistema **automáticamente** guarda el carrito como borrador. No hay botón manual "Guardar borrador".

**Visita efectiva**:
- *Sin compra*: el vendedor indica motivo (solo cobranza, no interesado, sin presupuesto, otro). El sistema guarda el borrador automáticamente.
- *Con compra parcial*: los productos no comprados se eliminan con motivo obligatorio. Los motivos se persisten para futuras visitas.
- *Con compra total*: al cargar el pedido, el sistema cierra la visita automáticamente (sin pedir motivo, el motivo es la venta) y **no** reutiliza el borrador en la próxima visita.

---

## 3. Lógica del borrador

Un cliente solo puede tener **un** borrador activo a la vez. El borrador se sobrescribe, no se apila. No hay historial de borradores.

- Si la visita actual termina sin compra total → se guarda el borrador.
- Si la visita actual termina con compra total → **no** se guarda borrador (el próximo Minuto de Oro arranca de cero).
- En la próxima visita al mismo cliente, si hay borrador, sus productos aparecen preseleccionados en el Minuto de Oro y el vendedor puede modificar la selección.

---

## 4. Historial de compra — regla de fechas

- **Cliente activo** (tiene compras en los últimos 6 meses): se muestran los últimos 12 meses contados desde hoy.
- **Cliente inactivo**: se muestran los 12 meses anteriores a la última fecha de compra del cliente.

---

## 5. Cálculos

**Media 12M**: promedio mensual de unidades compradas por el cliente en el rango de 12 meses. Suma de cantidades dividido por 12. Los meses sin compra cuentan como 0.

**Sugestão**: cantidad sugerida a ofrecer para los próximos 12 meses. Propuesta inicial: máximo entre la media mensual y la cantidad comprada en el mismo mes del año anterior. La fórmula debe ser parametrizable para ajustarse sin cambios de código (la fórmula exacta queda pendiente de validación comercial).

---

## 6. Motivos obligatorios

Cuando se elimina un producto que vino del Minuto de Oro, el sistema despliega un modal con estos motivos (parametrizables):

- Consume otra marca
- Precio
- Sin presupuesto
- Ya tiene
- Otro (habilita campo de texto libre)

Los motivos se persisten en el backend y quedan disponibles para reportería en el reactor, con estos campos: producto, código, fecha, vendedor, motivo, otro.

En la próxima visita al mismo cliente, los productos eliminados previamente pueden reaparecer organizados por motivo como secciones dinámicas adicionales del Minuto de Oro. Orden sugerido:
1. Primero los que hoy están en promoción (si antes se rechazaron por "precio").
2. Luego: Ya tiene, Sin presupuesto, Consume otra marca, Otro.

Esta funcionalidad debe ser parametrizable (activable/desactivable).

---

## 7. Cliente nuevo o con cambio de rubro

Si el cliente es nuevo o cambió de rubro:
- El vendedor debe seleccionar primero el rubro válido.
- No se aplica la lógica de reposición (no hay historial).
- Se muestran Magníficos e Imprescindibles, Lanzamientos y Promociones, y Selección propia basados en el rubro seleccionado.
- Mínimos ajustados: 3 dinero nuevo + 3 lanzamientos/promos.

---

## 8. Cotización

Durante la visita el vendedor puede:
- **Enviar cotización**: genera un PDF descargable. El vendedor lo envía manualmente por mail o WhatsApp.
- **Cotizar y vender** simultáneamente.
- Consultar cotizaciones anteriores del cliente (pantalla a definir en WB-3075).

---

## 9. Indicadores en pantalla del carrito

Durante la visita se muestra un recuadro con tres indicadores con semáforo (rojo / amarillo / verde):

- Cantidad de líneas.
- Productos comprados por primera vez (dinero nuevo concretado).
- Productos magníficos e imprescindibles en el carrito.

Los umbrales de cada color deben ser parametrizables.

---

## 10. Operación offline

- La tablet descarga al inicio del día: cartera de clientes, historial de compras, catálogo de productos, magníficos por división, imprescindibles por rubro, lanzamientos de los últimos 6 meses, promociones vigentes, borradores pendientes y motivos parametrizables.
- Durante la jornada, todas las acciones se guardan localmente.
- Al recuperar conexión, sincroniza automáticamente en segundo plano. También se puede disparar manualmente.
- Cada registro transaccional debe tener un identificador local único para poder sincronizar sin duplicar.
- Los reintentos ante falla se hacen con backoff exponencial.
- El vendedor debe ver en todo momento el estado de sincronización.
- Para datos de catálogo, siempre prevalece la versión del servidor.
- Para borradores, prevalece el más reciente por cliente.

---

## 11. Integración con Warrior existente

- Habilitar el botón "Minuto de Oro" en el dashboard del cliente, con indicador visual cuando hay borrador pendiente.
- El flujo actual de "Iniciar Visita" se reemplaza (o complementa) por el nuevo flujo de Minuto de Oro.
- En la pantalla de pedido existente, agregar la barra amarilla vertical para ítems del Minuto de Oro y el recuadro de indicadores.
- Al eliminar un ítem de Minuto de Oro, interceptar con el modal de motivo obligatorio.
- Permitir navegación bidireccional entre Minuto de Oro y el pedido sin perder estado.
- Al cerrar la visita con carga de pedido exitosa, cerrar la visita automáticamente sin pedir motivo.

---

## 12. Reglas de negocio que no se negocian

1. El botón Iniciar Visita está deshabilitado hasta cumplir los mínimos.
2. Un cliente solo tiene un borrador activo. Se sobrescribe, no se apila.
3. Eliminar un ítem de Minuto de Oro exige motivo obligatorio.
4. Compra total no reutiliza borrador en la próxima visita.
5. Compra parcial o sin compra genera/actualiza borrador.
6. Cierre con venta no solicita motivo.
7. Cualquier cierre no efectivo guarda borrador automáticamente.
8. La preselección de Minuto de Oro se hace antes de iniciar la visita, no durante.
9. Durante la visita, el vendedor puede volver al Minuto de Oro sin perder el carrito.
10. Clientes nuevos o con cambio de rubro arrancan con el rubro definido antes del armado.

---

## 13. Pendientes de definición

Estos puntos requieren decisión antes o durante el desarrollo (deberían resolverse con el área comercial o de producto):

1. Fórmula exacta del cálculo de sugestão.
2. Umbrales exactos del semáforo de indicadores.
3. Campo de la tabla ADR a usar para abrir WhatsApp (dejar parametrizable).
4. Lista definitiva de imprescindibles por rubro para división METAL.
5. Comportamiento de consulta de cotizaciones anteriores cuando el vendedor es nuevo.
6. Volumen máximo de productos a mostrar en historial cuando hay mucho histórico. Definir si se pagina o se filtra.
7. Qué hacer con productos históricos que el cliente ya no consume.
8. Integración con ticket WB-3075 (cotizaciones).
9. Visualización de horarios de atención cargados (podría requerir ticket adicional).

---

## 14. Criterios de aceptación resumidos

- Cliente activo: armado con secciones pobladas correctamente según reglas, botón Iniciar Visita deshabilitado hasta cumplir mínimos (4/3/3), habilitación automática al cumplir.
- Cliente inactivo: historial usa rango de 12 meses anterior a última compra, no desde hoy.
- Cliente nuevo/cambio de rubro: sin Reposición, mínimos 3/3, rubro como criterio base.
- Eliminación con motivo: no se puede eliminar un ítem del Minuto de Oro sin motivo. Si selecciona "Otro", texto obligatorio.
- Borrador automático: cualquier cierre no efectivo guarda borrador sin acción del vendedor. No hay botón manual.
- Reutilización: si existe borrador, aparece preseleccionado en la próxima visita.
- Compra total: borrador se elimina y próxima visita arranca desde cero.
- Compra parcial: no se puede finalizar el pedido si hay ítems del Minuto de Oro sin motivo.
- Offline: funciona sin conexión con datos previamente sincronizados. Estado de sincronización visible.
- WhatsApp: en cierres no efectivos con motivo "No me atendió" o "Cerrado" abre chat directo al número (o WhatsApp convencional si no hay número).
- Indicadores: tres KPIs visibles con semáforo según umbrales parametrizados.
- Identificación visual: barra amarilla vertical a la izquierda en los ítems del Minuto de Oro.

---

## 15. Capacitación y roll-out (contexto para el equipo de implementación)

El sistema es una herramienta que habilita un proceso comercial, no lo reemplaza. Para que el proceso funcione, cada vendedor debe capacitarse individualmente en los productos que selecciona del Minuto de Oro, conociendo diferenciales competitivos y argumentos de venta (auto-treinamento de 2 a 3 productos por visita).

Al implementar el roll-out:

- Comunicar a los vendedores a las 7/7:30 hs del día de capacitación que actualicen los tablets (datos, base, versión de Connect) antes de iniciar.
- Presentar el material de contexto: PowerPoint "Dinheiro Novo" y video de Urbano (tutorial del ajuste sistémico).
- Luego del video, los vendedores abren sus tablets y se hace un entrenamiento práctico corriendo por todo el proceso juntos.

El sistema debe facilitar esta preparación dando acceso rápido desde la ficha del producto a: descripción técnica, características, videos, compartir por WhatsApp, productos relacionados de la misma familia, promociones vigentes asociadas.
