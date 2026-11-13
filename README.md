# Automatización de Extracción y Registro de Facturas con n8n y Gemini AI

Este proyecto consiste en el desarrollo de un pipeline de automatización ETL (Extract, Transform, Load) sin código (no-code/low-code) utilizando n8n. El sistema intercepta correos electrónicos entrantes en Gmail que contienen comprobantes de facturación (imágenes o documentos), procesa el archivo mediante un modelo de lenguaje multimodal con capacidades de visión (Google Gemini), extrae los campos clave de forma estructurada y los registra automáticamente en una planilla de Google Sheets.

---

## ¿Cómo funciona?

Como cada empresa arma sus comprobantes de forma distinta, armé un script (un parser) independiente para cada proveedor:
* **Edesur** (`parser-edesur.py`)
* **Río Uruguay Seguros** (`parser-RUS.py`)
* **Woker** (`parser-woker.py`)

A pesar de que cada PDF es un mundo, todos devuelven exactamente el mismo formato JSON con estos datos clave:
* `proveedor`
* `cuit_emisor`
* `fecha_emision`
* `total_a_pagar`

---

## Cosas que le fui sumando al código

* **Planes de respaldo ("Plan B"):** Si el PDF viene medio corrido o le falta algún separador clásico (como una barra vertical), el script tiene un segundo patrón alternativo para no fallar y rescatar igual el monto o la fecha.
* **Cero falsos positivos:** Usé límites de palabra (`\b`) en las expresiones regulares para, asegurarme de que busque la palabra exacta `TOTAL` y no me agarre un `SUBTOTAL` por error.
* **Pruebas:** Todos los scripts tienen el bloque `if __name__ == "__main__":` para poder correrlos de desde la terminal y ver el JSON impreso en pantalla con una factura de muestra.

---

## Lo que usé

* **Python**
* **`pdfplumber`** (para extraer todo el texto plano de las hojas del PDF)
* **Expresiones Regulares (RegEx)** (para extraer los patrones de CUIT, fechas y montos)
* **`json`** (para armar la salida)

---

## Cómo está armado el proyecto

```text
mi-portfolio-parsers/
│
├── samples/                  # Comprobantes de prueba en PDF
│   ├── Factura-Edesur.pdf
│   ├── OrdendePago-RUS.pdf
│   └── Comprobante-Woker.pdf
│
├── parser-edesur.py          # Script para Edesur
├── parser-RUS.py             # Script para Río Uruguay Seguros
├── parser-woker.py           # Script para Woker
├── requirements.txt          # Dependencias (solo pdfplumber)
└── README.md                 # Explicación del proyecto
