# Calendario App (Google Apps Script)

Aplicación web simple construida con Google Apps Script para gestionar solicitudes de eventos (vacaciones, reuniones, etc.), aprobarlas/rechazarlas y publicarlas automáticamente en Google Calendar.

## Características

* **Registro de Eventos:** Los usuarios pueden enviar solicitudes de eventos especificando tipo, título, fechas, descripción, calendario de destino y un archivo adjunto opcional.
* **Flujo de Aprobación:** Un usuario "Validador" designado recibe notificaciones por correo y puede aprobar o rechazar eventos pendientes desde la aplicación.
* **Integración con Google Calendar:** Los eventos aprobados se crean automáticamente como eventos de día completo en el calendario de Google seleccionado.
* **Integración con Google Drive:** Los archivos adjuntos se guardan en una carpeta específica de Google Drive.
* **Interfaz de Usuario:**
    * Vista de tabla para eventos Pendientes, Aprobados y Rechazados (usando DataTables).
    * Vista de Calendario para eventos Aprobados (usando FullCalendar).
    * Lista de próximos eventos aprobados.
    * Formulario modal para agregar/editar eventos.
    * Notificaciones visuales (usando SweetAlert2).
* **Roles de Usuario:**
    * **Solicitante:** Puede registrar eventos, ver sus propios eventos y editar/eliminar eventos pendientes o rechazados.
    * **Validador:** Puede ver todos los eventos, aprobar/rechazar eventos pendientes, y editar/eliminar cualquier evento.

## Estructura del Proyecto

* **`Code.gs`:** Lógica del lado del servidor (Apps Script). Maneja la interacción con Google Sheets, Calendar, Drive y las solicitudes del cliente.
* **`Index.html`:** Estructura principal de la página HTML.
* **`CSS.html`:** Estilos CSS para la interfaz de usuario (usando Bootstrap 5).
* **`JavaScript.html`:** Lógica del lado del cliente (JavaScript/jQuery). Maneja la interacción con el usuario, DataTables, FullCalendar, SweetAlert2 y las llamadas al servidor (`google.script.run`).
* **`env.gs`:** Archivo para variables de configuración global (ID de hoja, ID de carpeta, email validador, etc.). *Nota: Este archivo no estaba en la lista inicial pero es mencionado en el `README.md` original.*

## Configuración

1.  **Crear Hoja de Cálculo:** Crea una nueva Google Sheet. Nómbrala como desees.
2.  **Abrir Editor de Scripts:** Dentro de la hoja, ve a `Extensiones > Apps Script`.
3.  **Copiar Código:**
    * Copia el contenido de `Code.gs` en el archivo `Code.gs` del editor.
    * Crea nuevos archivos HTML (`Archivo > Nuevo > Archivo HTML`) para `Index.html`, `CSS.html` y `JavaScript.html` y copia el contenido correspondiente en cada uno.
    * (Opcional pero recomendado) Crea un nuevo archivo de script (`Archivo > Nuevo > Script`) llamado `env.gs` y copia el contenido de `env.gs.txt`.
4.  **Configurar Variables en `env.gs` (o `Code.gs` si no creaste `env.gs`):**
    * `SPREADSHEET_ID`: Generalmente se obtiene automáticamente con `SpreadsheetApp.getActiveSpreadsheet().getId()`.
    * `SHEET_NAME`: Asegúrate de que coincida EXACTAMENTE con el nombre de tu hoja de cálculo (la pestaña dentro del archivo). Por defecto es "Eventos".
    * `VALIDATOR_EMAIL`: Cambia el correo de ejemplo por el correo electrónico del usuario que actuará como validador.
    * `DRIVE_FOLDER_ID`: **MUY IMPORTANTE:**
        * Crea una carpeta en tu Google Drive donde se guardarán los adjuntos.
        * Abre la carpeta en Drive.
        * Copia el ID de la carpeta de la URL (la cadena de caracteres después de `folders/`).
        * Pega **SOLO el ID** en lugar del valor de ejemplo (`"123id"` o similar).
        * **Asegúrate de que la cuenta que ejecuta el script tenga permisos de Editor sobre esta carpeta.**
    * `AVAILABLE_CALENDARS`: Modifica la lista para incluir los calendarios donde se podrán publicar eventos.
        * `name`: Nombre que verá el usuario en el desplegable.
        * `id`: El ID del calendario de Google Calendar (puedes encontrarlo en la configuración del calendario en Google Calendar). **El usuario Validador DEBE tener permisos para "Hacer cambios en eventos" en CADA uno de estos calendarios.**
    * `EVENT_TYPES`: Ajusta la lista de tipos de evento según tus necesidades.
    * `UPCOMING_EVENT_DAYS`: Número de días hacia adelante para mostrar en "Próximos Eventos".
    * `MAX_UPCOMING_EVENTS`: Número máximo de eventos a mostrar en "Próximos Eventos".
5.  **Guardar Proyecto:** Guarda los cambios en el editor de scripts (icono de disquete).

## Despliegue

1.  **Crear Despliegue:** En el editor de Apps Script, haz clic en `Desplegar > Nuevo Despliegue`.
2.  **Tipo de Despliegue:** Selecciona `Aplicación Web`.
3.  **Configuración:**
    * **Descripción:** (Opcional) Añade una descripción.
    * **Ejecutar como:** Selecciona `Usuario que accede a la aplicación`. (Esto es importante para que los permisos se basen en quién usa la app).
    * **Quién tiene acceso:** Selecciona quién podrá usar la aplicación (por ejemplo, `Cualquier usuario con cuenta de Google` o usuarios específicos dentro de tu dominio). **Importante:** Para que otros usuarios puedan usarla, necesitas seleccionar una opción que les dé acceso.
4.  **Desplegar:** Haz clic en `Desplegar`.
5.  **Autorización:** La primera vez (y posiblemente después de cambios significativos), Google pedirá autorización a cada usuario para que el script acceda a sus datos en su nombre (Sheets, Calendar, Drive, enviar correos). Deben revisar los permisos y hacer clic en `Permitir`. Es posible que vean una advertencia de "Aplicación no verificada"; si confían en el código, pueden proceder haciendo clic en "Configuración avanzada" y luego "Ir a [nombre del proyecto] (no seguro)".
6.  **URL de la Aplicación Web:** Copia la URL proporcionada. Esta es la dirección para acceder a tu Calendario App.

## Uso

1.  **Acceder:** Abre la URL de la aplicación web desplegada.
2.  **Registrar Evento:** Haz clic en "Registrar Nuevo Evento", completa el formulario y guarda.
3.  **Validación (Validador):** El validador recibirá un correo. Al acceder a la aplicación, verá los eventos pendientes en su tabla y podrá aprobarlos o rechazarlos usando los botones de acción.
4.  **Visualización:** Todos los usuarios pueden ver eventos aprobados y rechazados en sus respectivas tablas y en la vista de calendario. Los usuarios solo pueden editar/eliminar sus propios eventos si están pendientes o rechazados (a menos que sean el validador).

## Documentación de Funciones (`Code.gs`)

Aquí se describen las principales funciones del lado del servidor:

* **`doGet(e)`**
    * **Propósito:** Responde a las solicitudes GET sirviendo la interfaz de usuario principal (`Index.html`).
    * **Parámetros:** `e` (Objeto de evento GET).
    * **Retorna:** `HtmlOutput` - El servicio HTML para mostrar la página.

* **`include(filename)`**
    * **Propósito:** Incluye el contenido de otros archivos HTML (como CSS o JavaScript) dentro del HTML principal.
    * **Parámetros:** `filename` (String) - Nombre del archivo HTML a incluir (sin extensión).
    * **Retorna:** `String` - El contenido del archivo HTML.

* **`getUserInfo()`**
    * **Propósito:** Obtiene el email y rol ('validator' o 'requester') del usuario que accede a la aplicación, basado en `VALIDATOR_EMAIL`.
    * **Parámetros:** Ninguno.
    * **Retorna:** `Object|null` - Objeto `{ email: string, role: string }` o `null` si no se identifica.

* **`getAvailableCalendars()`**
    * **Propósito:** Devuelve la lista de calendarios configurados en `AVAILABLE_CALENDARS` que son válidos y accesibles.
    * **Parámetros:** Ninguno.
    * **Retorna:** `Array<Object>` - Array de objetos `{ name: string, id: string }`.

* **`getEventTypes()`**
    * **Propósito:** Devuelve la lista de tipos de evento predefinidos en `EVENT_TYPES`.
    * **Parámetros:** Ninguno.
    * **Retorna:** `Array<String>` - La lista de tipos de evento.

* **`getAllVisibleEventsForUser()`**
    * **Propósito:** Obtiene TODOS los eventos de la hoja de cálculo. *Nota: Actualmente no filtra por usuario, devuelve todos los eventos.*
    * **Parámetros:** Ninguno.
    * **Retorna:** `Object` - Objeto `{ data: Array<Object> }` con la lista de eventos formateados para DataTables.

* **`getUpcomingApprovedEvents()`**
    * **Propósito:** Obtiene los eventos aprobados que ocurren próximamente (dentro de `UPCOMING_EVENT_DAYS`) hasta un máximo de `MAX_UPCOMING_EVENTS`.
    * **Parámetros:** Ninguno.
    * **Retorna:** `Array<Object>` - Array de objetos de evento próximos, ordenados por fecha.

* **`getEventDetailsForEdit(uniqueId)`**
    * **Propósito:** Obtiene los detalles completos de un evento específico por su ID para edición o visualización, incluyendo notas del validador.
    * **Parámetros:** `uniqueId` (String) - El ID único del evento.
    * **Retorna:** `Object|null` - Objeto con datos del evento formateados, o `null` si no se encuentra o hay error.

* **`addEvent(formData, fileData)`**
    * **Propósito:** Agrega un nuevo evento a la hoja de cálculo, guarda el adjunto (si existe) en Drive y notifica al validador.
    * **Parámetros:**
        * `formData` (Object) - Datos del formulario del cliente.
        * `fileData` (Object|null) - Información del archivo adjunto (`{blob, fileName, mimeType}`) o `null`.
    * **Retorna:** `Object` - Resultado `{success: boolean, message: string}`.

* **`respondToEventRequest(action, uniqueId, notes)`**
    * **Propósito:** Procesa la acción de aprobar o rechazar un evento pendiente. Actualiza la hoja, crea/elimina evento en GCal (si aprueba) y notifica al solicitante.
    * **Parámetros:**
        * `action` (String) - 'approve' o 'reject'.
        * `uniqueId` (String) - ID del evento.
        * `notes` (String) - Notas del validador (requeridas para rechazar).
    * **Retorna:** `Object` - Resultado `{success: boolean, message: string}`.

* **`updateEvent(formData, fileData)`**
    * **Propósito:** Actualiza un evento existente en la hoja, GCal (si estaba aprobado) y maneja adjuntos (elimina/reemplaza). Permite reenviar eventos rechazados.
    * **Parámetros:**
        * `formData` (Object) - Datos del formulario (incluye `id` y `removeAttachment`).
        * `fileData` (Object|null) - Nuevo archivo adjunto o `null`.
    * **Retorna:** `Object` - Resultado `{success: boolean, message: string}`.

* **`deleteEvent(uniqueId)`**
    * **Propósito:** Elimina un evento de la hoja, GCal (si estaba aprobado) y el adjunto de Drive.
    * **Parámetros:** `uniqueId` (String) - ID del evento a eliminar.
    * **Retorna:** `Object` - Resultado `{success: boolean, message: string}`.

* **`getFileUrl(fileId)`**
    * **Propósito:** Obtiene la URL de visualización/descarga de un archivo en Google Drive por su ID.
    * **Parámetros:** `fileId` (String) - El ID del archivo.
    * **Retorna:** `String|null` - La URL del archivo o `null` si hay error.

* **`findEventRowDataById(sheet, uniqueId)`** (Auxiliar Interna)
    * **Propósito:** Busca una fila por ID y devuelve su índice y datos parseados.
    * **Parámetros:** `sheet` (Sheet), `uniqueId` (String).
    * **Retorna:** `Object` - `{rowIndex: number, eventData: object|null}`.

* **`ensureSheetHeaders(sheet)`** (Auxiliar Interna)
    * **Propósito:** Asegura que la hoja tenga los encabezados correctos.
    * **Parámetros:** `sheet` (Sheet).
    * **Retorna:** Nada.

* **`getDriveFolderById(folderId)`** (Auxiliar Interna)
    * **Propósito:** Obtiene un objeto `Folder` de Drive por su ID, con manejo de errores.
    * **Parámetros:** `folderId` (String).
    * **Retorna:** `Folder` - El objeto Folder.
    * **Lanza:** `Error` si no se encuentra o no es accesible.

* **`parseDateObject(dateInput)`** (Auxiliar Interna)
    * **Propósito:** Parsea una entrada a un objeto `Date` válido, interpretando 'YYYY-MM-DD' en la zona horaria del script.
    * **Parámetros:** `dateInput` (*).
    * **Retorna:** `Date|null` - Objeto Date o `null`.

* **`formatISODate(dateObject)`** (Auxiliar Interna)
    * **Propósito:** Formatea un objeto Date a 'YYYY-MM-DD' usando la zona horaria del script.
    * **Parámetros:** `dateObject` (Date).
    * **Retorna:** `String` - Fecha formateada o "".

* **`formatDisplayDate(dateObject)`** (Auxiliar Interna)
    * **Propósito:** Formatea un objeto Date a 'DD/MM/YYYY' usando la zona horaria del script.
    * **Parámetros:** `dateObject` (Date).
    * **Retorna:** `String` - Fecha formateada o texto de error.

* **`getCalendarNameById(calendarId)`** (Auxiliar Interna)
    * **Propósito:** Obtiene el nombre de un calendario por su ID.
    * **Parámetros:** `calendarId` (String).
    * **Retorna:** `String` - Nombre del calendario o ID si falla.

* **`parseAndFormatDateISO(dateInput)`** (Auxiliar Interna)
    * **Propósito:** Combina `parseDateObject` y `formatISODate`.
    * **Parámetros:** `dateInput` (*).
    * **Retorna:** `String` - Fecha 'YYYY-MM-DD' o "".

## Dependencias (Lado del Cliente)

La interfaz utiliza las siguientes librerías externas (cargadas vía CDN):

* Bootstrap 5 (CSS & JS)
* jQuery
* DataTables (con integración Bootstrap 5)
* Font Awesome 6
* FullCalendar 6
* SweetAlert2
* Luxon (para manejo de fechas/horas)

## Notas

* El script se ejecuta bajo la autoridad del usuario especificado en el despliegue. Si se configura como "Usuario que accede a la aplicación", cada usuario deberá autorizarlo individualmente.
* Los permisos de acceso a los datos (eventos en las tablas) dependen de los permisos de compartición de la Hoja de Cálculo subyacente para el usuario que accede. El Validador necesita permisos de Editor en la Hoja para gestionar todo. Los solicitantes necesitan al menos permisos de Lector.
* El Validador necesita permisos de "Hacer cambios a eventos" en todos los calendarios listados en `AVAILABLE_CALENDARS`.
* La cuenta que ejecuta el script (si se despliega como "Yo") o la cuenta del usuario (si se despliega como "Usuario que accede") necesita permisos de Editor sobre la carpeta de Drive especificada en `DRIVE_FOLDER_ID`.

