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

## Configuración

1.  **Crear Hoja de Cálculo:** Crea una nueva Google Sheet. Nómbrala como desees.
2.  **Abrir Editor de Scripts:** Dentro de la hoja, ve a `Extensiones > Apps Script`.
3.  **Copiar Código:**
    * Copia el contenido de `Code.gs` en el archivo `Code.gs` del editor.
    * Crea nuevos archivos HTML (`Archivo > Nuevo > Archivo HTML`) para `Index.html`, `CSS.html` y `JavaScript.html` y copia el contenido correspondiente en cada uno.
4.  **Configurar Variables en `Code.gs`:**
    * `SHEET_NAME`: Asegúrate de que coincida EXACTAMENTE con el nombre de tu hoja de cálculo (la pestaña dentro del archivo). Por defecto es "Eventos".
    * `VALIDATOR_EMAIL`: Cambia `"david.glzcr@gmail.com"` por el correo electrónico del usuario que actuará como validador.
    * `DRIVE_FOLDER_ID`: **MUY IMPORTANTE:**
        * Crea una carpeta en tu Google Drive donde se guardarán los adjuntos.
        * Abre la carpeta en Drive.
        * Copia el ID de la carpeta de la URL (la cadena de caracteres después de `folders/`).
        * Pega **SOLO el ID** en lugar de `"1KBI6cUngbrV3a_YAojaORS67bQP64CGo"` (o el placeholder que haya).
        * **Asegúrate de que la cuenta que ejecuta el script tenga permisos de Editor sobre esta carpeta.**
    * `AVAILABLE_CALENDARS`: Modifica la lista para incluir los calendarios donde se podrán publicar eventos.
        * `name`: Nombre que verá el usuario en el desplegable.
        * `id`: El ID del calendario de Google Calendar (puedes encontrarlo en la configuración del calendario en Google Calendar). **El usuario Validador DEBE tener permisos para "Hacer cambios en eventos" en CADA uno de estos calendarios.**
    * `EVENT_TYPES`: Ajusta la lista de tipos de evento según tus necesidades.
5.  **Guardar Proyecto:** Guarda los cambios en el editor de scripts (icono de disquete).

## Despliegue

1.  **Crear Despliegue:** En el editor de Apps Script, haz clic en `Desplegar > Nuevo Despliegue`.
2.  **Tipo de Despliegue:** Selecciona `Aplicación Web`.
3.  **Configuración:**
    * **Descripción:** (Opcional) Añade una descripción.
    * **Ejecutar como:** Selecciona `Yo`.
    * **Quién tiene acceso:** Selecciona quién podrá usar la aplicación (por ejemplo, `Cualquier usuario con cuenta de Google` o `Solo yo`, o usuarios específicos dentro de tu dominio si aplica). **Importante:** Para que otros usuarios puedan usarla, necesitas seleccionar una opción que les dé acceso.
4.  **Desplegar:** Haz clic en `Desplegar`.
5.  **Autorización:** La primera vez (y posiblemente después de cambios significativos), Google te pedirá autorización para que el script acceda a tus datos (Sheets, Calendar, Drive, enviar correos). Revisa los permisos y haz clic en `Permitir`. Es posible que veas una advertencia de "Aplicación no verificada"; si confías en el código (que tú mismo has puesto), puedes proceder haciendo clic en "Configuración avanzada" y luego "Ir a [nombre del proyecto] (no seguro)".
6.  **URL de la Aplicación Web:** Copia la URL proporcionada. Esta es la dirección para acceder a tu Calendario App.

## Uso

1.  **Acceder:** Abre la URL de la aplicación web desplegada.
2.  **Registrar Evento:** Haz clic en "Registrar Nuevo Evento", completa el formulario y guarda.
3.  **Validación (Validador):** El validador recibirá un correo. Al acceder a la aplicación, verá los eventos pendientes en su tabla y podrá aprobarlos o rechazarlos usando los botones de acción.
4.  **Visualización:** Todos los usuarios pueden ver eventos aprobados y rechazados en sus respectivas tablas y en la vista de calendario. Los usuarios solo pueden editar/eliminar sus propios eventos si están pendientes o rechazados (a menos que sean el validador).

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

* El script se ejecuta bajo la autoridad del usuario especificado en el despliegue ("Ejecutar como: Yo"). Por lo tanto, este usuario necesita los permisos necesarios sobre la Hoja de Cálculo, la carpeta de Drive y los Calendarios listados.
* Los permisos de visualización de datos en las tablas dependen de los permisos de compartición de la Hoja de Cálculo subyacente para el usuario que accede a la aplicación. El Validador debería tener al menos permisos de Editor en la Hoja para ver y gestionar todo.
