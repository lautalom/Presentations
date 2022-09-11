# Herramientas Necesarias

- [Burpsuite Community Edition](https://portswigger.net/burp/communitydownload)
- Damn Vulnerable Web App: [DVWA](https://github.com/digininja/DVWA)
- [Hydra](https://github.com/vanhauser-thc/thc-hydra)
- [WFUZZ](https://github.com/xmendez/wfuzz)

# Burpsuite: Docker
- Ver la direccion IP del container.

## Proxy (opcion 1)
Para la demo usé FoxyProxy. Hay más opciones, usen la alternativa de su gusto.

### Docker
- Agregar proxy en la IP del container y puerto 8080  
- Activar el proxy
### Instalado Local
- Agregar un proxy en localhost (127.0.0.1) y puerto 8080
- Activar el proxy

## Navegador de Burp (opcion 2)
- Sin proxy se puede usar el navegador interno de Burp para ir a la dirección IP del server.

# En Burpsuite
- Añadir en Target la dirección IP en Scope
- Setear Proxy Listener en la IP:8080
- Interceptar Requests y Responses: tildar "URL IN SCOPE"
- HTTP History: filtrar por "in scope".
- Setear Intercept ON/OFF según necesidad.

# Levantar la app

Segun el metodo elegido. (docker run o localhost)

# Ataque.
- Interceptar una GET request -> Send to Intruder.

## Intruder
- Positions: Clear Login y User, Add Password.
- Clonar SecList (google Known Passwords).
- Payloads: Cualquier lista que incluya password.
- Options: Grep "Username and/or password incorrect"

# Hydra

- Leer el manual (`man hydra`) y `hydra -h`. Ver flags. Buscar casos de uso.
- Atacar con algún conjunto de passwords.
- Comando `hydra -I -L <FILE> -P <FILE> '<METHOD>://<IP>/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:F=incorrect:H=Cookie: PHPSESSID=<YOUR_SESSION_ID>; security=low'`
- Remover escapes antes de los caracteres especiales!

# WFuzz

- Leer el manual (`man WFUZZ`). Ver Flags. Ver casos de uso.
- unico payload: `wfuzz -c -w <PATH_TO_PASSWORDS> -b 'security=low; PHPSESSID=<YOUR_SESSID>' "<IP>/vulnerabilities/brute/?username=admin&password=FUZZ&Login=Login" `. Pro tip: filtrar salidas --hw 250, --hs "incorrect"
- varios payloads: `wfuzz --hs 'incorrect' -c -w <USERS> -w <PWDS> -b 'security=<SECLEVEL>; PHPSESSID=<SESSID>' "http://172.17.0.2/vulnerabilities/brute/?username=FUZZ&password=FUZ2Z&Login=Login"`


# Sec Level: High

- Notar el CSRF token. Los ataques previos no funcionan.
- Tomamos la última respuesta del proxy history de Burp. La enviamos al intruder.
- En intruder, seleccionamos pitchfork attack sobre pswd y token. Clear al resto.
- En el campo de password, lista simple como antes.
- En el de token, setear Payload Type: Recursive Grep
- Options > Grep Match > "incorrect"
- Options > Grep Extract > user_token. Tildar start at offset y fixed length.
- Options > Redirections > Always follow.

