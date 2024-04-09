  
En PHP, les wrappers sont des fonctionnalités qui permettent d'accéder à différents protocoles et ressources externes de manière transparente. Ils sont utilisés avec des fonctions comme `fopen()`, `file_get_contents()`, `include()` et d'autres pour travailler avec des fichiers, des URLs et d'autres types de ressources.

Voici quelques-uns des wrappers les plus couramment utilisés en PHP :

1. **`file://` :** Ce wrapper est utilisé pour accéder aux fichiers locaux sur le système de fichiers du serveur.
    
    Exemple :
```
`$file_content = file_get_contents('file:///chemin/vers/le/fichier.txt');`
```

2. **`http://` et `https://` :** Ces wrappers sont utilisés pour accéder à des ressources via les protocoles HTTP et HTTPS.
    
    Exemple :
    
    phpCopy code
    
    `$url_content = file_get_contents('http://www.example.com');`
    
3. **`ftp://` :** Ce wrapper est utilisé pour accéder à des fichiers via le protocole FTP (File Transfer Protocol).
    
    Exemple :
    
    phpCopy code
    
    `$ftp_content = file_get_contents('ftp://example.com/fichier.txt');`
    
4. **`php://` :** Ce wrapper est utilisé pour accéder à diverses ressources fournies par PHP lui-même, telles que les flux d'entrée, de sortie et d'erreur.

    `$input_data = file_get_contents('php://input');`

5. **`data://` :** Ce wrapper est utilisé pour accéder aux données en ligne, souvent utilisé pour les données encodées en base64.

`$base64_data = file_get_contents('data://text/plain;base64,' . base64_encode('Hello, world!'));`

### Payload:
`php://filter/convert.base64-encode/resource=/etc/passwd` *used to get through php wrapper the content of /etc/passwd and then to encode it in base64. Consider using %00 as well.*

