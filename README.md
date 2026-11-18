# Automatización de Extracción y Registro de Facturas con n8n y Gemini AI

Este proyecto consiste en el desarrollo de un pipeline de automatización ETL (Extract, Transform, Load) sin código (no-code/low-code) utilizando **n8n**. El sistema intercepta correos electrónicos entrantes en Gmail que contienen comprobantes de facturación (imágenes), procesa el archivo mediante un modelo de lenguaje multimodal con capacidades de visión (**Google Gemini**), extrae los campos clave de forma estructurada y los registra automáticamente en una planilla de **Google Sheets**.

## Objetivos del Proyecto
* Automatizar la recepción y lectura de comprobantes fiscales recibidos por correo electrónico.
* Eliminar la carga manual de datos, reduciendo errores humanos y tiempos operativos.
* Implementar procesamiento multimodal con IA para interpretar facturas de diferentes formatos y diseños visuales.
* Estructurar la información no estructurada (imágenes/PDFs) en un formato tabular limpio listo para la contabilidad o gestión.

---

## ¿Cómo funciona?

### 1. Disparador y Captura (Gmail Trigger)
* **Cómo opera:** El sistema monitorea la casilla de correo en tiempo real.
* **Qué hace:** Cuando entra un correo electrónico que contiene una factura adjunta (ya sea en formato de imagen o documento), el nodo se activa automáticamente, extrae los metadatos del correo (como la fecha y el asunto) y captura el archivo adjunto convirtiéndolo en un objeto binario dentro del flujo de n8n (identificado como `attachment_0`).

### 2. Análisis Multimodal e Inteligencia Artificial (Basic LLM Chain + Gemini)
* **Cómo opera:** El archivo binario capturado es enviado directamente como entrada al modelo de lenguaje con visión (`gemini-2.0-flash`).
* **Qué hace:**
  * La IA "lee" visualmente el comprobante (como si fuera una persona mirando la factura, incluso si está rotada).
  * Extrae los datos clave acordados: **Cliente, Número de Factura, Monto, Fecha de emisión y Proveedor**.
  * Mediante un prompt estricto, la IA estructura la salida de forma limpia y ordenada (protegiendo puntos y comas decimales) para que los valores pasen directos al siguiente nodo sin necesidad de pasos intermedios de código[cite: 10, 12].

### 3. Persistencia en Destino (Google Sheets - Append Row)
* **Cómo opera:** Es el paso final del pipeline.
* **Qué hace:** Toma directamente los campos extraídos por el modelo de IA y ejecuta una inserción de fila limpia (`Append Row`) en tu hoja de cálculo de Google, registrando la factura de forma ordenada junto con la fecha y hora en la que fue procesada por el sistema.

---

## Lo que usé
* **n8n:** Como la plataforma central de automatización (pipeline ETL), encargada de orquestar el flujo de 3 nodos desde que llega el correo hasta que se guarda el registro.
* **Gmail (Gmail Trigger):** Como el canal de entrada o disparador (*trigger*) para detectar automáticamente la llegada de nuevos correos con facturas adjuntas[cite: 9, 12].
* **Google Gemini (`gemini-2.0-flash`):** Como el motor de inteligencia artificial multimodal y OCR inteligente para leer, interpretar y estructurar los datos clave directamente de las imágenes o archivos de las facturas[cite: 10, 12].
* **Google Sheets:** Como la base de datos o destino final donde se almacenan y ordenan de manera estructurada todos los comprobantes procesados[cite: 12].

---

## Cómo está armado el proyecto

```text
[ Gmail Trigger ] ──> [ Basic LLM Chain (Gemini Chat Model) ] ──> [ Append Row in Sheet ]

# Requisitos y Configuración

Al tratarse de un flujo de automatización en **n8n**, no requiere instalación de dependencias de código local. Para replicar y ejecutar este pipeline se necesita:

* **Instancia de n8n:** Versión Cloud o Self-hosted (local/Docker) con soporte para nodos de LangChain (`Basic LLM Chain`).
* **Google Cloud Platform (GCP):** Proyecto configurado con las siguientes APIs habilitadas y credenciales OAuth2 activas en n8n:
  * **Gmail API:** Para la lectura de correos entrantes y descarga de archivos adjuntos en formato binario.
  * **Google Sheets API:** Para la escritura y registro automático de filas en la planilla de destino.
* **Google AI Studio (Gemini API):** Una API Key válida con acceso al modelo multimodal `gemini-2.0-flash`.
* **Planilla de Google Sheets:** Previamente creada con las siguientes columnas en la primera fila:
  * `Cliente` | `N° Factura` | `Monto` | `Fecha` | `Proveedor` | `Fecha de Procesamiento`
