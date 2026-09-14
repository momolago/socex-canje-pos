# SOCEX · Prototipo Fase 0 (canje en mesón)

Simulador operativo del estudio *Canje de lealtad y pago dividido en retail presencial chileno* (13.09.2026).

**No es un PSP.** No cobra. No reconfigura el POS. No convierte HOLD en dinero de libre disposición. No habla con Transbank, Klap, Pomelo ni Geopagos.

## Abrir

Abrir `index.html` en el navegador.

## Caso tipo

| Campo | Valor |
|---|---|
| Ticket POS | $40.000 (Campo 4 bloqueado) |
| Canje de lealtad | $30.000 |
| Cupo tarjeta bancaria | $10.000 |
| TTL HOLD | 180 s |

## Superficies

1. **Briefing** — veredicto S4 + catálogo S1–S9.
2. **Wallet socio** — saldo, QR, DPAN Visa prepago, pedir canje (AVAILABLE → HOLD).
3. **Mesón cajero** — menú nativo «Pago dividido». Un tender $40.000 = `RECHAZADO (71)` = ISO 8583 Field 39 = 51. Dual-tender = 00 + 00 y voucher $40.000.
4. **Kernel** — AVAILABLE → HOLD(TTL) → BURNED | ROLLBACK | EXPIRED. LR = 0.
5. **Comercio / S3** — pide reembolso al operador. No ejecuta TEF. Oficio SII 2729.
6. **Laboratorio** — C1 (51), C2 (S4 ok), C4 (remanente alto → 51 pata B).

## Invariantes

- El total $40.000 no se edita. No hay UI para cambiar Campo 4.
- `RECHAZADO (71)` ≠ «Boleta no emitida (51)» Res. 176 SII.
- Voucher documenta $40.000 (Res. 176), no $10.000.
- No existe acción «transferir al socio». HOLD ≠ dinero libre.
- Si pata B = 51: void same-day de pata A y HOLD → ROLLBACK.
- NCG 498: el adquirente no recibe ID de socio + saldo.
- Fase 0 = S7 closed-loop (puntos = pasivo del comercio) + S4 dual-tender.
- S5 BaaS (Ley 20.950) = 6–12 meses. S9 factura espejo = prohibida.

## Cómo demostrar en 90 segundos

1. Laboratorio → **C1**. Una trama $40.000 contra cupo $10.000 → Field 39 = 51.
2. Reset → **C2**. HOLD + pata A DPAN $30.000 (00) + pata B banco $10.000 (00) → voucher $40.000 + BURN.
3. Reset → **C4**. Canje $5.000 + remanente $35.000 → 51 en pata B + void + ROLLBACK.
4. Comercio → **Pedir reembolso al operador (S3)** solo después de C2. SOCEX registra; no ejecuta.

## Qué falta para producción

- Contrato operador: URL webhook + evento + canal de pedido de refund.
- Oficio SII propio (el 2729 es publicidad, no lealtad multi-comercio).
- Emisor prepago inscrito (Pomelo u otro Ley 20.950) si se sale de S7.
- Parque real del comercio (Smart POS con NFC; Mobile POS fase 1 de TBK no tiene Apple Pay).
