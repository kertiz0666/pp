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
  // DUPLICATE_RECORD removed for consistency
  /** Indicates an attempt to add a record with an ID that already exists. (Wskazuje na próbę dodania rekordu z ID, które już istnieje.) */
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

Konfiguruje globalne ustawienia biblioteki xdB. Pozwala na zmianę domyślnego katalogu, w którym biblioteka będzie przechowywać pliki baz danych. Domyślnie jest to katalog, w którym znajduje się sam moduł `xdb.js`.

(Configures the global settings for the xdB library. It allows changing the default directory where the library will store database files. By default, this is the directory where the `xdb.js` module itself resides.)

*   **Parametry (Parameters):**
    *   `options` (`object`): Obiekt zawierający opcje konfiguracyjne. (An object containing configuration options.)
        *   `basePath` (`string`, opcjonalnie): Ścieżka (absolutna lub względna względem bieżącego katalogu roboczego procesu Node.js) do katalogu, który ma być używany jako katalog bazowy dla wszystkich operacji plikowych xdB. Jeśli podana ścieżka jest względna, zostanie przekonwertowana na absolutną. (The path (absolute or relative to the current working directory of the Node.js process) to the directory to be used as the base directory for all xdB file operations. If a relative path is provided, it will be resolved to an absolute path.)
*   **Szczegóły (Details):**
    *   Wywołanie `config` bez argumentów lub z pustym obiektem nie ma efektu. (Calling `config` with no arguments or an empty object has no effect.)
    *   Zmiana `basePath` wpływa na wszystkie **przyszłe** operacje xdB w bieżącym procesie Node.js. (Changing `basePath` affects all **subsequent** xdB operations within the current Node.js process.)
*   **Przykłady (Examples):**

    ```javascript
    import xdB from './xdb.js';
    import path from 'path';
    import os from 'os';

    // Przykład 1: Użycie ścieżki względnej (zostanie przekonwertowana na absolutną)
    // Example 1: Using a relative path (will be resolved to absolute)
    xdB.config({ basePath: './data/db_files' });
    console.log('Ścieżka bazowa po konfiguracji 1:', xdB.getBasePath());
    // Output: Ścieżka bazowa po konfiguracji 1: /path/to/your/project/data/db_files

    // Przykład 2: Użycie ścieżki absolutnej
    // Example 2: Using an absolute path
    const absoluteDbPath = path.join(os.homedir(), 'my_app_data', 'database');
    xdB.config({ basePath: absoluteDbPath });
    console.log('Ścieżka bazowa po konfiguracji 2:', xdB.getBasePath());
    // Output: Ścieżka bazowa po konfiguracji 2: /home/user/my_app_data/database (lub odpowiednik w Windows)

    // Przykład 3: Wywołanie bez basePath (bez zmian)
    // Example 3: Calling without basePath (no change)
    const pathBefore = xdB.getBasePath();
    xdB.config({});
    const pathAfter = xdB.getBasePath();
    console.log('Czy ścieżka się zmieniła?', pathBefore === pathAfter); // true
    ```

### `xdB.getBasePath()`

Pobiera aktualnie skonfigurowaną, absolutną ścieżkę bazową, której xdB używa do lokalizowania plików i katalogów.

(Gets the currently configured, absolute base path that xdB uses for locating files and directories.)

*   **Zwraca (Returns):**
    *   `string`: Zawsze zwraca **absolutną** ścieżkę bazową. (Always returns the **absolute** base path.)
*   **Szczegóły (Details):**
    *   Przydatne do debugowania lub konstruowania ścieżek niezależnie od xdB, ale w odniesieniu do jego konfiguracji. (Useful for debugging or constructing paths independently of xdB but relative to its configuration.)
*   **Przykłady (Examples):**

    ```javascript
    import xdB from './xdb.js';
    import path from 'path';

    // Domyślna ścieżka (katalog modułu xdb.js)
    // Default path (directory of the xdb.js module)
    console.log("Domyślna ścieżka bazowa:", xdB.getBasePath());
    // Output: Domyślna ścieżka bazowa: /path/to/your/project/node_modules/xdB (lub gdziekolwiek jest xdb.js)

    // Po konfiguracji
    // After configuration
    xdB.config({ basePath: '../database_storage' });
    const currentBasePath = xdB.getBasePath();
    console.log("Aktualna ścieżka bazowa:", currentBasePath);
    // Output: Aktualna ścieżka bazowa: /path/to/your/database_storage

    // Użycie do stworzenia pełnej ścieżki do pliku
    // Using it to create a full file path
    const userFilePath = path.join(currentBasePath, 'users.json');
    console.log("Pełna ścieżka do pliku użytkowników:", userFilePath);
    // Output: Pełna ścieżka do pliku użytkowników: /path/to/your/database_storage/users.json
    ```

## Operacje na Katalogach (`xdB.dir`) (Directory Operations)

### `xdB.dir.add(dirPath)`

Asynchronicznie tworzy nowy katalog pod podaną ścieżką (względną do skonfigurowanej `basePath`). Jeśli katalogi nadrzędne nie istnieją, zostaną one również utworzone (działa jak `mkdir -p`). Jeśli katalog już istnieje, operacja zakończy się sukcesem bez wprowadzania zmian.

(Asynchronously creates a new directory at the specified path (relative to the configured `basePath`). If parent directories do not exist, they will also be created (similar to `mkdir -p`). If the directory already exists, the operation completes successfully without making changes.)

*   **Parametry (Parameters):**
    *   `dirPath` (`string`): Ścieżka do katalogu do utworzenia. Może zawierać wiele poziomów, np. `'data/images/avatars'`. Jest interpretowana względem `basePath`. (The path to the directory to create. Can include multiple levels, e.g., `'data/images/avatars'`. It's interpreted relative to `basePath`.)
*   **Zwraca (Returns):**
    *   `Promise<{path: string}>`: Obietnica, która po pomyślnym utworzeniu katalogu (lub jeśli już istniał) rozwiązuje się obiektem zawierającym **absolutną** ścieżkę do utworzonego katalogu. (A promise that, upon successful directory creation (or if it already existed), resolves with an object containing the **absolute** path to the created directory.)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku problemów z uprawnieniami lub innych błędów systemu plików podczas tworzenia katalogu. (In case of permission issues or other file system errors during directory creation.)
*   **Przykłady (Examples):**

    ```javascript
    import xdB from './xdb.js';

    // Przykład 1: Tworzenie prostego katalogu
    // Example 1: Creating a simple directory
    async function createSimpleDir() {
      try {
        const result = await xdB.dir.add('backups');
        console.log(`[Przykład 1] Utworzono/potwierdzono katalog: ${result.path}`);
        // Output: [Przykład 1] Utworzono/potwierdzono katalog: /path/to/base/backups
      } catch (error) {
        console.error("[Przykład 1] Błąd:", error.message);
      }
    }
    createSimpleDir();

    // Przykład 2: Tworzenie zagnieżdżonych katalogów
    // Example 2: Creating nested directories
    async function createNestedDirs() {
      try {
        const result = await xdB.dir.add('archive/2025/logs');
        console.log(`[Przykład 2] Utworzono/potwierdzono zagnieżdżony katalog: ${result.path}`);
        // Output: [Przykład 2] Utworzono/potwierdzono zagnieżdżony katalog: /path/to/base/archive/2025/logs
      } catch (error) {
        console.error("[Przykład 2] Błąd:", error.message);
      }
    }
    // Poczekaj chwilę przed uruchomieniem drugiego przykładu, aby uniknąć potencjalnych konfliktów logów
    // Wait a moment before running the second example to avoid potential log conflicts
    setTimeout(createNestedDirs, 100);

    // Przykład 3: Próba utworzenia istniejącego katalogu (powinno się powieść bez błędu)
    // Example 3: Attempting to create an existing directory (should succeed without error)
    async function createExistingDir() {
      try {
        // Najpierw utwórz
        // First create
        await xdB.dir.add('temp_data');
        console.log('[Przykład 3] Pierwsze utworzenie powiodło się.');
        // Spróbuj utworzyć ponownie
        // Try creating again
        const result = await xdB.dir.add('temp_data');
        console.log(`[Przykład 3] Ponowne "utworzenie" istniejącego katalogu powiodło się: ${result.path}`);
      } catch (error) {
        // Ten blok nie powinien zostać wykonany
        // This block should not be executed
        console.error("[Przykład 3] Niespodziewany błąd:", error.message);
      }
    }
    setTimeout(createExistingDir, 200);
    ```

### `xdB.dir.del(dirPath)`

Asynchronicznie usuwa katalog wraz z całą jego zawartością (pliki i podkatalogi) rekurencyjnie. Działa podobnie do `rm -rf`. Używa blokady na katalogu, aby zapobiec konfliktom podczas usuwania.

(Asynchronously deletes a directory along with all its contents (files and subdirectories) recursively. Works similarly to `rm -rf`. Uses a lock on the directory to prevent conflicts during deletion.)

*   **Parametry (Parameters):**
    *   `dirPath` (`string`): Ścieżka do katalogu do usunięcia (względna do `basePath`). (The path to the directory to delete (relative to `basePath`).)
*   **Zwraca (Returns):**
    *   `Promise<{path: string}>`: Obietnica, która po pomyślnym usunięciu katalogu rozwiązuje się obiektem zawierającym **absolutną** ścieżkę usuniętego katalogu. (A promise that, upon successful directory deletion, resolves with an object containing the **absolute** path of the deleted directory.)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.DIR_NOT_FOUND}`: Jeśli podany katalog nie istnieje w momencie próby usunięcia. (If the specified directory does not exist at the time of the deletion attempt.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku problemów z uprawnieniami lub innych błędów systemu plików podczas usuwania. (In case of permission issues or other file system errors during deletion.)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: W przypadku problemów z uzyskaniem blokady na katalogu (np. timeout). (In case of problems acquiring the lock on the directory (e.g., timeout).)
*   **Przykłady (Examples):**

    ```javascript
    import xdB, { XDB_ERROR_CODES } from './xdb.js';

    // Przygotowanie: Utwórz katalog do usunięcia
    // Preparation: Create a directory to delete
    async function prepareForDelete() {
        try {
            await xdB.dir.add('to_be_deleted/subdir');
            await xdB.add.all('to_be_deleted/some_file', { data: 1 });
            console.log('[Przygotowanie do del] Utworzono strukturę do usunięcia.');
        } catch (e) { console.error('Błąd przygotowania:', e); }
    }

    // Przykład 1: Usunięcie istniejącego katalogu
    // Example 1: Deleting an existing directory
    async function deleteExistingDir() {
      try {
        const result = await xdB.dir.del('to_be_deleted');
        console.log(`[Przykład 1 del] Usunięto katalog: ${result.path}`);
        // Output: [Przykład 1 del] Usunięto katalog: /path/to/base/to_be_deleted
      } catch (error) {
        console.error("[Przykład 1 del] Błąd:", error.message, error.code);
      }
    }

    // Przykład 2: Próba usunięcia nieistniejącego katalogu
    // Example 2: Attempting to delete a non-existent directory
    async function deleteNonExistentDir() {
      try {
        const result = await xdB.dir.del('non_existent_dir');
        // Ten kod nie powinien się wykonać
        // This code should not execute
        console.log(`[Przykład 2 del] Niespodziewany sukces: ${result.path}`);
      } catch (error) {
        if (error.code === XDB_ERROR_CODES.DIR_NOT_FOUND) {
          console.log(`[Przykład 2 del] Zgodnie z oczekiwaniami, katalog nie istnieje: ${error.message}`);
        } else {
          console.error("[Przykład 2 del] Inny błąd:", error.message, error.code);
        }
      }
    }

    // Uruchomienie przykładów sekwencyjnie
    // Run examples sequentially
    prepareForDelete()
        .then(() => new Promise(resolve => setTimeout(resolve, 100))) // Krótka pauza
        .then(deleteExistingDir)
        .then(() => new Promise(resolve => setTimeout(resolve, 100))) // Krótka pauza
        .then(deleteNonExistentDir);
    ```

### `xdB.dir.rename(oldPath, newPath)`

Asynchronicznie zmienia nazwę istniejącego katalogu lub przenosi go w inne miejsce w ramach skonfigurowanej `basePath`. Jeśli katalog docelowy nie istnieje, zostanie utworzony (wraz z katalogami nadrzędnymi). Operacja używa blokady na katalogu źródłowym.

(Asynchronously renames an existing directory or moves it to another location within the configured `basePath`. If the target parent directory does not exist, it will be created (including parent directories). The operation uses a lock on the source directory.)

*   **Parametry (Parameters):**
    *   `oldPath` (`string`): Obecna ścieżka katalogu (względna do `basePath`), który ma zostać przeniesiony/zmieniony. (The current path of the directory (relative to `basePath`) to be moved/renamed.)
    *   `newPath` (`string`): Nowa ścieżka dla katalogu (względna do `basePath`). (The new path for the directory (relative to `basePath`).)
*   **Zwraca (Returns):**
    *   `Promise<{oldPath: string, newPath: string}>`: Obietnica, która po pomyślnej operacji rozwiązuje się obiektem zawierającym starą i nową **absolutną** ścieżkę katalogu. (A promise that, upon successful operation, resolves with an object containing the old and new **absolute** paths of the directory.)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.DIR_NOT_FOUND}`: Jeśli katalog źródłowy (`oldPath`) nie istnieje. (If the source directory (`oldPath`) does not exist.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku problemów z uprawnieniami, jeśli `newPath` już istnieje i nie jest pustym katalogiem, lub innych błędów systemu plików. (In case of permission issues, if `newPath` already exists and is not an empty directory, or other file system errors.)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: W przypadku problemów z uzyskaniem blokady na katalogu źródłowym (np. timeout). (In case of problems acquiring the lock on the source directory (e.g., timeout).)
*   **Przykłady (Examples):**

    ```javascript
    import xdB, { XDB_ERROR_CODES } from './xdb.js';

    // Przygotowanie: Utwórz katalog do zmiany nazwy/przeniesienia
    // Preparation: Create a directory to rename/move
    async function prepareForRename() {
        try {
            await xdB.dir.add('source_dir/content');
            console.log('[Przygotowanie do rename] Utworzono katalog źródłowy.');
        } catch (e) { console.error('Błąd przygotowania:', e); }
    }

    // Przykład 1: Zmiana nazwy katalogu w tym samym miejscu
    // Example 1: Renaming a directory in the same location
    async function renameDir() {
      try {
        const result = await xdB.dir.rename('source_dir', 'renamed_dir');
        console.log(`[Przykład 1 rename] Zmieniono nazwę z ${result.oldPath} na ${result.newPath}`);
        // Output: [Przykład 1 rename] Zmieniono nazwę z /path/to/base/source_dir na /path/to/base/renamed_dir
      } catch (error) {
        console.error("[Przykład 1 rename] Błąd:", error.message, error.code);
      }
    }

    // Przykład 2: Przeniesienie katalogu do nowego miejsca (z utworzeniem katalogu nadrzędnego)
    // Example 2: Moving a directory to a new location (with parent directory creation)
    async function moveDir() {
      try {
        // Zakładając, że 'renamed_dir' istnieje po Przykładzie 1
        // Assuming 'renamed_dir' exists after Example 1
        const result = await xdB.dir.rename('renamed_dir', 'archive/moved_dir');
        console.log(`[Przykład 2 rename] Przeniesiono z ${result.oldPath} do ${result.newPath}`);
        // Output: [Przykład 2 rename] Przeniesiono z /path/to/base/renamed_dir do /path/to/base/archive/moved_dir
      } catch (error) {
        console.error("[Przykład 2 rename] Błąd:", error.message, error.code);
      }
    }

    // Przykład 3: Próba zmiany nazwy nieistniejącego katalogu
    // Example 3: Attempting to rename a non-existent directory
    async function renameNonExistent() {
        try {
            const result = await xdB.dir.rename('non_existent_source', 'some_target');
            console.log(`[Przykład 3 rename] Niespodziewany sukces: ${result.oldPath} -> ${result.newPath}`);
        } catch(error) {
            if (error.code === XDB_ERROR_CODES.DIR_NOT_FOUND) {
                console.log(`[Przykład 3 rename] Zgodnie z oczekiwaniami, katalog źródłowy nie istnieje: ${error.message}`);
            } else {
                console.error("[Przykład 3 rename] Inny błąd:", error.message, error.code);
            }
        }
    }

    // Uruchomienie przykładów sekwencyjnie
    // Run examples sequentially
    prepareForRename()
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(renameDir)
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(moveDir)
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(renameNonExistent);
    ```

## Operacje Przenoszenia (`xdB.move`) (Move Operations)

### `xdB.move.file(sourcePath, targetPath)`

Asynchronicznie przenosi istniejący plik bazy danych (`.json`) do nowej lokalizacji lub zmienia jego nazwę. Automatycznie dodaje rozszerzenie `.json` do ścieżki źródłowej i docelowej, jeśli go brakuje. Zapewnia, że katalog docelowy istnieje (tworzy go, jeśli jest to konieczne). Operacja używa blokady na pliku źródłowym.

(Asynchronously moves an existing database file (`.json`) to a new location or renames it. Automatically adds the `.json` extension to both source and target paths if missing. Ensures the target directory exists (creates it if necessary). The operation uses a lock on the source file.)

*   **Parametry (Parameters):**
    *   `sourcePath` (`string`): Obecna ścieżka pliku (względna do `basePath`). Rozszerzenie `.json` jest opcjonalne. (The current path of the file (relative to `basePath`). The `.json` extension is optional.)
    *   `targetPath` (`string`): Nowa ścieżka dla pliku (względna do `basePath`). Rozszerzenie `.json` jest opcjonalne. (The new path for the file (relative to `basePath`). The `.json` extension is optional.)
*   **Zwraca (Returns):**
    *   `Promise<{source: string, target: string}>`: Obietnica, która po pomyślnym przeniesieniu/zmianie nazwy rozwiązuje się obiektem zawierającym starą (`source`) i nową (`target`) **absolutną** ścieżkę pliku (z rozszerzeniem `.json`). (A promise that, upon successful move/rename, resolves with an object containing the old (`source`) and new (`target`) **absolute** paths of the file (with the `.json` extension).)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.FILE_NOT_FOUND}`: Jeśli plik źródłowy (`sourcePath` + `.json`) nie istnieje. (If the source file (`sourcePath` + `.json`) does not exist.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku problemów z uprawnieniami, błędów podczas tworzenia katalogu docelowego, lub innych błędów systemu plików podczas operacji `rename`. (In case of permission issues, errors during target directory creation, or other file system errors during the `rename` operation.)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: W przypadku problemów z uzyskaniem blokady na pliku źródłowym (np. timeout). (In case of problems acquiring the lock on the source file (e.g., timeout).)
*   **Przykłady (Examples):**

    ```javascript
    import xdB, { XDB_ERROR_CODES } from './xdb.js';

    // Przygotowanie: Utwórz plik do przeniesienia
    // Preparation: Create a file to move
    async function prepareForMove() {
        try {
            await xdB.add.all('move_source', { initial: true });
            console.log('[Przygotowanie do move] Utworzono plik źródłowy.');
        } catch (e) { console.error('Błąd przygotowania:', e); }
    }

    // Przykład 1: Zmiana nazwy pliku w tym samym katalogu
    // Example 1: Renaming a file in the same directory
    async function renameFile() {
      try {
        // Nie trzeba podawać .json
        // No need to provide .json
        const result = await xdB.move.file('move_source', 'renamed_file');
        console.log(`[Przykład 1 move] Zmieniono nazwę z ${result.source} na ${result.target}`);
        // Output: [Przykład 1 move] Zmieniono nazwę z /path/to/base/move_source.json na /path/to/base/renamed_file.json
      } catch (error) {
        console.error("[Przykład 1 move] Błąd:", error.message, error.code);
      }
    }

    // Przykład 2: Przeniesienie pliku do nowego katalogu (który zostanie utworzony)
    // Example 2: Moving a file to a new directory (which will be created)
    async function moveFileToNewDir() {
      try {
        // Zakładając, że 'renamed_file.json' istnieje po Przykładzie 1
        // Assuming 'renamed_file.json' exists after Example 1
        const result = await xdB.move.file('renamed_file', 'new_location/final_file_name');
        console.log(`[Przykład 2 move] Przeniesiono z ${result.source} do ${result.target}`);
        // Output: [Przykład 2 move] Przeniesiono z /path/to/base/renamed_file.json do /path/to/base/new_location/final_file_name.json
      } catch (error) {
        console.error("[Przykład 2 move] Błąd:", error.message, error.code);
      }
    }

    // Przykład 3: Próba przeniesienia nieistniejącego pliku
    // Example 3: Attempting to move a non-existent file
    async function moveNonExistent() {
        try {
            const result = await xdB.move.file('ghost_file', 'somewhere_else');
            console.log(`[Przykład 3 move] Niespodziewany sukces: ${result.source} -> ${result.target}`);
        } catch(error) {
            if (error.code === XDB_ERROR_CODES.FILE_NOT_FOUND) {
                console.log(`[Przykład 3 move] Zgodnie z oczekiwaniami, plik źródłowy nie istnieje: ${error.message}`);
            } else {
                console.error("[Przykład 3 move] Inny błąd:", error.message, error.code);
            }
        }
    }

    // Uruchomienie przykładów sekwencyjnie
    // Run examples sequentially
    prepareForMove()
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(renameFile)
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(moveFileToNewDir)
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(moveNonExistent);
    ```

## Operacje Edycji (`xdB.edit`) (Edit Operations)

### `xdB.edit.all(filePath, newData)`

Asynchronicznie nadpisuje **całą** zawartość pliku JSON podaną w `newData`. Operacja jest atomowa (plik jest najpierw zapisywany do tymczasowej lokalizacji, a następnie przenoszony). Jeśli plik nie istnieje, zostanie utworzony (wraz z niezbędnymi katalogami nadrzędnymi). Automatycznie dodaje rozszerzenie `.json` do `filePath`, jeśli go brakuje. Używa blokady na pliku.

(Asynchronously overwrites the **entire** content of a JSON file with the data provided in `newData`. The operation is atomic (the file is first written to a temporary location and then moved). If the file does not exist, it will be created (along with necessary parent directories). Automatically adds the `.json` extension to `filePath` if missing. Uses a lock on the file.)

*   **Parametry (Parameters):**
    *   `filePath` (`string`): Ścieżka do pliku JSON (względna do `basePath`). Rozszerzenie `.json` jest opcjonalne. (Path to the JSON file (relative to `basePath`). The `.json` extension is optional.)
    *   `newData` (`Array` | `object`): Nowe dane (tablica JavaScript lub obiekt), które całkowicie zastąpią zawartość pliku. Muszą być serializowalne do formatu JSON. (The new data (JavaScript array or object) that will completely replace the file's content. Must be serializable to JSON.)
*   **Zwraca (Returns):**
    *   `Promise<{path: string}>`: Obietnica, która po pomyślnym zapisie rozwiązuje się obiektem zawierającym **absolutną** ścieżkę zapisanego pliku (z rozszerzeniem `.json`). (A promise that, upon successful write, resolves with an object containing the **absolute** path of the written file (with the `.json` extension).)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: Jeśli `newData` nie jest tablicą ani obiektem, lub jeśli zawiera nieprawidłowe dane (np. rekordy w tablicy nie są obiektami). (If `newData` is not an array or an object, or if it contains invalid data (e.g., records in an array are not objects).)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku problemów z uprawnieniami, błędów podczas tworzenia katalogów, błędów zapisu do pliku tymczasowego lub operacji `rename`. (In case of permission issues, errors during directory creation, errors writing to the temporary file, or `rename` operation failures.)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: W przypadku problemów z uzyskaniem blokady na pliku (np. timeout). (In case of problems acquiring the lock on the file (e.g., timeout).)
*   **Przykłady (Examples):**

    ```javascript
    import xdB, { XDB_ERROR_CODES } from './xdb.js';

    // Przykład 1: Nadpisanie pliku tablicą obiektów
    // Example 1: Overwriting a file with an array of objects
    async function overwriteWithArray() {
      const users = [
        { id: 1, name: 'Alice', role: 'admin' },
        { id: 2, name: 'Bob', role: 'user' }
      ];
      try {
        const result = await xdB.edit.all('user_list', users); // Nadpisuje lub tworzy user_list.json
        console.log(`[Przykład 1 edit.all] Plik ${result.path} nadpisany tablicą.`);
      } catch (error) {
        console.error("[Przykład 1 edit.all] Błąd:", error.message, error.code);
      }
    }
    overwriteWithArray();

    // Przykład 2: Nadpisanie pliku obiektem konfiguracyjnym
    // Example 2: Overwriting a file with a configuration object
    async function overwriteWithObject() {
      const config = {
        apiUrl: '/api/v2',
        timeout: 10000,
        features: { newUI: true, logging: 'verbose' }
      };
      try {
        const result = await xdB.edit.all('app_config', config); // Nadpisuje lub tworzy app_config.json
        console.log(`[Przykład 2 edit.all] Plik ${result.path} nadpisany obiektem.`);
      } catch (error) {
        console.error("[Przykład 2 edit.all] Błąd:", error.message, error.code);
      }
    }
    setTimeout(overwriteWithObject, 100);

    // Przykład 3: Próba nadpisania nieprawidłowymi danymi (nie obiekt/tablica)
    // Example 3: Attempting to overwrite with invalid data (not object/array)
    async function overwriteWithInvalidData() {
        try {
            const result = await xdB.edit.all('invalid_data_file', "to jest string, nie obiekt");
            console.log(`[Przykład 3 edit.all] Niespodziewany sukces: ${result.path}`);
        } catch(error) {
            if (error.code === XDB_ERROR_CODES.OPERATION_FAILED) {
                console.log(`[Przykład 3 edit.all] Zgodnie z oczekiwaniami, błąd walidacji danych: ${error.message}`);
            } else {
                 console.error("[Przykład 3 edit.all] Inny błąd:", error.message, error.code);
            }
        }
    }
    setTimeout(overwriteWithInvalidData, 200);

    // Przykład 4: Próba nadpisania tablicą zawierającą nie-obiekty
    // Example 4: Attempting to overwrite with an array containing non-objects
    async function overwriteWithInvalidArray() {
        try {
            const result = await xdB.edit.all('invalid_array_file', [{id: 1}, "nie obiekt", {id: 3}]);
            console.log(`[Przykład 4 edit.all] Niespodziewany sukces: ${result.path}`);
        } catch(error) {
            if (error.code === XDB_ERROR_CODES.OPERATION_FAILED && error.message.includes('Invalid record')) {
                console.log(`[Przykład 4 edit.all] Zgodnie z oczekiwaniami, błąd walidacji rekordu w tablicy: ${error.message}`);
            } else {
                 console.error("[Przykład 4 edit.all] Inny błąd:", error.message, error.code);
            }
        }
    }
    setTimeout(overwriteWithInvalidArray, 300);
    ```

### `xdB.edit.id(filePath, id, newRecord)`

Asynchronicznie edytuje pojedynczy rekord w pliku JSON (który musi zawierać tablicę obiektów). Rekord do edycji jest identyfikowany przez jego unikalne pole `id`. Operacja atomowo odczytuje plik, znajduje rekord, łączy go z danymi z `newRecord` (nadpisując istniejące pola i dodając nowe), a następnie zapisuje całą zaktualizowaną tablicę z powrotem do pliku. Automatycznie dodaje rozszerzenie `.json` do `filePath`, jeśli go brakuje. Używa blokady na pliku.

(Asynchronously edits a single record within a JSON file (which must contain an array of objects). The record to edit is identified by its unique `id` field. The operation atomically reads the file, finds the record, merges it with the data from `newRecord` (overwriting existing fields and adding new ones), and then writes the entire updated array back to the file. Automatically adds the `.json` extension to `filePath` if missing. Uses a lock on the file.)

*   **Parametry (Parameters):**
    *   `filePath` (`string`): Ścieżka do pliku JSON (względna do `basePath`). Rozszerzenie `.json` jest opcjonalne. (Path to the JSON file (relative to `basePath`). The `.json` extension is optional.)
    *   `id` (`number`): Numeryczne ID rekordu, który ma zostać zaktualizowany. (The numeric ID of the record to be updated.)
    *   `newRecord` (`object`): Obiekt zawierający pola i wartości do zaktualizowania w znalezionym rekordzie. Pola z `newRecord` nadpiszą istniejące pola w rekordzie. Pole `id` w tym obiekcie jest ignorowane; ID rekordu pozostaje niezmienione. (An object containing the fields and values to update in the found record. Fields from `newRecord` will overwrite existing fields in the record. The `id` field within this object is ignored; the record's ID remains unchanged.)
*   **Zwraca (Returns):**
    *   `Promise<{path: string, record: object}>`: Obietnica, która po pomyślnej edycji rozwiązuje się obiektem zawierającym **absolutną** ścieżkę pliku (z rozszerzeniem `.json`) oraz **cały zaktualizowany obiekt rekordu**. (A promise that, upon successful edit, resolves with an object containing the **absolute** file path (with the `.json` extension) and the **entire updated record object**.)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: Jeśli `id` nie jest liczbą lub `newRecord` nie jest obiektem (lub jest `null`/tablicą). (If `id` is not a number or `newRecord` is not an object (or is `null`/array).)
    *   `Error & {code: XDB_ERROR_CODES.FILE_NOT_FOUND}`: Jeśli plik określony przez `filePath` nie istnieje. (If the file specified by `filePath` does not exist.)
    *   `Error & {code: XDB_ERROR_CODES.INVALID_JSON}`: Jeśli plik istnieje, ale zawiera nieprawidłowe dane JSON. (If the file exists but contains invalid JSON data.)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: Jeśli plik nie zawiera tablicy JSON. (If the file does not contain a JSON array.)
    *   `Error & {code: XDB_ERROR_CODES.RECORD_NOT_FOUND}`: Jeśli w pliku nie znaleziono rekordu o podanym `id`. (If no record with the specified `id` was found in the file.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku problemów z uprawnieniami lub innych błędów systemu plików podczas odczytu lub zapisu. (In case of permission issues or other file system errors during reading or writing.)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: W przypadku problemów z uzyskaniem blokady na pliku (np. timeout). (In case of problems acquiring the lock on the file (e.g., timeout).)
*   **Przykłady (Examples):**

    ```javascript
    import xdB, { XDB_ERROR_CODES } from './xdb.js';

    // Przygotowanie: Utwórz plik z rekordami
    // Preparation: Create a file with records
    async function prepareForEditId() {
        try {
            const initialUsers = [
                { id: 1, name: 'Adam', email: 'adam@example.com', status: 'pending' },
                { id: 2, name: 'Ewa', email: 'ewa@example.com', status: 'active' }
            ];
            await xdB.add.all('users_to_edit', initialUsers);
            console.log('[Przygotowanie do edit.id] Utworzono plik users_to_edit.json.');
        } catch (e) { console.error('Błąd przygotowania:', e); }
    }

    // Przykład 1: Aktualizacja istniejącego rekordu
    // Example 1: Updating an existing record
    async function updateExistingRecord() {
      const updates = { status: 'inactive', lastLogin: Date.now() };
      try {
        const result = await xdB.edit.id('users_to_edit', 1, updates);
        console.log(`[Przykład 1 edit.id] Zaktualizowano rekord ID ${result.record.id}:`, result.record);
        // Output: [Przykład 1 edit.id] Zaktualizowano rekord ID 1: { id: 1, name: 'Adam', email: 'adam@example.com', status: 'inactive', lastLogin: 17... }
      } catch (error) {
        console.error("[Przykład 1 edit.id] Błąd:", error.message, error.code);
      }
    }

    // Przykład 2: Próba aktualizacji nieistniejącego rekordu
    // Example 2: Attempting to update a non-existent record
    async function updateNonExistentRecord() {
      const updates = { status: 'archived' };
      try {
        const result = await xdB.edit.id('users_to_edit', 999, updates);
        console.log(`[Przykład 2 edit.id] Niespodziewany sukces:`, result.record);
      } catch (error) {
        if (error.code === XDB_ERROR_CODES.RECORD_NOT_FOUND) {
          console.log(`[Przykład 2 edit.id] Zgodnie z oczekiwaniami, rekord nie znaleziony: ${error.message}`);
        } else {
          console.error("[Przykład 2 edit.id] Inny błąd:", error.message, error.code);
        }
      }
    }

    // Przykład 3: Próba aktualizacji w pliku, który nie jest tablicą
    // Example 3: Attempting to update in a file that is not an array
    async function updateInNonArrayFile() {
        try {
            await xdB.add.all('not_an_array_file', { config: true }); // Utwórz plik z obiektem
            const result = await xdB.edit.id('not_an_array_file', 1, { data: 'test' });
            console.log(`[Przykład 3 edit.id] Niespodziewany sukces:`, result.record);
        } catch(error) {
             if (error.code === XDB_ERROR_CODES.OPERATION_FAILED && error.message.includes('not contain a JSON array')) {
                console.log(`[Przykład 3 edit.id] Zgodnie z oczekiwaniami, błąd - plik nie zawiera tablicy: ${error.message}`);
            } else {
                 console.error("[Przykład 3 edit.id] Inny błąd:", error.message, error.code);
            }
        } finally {
            // Sprzątanie
            // Cleanup
            try { await xdB.del.all('not_an_array_file'); } catch {}
        }
    }

    // Uruchomienie przykładów sekwencyjnie
    // Run examples sequentially
    prepareForEditId()
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(updateExistingRecord)
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(updateNonExistentRecord)
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(updateInNonArrayFile);
    ```

## Operacje Usuwania (`xdB.del`) (Delete Operations)

### `xdB.del.all(filePath)`

Asynchronicznie usuwa **całą zawartość** istniejącego pliku JSON, zastępując ją pustą tablicą `[]`. Operacja jest atomowa i używa blokady na pliku. **Ważne:** Ta metoda **nie usuwa** samego pliku, jedynie czyści jego zawartość. Automatycznie dodaje rozszerzenie `.json` do `filePath`, jeśli go brakuje.

(Asynchronously deletes **all content** of an existing JSON file, replacing it with an empty array `[]`. The operation is atomic and uses a lock on the file. **Important:** This method **does not delete** the file itself, it only clears its content. Automatically adds the `.json` extension to `filePath` if missing.)

*   **Parametry (Parameters):**
    *   `filePath` (`string`): Ścieżka do pliku JSON, którego zawartość ma zostać usunięta (względna do `basePath`). Rozszerzenie `.json` jest opcjonalne. (Path to the JSON file whose content is to be deleted (relative to `basePath`). The `.json` extension is optional.)
*   **Zwraca (Returns):**
    *   `Promise<{path: string}>`: Obietnica, która po pomyślnym opróżnieniu pliku rozwiązuje się obiektem zawierającym **absolutną** ścieżkę opróżnionego pliku (z rozszerzeniem `.json`). (A promise that, upon successful emptying of the file, resolves with an object containing the **absolute** path of the emptied file (with the `.json` extension).)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.FILE_NOT_FOUND}`: Jeśli plik określony przez `filePath` nie istnieje w momencie próby opróżnienia. (If the file specified by `filePath` does not exist at the time of the emptying attempt.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku problemów z uprawnieniami lub innych błędów systemu plików podczas zapisu pustej tablicy. (In case of permission issues or other file system errors during the writing of the empty array.)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: W przypadku problemów z uzyskaniem blokady na pliku (np. timeout). (In case of problems acquiring the lock on the file (e.g., timeout).)
*   **Przykłady (Examples):**

    ```javascript
    import xdB, { XDB_ERROR_CODES } from './xdb.js';

    // Przygotowanie: Utwórz plik z danymi
    // Preparation: Create a file with data
    async function prepareForDelAll() {
        try {
            await xdB.add.all('temp_log', [{ts: Date.now(), msg: 'Log entry 1'}]);
            console.log('[Przygotowanie do del.all] Utworzono plik temp_log.json.');
        } catch (e) { console.error('Błąd przygotowania:', e); }
    }

    // Przykład 1: Opróżnienie istniejącego pliku
    // Example 1: Emptying an existing file
    async function emptyExistingFile() {
      try {
        const result = await xdB.del.all('temp_log');
        console.log(`[Przykład 1 del.all] Opróżniono plik: ${result.path}`);
        // Sprawdź zawartość (powinna być [])
        // Check content (should be [])
        const content = await xdB.view.all('temp_log');
        console.log(`[Przykład 1 del.all] Zawartość po opróżnieniu:`, content.data); // Output: []
      } catch (error) {
        console.error("[Przykład 1 del.all] Błąd:", error.message, error.code);
      }
    }

    // Przykład 2: Próba opróżnienia nieistniejącego pliku
    // Example 2: Attempting to empty a non-existent file
    async function emptyNonExistentFile() {
      try {
        const result = await xdB.del.all('ghost_log_file');
        console.log(`[Przykład 2 del.all] Niespodziewany sukces: ${result.path}`);
      } catch (error) {
        if (error.code === XDB_ERROR_CODES.FILE_NOT_FOUND) {
          console.log(`[Przykład 2 del.all] Zgodnie z oczekiwaniami, plik nie istnieje: ${error.message}`);
        } else {
          console.error("[Przykład 2 del.all] Inny błąd:", error.message, error.code);
        }
      }
    }

    // Uruchomienie przykładów sekwencyjnie
    // Run examples sequentially
    prepareForDelAll()
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(emptyExistingFile)
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(emptyNonExistentFile);
    ```

### `xdB.del.id(filePath, id)`

Asynchronicznie usuwa pojedynczy rekord z pliku JSON (który musi zawierać tablicę obiektów), identyfikowany przez jego unikalne pole `id`. Operacja atomowo odczytuje plik, filtruje tablicę, usuwając rekord o podanym `id`, a następnie zapisuje zmodyfikowaną tablicę z powrotem do pliku. Automatycznie dodaje rozszerzenie `.json` do `filePath`, jeśli go brakuje. Używa blokady na pliku.

(Asynchronously deletes a single record from a JSON file (which must contain an array of objects), identified by its unique `id` field. The operation atomically reads the file, filters the array removing the record with the specified `id`, and then writes the modified array back to the file. Automatically adds the `.json` extension to `filePath` if missing. Uses a lock on the file.)

*   **Parametry (Parameters):**
    *   `filePath` (`string`): Ścieżka do pliku JSON (względna do `basePath`). Rozszerzenie `.json` jest opcjonalne. (Path to the JSON file (relative to `basePath`). The `.json` extension is optional.)
    *   `id` (`number`): Numeryczne ID rekordu, który ma zostać usunięty. (The numeric ID of the record to be deleted.)
*   **Zwraca (Returns):**
    *   `Promise<{path: string, deletedId: number}>`: Obietnica, która po pomyślnym usunięciu rekordu rozwiązuje się obiektem zawierającym **absolutną** ścieżkę pliku (z rozszerzeniem `.json`) oraz **ID** usuniętego rekordu. (A promise that, upon successful record deletion, resolves with an object containing the **absolute** file path (with the `.json` extension) and the **ID** of the deleted record.)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: Jeśli `id` nie jest liczbą. (If `id` is not a number.)
    *   `Error & {code: XDB_ERROR_CODES.FILE_NOT_FOUND}`: Jeśli plik określony przez `filePath` nie istnieje. (If the file specified by `filePath` does not exist.)
    *   `Error & {code: XDB_ERROR_CODES.INVALID_JSON}`: Jeśli plik istnieje, ale zawiera nieprawidłowe dane JSON. (If the file exists but contains invalid JSON data.)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: Jeśli plik nie zawiera tablicy JSON. (If the file does not contain a JSON array.)
    *   `Error & {code: XDB_ERROR_CODES.RECORD_NOT_FOUND}`: Jeśli w pliku nie znaleziono rekordu o podanym `id` do usunięcia. (If no record with the specified `id` was found in the file for deletion.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku problemów z uprawnieniami lub innych błędów systemu plików podczas odczytu lub zapisu. (In case of permission issues or other file system errors during reading or writing.)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: W przypadku problemów z uzyskaniem blokady na pliku (np. timeout). (In case of problems acquiring the lock on the file (e.g., timeout).)
*   **Przykłady (Examples):**

    ```javascript
    import xdB, { XDB_ERROR_CODES } from './xdb.js';

    // Przygotowanie: Utwórz plik z rekordami
    // Preparation: Create a file with records
    async function prepareForDelId() {
        try {
            const initialData = [
                { id: 10, item: 'stare zadanie', done: true },
                { id: 20, item: 'ważne zadanie', done: false },
                { id: 30, item: 'inne zadanie', done: false }
            ];
            await xdB.add.all('tasks_to_delete_from', initialData);
            console.log('[Przygotowanie do del.id] Utworzono plik tasks_to_delete_from.json.');
        } catch (e) { console.error('Błąd przygotowania:', e); }
    }

    // Przykład 1: Usunięcie istniejącego rekordu
    // Example 1: Deleting an existing record
    async function deleteExistingRecord() {
      try {
        const result = await xdB.del.id('tasks_to_delete_from', 10);
        console.log(`[Przykład 1 del.id] Usunięto rekord o ID ${result.deletedId} z pliku ${result.path}`);
        // Sprawdź zawartość (rekord 10 powinien zniknąć)
        // Check content (record 10 should be gone)
        const content = await xdB.view.all('tasks_to_delete_from');
        console.log(`[Przykład 1 del.id] Zawartość po usunięciu:`, content.data);
        // Output: [ { id: 20, ... }, { id: 30, ... } ]
      } catch (error) {
        console.error("[Przykład 1 del.id] Błąd:", error.message, error.code);
      }
    }

    // Przykład 2: Próba usunięcia nieistniejącego rekordu
    // Example 2: Attempting to delete a non-existent record
    async function deleteNonExistentRecord() {
      try {
        const result = await xdB.del.id('tasks_to_delete_from', 99);
        console.log(`[Przykład 2 del.id] Niespodziewany sukces:`, result.deletedId);
      } catch (error) {
        if (error.code === XDB_ERROR_CODES.RECORD_NOT_FOUND) {
          console.log(`[Przykład 2 del.id] Zgodnie z oczekiwaniami, rekord nie znaleziony: ${error.message}`);
        } else {
          console.error("[Przykład 2 del.id] Inny błąd:", error.message, error.code);
        }
      }
    }

    // Przykład 3: Próba usunięcia z nieprawidłowym ID (nie-liczba)
    // Example 3: Attempting to delete with an invalid ID (non-number)
    async function deleteWithInvalidId() {
        try {
            const result = await xdB.del.id('tasks_to_delete_from', 'abc');
            console.log(`[Przykład 3 del.id] Niespodziewany sukces:`, result.deletedId);
        } catch(error) {
             if (error.code === XDB_ERROR_CODES.OPERATION_FAILED && error.message.includes('ID must be a number')) {
                console.log(`[Przykład 3 del.id] Zgodnie z oczekiwaniami, błąd walidacji ID: ${error.message}`);
            } else {
                 console.error("[Przykład 3 del.id] Inny błąd:", error.message, error.code);
            }
        }
    }

    // Uruchomienie przykładów sekwencyjnie
    // Run examples sequentially
    prepareForDelId()
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(deleteExistingRecord)
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(deleteNonExistentRecord)
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(deleteWithInvalidId);
    ```

## Operacje Dodawania (`xdB.add`) (Add Operations)

### `xdB.add.all(filePath, initialData, options)`

Asynchronicznie tworzy nowy plik JSON lub nadpisuje istniejący, zapisując w nim podane `initialData`. Operacja jest atomowa. Domyślnie nadpisuje plik, jeśli istnieje, ale można to zmienić za pomocą opcji `overwrite`. Automatycznie dodaje rozszerzenie `.json` do `filePath`, jeśli go brakuje, i tworzy niezbędne katalogi nadrzędne. Używa blokady na pliku.

(Asynchronously creates a new JSON file or overwrites an existing one, writing the provided `initialData` into it. The operation is atomic. By default, it overwrites the file if it exists, but this can be changed using the `overwrite` option. Automatically adds the `.json` extension to `filePath` if missing and creates necessary parent directories. Uses a lock on the file.)

*   **Parametry (Parameters):**
    *   `filePath` (`string`): Ścieżka do pliku JSON (względna do `basePath`). Rozszerzenie `.json` jest opcjonalne. (Path to the JSON file (relative to `basePath`). The `.json` extension is optional.)
    *   `initialData` (`Array` | `object`, opcjonalnie, domyślnie `[]`): Dane (tablica JavaScript lub obiekt), które mają zostać zapisane jako zawartość pliku. Muszą być serializowalne do formatu JSON. Jeśli `initialData` jest tablicą, jej elementy powinny być obiektami (walidacja `validateRecord`). (The data (JavaScript array or object) to be written as the file content. Must be serializable to JSON. If `initialData` is an array, its elements should be objects (validated by `validateRecord`).)
    *   `options` (`object`, opcjonalnie, domyślnie `{ overwrite: true }`): Opcje operacji. (Options for the operation.)
        *   `overwrite` (`boolean`, domyślnie `true`): Jeśli `true`, istniejący plik zostanie nadpisany. Jeśli `false`, a plik już istnieje, operacja zakończy się błędem `XDB_ERROR_CODES.OPERATION_FAILED`. (If `true`, an existing file will be overwritten. If `false` and the file already exists, the operation will fail with an `XDB_ERROR_CODES.OPERATION_FAILED` error.)
*   **Zwraca (Returns):**
    *   `Promise<{path: string}>`: Obietnica, która po pomyślnym utworzeniu/nadpisaniu pliku rozwiązuje się obiektem zawierającym **absolutną** ścieżkę zapisanego pliku (z rozszerzeniem `.json`). (A promise that, upon successful file creation/overwrite, resolves with an object containing the **absolute** path of the written file (with the `.json` extension).)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: Jeśli `initialData` nie jest tablicą ani obiektem, lub jeśli zawiera nieprawidłowe dane (np. rekordy w tablicy nie są obiektami). (If `initialData` is not an array or an object, or if it contains invalid data (e.g., records in an array are not objects).)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: Jeśli `options.overwrite` jest `false`, a plik określony przez `filePath` już istnieje. (If `options.overwrite` is `false` and the file specified by `filePath` already exists.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku problemów z uprawnieniami, błędów podczas tworzenia katalogów, błędów zapisu do pliku tymczasowego lub operacji `rename`. (In case of permission issues, errors during directory creation, errors writing to the temporary file, or `rename` operation failures.)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: W przypadku problemów z uzyskaniem blokady na pliku (np. timeout). (In case of problems acquiring the lock on the file (e.g., timeout).)
*   **Przykłady (Examples):**

    ```javascript
    import xdB, { XDB_ERROR_CODES } from './xdb.js';

    // Przykład 1: Utworzenie nowego pliku z tablicą (domyślnie nadpisuje)
    // Example 1: Creating a new file with an array (overwrites by default)
    async function createOrOverwriteArray() {
      const initialProducts = [ { id: 1, name: 'Laptop', price: 1200 }, { id: 2, name: 'Mouse', price: 25 } ];
      try {
        const result = await xdB.add.all('inventory/products', initialProducts);
        console.log(`[Przykład 1 add.all] Utworzono/nadpisano plik: ${result.path}`);
      } catch (error) {
        console.error("[Przykład 1 add.all] Błąd:", error.message, error.code);
      }
    }
    createOrOverwriteArray();

    // Przykład 2: Utworzenie nowego pliku z obiektem, bez nadpisywania
    // Example 2: Creating a new file with an object, without overwriting
    async function createFileNoOverwrite() {
      const siteConfig = { title: 'Moja Strona', lang: 'pl' };
      try {
        const result = await xdB.add.all('site_config', siteConfig, { overwrite: false });
        console.log(`[Przykład 2 add.all] Utworzono plik: ${result.path}`);
      } catch (error) {
        if (error.code === XDB_ERROR_CODES.OPERATION_FAILED && error.message.includes('already exists')) {
          console.log(`[Przykład 2 add.all] Plik site_config.json już istnieje, nie nadpisano.`);
        } else {
          console.error("[Przykład 2 add.all] Błąd:", error.message, error.code);
        }
      }
    }
    // Uruchom drugi raz, aby zobaczyć komunikat o istnieniu pliku
    // Run a second time to see the file exists message
    setTimeout(() => {
        createFileNoOverwrite().then(() => setTimeout(createFileNoOverwrite, 100));
    }, 200);


    // Przykład 3: Próba utworzenia pliku z nieprawidłowymi danymi w tablicy
    // Example 3: Attempting to create a file with invalid data in the array
    async function createWithInvalidRecord() {
        const invalidData = [{ id: 1 }, "nie obiekt"];
        try {
            const result = await xdB.add.all('invalid_records', invalidData);
            console.log(`[Przykład 3 add.all] Niespodziewany sukces: ${result.path}`);
        } catch(error) {
            if (error.code === XDB_ERROR_CODES.OPERATION_FAILED && error.message.includes('Invalid record')) {
                console.log(`[Przykład 3 add.all] Zgodnie z oczekiwaniami, błąd walidacji rekordu: ${error.message}`);
            } else {
                 console.error("[Przykład 3 add.all] Inny błąd:", error.message, error.code);
            }
        }
    }
     setTimeout(createWithInvalidRecord, 400);
    ```

### `xdB.add.id(filePath, newRecord)`

Asynchronicznie dodaje nowy rekord (obiekt) do pliku JSON, który musi zawierać tablicę. Operacja jest atomowa.
Jeśli plik nie istnieje, zostanie utworzony z tablicą zawierającą tylko nowy rekord.
Jeśli `newRecord` nie zawiera pola `id` (lub jest ono `undefined`/`null`), biblioteka **automatycznie wygeneruje unikalne, kolejne numeryczne ID** dla tego rekordu w kontekście danego pliku. Mechanizm ten opiera się na odczycie pliku `_xdB_counters.json` w katalogu `basePath` oraz skanowaniu istniejących ID w pliku docelowym, aby zapewnić unikalność i ciągłość (znajduje maksymalne istniejące ID i dodaje 1).
Jeśli `newRecord` zawiera pole `id`, biblioteka sprawdzi, czy rekord o takim ID już istnieje w pliku. Jeśli tak, rzuci błąd `RECORD_EXISTS`.
Automatycznie dodaje rozszerzenie `.json` do `filePath`, jeśli go brakuje. Używa blokady na pliku docelowym oraz potencjalnie na pliku `_xdB_counters.json`.

(Asynchronously adds a new record (object) to a JSON file, which must contain an array. The operation is atomic.
If the file does not exist, it will be created with an array containing only the new record.
If `newRecord` does not include an `id` field (or it is `undefined`/`null`), the library will **automatically generate a unique, sequential numeric ID** for this record within the context of the given file. This mechanism relies on reading the `_xdB_counters.json` file in the `basePath` directory and scanning existing IDs in the target file to ensure uniqueness and continuity (it finds the maximum existing ID and adds 1).
If `newRecord` includes an `id` field, the library will check if a record with that ID already exists in the file. If so, it will throw a `RECORD_EXISTS` error.
Automatically adds the `.json` extension to `filePath` if missing. Uses a lock on the target file and potentially on the `_xdB_counters.json` file.)

*   **Parametry (Parameters):**
    *   `filePath` (`string`): Ścieżka do pliku JSON (względna do `basePath`). Rozszerzenie `.json` jest opcjonalne. (Path to the JSON file (relative to `basePath`). The `.json` extension is optional.)
    *   `newRecord` (`object`): Obiekt rekordu do dodania. Musi być prawidłowym obiektem JavaScript. Pole `id` jest opcjonalne dla automatycznego generowania. (The record object to add. Must be a valid JavaScript object. The `id` field is optional for auto-generation.)
*   **Zwraca (Returns):**
    *   `Promise<{path: string, record: object}>`: Obietnica, która po pomyślnym dodaniu rekordu rozwiązuje się obiektem zawierającym **absolutną** ścieżkę pliku (z rozszerzeniem `.json`) oraz **dodany obiekt rekordu** (wraz z przypisanym lub podanym `id`). (A promise that, upon successful record addition, resolves with an object containing the **absolute** file path (with the `.json` extension) and the **added record object** (including its assigned or provided `id`).)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: Jeśli `newRecord` nie jest obiektem (lub jest `null`/tablicą). (If `newRecord` is not an object (or is `null`/array).)
    *   `Error & {code: XDB_ERROR_CODES.INVALID_JSON}`: Jeśli istniejący plik (`filePath` lub `_xdB_counters.json`) zawiera nieprawidłowe dane JSON. (If an existing file (`filePath` or `_xdB_counters.json`) contains invalid JSON data.)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: Jeśli plik (`filePath`) istnieje, ale nie zawiera tablicy JSON. (If the file (`filePath`) exists but does not contain a JSON array.)
    *   `Error & {code: XDB_ERROR_CODES.RECORD_EXISTS}`: Jeśli `newRecord` zawiera `id`, a rekord o tym `id` już istnieje w pliku. (If `newRecord` includes an `id`, and a record with that `id` already exists in the file.)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: Jeśli podane `id` w `newRecord` nie jest liczbą. (If the provided `id` in `newRecord` is not a number.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku problemów z uprawnieniami lub innych błędów systemu plików podczas odczytu/zapisu plików (w tym `_xdB_counters.json`). (In case of permission issues or other file system errors during file read/write operations (including `_xdB_counters.json`).)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: W przypadku problemów z uzyskaniem blokady na plikach (np. timeout). (In case of problems acquiring locks on the files (e.g., timeout).)
*   **Przykłady (Examples):**

    ```javascript
    import xdB, { XDB_ERROR_CODES } from './xdb.js';

    // Przykład 1: Dodanie rekordu z automatycznie generowanym ID (do nowego pliku)
    // Example 1: Adding a record with auto-generated ID (to a new file)
    async function addWithAutoIdNewFile() {
      const task = { description: 'Kup mleko', priority: 'high' };
      try {
        const result = await xdB.add.id('shopping_list', task);
        console.log(`[Przykład 1 add.id] Dodano rekord z ID ${result.record.id} do nowego pliku ${result.path}`);
        // Output: [Przykład 1 add.id] Dodano rekord z ID 1 do nowego pliku /path/to/base/shopping_list.json
      } catch (error) {
        console.error("[Przykład 1 add.id] Błąd:", error.message, error.code);
      }
    }
    addWithAutoIdNewFile();

    // Przykład 2: Dodanie kolejnego rekordu z automatycznym ID (do istniejącego pliku)
    // Example 2: Adding another record with auto ID (to an existing file)
    async function addWithAutoIdExistingFile() {
      const task = { description: 'Zapłać rachunki', priority: 'medium' };
      try {
        // Zakładając, że plik shopping_list.json istnieje po Przykładzie 1
        // Assuming shopping_list.json exists after Example 1
        const result = await xdB.add.id('shopping_list', task);
        console.log(`[Przykład 2 add.id] Dodano kolejny rekord z ID ${result.record.id} do pliku ${result.path}`);
         // Output: [Przykład 2 add.id] Dodano kolejny rekord z ID 2 do pliku /path/to/base/shopping_list.json
      } catch (error) {
        console.error("[Przykład 2 add.id] Błąd:", error.message, error.code);
      }
    }
    setTimeout(addWithAutoIdExistingFile, 100);

    // Przykład 3: Dodanie rekordu z konkretnym ID
    // Example 3: Adding a record with a specific ID
    async function addWithSpecificId() {
      const user = { id: 100, username: 'admin', level: 99 };
      try {
        const result = await xdB.add.id('admin_users', user); // Tworzy plik, jeśli nie istnieje
        console.log(`[Przykład 3 add.id] Dodano rekord z podanym ID ${result.record.id} do pliku ${result.path}`);
      } catch (error) {
        console.error("[Przykład 3 add.id] Błąd:", error.message, error.code);
      }
    }
     setTimeout(addWithSpecificId, 200);

    // Przykład 4: Próba dodania rekordu z ID, które już istnieje
    // Example 4: Attempting to add a record with an ID that already exists
    async function addWithExistingId() {
      const user = { id: 100, username: 'another_admin' }; // ID 100 już istnieje z Przykładu 3
      try {
        const result = await xdB.add.id('admin_users', user);
        console.log(`[Przykład 4 add.id] Niespodziewany sukces:`, result.record);
      } catch (error) {
        if (error.code === XDB_ERROR_CODES.RECORD_EXISTS) {
          console.log(`[Przykład 4 add.id] Zgodnie z oczekiwaniami, rekord o ID 100 już istnieje: ${error.message}`);
        } else {
          console.error("[Przykład 4 add.id] Inny błąd:", error.message, error.code);
        }
      }
    }
    setTimeout(addWithExistingId, 300);

     // Przykład 5: Próba dodania nie-obiektu jako rekordu
    // Example 5: Attempting to add a non-object as a record
    async function addInvalidRecordType() {
        try {
            const result = await xdB.add.id('some_file', "to nie jest obiekt");
            console.log(`[Przykład 5 add.id] Niespodziewany sukces:`, result.record);
        } catch(error) {
             if (error.code === XDB_ERROR_CODES.OPERATION_FAILED && error.message.includes('non-null object')) {
                console.log(`[Przykład 5 add.id] Zgodnie z oczekiwaniami, błąd walidacji typu rekordu: ${error.message}`);
            } else {
                 console.error("[Przykład 5 add.id] Inny błąd:", error.message, error.code);
            }
        }
    }
    setTimeout(addInvalidRecordType, 400);
    ```

## Operacje Odczytu (`xdB.view`) (View Operations)

### `xdB.view.all(filePath)`

Asynchronicznie odczytuje całą zawartość pliku JSON i zwraca ją jako sparsowany obiekt JavaScript (tablicę lub obiekt). Automatycznie dodaje rozszerzenie `.json` do `filePath`, jeśli go brakuje. Ta operacja nie używa blokady pliku.

(Asynchronously reads the entire content of a JSON file and returns it as a parsed JavaScript object (array or object). Automatically adds the `.json` extension to `filePath` if missing. This operation does not use a file lock.)

*   **Parametry (Parameters):**
    *   `filePath` (`string`): Ścieżka do pliku JSON (względna do `basePath`). Rozszerzenie `.json` jest opcjonalne. (Path to the JSON file (relative to `basePath`). The `.json` extension is optional.)
*   **Zwraca (Returns):**
    *   `Promise<{path: string, data: Array|object}>`: Obietnica, która po pomyślnym odczycie i sparsowaniu rozwiązuje się obiektem zawierającym **absolutną** ścieżkę pliku (z rozszerzeniem `.json`) oraz sparsowane dane (`data`). (A promise that, upon successful read and parse, resolves with an object containing the **absolute** file path (with the `.json` extension) and the parsed data (`data`).)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.FILE_NOT_FOUND}`: Jeśli plik określony przez `filePath` nie istnieje. (If the file specified by `filePath` does not exist.)
    *   `Error & {code: XDB_ERROR_CODES.INVALID_JSON}`: Jeśli plik istnieje, ale zawiera nieprawidłowe dane JSON, które nie mogą zostać sparsowane. (If the file exists but contains invalid JSON data that cannot be parsed.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku problemów z uprawnieniami lub innych błędów systemu plików podczas odczytu. (In case of permission issues or other file system errors during reading.)
*   **Przykłady (Examples):**

    ```javascript
    import xdB, { XDB_ERROR_CODES } from './xdb.js';

    // Przygotowanie: Utwórz pliki z różną zawartością
    // Preparation: Create files with different content
    async function prepareForViewAll() {
        try {
            await xdB.add.all('view_all_array', [{ id: 1, value: 'a' }, { id: 2, value: 'b' }]);
            await xdB.add.all('view_all_object', { setting: 'enabled', level: 5 });
            await xdB.add.all('view_all_empty_array', []);
            console.log('[Przygotowanie do view.all] Utworzono pliki testowe.');
        } catch (e) { console.error('Błąd przygotowania:', e); }
    }

    // Przykład 1: Odczyt pliku zawierającego tablicę
    // Example 1: Reading a file containing an array
    async function readArrayFile() {
      try {
        const result = await xdB.view.all('view_all_array');
        console.log(`[Przykład 1 view.all] Odczytano z ${result.path}:`, result.data);
        // Output: [Przykład 1 view.all] Odczytano z /path/to/base/view_all_array.json: [ { id: 1, value: 'a' }, { id: 2, value: 'b' } ]
      } catch (error) {
        console.error("[Przykład 1 view.all] Błąd:", error.message, error.code);
      }
    }

    // Przykład 2: Odczyt pliku zawierającego obiekt
    // Example 2: Reading a file containing an object
    async function readObjectFile() {
      try {
        const result = await xdB.view.all('view_all_object'); // .json dodane automatycznie
        console.log(`[Przykład 2 view.all] Odczytano z ${result.path}:`, result.data);
        // Output: [Przykład 2 view.all] Odczytano z /path/to/base/view_all_object.json: { setting: 'enabled', level: 5 }
      } catch (error) {
        console.error("[Przykład 2 view.all] Błąd:", error.message, error.code);
      }
    }

    // Przykład 3: Odczyt pliku z pustą tablicą
    // Example 3: Reading a file with an empty array
    async function readEmptyArrayFile() {
      try {
        const result = await xdB.view.all('view_all_empty_array');
        console.log(`[Przykład 3 view.all] Odczytano z ${result.path}:`, result.data);
        // Output: [Przykład 3 view.all] Odczytano z /path/to/base/view_all_empty_array.json: []
      } catch (error) {
        console.error("[Przykład 3 view.all] Błąd:", error.message, error.code);
      }
    }

    // Przykład 4: Próba odczytu nieistniejącego pliku
    // Example 4: Attempting to read a non-existent file
    async function readNonExistentFile() {
        try {
            const result = await xdB.view.all('non_existent_file_view');
            console.log(`[Przykład 4 view.all] Niespodziewany sukces:`, result.data);
        } catch(error) {
            if (error.code === XDB_ERROR_CODES.FILE_NOT_FOUND) {
                console.log(`[Przykład 4 view.all] Zgodnie z oczekiwaniami, plik nie znaleziony: ${error.message}`);
            } else {
                 console.error("[Przykład 4 view.all] Inny błąd:", error.message, error.code);
            }
        }
    }

    // Uruchomienie przykładów sekwencyjnie
    // Run examples sequentially
    prepareForViewAll()
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(readArrayFile)
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(readObjectFile)
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(readEmptyArrayFile)
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(readNonExistentFile);
    ```

### `xdB.view.id(filePath, id)`

Asynchronicznie odczytuje plik JSON (który musi zawierać tablicę obiektów), znajduje rekord o podanym `id` i zwraca go. Automatycznie dodaje rozszerzenie `.json` do `filePath`, jeśli go brakuje. Ta operacja nie używa blokady pliku.

(Asynchronously reads a JSON file (which must contain an array of objects), finds the record with the specified `id`, and returns it. Automatically adds the `.json` extension to `filePath` if missing. This operation does not use a file lock.)

*   **Parametry (Parameters):**
    *   `filePath` (`string`): Ścieżka do pliku JSON (względna do `basePath`). Rozszerzenie `.json` jest opcjonalne. (Path to the JSON file (relative to `basePath`). The `.json` extension is optional.)
    *   `id` (`number`): Numeryczne ID rekordu, który ma zostać wyszukany i zwrócony. (The numeric ID of the record to find and return.)
*   **Zwraca (Returns):**
    *   `Promise<{path: string, record: object}>`: Obietnica, która po pomyślnym znalezieniu rekordu rozwiązuje się obiektem zawierającym **absolutną** ścieżkę pliku (z rozszerzeniem `.json`) oraz znaleziony **obiekt rekordu**. (A promise that, upon successfully finding the record, resolves with an object containing the **absolute** file path (with the `.json` extension) and the found **record object**.)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: Jeśli `id` nie jest liczbą. (If `id` is not a number.)
    *   `Error & {code: XDB_ERROR_CODES.FILE_NOT_FOUND}`: Jeśli plik określony przez `filePath` nie istnieje. (If the file specified by `filePath` does not exist.)
    *   `Error & {code: XDB_ERROR_CODES.INVALID_JSON}`: Jeśli plik istnieje, ale zawiera nieprawidłowe dane JSON. (If the file exists but contains invalid JSON data.)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: Jeśli plik nie zawiera tablicy JSON. (If the file does not contain a JSON array.)
    *   `Error & {code: XDB_ERROR_CODES.RECORD_NOT_FOUND}`: Jeśli w pliku nie znaleziono rekordu o podanym `id`. (If no record with the specified `id` was found in the file.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku problemów z uprawnieniami lub innych błędów systemu plików podczas odczytu. (In case of permission issues or other file system errors during reading.)
*   **Przykłady (Examples):**

    ```javascript
    import xdB, { XDB_ERROR_CODES } from './xdb.js';

    // Przygotowanie: Utwórz plik z rekordami
    // Preparation: Create a file with records
    async function prepareForViewId() {
        try {
            const initialData = [
                { id: 5, product: 'Klawiatura', price: 75 },
                { id: 15, product: 'Monitor', price: 300 },
                { id: 25, product: 'Mysz', price: 25 }
            ];
            await xdB.add.all('products_to_view', initialData);
            console.log('[Przygotowanie do view.id] Utworzono plik products_to_view.json.');
        } catch (e) { console.error('Błąd przygotowania:', e); }
    }

    // Przykład 1: Znalezienie istniejącego rekordu
    // Example 1: Finding an existing record
    async function findExistingRecord() {
      try {
        const result = await xdB.view.id('products_to_view', 15);
        console.log(`[Przykład 1 view.id] Znaleziono rekord w ${result.path}:`, result.record);
        // Output: [Przykład 1 view.id] Znaleziono rekord w /path/to/base/products_to_view.json: { id: 15, product: 'Monitor', price: 300 }
      } catch (error) {
        console.error("[Przykład 1 view.id] Błąd:", error.message, error.code);
      }
    }

    // Przykład 2: Próba znalezienia nieistniejącego rekordu
    // Example 2: Attempting to find a non-existent record
    async function findNonExistentRecord() {
      try {
        const result = await xdB.view.id('products_to_view', 99);
        console.log(`[Przykład 2 view.id] Niespodziewany sukces:`, result.record);
      } catch (error) {
        if (error.code === XDB_ERROR_CODES.RECORD_NOT_FOUND) {
          console.log(`[Przykład 2 view.id] Zgodnie z oczekiwaniami, rekord nie znaleziony: ${error.message}`);
        } else {
          console.error("[Przykład 2 view.id] Inny błąd:", error.message, error.code);
        }
      }
    }

    // Przykład 3: Próba odczytu z nieistniejącego pliku
    // Example 3: Attempting to read from a non-existent file
    async function readFromNonExistentFile() {
        try {
            const result = await xdB.view.id('imaginary_products', 1);
            console.log(`[Przykład 3 view.id] Niespodziewany sukces:`, result.record);
        } catch(error) {
             if (error.code === XDB_ERROR_CODES.FILE_NOT_FOUND) {
                console.log(`[Przykład 3 view.id] Zgodnie z oczekiwaniami, plik nie znaleziony: ${error.message}`);
            } else {
                 console.error("[Przykład 3 view.id] Inny błąd:", error.message, error.code);
            }
        }
    }

    // Uruchomienie przykładów sekwencyjnie
    // Run examples sequentially
    prepareForViewId()
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(findExistingRecord)
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(findNonExistentRecord)
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(readFromNonExistentFile);
    ```

### `xdB.view.more(filePath, options)`

Asynchronicznie odczytuje dane z pliku JSON (który musi zawierać tablicę obiektów) i stosuje zaawansowane opcje filtrowania, sortowania oraz paginacji (skip/limit) przed zwróceniem wyników. Automatycznie dodaje rozszerzenie `.json` do `filePath`, jeśli go brakuje. Ta operacja nie używa blokady pliku.

(Asynchronously reads data from a JSON file (which must contain an array of objects) and applies advanced filtering, sorting, and pagination (skip/limit) options before returning the results. Automatically adds the `.json` extension to `filePath` if missing. This operation does not use a file lock.)

*   **Parametry (Parameters):**
    *   `filePath` (`string`): Ścieżka do pliku JSON (względna do `basePath`). Rozszerzenie `.json` jest opcjonalne. (Path to the JSON file (relative to `basePath`). The `.json` extension is optional.)
    *   `options` (`object`, opcjonalnie, domyślnie `{}`): Obiekt zawierający opcje zapytania. (An object containing query options.)
        *   `filter` (`function(record: object): boolean`, opcjonalnie): Funkcja, która otrzymuje każdy rekord jako argument. Powinna zwrócić `true`, jeśli rekord ma zostać uwzględniony w wynikach, w przeciwnym razie `false`. (A function that receives each record as an argument. Should return `true` if the record should be included in the results, `false` otherwise.)
        *   `sort` (`object` | `Array<object>`, opcjonalnie): Kryteria sortowania.
            *   Pojedynczy obiekt: `{key: string, order?: 'asc'|'desc', comparator?: function(a: any, b: any): number}`. Sortuje po jednym kluczu. `comparator` ma pierwszeństwo przed `key` i `order`. (Single object: `{key: string, order?: 'asc'|'desc', comparator?: function(a: any, b: any): number}`. Sorts by a single key. `comparator` takes precedence over `key` and `order`.)
            *   Tablica obiektów: `[{key: 'key1', order: 'asc'}, {key: 'key2', order: 'desc'}]`. Umożliwia sortowanie po wielu kluczach w podanej kolejności. (Array of objects: `[{key: 'key1', order: 'asc'}, {key: 'key2', order: 'desc'}]`. Allows sorting by multiple keys in the specified order.)
            *   `key` (`string`): Nazwa pola (klucza) w rekordzie, po którym ma nastąpić sortowanie. Wymagane, jeśli nie podano `comparator`. (The name of the field (key) in the record to sort by. Required if `comparator` is not provided.)
            *   `order` (`'asc'` | `'desc'`, opcjonalnie, domyślnie `'asc'`): Kierunek sortowania ('asc' - rosnąco, 'desc' - malejąco). (Sort order ('asc' - ascending, 'desc' - descending).)
            *   `comparator` (`function(a: object, b: object): number`, opcjonalnie): Niestandardowa funkcja porównująca, która otrzymuje dwa **całe rekordy** (`a` i `b`) i powinna zwrócić liczbę ujemną, zero lub dodatnią, zgodnie ze standardem `Array.prototype.sort`. (Optional custom comparison function that receives two **entire records** (`a` and `b`) and should return a negative number, zero, or a positive number, following the `Array.prototype.sort` standard.)
        *   `skip` (`number`, opcjonalnie): Liczba rekordów do pominięcia od początku **po** przefiltrowaniu i posortowaniu. Musi być liczbą nieujemną. (Number of records to skip from the beginning **after** filtering and sorting. Must be a non-negative number.)
        *   `limit` (`number`, opcjonalnie): Maksymalna liczba rekordów do zwrócenia **po** przefiltrowaniu, posortowaniu i pominięciu (`skip`). Musi być liczbą nieujemną. (Maximum number of records to return **after** filtering, sorting, and skipping (`skip`). Must be a non-negative number.)
*   **Zwraca (Returns):**
    *   `Promise<{path: string, data: Array<object>}>`: Obietnica, która po pomyślnym odczycie, przetworzeniu i sparsowaniu rozwiązuje się obiektem zawierającym **absolutną** ścieżkę pliku (z rozszerzeniem `.json`) oraz wynikową **tablicę rekordów** (`data`) pasujących do kryteriów. (A promise that, upon successful read, processing, and parse, resolves with an object containing the **absolute** file path (with the `.json` extension) and the resulting **array of records** (`data`) matching the criteria.)
*   **Rzuca (Throws):**
    *   `Error & {code: XDB_ERROR_CODES.FILE_NOT_FOUND}`: Jeśli plik określony przez `filePath` nie istnieje. (If the file specified by `filePath` does not exist.)
    *   `Error & {code: XDB_ERROR_CODES.INVALID_JSON}`: Jeśli plik istnieje, ale zawiera nieprawidłowe dane JSON. (If the file exists but contains invalid JSON data.)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: Jeśli plik nie zawiera tablicy JSON. (If the file does not contain a JSON array.)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: Jeśli podane opcje (`filter`, `sort`, `skip`, `limit`) są nieprawidłowego typu lub formatu (np. `filter` nie jest funkcją, `skip` jest ujemny, kryterium `sort` jest niepoprawne). (If the provided options (`filter`, `sort`, `skip`, `limit`) are of invalid type or format (e.g., `filter` is not a function, `skip` is negative, `sort` criterion is incorrect).)
    *   `Error & {code: XDB_ERROR_CODES.OPERATION_FAILED}`: Jeśli wystąpi błąd podczas wykonywania funkcji `filter` lub `comparator`. (If an error occurs during the execution of the `filter` or `comparator` function.)
    *   `Error & {code: XDB_ERROR_CODES.IO_ERROR}`: W przypadku problemów z uprawnieniami lub innych błędów systemu plików podczas odczytu. (In case of permission issues or other file system errors during reading.)
*   **Przykłady (Examples):**

    ```javascript
    import xdB, { XDB_ERROR_CODES } from './xdb.js';

    // Przygotowanie: Utwórz plik z danymi
    // Preparation: Create a file with data
    async function prepareForViewMore() {
        try {
            const data = [
                { id: 3, name: 'Charlie', score: 85, group: 'A', registered: new Date('2023-01-15') },
                { id: 1, name: 'Alice', score: 90, group: 'B', registered: new Date('2023-01-10') },
                { id: 4, name: 'David', score: 85, group: 'B', registered: new Date('2023-02-01') },
                { id: 2, name: 'Bob', score: 95, group: 'A', registered: new Date('2023-01-20') },
                { id: 5, name: 'Eve', score: 90, group: 'A', registered: new Date('2023-01-05') },
            ];
            await xdB.add.all('scores_data', data);
            console.log('[Przygotowanie do view.more] Utworzono plik scores_data.json.');
        } catch (e) { console.error('Błąd przygotowania:', e); }
    }

    // Przykład 1: Filtrowanie i sortowanie po jednym kluczu
    // Example 1: Filtering and sorting by a single key
    async function filterAndSort() {
      try {
        const options = {
          filter: record => record.group === 'A', // Tylko grupa A
          sort: { key: 'score', order: 'desc' } // Sortuj wg wyniku malejąco
        };
        const result = await xdB.view.more('scores_data', options);
        console.log(`[Przykład 1 view.more] Grupa A posortowana wg wyniku (desc):`, result.data);
        // Output: [ { id: 2, ..., score: 95 }, { id: 5, ..., score: 90 }, { id: 3, ..., score: 85 } ]
      } catch (error) {
        console.error("[Przykład 1 view.more] Błąd:", error.message, error.code);
      }
    }

    // Przykład 2: Sortowanie po wielu kluczach (wynik malejąco, potem nazwa rosnąco)
    // Example 2: Sorting by multiple keys (score descending, then name ascending)
    async function multiSort() {
      try {
        const options = {
          sort: [
            { key: 'score', order: 'desc' },
            { key: 'name', order: 'asc' }
          ]
        };
        const result = await xdB.view.more('scores_data', options);
        console.log(`[Przykład 2 view.more] Posortowane wg wyniku (desc), potem nazwy (asc):`, result.data);
        // Output: [ { id: 2, name: 'Bob', score: 95,... }, { id: 1, name: 'Alice', score: 90,... }, { id: 5, name: 'Eve', score: 90,... }, { id: 3, name: 'Charlie', score: 85,... }, { id: 4, name: 'David', score: 85,... } ]
      } catch (error) {
        console.error("[Przykład 2 view.more] Błąd:", error.message, error.code);
      }
    }

    // Przykład 3: Paginacja (druga strona, 2 elementy na stronie)
    // Example 3: Pagination (second page, 2 items per page)
    async function paginateResults() {
      try {
        const options = {
          sort: { key: 'id', order: 'asc' }, // Sortuj wg ID dla spójnej kolejności
          skip: 2, // Pomiń pierwsze 2 rekordy
          limit: 2  // Zwróć następne 2 rekordy
        };
        const result = await xdB.view.more('scores_data', options);
        console.log(`[Przykład 3 view.more] Druga strona wyników (limit 2):`, result.data);
         // Output: [ { id: 3, ... }, { id: 4, ... } ]
      } catch (error) {
        console.error("[Przykład 3 view.more] Błąd:", error.message, error.code);
      }
    }

    // Przykład 4: Użycie niestandardowego komparatora (sortowanie wg daty rejestracji)
    // Example 4: Using a custom comparator (sorting by registration date)
    async function customSort() {
        try {
            const options = {
                sort: {
                    // Klucz 'registered' jest obiektem Date, domyślne porównanie może nie działać jak oczekiwano
                    // The 'registered' key is a Date object, default comparison might not work as expected
                    // Użyj komparatora do porównania timestampów
                    // Use a comparator to compare timestamps
                    comparator: (a, b) => new Date(a.registered).getTime() - new Date(b.registered).getTime(),
                    order: 'asc' // Mimo że komparator definiuje porządek, 'order' jest nadal sprawdzane pod kątem poprawności
                }
            };
            const result = await xdB.view.more('scores_data', options);
            console.log(`[Przykład 4 view.more] Posortowane wg daty rejestracji (asc):`, result.data.map(r => ({id: r.id, registered: r.registered})));
             // Output: [ { id: 5, ... }, { id: 1, ... }, { id: 3, ... }, { id: 2, ... }, { id: 4, ... } ] (kolejność wg daty)
        } catch(error) {
            console.error("[Przykład 4 view.more] Błąd:", error.message, error.code);
        }
    }

    // Uruchomienie przykładów sekwencyjnie
    // Run examples sequentially
    prepareForViewMore()
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(filterAndSort)
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(multiSort)
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(paginateResults)
        .then(() => new Promise(resolve => setTimeout(resolve, 100)))
        .then(customSort);
    ```

## Kontrola Współbieżności (Concurrency Control)

xdB używa wewnętrznego mechanizmu blokad plików (`acquireLock`, `releaseLock`) opartego na `Map` i `Promise`, aby zapobiegać konfliktom podczas jednoczesnych operacji zapisu do tego samego pliku. Blokady są stosowane automatycznie w metodach modyfikujących pliki (`edit.*`, `del.*`, `add.*`, `move.*`, `dir.del`, `dir.rename`). Operacje odczytu (`view.*`) generalnie nie wymagają blokad, ale odczytują spójny stan pliku w momencie operacji.

(xdB uses an internal file locking mechanism (`acquireLock`, `releaseLock`) based on `Map` and `Promise` to prevent conflicts during concurrent write operations to the same file. Locks are applied automatically in methods that modify files (`edit.*`, `del.*`, `add.*`, `move.*`, `dir.del`, `dir.rename`). Read operations (`view.*`) generally do not require locks but read a consistent state of the file at the time of the operation.)

## Funkcje Pomocnicze (Helper Functions)

Biblioteka zawiera również wewnętrzne funkcje pomocnicze (np. `ensureJsonExtension`, `ensureDirectoryExists`, `safeParseJSON`, `validateId`, `validateRecord`, `createXdbError`), które generalnie nie są przeznaczone do bezpośredniego użytku zewnętrznego, ale wspierają działanie głównych metod API.

(The library also includes internal helper functions (e.g., `ensureJsonExtension`, `ensureDirectoryExists`, `safeParseJSON`, `validateId`, `validateRecord`, `createXdbError`) which are generally not intended for direct external use but support the operation of the main API methods.)

## Historia Zmian (Change History)

**2025-04-04:**
*   **Usprawnienia Zapisu Atomowego (Atomic Write Improvements):**
    *   Wprowadzono nową, scentralizowaną funkcję pomocniczą `atomicWrite` wykorzystującą uchwyty plików (`fs.open`) i `fileHandle.sync()` w celu zapewnienia większej pewności zapisu danych do pliku tymczasowego przed operacją `rename`. (Introduced a new, centralized helper function `atomicWrite` utilizing file handles (`fs.open`) and `fileHandle.sync()` to provide greater assurance of data being written to the temporary file before the `rename` operation.)
    *   Zrefaktoryzowano wszystkie metody zapisu (`edit.all`, `edit.id`, `del.all`, `del.id`, `add.all`, `add.id`) do użycia nowej funkcji `atomicWrite`. (Refactored all write methods (`edit.all`, `edit.id`, `del.all`, `del.id`, `add.all`, `add.id`) to use the new `atomicWrite` function.)
*   **Poprawa Komunikatów Błędów (Improved Error Messages):**
    *   Zaktualizowano komunikaty błędów w wielu metodach, aby zawierały więcej kontekstu (np. nazwę metody), co ułatwia diagnozowanie problemów. (Updated error messages in several methods to include more context (e.g., method name), facilitating easier problem diagnosis.)
*   **Logowanie Sprzątania (Cleanup Logging):**
    *   Zmieniono poziom logowania błędów (innych niż `ENOENT`) w funkcji `cleanupTempFile` z `console.error` na `console.warn`, aby lepiej odzwierciedlał charakter ostrzeżenia. (Changed the logging level for errors (other than `ENOENT`) in the `cleanupTempFile` function from `console.error` to `console.warn` to better reflect its nature as a warning.)

**2025-04-05:**
*   **Pełne Komentowanie Kodu (Comprehensive Code Commenting):**
    *   Usunięto wszystkie istniejące komentarze z pliku `xdb.js`. (Removed all existing comments from the `xdb.js` file.)
    *   Dodano szczegółowe komentarze w formacie JSDoc (`/** ... */`) do **każdej** funkcji, stałej i metody w bibliotece, włączając opisy, parametry (`@param`), zwracane wartości (`@returns`) i potencjalne błędy (`@throws`). (Added detailed JSDoc comments (`/** ... */`) to **every** function, constant, and method in the library, including descriptions, parameters (`@param`), return values (`@returns`), and potential errors (`@throws`).)
