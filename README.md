# Darwin Capacity Optimiser - Versión MQL4

**Port / Traducción a MQL4** del [Darwin Capacity Optimiser](https://github.com/marticastany/darwin-capacity-optimiser) original (desarrollado para MetaTrader 5).

Esta versión adapta la librería de **splitting de órdenes** para que funcione en **MetaTrader 4**, ayudando a reducir el slippage en ejecuciones de gran volumen (especialmente útil en copy-trading y Darwins).

---

## ⚠️ Advertencia - Software Alpha

**Software en fase Alpha — úselo bajo su propia responsabilidad.**

Este código se encuentra en fase alpha y no ha sido exhaustivamente probado en diferentes brokers, símbolos ni en condiciones reales de mercado.

Es **su responsabilidad** revisar detenidamente el código fuente, probar la librería en una cuenta demo y verificar su comportamiento completo antes de utilizarla con dinero real.

Los autores y colaboradores **no aceptan ninguna responsabilidad** por pérdidas en trading, errores de ejecución, rellenos parciales, cierres no realizados, slippage, rechazos del broker, ni cualquier otra consecuencia financiera o operativa derivada del uso de este software.

La librería se proporciona **"TAL CUAL"** ("AS IS"), sin ningún tipo de garantía — consulte el archivo [LICENSE](LICENSE) (Apache License 2.0, Secciones 7 y 8) para leer el descargo de responsabilidad completo.

## ⚠️ Warning - Alpha Software

**Alpha software — use at your own risk.**

This is alpha-stage code and has not been battle-tested across brokers, symbols, or live market conditions. 

It is your responsibility to thoroughly review the source, test the library on a demo account, and verify its behaviour end-to-end before deploying it with real funds.

The authors and contributors accept **no liability** for trading losses, execution errors, partial fills, missed closes, slippage, broker rejections, or any other financial or operational consequence arising from the use of this software.

The library is provided **"AS IS"**, without warranties of any kind — see the [LICENSE](LICENSE) (Apache License 2.0, Sections 7 and 8) for the full disclaimer.

---

## 📌 ¿Qué hace esta librería?

Cuando el volumen de una operación es grande, enviar una sola orden al mercado genera slippage. Esta librería divide la orden en **N órdenes más pequeñas** enviadas de forma secuencial con un retraso configurable, permitiendo que el libro de órdenes recupere liquidez entre ejecuciones.

Ideal para:
- Estrategias copiadas con alto volumen
- Mejora de la capacidad de Darwins
- Reducción de diferencia de rendimiento por slippage

---

## 🎯 Créditos y Fuente Original

- **Autor original**: [marticastany](https://github.com/marticastany)
- **Repositorio original**: [https://github.com/marticastany/darwin-capacity-optimiser](https://github.com/marticastany/darwin-capacity-optimiser)
- **Licencia**: Apache License 2.0 (misma que el proyecto original)

**Esta es una obra derivada**. Se mantiene la licencia Apache 2.0 y se da el crédito correspondiente al autor original.

---

## 📥 Instalación

1. Descarga o clona este repositorio.
2. Copia la carpeta `Include/SplitOrder/` completa a tu carpeta de MetaTrader 4: `MQL4/Include/SplitOrder/`
3. (Opcional) Copia los ejemplos que están en `Experts/Examples/` para probar el funcionamiento.

---

## 🚀 Uso Básico (Quick Start) 
### Pasos para integrar SplitOrder en Estrategias MQL4
(Pueden ser aplicados manualmente o con ayuda de cualquier LLM)

**Paso 1.** Añadir la librería en `Include` (al principio del archivo):
```mql4
#include <SplitOrder/SplitOrder.mqh>
```

**Paso 2.** Añadir en `variables globales` (fuera de cualquier función):
```mql4
SplitConfig splitCfg;
SplitState  splitLong;
SplitState  splitShort;
```

**Paso 3.** Añadir en `OnInit()` la configuración del split:
```mql4
SplitConfigInit(splitCfg);
splitCfg.splitCount   = 3;   // Dividir en 3 órdenes
splitCfg.delaySeconds = 10;  // 10 segundos entre órdenes
splitCfg.magic        = MagicNumber; //No cambiar para que coincida con la orden original
EventSetTimer(1);
```

**Paso 4.** Añadir en `OnDeinit()`:
```mql4
EventKillTimer();
```

**Paso 5.** Añadir en `OnTick()` justo al principio del cuerpo:
```mql4
// Gestión de las colas de órdenes de SplitOrder
SplitManage(splitLong,  splitCfg);
SplitManage(splitShort, splitCfg);
```

**Paso 6.** En cada orden, reemplazar la apertura de orden por la versión split. 

Ejemplo de apertura de orden en una estrategia sin modificar:
```mql4
_ticket = OrderSend(OP_BUYSTOP, ..., mmLots, ...);
```

Versión split para las órdenes pendientes (STOP o LIMIT)
```mql4
double pos0Lots = SplitAdjustLots(Symbol(), mmLots, splitCfg);
_ticket = OrderSend(OP_BUYSTOP, ..., pos0Lots, ...);
if(_ticket > 0)
    SplitRegister(splitLong, _ticket, splitCfg, mmLots, true);
```

Versión split para órdenes a mercado (MARKET)
```mql4
double pos0Lots = SplitAdjustLots(Symbol(), mmLots, splitCfg);
_ticket = OrderSend(OP_BUYSTOP, ..., pos0Lots, ...);
if(_ticket > 0)
    SplitRegister(splitLong, _ticket, splitCfg, mmLots, false);
```

Nota: Usa splitLong para ordenes de compra y splitShort para órdenes de venta.

---

## ⚠️ Limitaciones

- Soporta un split activo por dirección (Long / Short)
- No persiste tras reinicio del terminal
- Las órdenes hijo no llevan Take Profit ni Stop Loss individual (se gestiona la salida de forma secuencial)
- Requiere un timer para gestionar el envío de las órdenes restantes

---

## 📝 Licencia

Este proyecto está licenciado bajo la **Apache License 2.0** (la misma que el proyecto original).
Ver el archivo [`LICENSE`](LICENSE) para más detalles.

---

## 🙏 Agradecimientos

Gracias a **[marticastany](https://github.com/marticastany)** por publicar el código original bajo licencia abierta, lo que ha permitido realizar esta adaptación a MQL4.
Sin su trabajo esto no habría sido posible.

---
