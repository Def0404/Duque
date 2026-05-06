# 🐳 DockerLabs Writeup - Duque

## 📌 Resumen

En este laboratorio se compromete una máquina mediante:

* Credenciales por defecto
* Enumeración de parámetros (IDOR)
* Reutilización de credenciales
* Escalada de privilegios por SUID

---

## 🔍 Reconocimiento

Se realizó un escaneo con Nmap:

```bash
nmap -p- -sS -sV -Pn --min-rate 5000 172.17.0.2
```

<img width="766" height="292" alt="nmap" src="https://github.com/user-attachments/assets/fe037f7e-a837-4674-93b6-23f023c2e710" />

**Puertos encontrados:**

* 22 → SSH
* 80 → HTTP

---

## 🌐 Enumeración Web

Se accedió al sitio web:

```
http://172.17.0.2
```

<img width="1328" height="640" alt="dashboard" src="https://github.com/user-attachments/assets/c987c8c0-95d3-4887-9c68-fc46b985f924" />

Se identificaron varias secciones, algunas restringidas (**Empleados, Intranet**), lo que indica control de acceso.

<img width="1328" height="635" alt="restriccion" src="https://github.com/user-attachments/assets/b051a39b-37ef-4b0b-be15-373ba3ff8d18" />

---

## 📂 Descubrimiento de rutas

Se utilizó Gobuster:

```bash
gobuster dir -u http://172.17.0.2/ -w wordlist.txt
```

<img width="1304" height="380" alt="gobuster" src="https://github.com/user-attachments/assets/fbbcac00-d853-4401-818d-8ba222186f0c" />

Ruta interesante encontrada:

* `/bills`

---

## 🔐 Acceso al sistema

Se accedió a `/bills`, donde había un login:

<img width="1322" height="634" alt="login" src="https://github.com/user-attachments/assets/096b4514-006d-4ccd-8184-d60fabee48c0" />

Se probaron credenciales por defecto:

```
admin : admin123
```

<img width="1324" height="636" alt="imagen" src="https://github.com/user-attachments/assets/3f28bcc3-2f5b-465f-8950-97694050b2e0" />

---

## 📄 Parámetro vulnerable

Dentro del panel:

<img width="1323" height="634" alt="imagen" src="https://github.com/user-attachments/assets/744b75b9-700d-4ff8-8f8b-f35c30059fd0" />

Se detectó el parámetro:

```
panel.php?id=xya456
```

Esto indica posible carga dinámica → candidato a IDOR.

---

## 🚀 Fuzzing

Se generó una wordlist basada en el patrón `xy[a-z][0-9]` y se utilizó ffuf:

```bash
ffuf -u 'http://172.17.0.2/bills/panel.php?id=FUZZ' \
-w fuzz.txt \
-H "Cookie: PHPSESSID=..." \
-ac -c
```

<img width="1320" height="631" alt="imagen" src="https://github.com/user-attachments/assets/86627511-8cb6-4eaa-9ad7-1718de731e97" />

Se identificaron múltiples IDs válidos.

---

## ⭐ Hallazgo clave (IDOR)

El ID más interesante fue:

```
xyc724
```

<img width="1322" height="637" alt="imagen" src="https://github.com/user-attachments/assets/b8958e1c-a795-439b-ba0a-dafb09605aa5" />

Reveló credenciales en texto plano:

```
duque : duquelaje81029557!
```

---

## 🔐 Acceso por SSH

```bash
ssh duque@172.17.0.2
```

<img width="1318" height="633" alt="imagen" src="https://github.com/user-attachments/assets/20663769-6ad3-4b24-96d5-3f08e3082ffb" />

Acceso exitoso → usuario `duque`.

---

## ⬆️ Escalada de privilegios

Se listaron binarios SUID:

```bash id="7yq3nt"
find / -perm -4000 2>/dev/null
```

<img width="426" height="238" alt="imagen" src="https://github.com/user-attachments/assets/ea82ee76-af62-44ac-8f35-8b65cd20517b" />

Se identificó `/usr/bin/env` como vector potencial de explotación.

---

### ⚔️ Explotación

El binario `env` permite ejecutar programas en un entorno modificado.
Cuando tiene el bit **SUID**, puede ejecutarse con privilegios elevados.

Investigando en **GTFOBins**, se encontró que es posible abusar de este binario para invocar una shell manteniendo privilegios:

```bash id="2f2y1g"
env /bin/sh -p
```

### 🔍 ¿Qué hace este comando?

* `env` → ejecuta un programa dentro de su entorno
* `/bin/sh` → lanza una shell
* `-p` → **preserva los privilegios efectivos (EUID)**

👉 Esto es clave, ya que evita que la shell reduzca privilegios al usuario actual.

---

### ✅ Resultado

<img width="1321" height="594" alt="imagen" src="https://github.com/user-attachments/assets/eccfeefe-b0ce-4418-a824-847ea7d73399" />

<img width="315" height="126" alt="imagen" src="https://github.com/user-attachments/assets/2415be45-8bb1-4284-a685-ff8655c20c8c" />

Al ejecutar el comando, se obtuvo una shell con privilegios de **root**, confirmando la escalada exitosa.

---

### 🧠 Análisis

* ⚠️ El binario `/usr/bin/env` con SUID representa un riesgo crítico si no está correctamente restringido.
* 🔓 Permite ejecutar comandos arbitrarios con privilegios elevados.
* 🧪 Este tipo de vectores es común en entornos mal configurados.

---


---

# 🧠 Conclusiones y aprendizaje

Este laboratorio demuestra varias fallas comunes:

* Uso de **credenciales por defecto**
* Vulnerabilidad **IDOR**
* Exposición de credenciales en texto plano
* Reutilización de credenciales entre servicios
* Binarios SUID mal configurados

---

# 🔐 Recomendaciones

* ❌ Evitar credenciales por defecto en producción
* 🔒 Implementar controles de autorización (evitar IDOR)
* 🔑 No almacenar credenciales en texto plano
* 🔁 No reutilizar credenciales entre servicios
* ⚙️ Auditar binarios SUID innecesarios
* 🧪 Validar correctamente parámetros de entrada

---

🚀 Con esto se logra el compromiso total del sistema.
