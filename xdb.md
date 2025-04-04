
# xdB - Dokumentacja Techniczna

## Wprowadzenie (Introduction)

xdB to biblioteka Node.js do zarządzania prostymi bazami danych opartymi na plikach JSON. Umożliwia operacje CRUD (Create, Read, Update, Delete) na plikach i rekordach w nich zawartych, a także zarządzanie katalogami. Biblioteka wykorzystuje asynchroniczne operacje na systemie plików i zapewnia podstawową kontrolę współbieżności za pomocą blokad plików.

(xdB is a Node.js library for managing simple JSON file-based databases. It allows CRUD operations on files and records within them, as well as directory management. The library uses asynchronous file system operations and provides basic concurrency control using file locks.)

## Kody Błędów (Error Codes)

Biblioteka definiuje zestaw specyficznych kodów błędów, aby ułatwić obsługę błędów.

(The library defines a set of specific error codes to facilitate error handling.)

```javascript
const XDB_ERROR_CODES = {
  /** Indicates that a specified file was not found. (Wskazuje, że określony plik nie został znaleziony.) */
  FILE_NOT_FOUND: 'XDB_FILE_NOT_FOUND',
  /** Indicates that a specified directory was not found. (Wskazuje, że określony katalog nie został znaleziony.) */
  DIR_NOT_FOUND: 'XDB_DIR_NOT_FOUND',
  /** Indicates a general Input/Output error occurred. (Wskazuje, że wystąpił ogólny błąd Wejścia/Wyjścia.) */
  IO_ERROR: 'XDB_IO_ERROR',
  /** Indicates that the content of a file is not valid JSON. (Wskazuje, że zawartość pliku nie jest poprawnym JSON-em.) */
  INVALID_JSON: 'XDB_INVALID_JSON',
  /** Indicates that a record with a specific ID was not found. (Wskazuje, że rekord o określonym ID nie został znaleziony.) */
  RECORD_NOT_FOUND: 'XDB_RECORD_NOT_FOUND',
  /** Indicates an attempt to add a record with an ID that already exists. (Wskazuje na próbę dodania rekordu z ID, które już istnieje.) */
  DUPLICATE_RECORD: 'XDB_DUPLICATE_RECORD', // Kept for potential internal use
  /** Indicates an attempt to add a record with an ID that already exists (alternative code, used by tests). (Wskazuje na próbę dodania rekordu z ID, które już istnieje (alternatywny kod, używany przez testy).) */
  RECORD_EXISTS: 'XDB_RECORD_EXISTS',
  /** Generic error code for failed operations not covered by other codes. (Ogólny kod błędu dla nieudanych operacji nieobjętych innymi kodami.) */
  OPERATION_FAILED: 'XDB_OPERATION_FAILED',
};
```

**Użycie (Usage):**

```javascript
import xdB, { XDB_ERROR_CODES } from './xdb.js';

try {
  // Some xdB operation
} catch (error) {
  if (error.code === XDB_ERROR_CODES.FILE_NOT_FOUND) {
    console.error("Plik nie istnieje:", error.message);
  } else {
    console.error("Wystąpił błąd xdB:", error.code, error.message);
  }
}
```

## Konfiguracja (Configuration)

### `xdB.config(options)`

Konfiguruje ustawienia biblioteki xdB.

(Configures the xdB library settings.)

*   **Parametry (Parameters):**
    *   `options` (`object`): Opcje konfiguracyjne. (Configuration options.)
        *   `basePath` (`string`, opcjonalnie): Absolutna lub względna ścieżka używana jako baza dla wszystkich operacji plikowych. (The absolute or relative path to use as the base for all file operations.)
*   **Przykład (Example):**
    ```javascript
    import xdB from './xdb.js';
    import path from 'path';

    // Ustawienie niestandardowej ścieżki bazowej
    // Set a custom base path
    xdB.config({ basePath: path.resolve('./my_database_data') });
    ```

### `xdB.getBasePath()`

Pobiera aktualnie skonfigurowaną ścieżkę bazową dla operacji plikowych.

(Gets the currently configured base path for file operations.)

*   **Zwraca (Returns):**
    *   `string`: Absolutna ścieżka bazowa. (The absolute base path.)
*   **Przykład (Example):**
    ```javascript
    const currentBasePath = xdB.getBasePath();
    console.log("Aktualna ścieżka bazowa:", currentBasePath);
    // Output: Aktualna ścieżka bazowa: /path/to/your/project/my_database_data
    ```

## Operacje na Katalogach (`xdB.dir`) (Directory Operations)

### `xdB.dir.add(dirPath)`

Tworzy katalog, włączając wszelkie niezbędne katalogi nadrzędne.

(Creates a directory, including any necessary parent directories.)

*   **Parametry (Parameters):**
    *   `dirPath` (`string`): Ścieżka do katalogu do utworzenia (względna do `basePath`). (The path to the directory to create (relative to `basePath`).)
*   **Zwraca (Returns):**
    *   `Promise<{path: string}>`: Obietnica rozwiązująca się obiektem zawierającym absolutną ścieżkę utworzonego katalogu. (A promise resolving with an object containing the absolute path of the created directory.)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku niepowodzenia tworzenia katalogu. (On directory creation failure.)
*   **Przykład (Example):**
    ```javascript
    try {
      const result = await xdB.dir.add('users/profiles');
      console.log("Utworzono katalog:", result.path);
      // Output: Utworzono katalog: /path/to/base/users/profiles
    } catch (error) {
      console.error("Błąd tworzenia katalogu:", error.message);
    }
    ```

### `xdB.dir.del(dirPath)`

Usuwa katalog rekurencyjnie.

(Deletes a directory recursively.)

*   **Parametry (Parameters):**
    *   `dirPath` (`string`): Ścieżka do katalogu do usunięcia (względna do `basePath`). (The path to the directory to delete (relative to `basePath`).)
*   **Zwraca (Returns):**
    *   `Promise<{path: string}>`: Obietnica rozwiązująca się obiektem zawierającym absolutną ścieżkę usuniętego katalogu. (A promise resolving with an object containing the absolute path of the deleted directory.)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.DIR_NOT_FOUND}`: Jeśli katalog nie istnieje. (If the directory doesn't exist.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku innych błędów usuwania. (On other deletion failures.)
*   **Przykład (Example):**
    ```javascript
    try {
      const result = await xdB.dir.del('temporary_files');
      console.log("Usunięto katalog:", result.path);
    } catch (error) {
      if (error.code === XDB_ERROR_CODES.DIR_NOT_FOUND) {
        console.warn("Katalog do usunięcia nie istnieje.");
      } else {
        console.error("Błąd usuwania katalogu:", error.message);
      }
    }
    ```

### `xdB.dir.rename(oldPath, newPath)`

Zmienia nazwę lub przenosi katalog.

(Renames or moves a directory.)

*   **Parametry (Parameters):**
    *   `oldPath` (`string`): Obecna ścieżka katalogu (względna do `basePath`). (The current path of the directory (relative to `basePath`).)
    *   `newPath` (`string`): Nowa ścieżka dla katalogu (względna do `basePath`). (The new path for the directory (relative to `basePath`).)
*   **Zwraca (Returns):**
    *   `Promise<{oldPath: string, newPath: string}>`: Obietnica rozwiązująca się obiektem zawierającym starą i nową absolutną ścieżkę. (A promise resolving with an object containing the old and new absolute paths.)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.DIR_NOT_FOUND}`: Jeśli katalog źródłowy nie istnieje. (If the source directory doesn't exist.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku innych błędów zmiany nazwy/przenoszenia. (On other rename/move failures.)
*   **Przykład (Example):**
    ```javascript
    try {
      const result = await xdB.dir.rename('data/old_logs', 'data/archived_logs');
      console.log(`Przeniesiono katalog z ${result.oldPath} do ${result.newPath}`);
    } catch (error) {
      console.error("Błąd przenoszenia katalogu:", error.message);
    }
    ```

## Operacje Przenoszenia (`xdB.move`) (Move Operations)

### `xdB.move.file(sourcePath, targetPath)`

Przenosi lub zmienia nazwę pliku. Zapewnia istnienie katalogu docelowego. Automatycznie dodaje rozszerzenie `.json`, jeśli go brakuje.

(Moves or renames a file. Ensures the target directory exists. Automatically adds the `.json` extension if missing.)

*   **Parametry (Parameters):**
    *   `sourcePath` (`string`): Obecna ścieżka pliku (względna do `basePath`, `.json` dodawane, jeśli brakuje). (The current path of the file (relative to `basePath`, `.json` added if missing).)
    *   `targetPath` (`string`): Nowa ścieżka dla pliku (względna do `basePath`, `.json` dodawane, jeśli brakuje). (The new path for the file (relative to `basePath`, `.json` added if missing).)
*   **Zwraca (Returns):**
    *   `Promise<{source: string, target: string}>`: Obietnica rozwiązująca się obiektem zawierającym źródłową i docelową absolutną ścieżkę. (A promise resolving with an object containing the source and target absolute paths.)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.FILE_NOT_FOUND}`: Jeśli plik źródłowy nie istnieje. (If the source file doesn't exist.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku innych błędów przenoszenia. (On other move failures.)
*   **Przykład (Example):**
    ```javascript
    try {
      // Przeniesienie pliku 'users.json' do 'backup/users_old.json'
      // Move 'users.json' to 'backup/users_old.json'
      const result = await xdB.move.file('users', 'backup/users_old'); // .json zostanie dodane automatycznie
      console.log(`Przeniesiono plik z ${result.source} do ${result.target}`);
    } catch (error) {
      console.error("Błąd przenoszenia pliku:", error.message);
    }
    ```

### `xdB.move.dir(sourcePath, targetPath)`

Przenosi lub zmienia nazwę katalogu. Zapewnia istnienie nadrzędnego katalogu docelowego. (Identyczne jak `xdB.dir.rename`).

(Moves or renames a directory. Ensures the target parent directory exists. (Identical to `xdB.dir.rename`).)

*   **Parametry (Parameters):**
    *   `sourcePath` (`string`): Obecna ścieżka katalogu (względna do `basePath`). (The current path of the directory (relative to `basePath`).)
    *   `targetPath` (`string`): Nowa ścieżka dla katalogu (względna do `basePath`). (The new path for the directory (relative to `basePath`).)
*   **Zwraca (Returns):**
    *   `Promise<{source: string, target: string}>`: Obietnica rozwiązująca się obiektem zawierającym źródłową i docelową absolutną ścieżkę. (A promise resolving with an object containing the source and target absolute paths.)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.DIR_NOT_FOUND}`: Jeśli katalog źródłowy nie istnieje. (If the source directory doesn't exist.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku innych błędów przenoszenia. (On other move failures.)
*   **Przykład (Example):**
    ```javascript
    try {
      const result = await xdB.move.dir('pending_uploads', 'processed_uploads');
      console.log(`Przeniesiono katalog z ${result.source} do ${result.target}`);
    } catch (error) {
      console.error("Błąd przenoszenia katalogu:", error.message);
    }
    ```

## Operacje Edycji (`xdB.edit`) (Edit Operations)

### `xdB.edit.all(filePath, newData)`

Nadpisuje całą zawartość pliku JSON atomowo. Tworzy plik, jeśli nie istnieje. Automatycznie dodaje rozszerzenie `.json`, jeśli go brakuje.

(Overwrites the entire content of a JSON file atomically. Creates the file if it doesn't exist. Automatically adds the `.json` extension if missing.)

*   **Parametry (Parameters):**
    *   `filePath` (`string`): Ścieżka do pliku JSON (względna do `basePath`, `.json` dodawane, jeśli brakuje). (Path to the JSON file (relative to `basePath`, `.json` added if missing).)
    *   `newData` (`Array` | `object`): Nowe dane (tablica lub obiekt) do zapisania w pliku. (The new data (array or object) to write to the file.)
*   **Zwraca (Returns):**
    *   `Promise<{path: string}>`: Obietnica rozwiązująca się obiektem zawierającym absolutną ścieżkę zapisanego pliku. (A promise resolving with an object containing the absolute path of the written file.)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku błędów zapisu. (On write failures.)
*   **Przykład (Example):**
    ```javascript
    const newSettings = { theme: 'dark', notifications: false };
    try {
      const result = await xdB.edit.all('config/settings', newSettings); // .json zostanie dodane
      console.log("Zapisano nowe ustawienia do:", result.path);
    } catch (error) {
      console.error("Błąd zapisu pliku:", error.message);
    }
    ```

### `xdB.edit.id(filePath, id, newRecord)`

Edytuje określony rekord w pliku JSON (który musi być tablicą obiektów) identyfikowany przez jego ID. Automatycznie dodaje rozszerzenie `.json`, jeśli go brakuje.

(Edits a specific record within a JSON file (which must be an array of objects) identified by its ID. Automatically adds the `.json` extension if missing.)

*   **Parametry (Parameters):**
    *   `filePath` (`string`): Ścieżka do pliku JSON (względna do `basePath`, `.json` dodawane, jeśli brakuje). (Path to the JSON file (relative to `basePath`, `.json` added if missing).)
    *   `id` (`number`): ID rekordu do edycji. (The ID of the record to edit.)
    *   `newRecord` (`object`): Obiekt zawierający pola do zaktualizowania w rekordzie. **Nie** musi zawierać `id`. (An object containing the fields to update in the record. Does **not** need to include `id`.)
*   **Zwraca (Returns):**
    *   `Promise<{path: string, record: object}>`: Obietnica rozwiązująca się obiektem zawierającym ścieżkę pliku i zaktualizowany rekord. (A promise resolving with an object containing the file path and the updated record.)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.FILE_NOT_FOUND}`: Jeśli plik nie istnieje. (If the file doesn't exist.)
    *   `Error & {code: XDB_ERROR_CODES.INVALID_JSON}`: Jeśli plik zawiera nieprawidłowy JSON. (If the file contains invalid JSON.)
    *   `Error & {code: XDB_ERROR_CODES.RECORD_NOT_FOUND}`: Jeśli rekord o podanym ID nie został znaleziony. (If a record with the specified ID was not found.)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: Jeśli `id` nie jest liczbą lub `newRecord` nie jest obiektem. (If `id` is not a number or `newRecord` is not an object.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku innych błędów zapisu. (On other write failures.)
*   **Przykład (Example):**
    ```javascript
    const updates = { email: 'new.email@example.com', status: 'active' };
    try {
      const result = await xdB.edit.id('users', 123, updates); // Edycja rekordu w users.json
      console.log(`Zaktualizowano rekord ${result.record.id} w pliku ${result.path}`);
      console.log("Nowe dane rekordu:", result.record);
    } catch (error) {
      console.error("Błąd edycji rekordu:", error.message, error.code);
    }
    ```

## Operacje Usuwania (`xdB.del`) (Delete Operations)

### `xdB.del.all(filePath)`

Usuwa (opróżnia) zawartość pliku JSON atomowo, zastępując ją pustą tablicą `[]`. **Nie usuwa samego pliku.** Automatycznie dodaje rozszerzenie `.json`, jeśli go brakuje.

(Deletes (empties) the content of a JSON file atomically by replacing it with an empty array `[]`. **Does not delete the file itself.** Automatically adds the `.json` extension if missing.)

*   **Parametry (Parameters):**
    *   `filePath` (`string`): Ścieżka do pliku JSON do opróżnienia (względna do `basePath`, `.json` dodawane, jeśli brakuje). (Path to the JSON file to empty (relative to `basePath`, `.json` added if missing).)
*   **Zwraca (Returns):**
    *   `Promise<{path: string}>`: Obietnica rozwiązująca się obiektem zawierającym absolutną ścieżkę opróżnionego pliku. (A promise resolving with an object containing the absolute path of the emptied file.)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.FILE_NOT_FOUND}`: Jeśli plik nie istnieje. (If the file doesn't exist.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku innych błędów zapisu. (On other write failures.)
*   **Przykład (Example):**
    ```javascript
    try {
      const result = await xdB.del.all('logs/today_log'); // Opróżnia logs/today_log.json
      console.log("Opróżniono plik:", result.path);
    } catch (error) {
      console.error("Błąd opróżniania pliku:", error.message);
    }
    ```

### `xdB.del.id(filePath, id)`

Usuwa określony rekord z pliku JSON (który musi być tablicą obiektów) identyfikowany przez jego ID atomowo. Automatycznie dodaje rozszerzenie `.json`, jeśli go brakuje.

(Deletes a specific record from a JSON file (which must be an array of objects) identified by its ID atomically. Automatically adds the `.json` extension if missing.)

*   **Parametry (Parameters):**
    *   `filePath` (`string`): Ścieżka do pliku JSON (względna do `basePath`, `.json` dodawane, jeśli brakuje). (Path to the JSON file (relative to `basePath`, `.json` added if missing).)
    *   `id` (`number`): ID rekordu do usunięcia. (The ID of the record to delete.)
*   **Zwraca (Returns):**
    *   `Promise<{path: string, deletedId: number}>`: Obietnica rozwiązująca się obiektem zawierającym ścieżkę pliku i ID usuniętego rekordu. (A promise resolving with an object containing the file path and the ID of the deleted record.)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.FILE_NOT_FOUND}`: Jeśli plik nie istnieje. (If the file doesn't exist.)
    *   `Error & {code: XDB_ERROR_CODES.INVALID_JSON}`: Jeśli plik zawiera nieprawidłowy JSON. (If the file contains invalid JSON.)
    *   `Error & {code: XDB_ERROR_CODES.RECORD_NOT_FOUND}`: Jeśli rekord o podanym ID nie został znaleziony. (If a record with the specified ID was not found.)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: Jeśli `id` nie jest liczbą. (If `id` is not a number.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku innych błędów zapisu. (On other write failures.)
*   **Przykład (Example):**
    ```javascript
    try {
      const result = await xdB.del.id('products', 45); // Usuwa rekord z products.json
      console.log(`Usunięto rekord o ID ${result.deletedId} z pliku ${result.path}`);
    } catch (error) {
      console.error("Błąd usuwania rekordu:", error.message, error.code);
    }
    ```

## Operacje Dodawania (`xdB.add`) (Add Operations)

### `xdB.add.all(filePath, initialData, options)`

Tworzy nowy plik JSON z danymi początkowymi atomowo, opcjonalnie nadpisując, jeśli istnieje. Automatycznie dodaje rozszerzenie `.json`, jeśli go brakuje.

(Creates a new JSON file with initial data atomically, optionally overwriting if it exists. Automatically adds the `.json` extension if missing.)

*   **Parametry (Parameters):**
    *   `filePath` (`string`): Ścieżka dla nowego pliku JSON (względna do `basePath`, `.json` dodawane, jeśli brakuje). (Path for the new JSON file (relative to `basePath`, `.json` added if missing).)
    *   `initialData` (`Array` | `object`, opcjonalnie, domyślnie `[]`): Początkowe dane do zapisania w pliku. (The initial data to write to the file.)
    *   `options` (`object`, opcjonalnie, domyślnie `{ overwrite: true }`): Opcje operacji. (Options for the operation.)
        *   `overwrite` (`boolean`, domyślnie `true`): Czy nadpisać plik, jeśli już istnieje. (Whether to overwrite the file if it already exists.)
*   **Zwraca (Returns):**
    *   `Promise<{path: string}>`: Obietnica rozwiązująca się obiektem zawierającym absolutną ścieżkę utworzonego pliku. (A promise resolving with an object containing the absolute path of the created file.)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: Jeśli `overwrite` jest `false` i plik istnieje, lub jeśli `initialData` nie jest tablicą/obiektem. (If `overwrite` is `false` and the file exists, or if `initialData` is not an array/object.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku innych błędów zapisu. (On other write failures.)
*   **Przykład (Example):**
    ```javascript
    // Utwórz nowy plik 'tasks.json' z pustą tablicą, nie nadpisuj jeśli istnieje
    // Create a new 'tasks.json' file with an empty array, do not overwrite if exists
    try {
      const result = await xdB.add.all('tasks', [], { overwrite: false });
      console.log("Utworzono plik:", result.path);
    } catch (error) {
      if (error.code === XDB_ERROR_CODES.OPERATION_FAILED && error.message.includes('already exists')) {
         console.warn("Plik 'tasks.json' już istnieje.");
      } else {
         console.error("Błąd tworzenia pliku:", error.message);
      }
    }

    // Utwórz/nadpisz plik 'metadata.json' z obiektem
    // Create/overwrite 'metadata.json' with an object
    try {
      const result = await xdB.add.all('metadata', { version: '1.0', created: Date.now() });
      console.log("Utworzono/nadpisano plik:", result.path);
    } catch (error) {
      console.error("Błąd tworzenia pliku metadata:", error.message);
    }
    ```

### `xdB.add.id(filePath, newRecord)`

Dodaje nowy rekord do pliku JSON (który musi być tablicą obiektów) atomowo. Tworzy plik, jeśli nie istnieje. Generuje unikalny numeryczny ID, jeśli `newRecord.id` nie jest podane. Automatycznie dodaje rozszerzenie `.json`, jeśli go brakuje.

(Adds a new record to a JSON file (which must be an array of objects) atomically. Creates the file if it doesn't exist. Generates a unique numeric ID if `newRecord.id` is not provided. Automatically adds the `.json` extension if missing.)

*   **Parametry (Parameters):**
    *   `filePath` (`string`): Ścieżka do pliku JSON (względna do `basePath`, `.json` dodawane, jeśli brakuje). (Path to the JSON file (relative to `basePath`, `.json` added if missing).)
    *   `newRecord` (`object`): Obiekt rekordu do dodania. Może zawierać właściwość `id`. Jeśli `id` nie jest podane lub jest `undefined`/`null`, zostanie wygenerowane automatycznie. (The record object to add. May include an `id` property. If `id` is not provided or is `undefined`/`null`, it will be auto-generated.)
*   **Zwraca (Returns):**
    *   `Promise<{path: string, record: object}>`: Obietnica rozwiązująca się obiektem zawierającym ścieżkę pliku i dodany rekord (wraz z jego ID). (A promise resolving with an object containing the file path and the added record (including its ID).)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.INVALID_JSON}`: Jeśli istniejący plik zawiera nieprawidłowy JSON. (If the existing file contains invalid JSON.)
    *   `Error & {code: XDB_ERROR_CODES.RECORD_EXISTS}`: Jeśli podano `id` w `newRecord` i rekord o takim ID już istnieje. (If an `id` is provided in `newRecord` and a record with that ID already exists.)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: Jeśli `newRecord` nie jest obiektem. (If `newRecord` is not an object.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku błędów odczytu/zapisu plików (w tym pliku licznika `_xdB_counters.json`). (On file read/write errors (including the counter file `_xdB_counters.json`).)
*   **Przykład (Example):**
    ```javascript
    const newUser = { name: 'Jan Kowalski', city: 'Warszawa' };
    try {
      // Dodaj użytkownika, ID zostanie wygenerowane automatycznie
      // Add user, ID will be auto-generated
      const result = await xdB.add.id('users', newUser);
      console.log(`Dodano rekord z ID ${result.record.id} do pliku ${result.path}`);
      console.log("Dodany rekord:", result.record);
    } catch (error) {
      console.error("Błąd dodawania rekordu:", error.message, error.code);
    }

    const specificProduct = { id: 101, name: 'Produkt Specjalny', price: 99.99 };
    try {
      // Dodaj produkt z konkretnym ID
      // Add product with a specific ID
      const result = await xdB.add.id('products', specificProduct);
      console.log(`Dodano rekord z ID ${result.record.id} do pliku ${result.path}`);
    } catch (error) {
      if (error.code === XDB_ERROR_CODES.RECORD_EXISTS) {
         console.warn(`Produkt o ID ${specificProduct.id} już istnieje.`);
      } else {
         console.error("Błąd dodawania produktu:", error.message, error.code);
      }
    }
    ```

## Operacje Odczytu (`xdB.view`) (View Operations)

### `xdB.view.all(filePath)`

Odczytuje i zwraca całą zawartość pliku JSON. Automatycznie dodaje rozszerzenie `.json`, jeśli go brakuje.

(Reads and returns the entire content of a JSON file. Automatically adds the `.json` extension if missing.)

*   **Parametry (Parameters):**
    *   `filePath` (`string`): Ścieżka do pliku JSON (względna do `basePath`, `.json` dodawane, jeśli brakuje). (Path to the JSON file (relative to `basePath`, `.json` added if missing).)
*   **Zwraca (Returns):**
    *   `Promise<{path: string, data: Array|object}>`: Obietnica rozwiązująca się obiektem zawierającym absolutną ścieżkę pliku i sparsowane dane JSON. (A promise resolving with an object containing the absolute file path and the parsed JSON data.)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.FILE_NOT_FOUND}`: Jeśli plik nie istnieje. (If the file doesn't exist.)
    *   `Error & {code: XDB_ERROR_CODES.INVALID_JSON}`: Jeśli plik zawiera nieprawidłowy JSON. (If the file contains invalid JSON.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku innych błędów odczytu. (On other read errors.)
*   **Przykład (Example):**
    ```javascript
    try {
      const result = await xdB.view.all('config/settings'); // Odczytuje config/settings.json
      console.log("Zawartość pliku:", result.path, result.data);
    } catch (error) {
      console.error("Błąd odczytu pliku:", error.message, error.code);
    }
    ```

### `xdB.view.id(filePath, id)`

Odczytuje i zwraca określony rekord z pliku JSON (który musi być tablicą obiektów) identyfikowany przez jego ID. Automatycznie dodaje rozszerzenie `.json`, jeśli go brakuje.

(Reads and returns a specific record from a JSON file (which must be an array of objects) identified by its ID. Automatically adds the `.json` extension if missing.)

*   **Parametry (Parameters):**
    *   `filePath` (`string`): Ścieżka do pliku JSON (względna do `basePath`, `.json` dodawane, jeśli brakuje). (Path to the JSON file (relative to `basePath`, `.json` added if missing).)
    *   `id` (`number`): ID rekordu do pobrania. (The ID of the record to retrieve.)
*   **Zwraca (Returns):**
    *   `Promise<{path: string, record: object}>`: Obietnica rozwiązująca się obiektem zawierającym ścieżkę pliku i znaleziony obiekt rekordu. (A promise resolving with an object containing the file path and the found record object.)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.FILE_NOT_FOUND}`: Jeśli plik nie istnieje. (If the file doesn't exist.)
    *   `Error & {code: XDB_ERROR_CODES.INVALID_JSON}`: Jeśli plik zawiera nieprawidłowy JSON. (If the file contains invalid JSON.)
    *   `Error & {code: XDB_ERROR_CODES.RECORD_NOT_FOUND}`: Jeśli rekord o podanym ID nie został znaleziony. (If a record with the specified ID was not found.)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: Jeśli `id` nie jest liczbą. (If `id` is not a number.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku innych błędów odczytu. (On other read errors.)
*   **Przykład (Example):**
    ```javascript
    try {
      const result = await xdB.view.id('users', 123); // Pobiera użytkownika z users.json
      console.log(`Znaleziono rekord w ${result.path}:`, result.record);
    } catch (error) {
      console.error("Błąd odczytu rekordu:", error.message, error.code);
    }
    ```

### `xdB.view.more(filePath, options)`

Odczytuje dane z pliku JSON (który musi być tablicą obiektów) z zaawansowanymi opcjami filtrowania, sortowania i paginacji. Automatycznie dodaje rozszerzenie `.json`, jeśli go brakuje.

(Reads data from a JSON file (which must be an array of objects) with advanced filtering, sorting, and pagination options. Automatically adds the `.json` extension if missing.)

*   **Parametry (Parameters):**
    *   `filePath` (`string`): Ścieżka do pliku JSON (względna do `basePath`, `.json` dodawane, jeśli brakuje). (Path to the JSON file (relative to `basePath`, `.json` added if missing).)
    *   `options` (`object`, opcjonalnie, domyślnie `{}`): Opcje zapytania do danych. (Options for querying the data.)
        *   `filter` (`function(object): boolean`, opcjonalnie): Funkcja do filtrowania rekordów. Otrzymuje rekord jako argument, powinna zwrócić `true`, aby go zachować. (A function to filter records. Receives the record as an argument, should return `true` to keep it.)
        *   `sort` (`object` | `Array<object>`, opcjonalnie): Kryteria sortowania. Może być pojedynczym obiektem `{key: string, order?: 'asc'|'desc', comparator?: function}` lub tablicą takich obiektów dla sortowania po wielu kluczach. (Sorting criteria. Can be a single object `{key: string, order?: 'asc'|'desc', comparator?: function}` or an array of such objects for multi-key sorting.)
            *   `key` (`string`): Klucz do sortowania. (The key to sort by.)
            *   `order` (`'asc'` | `'desc'`, opcjonalnie, domyślnie `'asc'`): Kierunek sortowania. (Sort order.)
            *   `comparator` (`function(any, any): number`, opcjonalnie): Niestandardowa funkcja porównująca (jak dla `Array.sort`). (Optional custom comparison function (like for `Array.sort`).)
        *   `skip` (`number`, opcjonalnie): Liczba rekordów do pominięcia (dla paginacji). (Number of records to skip (for pagination).)
        *   `limit` (`number`, opcjonalnie): Maksymalna liczba rekordów do zwrócenia (dla paginacji). (Maximum number of records to return (for pagination).)
*   **Zwraca (Returns):**
    *   `Promise<Array<object>>`: Obietnica rozwiązująca się tablicą rekordów pasujących do kryteriów. (A promise resolving to an array of records matching the criteria.)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.FILE_NOT_FOUND}`: Jeśli plik nie istnieje. (If the file doesn't exist.)
    *   `Error & {code: XDB_ERROR_CODES.INVALID_JSON}`: Jeśli plik zawiera nieprawidłowy JSON. (If the file contains invalid JSON.)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: Jeśli wystąpi błąd podczas stosowania funkcji `filter`. (If an error occurs while applying the `filter` function.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku innych błędów odczytu. (On other read errors.)
*   **Przykład (Example):**
    ```javascript
    try {
      // Znajdź aktywnych użytkowników z Warszawy, posortuj wg nazwiska (desc), pobierz pierwszych 10
      // Find active users from Warsaw, sort by last name (desc), get the first 10
      const options = {
        filter: user => user.status === 'active' && user.city === 'Warszawa',
        sort: { key: 'lastName', order: 'desc' }, // Zakładając, że jest pole 'lastName'
        limit: 10
      };
      const users = await xdB.view.more('users', options);
      console.log("Aktywni użytkownicy z Warszawy (max 10):", users);

      // Znajdź produkty droższe niż 100, posortuj wg ceny (asc), pomiń pierwsze 5
      // Find products more expensive than 100, sort by price (asc), skip the first 5
      const productOptions = {
        filter: product => product.price > 100,
        sort: { key: 'price', order: 'asc' },
        skip: 5
      };
      const expensiveProducts = await xdB.view.more('products', productOptions);
      console.log("Drogie produkty (pomijając 5 najtańszych):", expensiveProducts);

    } catch (error) {
      console.error("Błąd zaawansowanego odczytu:", error.message, error.code);
    }
    ```

## Kontrola Współbieżności (Concurrency Control)

xdB używa wewnętrznego mechanizmu blokad plików (`acquireLock`, `releaseLock`) opartego na `Map` i `Promise`, aby zapobiegać konfliktom podczas jednoczesnych operacji zapisu do tego samego pliku. Blokady są stosowane automatycznie w metodach modyfikujących pliki (`edit.*`, `del.*`, `add.*`, `move.*`, `dir.del`, `dir.rename`). Operacje odczytu (`view.*`) generalnie nie wymagają blokad, ale odczytują spójny stan pliku w momencie operacji.

(xdB uses an internal file locking mechanism (`acquireLock`, `releaseLock`) based on `Map` and `Promise` to prevent conflicts during concurrent write operations to the same file. Locks are applied automatically in methods that modify files (`edit.*`, `del.*`, `add.*`, `move.*`, `dir.del`, `dir.rename`). Read operations (`view.*`) generally do not require locks but read a consistent state of the file at the time of the operation.)

## Funkcje Pomocnicze (Helper Functions)

Biblioteka zawiera również wewnętrzne funkcje pomocnicze (np. `ensureJsonExtension`, `ensureDirectoryExists`, `safeParseJSON`, `validateId`, `validateRecord`, `createXdbError`), które generalnie nie są przeznaczone do bezpośredniego użytku zewnętrznego, ale wspierają działanie głównych metod API.

(The library also includes internal helper functions (e.g., `ensureJsonExtension`, `ensureDirectoryExists`, `safeParseJSON`, `validateId`, `validateRecord`, `createXdbError`) which are generally not intended for direct external use but support the operation of the main API methods.)

