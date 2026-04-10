# Guía de Integración con Google Sites y Drive

Para que la aplicación WPS Interactivo se pueda rellenar desde tu Google Sites y las notas vayan a tu Google Drive, debes seguir tres grandes pasos.

---

## PASO 1: Subir la aplicación a Internet (GitHub Pages)
Google Sites no permite alojar el código directamente, por lo que debes subir tu carpeta a un alojamiento gratuito como GitHub Pages.

1. Sube los archivos (`index.html`, `script.css`, `script.js` y las imágenes) a un repositorio público en GitHub.
2. En GitHub, ve a **Settings** > **Pages**.
3. En la sección "Build and deployment", selecciona la rama `main` y guarda.
4. En un par de minutos, GitHub te dará un enlace público (ej. `https://tu-usuario.github.io/tu-repo/`). Cópialo.

---

## PASO 2: Insertarlo en Google Sites
1. Abre tu página de Google Sites.
2. En el menú de la derecha, haz clic en **Insertar** > **Incorporar** (o *Embed*).
3. Elige la opción **Por URL**.
4. Pega el enlace de tu GitHub Pages que copiaste en el paso anterior.
5. Ajusta el tamaño de la ventana en Google Sites para que se vea el formulario completo.
6. Publica los cambios en tu Google Sites.

---

## PASO 3: Conectar a Google Drive (Google Apps Script)
He dejado preparado un botón en la aplicación ("Enviar a Google Drive"). Ahora necesitamos crear la "base de datos" que recibirá la información.

### 3.1. Crear la Hoja de Cálculo
1. Ve a tu Google Drive y crea un nuevo **Google Sheets** (Hoja de cálculo).
2. Llámalo, por ejemplo, "Resultados WPS".
3. En la fila 1, pon los siguientes encabezados en las columnas de la A a la G:
   - `Fecha de envío`
   - `Nombre Alumno`
   - `Fecha Práctica`
   - `Curso`
   - `Documento WPS`
   - `Puntuación`
   - `Prácticas`

### 3.2. Crear el Script (API)
1. En esa misma hoja de cálculo, ve al menú superior: **Extensiones** > **Apps Script**.
2. Borra todo el código que haya y pega exactamente esto:

```javascript
// La función doPost se ejecuta cuando la web de Github envía los datos
function doPost(e) {
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    var data = JSON.parse(e.postData.contents);
    
    // Las columnas de tu hoja actual: 
    // A: Evaluación | B: RA | C: CE | D: Fecha | E: Correo | F: Nombre | G: Ejercicio | H: Nota
    sheet.appendRow([
      "",                // A: Evaluación (en blanco por ahora)
      "",                // B: RA (en blanco por ahora)
      "",                // C: CE (en blanco por ahora)
      data.fecha,        // D: Fecha
      "",                // E: Correo (en blanco por ahora)
      data.nombre,       // F: Nombre
      data.ejercicio,    // G: Ejercicio (Ej: "WPS 1 (P2 TIG)")
      data.nota          // H: Nota
    ]);
    
    // Respuesta para que fetch no lance errores
    return ContentService.createTextOutput(JSON.stringify({"status": "success"}))
      .setMimeType(ContentService.MimeType.JSON);
      
  } catch(error) {
    return ContentService.createTextOutput(JSON.stringify({"error": error.toString()}))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

### 3.3. Publicar el Script
1. En la parte superior derecha de Apps Script, da clic en el botón azul **Implementar** (o *Deploy*) > **Nueva implementación**.
2. Dale al engranaje ⚙️ y selecciona **Aplicación Web**.
3. Rellena lo siguiente:
   - **Descripción**: API WPS
   - **Ejecutar como**: *Tú (tu email)*
   - **Quién tiene acceso**: *Cualquier persona* (¡Muy importante!)
4. Haz clic en **Implementar**.
5. Te pedirá **Autorizar acceso**. Sigue los pasos (Google te dirá que "no es seguro", pulsa en "Configuración avanzada" e ir de todos modos).
6. Al final, te dará y seleccionas copiar la **URL de la aplicación web** (`https://script.google.com/macros/s/.../exec`). ¡Guarda bien ese enlace!

---

## PASO 4: Pegar el enlace en tu código
1. Abre el archivo `script.js` en tu ordenador.
2. Ve a la línea donde insertamos el código nuevo (aproximadamente la línea `1030`, la verás en un bloque llamado `ENVÍO A GOOGLE DRIVE`).
3. Busca esta línea:
   ```javascript
   const GAS_URL = "URL_DE_TU_SCRIPT_AQUI"; 
   ```
4. Reemplaza `"URL_DE_TU_SCRIPT_AQUI"` por la URL larga real que copiaste en el paso anterior.
5. Guarda el archivo, vuelve a subirlo a GitHub para que los cambios surtan efecto en tu Google Sites.

¡Y ya estaría! Cuando los alumnos usen la app embebida en Google Sites y le den al botón, verás los datos aparecer mágicamente en tu hoja de Google Drive.
