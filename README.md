## Análisis de Ventas e Inventario – Flujo n8n para “Juguetería Mandy”

Este repositorio contiene un **workflow** de n8n que automatiza el análisis de ventas históricas, proyección para Q4 2024, evaluación de inventarios y generación de un informe profesional en Google Docs. Está pensado para la línea de juguetes **Mandy**.

---

## 1. Pre requisitos

* **Docker** instalado en tu máquina.
* **Git** (para clonar este repositorio, opcional).
* Cuentas y credenciales de:

  * Google Drive (para descargar el `.xlsx`)
  * Google Docs (para crear el informe)
  * OpenAI (para los nodos de IA)

---

## 2. Instalación y ejecución con Docker

1. **Clona o descarga** este repositorio en tu máquina:

   ```bash
   git clone https://github.com/tu-usuario/mandy-inventory-workflow.git
   cd mandy-inventory-workflow
   ```

2. **Crea un volumen** para persistir datos de n8n:

   ```bash
   docker volume create n8n_data
   ```

3. **Levanta** el contenedor de n8n:

   ```bash
   docker run -it --rm \
     --name n8n \
     -p 5678:5678 \
     -v n8n_data:/home/node/.n8n \
     docker.n8n.io/n8nio/n8n
   ```

   * La UI de n8n quedará accesible en `http://localhost:5678`.
   * El volumen `n8n_data` guardará tus workflows y credenciales entre reinicios.

---

## 3. Importar el workflow

1. Accede a `http://localhost:5678`.
2. En el menú lateral, elige **Workflows → Import from file**.
3. Selecciona el JSON incluido aquí (`workflow-mandy.json`).
4. Activa el workflow (switch en la esquina superior derecha).

---

## 4. Configurar credenciales

En **Settings → API Credentials** de n8n, crea:

* **Google Drive OAuth2**

  * ID de cliente y secreto obtenidos de la Consola de Google Cloud.
  * Permisos: `drive.readonly`.

* **Google Docs OAuth2**

  * ID de cliente y secreto desde Google Cloud; permisos: `docs.documents`.

* **OpenAI API**

  * Tu **API Key** de OpenAI.

Para cada credencial, introduce un nombre descriptivo y los valores (`Client ID`, `Client Secret`, `API Key`) y guarda.

---

## 5. Descripción del flujo

El workflow sigue estos bloque principales:

1. **Disparador Manual**

   * Inicia al hacer clic en “Execute workflow”.

2. **Descarga del `.xlsx` desde Google Drive**

   * Nodo **Google Drive**: baja el archivo con las hojas:

     * *Ventas 2023*
     * *Ventas Reales 2024*
     * *Inventarios*

3. **Conversión y extracción de JSON**

   * Para cada hoja: `Convert to File → Extract from File`.

4. **Normalización de claves**

   * Tres nodos **Code** convierten nombres de columnas a camelCase.

5. **Merge de los tres conjuntos**

   * Nodo **Merge** combina las tres listas en un solo array.

6. **Formateo general**

   * Nodo **Code** agrupa ventas 2023, ventas 2024 e inventarios por producto.

7. **Cálculo de proyecciones**

   * Nodo **Code** aplica la lógica de temporalidad y genera:

     * `%` mensuales
     * Proyección anual y Q4
     * Análisis de inventario vs demanda
     * Crecimiento comparativo

8. **Llamadas a IA**

   * **Asistente analista de inventario**
   * **Asistente generador de plan de acción**
   * **Asistente generador de informe**: produce el Markdown final.

9. **Sanitización y unión de Markdown**

   * Nodo **Code** elimina backticks y la palabra “markdown”, fusiona los bloques.

10. **Creación de Google Doc**

    * Nodo **HTTP Request** envía el Markdown al endpoint de generación de documentos.

---

## 6. Cómo probarlo

1. **Levanta** n8n con Docker (ver sección 2).
2. **Importa** el workflow (ver sección 3).
3. **Configura** tus credenciales (ver sección 4).
4. **Ejecuta** manualmente el workflow; observa las salidas en cada nodo.
5. Al finalizar, tendrás un **Google Doc** con el informe completo.

---

## 7. Personalizaciones

* **Endpoint de Markdown → Doc**: reemplaza la URL en el nodo **Crear Google Doc** por tu propio servicio o por la API nativa de Google Docs.
* **Modelo de IA**: ajusta `gpt-4.1` por cualquier otro que prefieras.
* **Formato y estilo**: modifica la plantilla Markdown en el nodo **Asistente generador de informe**.

---

## 8. FAQs

* **¿Cómo agregar más productos?**
  Solo actualiza la hoja de Excel en tu Google Drive; el flujo es dinámico.

* **¿Automatización periódica?**
  Sustituye el disparador Manual por un **Schedule Trigger**.

* **¿Dónde ver errores?**
  Cada nodo muestra logs en el panel derecho bajo **Execution Data**.

---

Con esto tienes todo lo necesario para desplegar y probar localmente el análisis automatizado de ventas e inventarios para la “Juguetería Mandy”. ¡Éxitos!
