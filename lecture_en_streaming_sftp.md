## Lire en streaming sur un serveur avec une connection SFTP

### 1. Télécharger la bibliothèque `flysystem-sftp-v3`
* De quoi s'agit-il exactement ?
   - Laravel offre une puissante couche d'abstraction pour travailler avec des fichiers. Il prend en charge plusieurs systèmes de stockage de fichiers par défaut et peut être facilement étendu. Pour utiliser SFTP comme système de fichiers Laravel, nous devons créer un pilote personnalisé qui utilise le package league/flysystem-sftp-v3. Flysystem est une bibliothèque d'abstraction de système de fichiers pour PHP qui fournit une API simple et unifiée pour travailler avec des fichiers à travers différents systèmes de stockage.

### 2. Dans les variables d'environement , ajouter les informations de connection : 
```.env
SFTP_HOST= bms-server.dev #host de votre serveur
SFTP_USERNAME="bms" #le nom d'utilisasteur
SFTP_PASSWORD="Azerty" #le mot de passe
SFTP_ROOT_PATH="/" #le chemin vers le dossier des fichiers 
```

### 3. Mise en place du disk dans le filsystem 
```php
//../config/filesystems.php

 'disks' => [
  //Ancien conteunu ....
        
        'sftp' => [
            'driver' => 'sftp',
            'host' => env('SFTP_HOST'),
            'port' => env('SFTP_PORT', 22),
            'username' => env('SFTP_USERNAME'),
            'password' => env('SFTP_PASSWORD'),
            'root' => env('SFTP_REC_PATH'),
        ],

    ],
```

### 4. Mise en place du controller : 
```bash
php artisan make:controller AudioController
```
```php
class AudioController extends Controller
{
    public function stream($filename)
    {
        // Vérifier si le fichier existe sur le serveur distant
        if (!Storage::disk('sftp')->exists($filename)) {
            abort(404, 'Fichier non trouvé');
        }

        // Diffuser le fichier en continu
        return new StreamedResponse(function () use ($filename) {
            $stream = Storage::disk('sftp')->readStream($filename);

            while (!feof($stream)) {
                echo fread($stream, 1024 * 8);
                ob_flush();
                flush();
            }
            fclose($stream);
        }, 200, [
            'Content-Type' => 'audio/mpeg',
            'Content-Disposition' => 'inline; filename="' . $filename . '"'
        ]);
    }
}
```
### 5. Mise en place de la route : 
```php
Route::get('/stream-audio/{filepath}',[AudioController::class,"stream"]);
```
* NB: n'oubliez pas d'importer le controlleur dans le fichier de route

* 😎C'est terminé.
  Maintenant testez en utilisant : http://127.0.0.1:8000/api/stream-audio/chemin-du-fichier
