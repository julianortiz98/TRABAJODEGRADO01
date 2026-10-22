# Transformadores:gemelo digital

Dos simuladores web interactivos para entender cómo se carga, se calienta y envejece un transformador, y cómo un gemelo digital puede avisar antes de una falla.

**Autor:** Julian David Ortiz Gaviria
**Año:** 2026
**Contexto:** proyecto asociado a una idea de investigación para trabajo de grado.
**Licencia:** pendiente de definir por el autor.

> **Aviso:** todos los datos son simulados y los valores de los equipos son de demostración. Ninguna lectura proviene de sensores reales y el proyecto no sirve para operar ni diagnosticar equipos reales.

---

## Contenido

| Archivo | Qué es |
|---|---|
| `index.html` | **Transformador T-01 por dentro (3D).** Gemelo digital de un transformador de potencia de 30 MVA, 115/13,8 kV, en aceite (ONAN/ONAF). |
| `simulador.html` | **Simulador de transformador de poste.** Un transformador de distribución y las casas que alimenta. |
| `docs/presentacion_transformadores.pdf` | Presentación de 7 láminas para exponer el proyecto. |
| `docs/Idea_de_Investigacion_Gemelo_Digital_Transformadores.docx` | Plantilla de idea de investigación diligenciada. |
| `docs/datos_verificacion_voltajes.txt` | Voltajes, corrientes, umbrales y constantes usados, contrastados con normas y práctica real. |

Los dos simuladores están enlazados entre sí desde la barra de navegación.

## Cómo ejecutarlo

No requiere instalación ni servidor. Abre `index.html` o `simulador.html` en un navegador moderno (Chrome, Edge, Firefox o Safari).

- `simulador.html` no tiene dependencias externas.
- `index.html` necesita WebGL y conexión a internet para cargar **three.js r128** (cdnjs) y la fuente **IBM Plex Sans** (Google Fonts). La librería **jsPDF 2.5.1** se descarga solo cuando se pide el informe en PDF. Sin conexión, la fuente cae a una del sistema y el simulador 3D no arranca, porque no se carga three.js.
- Si el navegador no puede iniciar WebGL, el visor muestra un aviso. Activa la aceleración por hardware o prueba otro navegador.

---

## 1. Simulador de transformador de poste (`simulador.html`)

Muestra cuánto consume cada casa, qué parte del transformador ocupa y cómo reaccionan la carga y la temperatura.

**Qué incluye**

- Transformador de **13 200 V a 240 V / 120 V** con capacidad elegible: 15, 25 (por defecto), 37,5, 50 o 75 kVA.
- De 1 a 6 casas (3 por defecto), con 8 aparatos cada una.
- Corriente de entrada, de las líneas L1 y L2 y del neutro, con animación de flujo.
- Medidor de carga, punto caliente del devanado, aceite superior y desgaste del aislamiento.
- Gráfica de los últimos 90 s con la carga apilada por casa y la temperatura.
- Reparto de la capacidad por casa, en barra y en tabla.
- Escenarios: Madrugada, Hora de la cena, Todo encendido, Apagar todo y Apagar solo los de 240 V.
- Modo **un día completo** (lento: 2 min, normal: 1 min, rápido: 30 s), con horarios de encendido propios de cada casa.

**Aparatos simulados**

| Aparato | Tensión | Potencia |
|---|---|---|
| Luces | 120 V | 100 W |
| Televisor | 120 V | 150 W |
| Nevera | 120 V | 200 W |
| Microondas | 120 V | 1200 W |
| Plancha | 120 V | 1000 W |
| Aire acondicionado | 240 V | 1800 W |
| Estufa eléctrica | 240 V | 3000 W |
| Ducha eléctrica | 240 V | 4500 W |

**Niveles de estado:** Normal, Atención, Alerta y Crítico, según el punto caliente (98, 110 y 130 °C) y la carga (85, 100 y 120 %).

---

## 2. Transformador T-01 por dentro (`index.html`)

**Visor 3D**

- Tres vistas: **Exterior**, **Corte** y **Explotada** (con control para separar las partes).
- Arrastrar para girar, rueda o dos dedos para acercar, tocar una parte para ver su función. También funciona con teclado (flechas, `+` y `-`).
- Opciones: tanque transparente, sensores del gemelo, flujo magnético, circulación del aceite, vista térmica y sombras reales.
- Calidad gráfica alta, media o baja, con bajada automática si el equipo no da la talla.

**Doce partes explicadas:** núcleo magnético, devanados, salidas y tierras, tanque y aceite, aisladores pasantes, conservador y respirador, relé Buchholz, radiadores, ventiladores, cambiador de tomas (OLTC), instrumentos y caja de control, válvulas y puntos de muestreo.

**Gemelo digital**

- **11 sensores simulados:** TOP-OIL, FIBRA-HS, DGA, VIB, TAN-D, NIVEL, HUM, PD-UHF, TORQUE, I-FAN e IR-CONEX.
- **33 variables** en cuatro grupos (operación, térmico y refrigeración, aislamiento y gases, mecánico y accesorios), de tipo **M** medida, **E** estimada, **C** calculada o **S** estado, con referencia, umbral de alarma y estado.
- **Lo medido frente a lo esperado:** el gemelo calcula la temperatura del aceite que esperaría con refrigeración sana y la compara con la medida.
- Tendencias de los últimos 2 minutos y proyección de **vida útil** del aislamiento según la carga.
- **8 fallas simulables:** ventiladores averiados, fuga de aceite, corrosión del tanque, humedad en el aceite, descargas parciales, aislador pasante deteriorado, conexión floja y contactos del OLTC desgastados.
- Diagnóstico con hallazgos que incluyen evidencia, hipótesis, verificación sugerida y acción de mantenimiento, y un registro de eventos.

**Operación:** carga de 0 a 42 MVA (21 por defecto), ambiente de 0 a 45 °C (30 por defecto) y opción de desenergizar el equipo.

**Exportación**

- Tendencias en CSV (hasta 1 hora, una muestra cada 0,5 s).
- Registro de eventos en CSV.
- Informe de mantenimiento en PDF, o para copiar e imprimir.
- Formato CSV para Excel en español (`;` y coma decimal) o estándar (`,` y punto).

---

## Modelo físico

Los dos simuladores usan la misma forma del modelo térmico y de envejecimiento, con `K = carga / capacidad`:

```
Calentamiento del aceite = 55 · ((1 + 5·K²) / 6)^0,8
Gradiente del devanado   = 23 · K^1,6
Punto caliente           = ambiente + calentamiento del aceite + gradiente
FAA                      = exp(15000/383 − 15000/(punto caliente + 273))
Vida restante (años)     = (180 000 − 72 000 − envejecimiento) / FAA / 8760
```

- **FAA** es la aceleración del envejecimiento: vale 1 con 110 °C en el punto caliente.
- En el T-01, los ventiladores reducen el calentamiento del aceite en un 15 % y arrancan por histéresis (aceite > 70 °C o punto caliente > 100 °C).
- La temperatura sube y baja con **inercia térmica**: al apagar aparatos la carga cae al instante, pero el transformador tarda en enfriarse.
- La tensión de salida baja un poco con la carga (3,5 % a plena carga en el poste, 1 % en el T-01).

Referencia: guía de carga IEEE C57.91 (la fórmula del envejecimiento, los 110 °C y las 180 000 h coinciden con ella).

## Parámetros principales

| Parámetro | Poste | T-01 |
|---|---|---|
| Tensión de entrada | 13 200 V | 115 kV |
| Tensión de salida | 240 V / 120 V | 13,8 kV |
| Capacidad | 25 kVA (15 a 75) | 30 MVA (18 MVA en ONAN) |
| Corriente nominal | 104,2 A por línea | 150,6 A (alta) y 1255 A (baja) |
| Frecuencia | n/d | 60 Hz |
| Conexión e impedancia | n/d | YNd11, 9,5 % |
| Ambiente | 30 °C | 30 °C (0 a 45) |
| Vida consumida inicial | n/d | 40 % |

La verificación de cada valor contra normas y práctica real está en [`docs/datos_verificacion_voltajes.txt`](docs/datos_verificacion_voltajes.txt).

## Verificación frente al mundo real (resumen)

- **Corregido:** el secundario del poste usa **240/120 V**, como las especificaciones de los operadores de red colombianos (en el habla común se dice «220/110»). Con 240 V la relación es 55:1 y la corriente nominal de 25 kVA es 104,2 A por línea.
- **Coinciden:** 13 200 V, 13,8 kV y 115 kV como niveles de tensión; los tamaños de 15 a 75 kVA y 30 MVA; la fórmula y las constantes del envejecimiento.
- **Simplificaciones:** parámetros térmicos típicos pero de demostración, constantes de tiempo aceleradas, caída de tensión del T-01 algo baja, corriente de entrada sin factor de potencia.
- **Por confirmar con las normas de pago:** umbrales de gases disueltos, humedad, rigidez dieléctrica y descargas parciales.

## Limitaciones

- Todas las lecturas son simuladas; no hay conexión con sensores ni equipos reales.
- El flujo magnético y la circulación del aceite son representaciones conceptuales, no una simulación de fluidos (CFD) ni de campos.
- Los diagnósticos son hipótesis: una sola variable no confirma una falla, y se verifican con pruebas y criterio técnico.
- Modelo simplificado: aparatos resistivos en el poste, ambiente fijo de 30 °C en el poste y régimen permanente en las proyecciones.

## Accesibilidad

Navegación con teclado, tema claro y oscuro según el sistema, respeto de la preferencia de movimiento reducido, estados con texto además de color y diseño adaptable a celular.

## Tecnologías

HTML, CSS y JavaScript sin servidor · three.js r128 (WebGL) · canvas 2D y SVG · jsPDF 2.5.1 · IBM Plex Sans.

## Referencias

- IEEE. (2025). *IEEE Std C57.91-2025. IEEE guide for loading mineral-oil-immersed transformers and step-voltage regulators.* IEEE.
- IEEE. (2019). *IEEE Std C57.104-2019. IEEE guide for the interpretation of gases generated in mineral oil-immersed transformers.* IEEE.
- IEC. (2018). *IEC 60076-7:2018. Power transformers. Part 7: Loading guide for mineral-oil-immersed power transformers.*
- Duval, M. (2002). A review of faults detectable by gas-in-oil analysis in transformers. *IEEE Electrical Insulation Magazine, 18*(3), 8-17.
- Grieves, M. (2014). *Digital twin: Manufacturing excellence through virtual factory replication* [White paper].

## Autor

**Julian David Ortiz Gaviria**
