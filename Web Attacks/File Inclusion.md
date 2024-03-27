Dividios en Remote y Local dependiendo sobre donde el archivo para incluir esta localizado

# Local File Inclusion

![](file:///C:/Users/fredd/AppData/Local/Temp/msohtmlclip1/02/clip_image001.png)

![](file:///C:/Users/fredd/AppData/Local/Temp/msohtmlclip1/02/clip_image002.png)

![](file:///C:/Users/fredd/AppData/Local/Temp/msohtmlclip1/02/clip_image003.png)

![](file:///C:/Users/fredd/AppData/Local/Temp/msohtmlclip1/02/clip_image004.png)

# Remote File Inclusion

Funciona igual que el local, solo que el file a ser incluido es jalado remotamente. Para que funcione los developer debieron configurar el php.ini lo que lo hace menos probable que LFI. Nuestro objetivo no es solo leer, sino incluir nuestro propio código en ejecución. Un URL explotable podría ser:

![](file:///C:/Users/fredd/AppData/Local/Temp/msohtmlclip1/02/clip_image005.png)

![](file:///C:/Users/fredd/AppData/Local/Temp/msohtmlclip1/02/clip_image006.png)

![](file:///C:/Users/fredd/AppData/Local/Temp/msohtmlclip1/02/clip_image007.png)

![](file:///C:/Users/fredd/AppData/Local/Temp/msohtmlclip1/02/clip_image008.png)

![](file:///C:/Users/fredd/AppData/Local/Temp/msohtmlclip1/02/clip_image009.png)

### Test

![](file:///C:/Users/fredd/AppData/Local/Temp/msohtmlclip1/02/clip_image010.png)

![](file:///C:/Users/fredd/AppData/Local/Temp/msohtmlclip1/02/clip_image011.png)

### Caso práctico

![](file:///C:/Users/fredd/AppData/Local/Temp/msohtmlclip1/02/clip_image012.png)

Tenemos el archivo shell.txt en una web del atacante que muestra el directorio actual del servidor

![](file:///C:/Users/fredd/AppData/Local/Temp/msohtmlclip1/02/clip_image013.png)

La file extesion es txt a pesar de que es un código de PHP. No usamos extensión PHP en RFI porque el código será ejecutado en el lado del atacante mas no en el de la victima

![](file:///C:/Users/fredd/AppData/Local/Temp/msohtmlclip1/02/clip_image014.png)

Si usamos la extensión PHP

![](file:///C:/Users/fredd/AppData/Local/Temp/msohtmlclip1/02/clip_image015.png)