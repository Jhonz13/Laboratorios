# Escaneo y explotación rápida 🛠️🔍

## Resumen ejecutivo 🏁
- Objetivo: Reconocimiento y acceso inicial a la máquina 172.17.0.2  
- Sistema: Linux (reconocimiento)
- Puertos abiertos relevantes: 21 (FTP), 80 (HTTP)
- Resultado: No hay contenido web interesante → se opta por atacar FTP / backdoor con Netcat

---

## 1) Reconocimiento 🔎
Comprobación inicial del SO y red.

- Detección: Linux

---

## 2) Escaneo de puertos (Nmap) 🧭
Comando usado:
```bash
sudo nmap -p- --open -sS -sV --min-rate 2000 -n -Pn 172.17.0.2
```

Resultado relevante:
- 21/tcp  — FTP abierto — vsftpd 2.3.4
- 80/tcp  — HTTP abierto — Apache httpd 2.4.58 ((Ubuntu))

---

## 3) Enumeración web 🌐
Acceso vía navegador o curl:
- URL: http://172.17.0.2

Resultado:
- Página web sin contenido relevante o información explotable.

Decisión:
- Como no hay vectores útiles en la web, se procede a explorar FTP / conexiones con Netcat.

---

## 4) Acceso con Netcat (backdoor / FTP) 🛰️
Primera conexión al puerto FTP para enumeración básica:
```bash
nc -nv 172.17.0.2 21
```
- Comando abierto en modo verbose para ver banners y respuestas.

Apertura de una "backdoor" mediante Netcat en puerto 6200 (según procedimiento descrito):
```bash
nc -nv 172.17.0.2 6200
```
- En este ejemplo, se indica que se escribió "User cualquiera:)" para provocar la apertura de una shell/puerta en el puerto 6200.

Significado de flags de netcat:
- -n : no realiza resolución DNS (usa IPs numéricas) 🧭
- -v : verbose (modo detallado) 📣
- -N / otros flags no usados aquí, revisar versión de netcat si se necesita cerrar conexión o mantener modo escucha

Nota:
- Verifica siempre si la máquina responde a la segunda conexión y si el puerto 6200 realmente quedó abierto como backdoor. Ajusta el puerto según la salida recibida.

---

## 5) Resumen final ✅
1. Reconocimiento → Linux  
2. Nmap → Puertos abiertos: 21 (FTP) y 80 (HTTP)  
3. Enumeración web → Sin contenido útil  
4. Netcat → Conexión al FTP y prueba de backdoor en puerto 6200  
5. Acción recomendada siguiente → Enumerar FTP (usuarios, listados, archivos); si se consigue credencial o archivo con info sensible, pivotear desde ahí.

---

## Notas y recomendaciones 📝
- Registrar banners y respuestas completas de Nmap y Netcat para auditoría.  
- Evitar acciones destructivas sin autorización.  
- Si necesitas, puedo preparar un playbook con los comandos para enumerar FTP
