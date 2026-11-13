# Automatización de Extracción y Registro de Facturas con n8n y Gemini AI

Este proyecto consiste en el desarrollo de un pipeline de automatización ETL (Extract, Transform, Load) sin código (no-code/low-code) utilizando n8n. El sistema intercepta correos electrónicos entrantes en Gmail que contienen comprobantes de facturación (imágenes), procesa el archivo mediante un modelo de lenguaje multimodal con capacidades de visión (Google Gemini), extrae los campos clave de forma estructurada y los registra automáticamente en una planilla de Google Sheets.

---
# Objetivos del Proyecto

Automatizar la recepción y lectura de comprobantes fiscales recibidos por correo electrónico.

Eliminar la carga manual de datos, reduciendo errores humanos y tiempos operativos.

Implementar procesamiento multimodal con IA para interpretar facturas de diferentes formatos y diseños visuales.

Estructurar la información no estructurada (imágenes/PDFs) en un formato tabular limpio listo para la contabilidad o gestión.

---
## ¿Cómo funciona?

1. Disparador y Captura (Gmail Trigger)
Cómo opera: El sistema monitorea la casilla de correo en tiempo real.

Qué hace: Cuando entra un correo electrónico que contiene una factura adjunta (ya sea en formato de imagen o documento), el nodo se activa automáticamente, extrae los metadatos del correo (como la fecha y el asunto) y captura el archivo adjunto convirtiéndolo en un objeto binario dentro del flujo de n8n (identificado como attachment_0).

2. Análisis Multimodal e Inteligencia Artificial (Basic LLM Chain + Gemini)
Cómo opera: El archivo binario capturado es enviado como entrada al modelo de lenguaje con visión (gemini-2.0-flash).

Qué hace:

La IA "lee" visualmente el comprobante (como si fuera una persona mirando la factura).

Extrae los datos clave acordados: Cliente, Número de Factura, Monto, Fecha de emisión y Proveedor.

Para evitar errores con los puntos y comas de los números (por ejemplo, montos como 2.183.989,50), la IA está instruida mediante un prompt estricto para entregar el resultado en una sola línea de texto separada exclusivamente por barras verticales (|), con este formato:Cliente | N° Factura | Monto | Fecha | Proveedor

3. Transformación y Limpieza de Datos (Expressions con JavaScript)
Cómo opera: Antes de escribir en la planilla, cada campo de destino en el nodo de Google Sheets utiliza funciones nativas de JavaScript para trocear la cadena de texto que devolvió la IA.

Qué hace:

Utiliza .split('|') para dividir el texto en partes tomando la barra vertical como límite.

Utiliza el índice correspondiente ([0], [1], [2], etc.) para asignarle su valor exacto a cada columna.

Aplica .trim() para eliminar cualquier espacio en blanco sobrante al inicio o final de cada palabra u número.

4. Persistencia en Destino (Google Sheets - Append Row)
Cómo opera: Es el paso final del pipeline.

Qué hace: Toma todos los campos ya limpios y estructurados y ejecuta una inserción de fila limpia (Append Row) en tu hoja de cálculo de Google, registrando la factura de forma ordenada junto con la fecha y hora en la que fue procesada por el sistema.

---

## Lo que usé

n8n: Como la plataforma central de automatización (pipeline ETL), encargada de orquestar el flujo desde que llega el correo hasta que se guarda el registro.

Gmail (Gmail Trigger): Como el canal de entrada o disparador (trigger) para detectar automáticamente la llegada de nuevos correos con facturas adjuntas.

Google Gemini (gemini-2.0-flash): Como el motor de inteligencia artificial multimodal y OCR inteligente para leer, interpretar y extraer los datos clave directamente de las imágenes o archivos de las facturas.

Google Sheets: Como la base de datos o destino final donde se almacenan y ordenan de manera estructurada todos los comprobantes procesados.

Funciones de JavaScript / Expresiones: Integradas dentro de n8n para aplicar métodos de limpieza y separación de texto (.split('|') y .trim()) utilizando barras verticales como delimitadores seguros.

---

## Cómo está armado el proyecto

Disparador de Entrada (Gmail Trigger):

Monitorea continuamente la casilla de correo. Cuando ingresa un nuevo correo con una factura adjunta (imagen o documento), se activa de forma automática y extrae el archivo adjunto en formato binario.

Motor de Inteligencia Artificial Multimodal (Basic LLM Chain + Gemini):

El archivo binario pasa al nodo de IA utilizando el modelo gemini-2.0-flash.

La IA actúa como un OCR inteligente: "lee" visualmente el comprobante y extrae los datos clave (Cliente, Nº Factura, Monto, Fecha de emisión y Proveedor).

Mediante un prompt estructurado, se le ordena que devuelva la información en una sola línea de texto utilizando barras verticales (|) como separadores para evitar conflictos con los puntos y comas decimales de los montos.

Transformación y Limpieza de Datos (Expresiones con JavaScript):

Antes de guardarlos, los campos aplican funciones de texto nativas de n8n (.split('|') y .trim()).

Esto permite "cortar" la cadena de texto de la IA en cada barra vertical, eliminar espacios vacíos sobrantes y ubicar cada dato en su columna correspondiente de forma limpia.

Persistencia en Destino (Google Sheets):

El paso final toma todos los datos ya limpios y estructurados y ejecuta una inserción de fila (Append Row) en tu planilla de Google Sheets, registrando la factura de manera ordenada junto con la fecha y hora de procesamiento.
