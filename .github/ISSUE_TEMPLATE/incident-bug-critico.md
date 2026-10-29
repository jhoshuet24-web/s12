---
name: Reporte de Incidente
about: Usa este formulario para reportar un incidente crítico
title: "[INCIDENTE] Error 500 al procesar pagos con Stripe en checkout"
labels: [incident, urgent]
assignees: 'sofia-dev, carlos-ops'
---

## Descripción del incidente
Los usuarios están experimentando un error interno del servidor (HTTP 500) al intentar finalizar sus compras utilizando la pasarela de pagos de Stripe. El dinero es retenido en el banco del usuario, pero la orden no se crea en nuestra base de datos.

## Impacto
* **Afecta a:** Todos los clientes de la plataforma web en entorno de producción.
* **Gravedad:** Bloqueo total del flujo de ingresos (conversión caída al 0%).
* **Métricas:** ~150 transacciones fallidas en la última hora.

## Tiempo de detección
* **Ocurrencia:** 28 de Octubre de 2026, 07:15 AM UTC
* **Detección:** 28 de Octubre de 2026, 07:22 AM UTC (Alerta de Datadog por pico de errores 500).

## Pasos para reproducir
1. Ir al carrito de compras con cualquier producto.
2. Hacer clic en "Proceder al pago".
3. Introducir los datos de una tarjeta válida de prueba/producción.
4. Hacer clic en "Confirmar Compra".
5. Ver la pantalla de error genérica "Algo salió mal (Código 500)".

## Logs / Evidencia
```json
[2026-10-28T07:15:32.123Z] ERROR (PaymentService): Stripe webhook signature verification failed.
TypeError: Cannot read property 'id' of undefined at stripe.js:42:18
```

## Acciones tomadas
* Se desactivó temporalmente el método de pago Stripe para evitar cobros dobles fantasma.
* Se habilitó un banner de mantenimiento en la sección de checkout.
* El equipo de backend está revisando el último despliegue de las 07:00 AM.

## Responsable(s)
* `@sofia-dev` (Líder de Backend)
* `@carlos-ops` (SRE / Infraestructura)

## Próximos pasos
- [x] Mitigación inicial (Desactivar Stripe en frontend)
- [ ] Rollback del microservicio de pagos a la versión `v2.4.1`
- [ ] Validar compatibilidad de la firma de webhooks con la nueva SDK de Stripe
- [ ] Ejecutar script de conciliación para los usuarios con cobros duplicados sin orden generada
