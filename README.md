# 🐳 DockerLabs Writeup - Duque

## 📌 Descripción

Resolución de la máquina **Duque** de DockerLabs.

---

## 🔍 Reconocimiento

Para comenzar, se realizó un escaneo de puertos utilizando **Nmap**, con detección de servicios, versiones y ejecución de scripts por defecto:

```bash
nmap -p- -sS -sV -Pn --min-rate 5000 172.17.0.2
```

<img width="766" height="292" alt="nmap" src="https://github.com/user-attachments/assets/fe037f7e-a837-4674-93b6-23f023c2e710" />

### 📊 Resultados

El escaneo reveló que el host objetivo se encuentra activo y expone los siguientes servicios:
* **Puerto 22 (SSH)** → OpenSSH (Ubuntu)
* **Puerto 80 (HTTP)** → Apache (Ubuntu)

Dado que el puerto 80 expone una aplicación web, se decidió enfocar el análisis en este servicio.

---

## 🌐 Enumeración Web

Se accedió a la aplicación desde el navegador:

```
http://172.17.0.2
```

### 🖥️ Interfaz inicial

La página principal corresponde a un dashboard corporativo llamado **NaturGas Solutions**, el cual muestra estadísticas e información general.

<img width="1328" height="640" alt="dashboard" src="https://github.com/user-attachments/assets/c987c8c0-95d3-4887-9c68-fc46b985f924" />

Se identificaron las siguientes secciones:

* Inicio
* Empleados
* Normativa
* Intranet
* Proveedores

---

## 🔐 Restricciones de acceso

Al intentar acceder a secciones como **Empleados** e **Intranet**, se observó que están restringidas.

<img width="1328" height="635" alt="restriccion" src="https://github.com/user-attachments/assets/b051a39b-37ef-4b0b-be15-373ba3ff8d18" />


Esto indica la presencia de mecanismos de autenticación o control de acceso, lo que sugiere que existen áreas internas protegidas.

---

## 📂 Enumeración de Directorios

Con el objetivo de descubrir rutas ocultas, se utilizó **Gobuster**:

```bash
gobuster dir -u http://172.17.0.2/ \
-w /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt \
-x html,txt,zip,php,bak,py,png,tar,db,sh,jpg,jpeg
```

<img width="1304" height="380" alt="gobuster" src="https://github.com/user-attachments/assets/fbbcac00-d853-4401-818d-8ba222186f0c" />

### 📊 Resultados

Se identificaron las siguientes rutas relevantes:

* `/index.php`
* `/intranet`
* `/bills`

La ruta `/bills` resulta especialmente interesante, ya que podría estar relacionada con facturación o gestión interna, lo cual suele implicar información sensible.

---

## 💰 Acceso a `/bills`

Se accedió directamente al endpoint:

```
http://172.17.0.2/bills/
```

### 🖥️ Interfaz encontrada

Se presentó un panel de autenticación correspondiente al sistema de facturación de **NaturGas Solutions**.

<img width="1322" height="634" alt="login" src="https://github.com/user-attachments/assets/096b4514-006d-4ccd-8184-d60fabee48c0" />

El formulario solicita:

* Usuario
* Contraseña

---

## 🧠 Análisis

* 🔐 La ruta `/bills` está protegida mediante autenticación.
* 💰 Al tratarse de un sistema de facturación, es probable que maneje información sensible.
* ⚠️ Este tipo de paneles puede ser vulnerable a:

  * Fuerza bruta
  * Inyección SQL
  * Fallos de autorización

Por lo tanto, el siguiente paso será buscar formas de interactuar con este sistema sin autenticación o identificar posibles vulnerabilidades en sus endpoints.

---
## 🔓 Acceso mediante credenciales por defecto

Tras analizar el panel de autenticación en la ruta `/bills`, se procedió a probar credenciales comunes o por defecto.

Se logró acceder exitosamente utilizando las siguientes credenciales:

```id="k2d9sl"
Usuario: admin
Contraseña: admin123
```
<img width="1324" height="636" alt="imagen" src="https://github.com/user-attachments/assets/3f28bcc3-2f5b-465f-8950-97694050b2e0" />

### 🧠 Análisis

* ⚠️ El uso de credenciales por defecto representa una mala práctica de seguridad.
* 🔓 Esto permitió el acceso directo al sistema de facturación sin necesidad de técnicas más complejas.

---

## 📄 Panel interno y análisis de parámetros

Una vez autenticado, se obtuvo acceso al panel interno del sistema de facturación.

<img width="1323" height="634" alt="imagen" src="https://github.com/user-attachments/assets/744b75b9-700d-4ff8-8f8b-f35c30059fd0" />

Dentro de la aplicación se encontraron dos documentos disponibles:

* 📄 **Factura Consumo (Enero)**
* 📄 **Factura Instalación (Febrero)**

Al interactuar con estos elementos, se observó que la aplicación utiliza un parámetro en la URL para gestionar el contenido mostrado:

```id="a9xw2k"
http://172.17.0.2/bills/panel.php?id=xya456
```

### 🧠 Análisis

* 🔎 El uso del parámetro `id` sugiere que la aplicación carga información de forma dinámica.
* ⚠️ Este tipo de implementación puede ser vulnerable si no existe una validación adecuada.
* 🧪 Esto abre la posibilidad de realizar pruebas de enumeración o manipulación del parámetro para acceder a otros recursos.

---

A partir de este punto, se procedió a analizar y fuzzear el parámetro `id` en busca de posibles vulnerabilidades.

## 🧪 Generación de wordlist personalizada

Dado que el parámetro `id` seguía un patrón específico (por ejemplo: `xya456`), se optó por generar una **wordlist personalizada** con posibles combinaciones para realizar fuzzing.

Para ello, se creó un diccionario basado en:

* Prefijo fijo: `xy`
* Variación de caracteres (letras)
* Rango numérico (0–1000)

Ejemplo de valores generados:

```id="p9d3ka"
xya0
xya1
xya2
...
xyb0
xyb1
...
xyz1000
```

Este enfoque permitió adaptar el fuzzing al comportamiento observado en la aplicación, incrementando las probabilidades de encontrar identificadores válidos.

### 🧠 Análisis

* 🔎 La creación de wordlists específicas es más efectiva que el uso de listas genéricas.
* 🧪 Permite atacar directamente patrones identificados durante la enumeración.
* ⚡ Mejora significativamente la eficiencia del fuzzing.

---

Posteriormente, esta wordlist fue utilizada para realizar pruebas sobre el parámetro `id`.

## 🚀 Fuzzing del parámetro `id`

Una vez generada la wordlist personalizada, se procedió a realizar fuzzing sobre el parámetro `id` utilizando la herramienta **ffuf**.

```bash id="v3m8zk"
ffuf -u 'http://172.17.0.2/bills/panel.php?id=FUZZ' \
-w fuzz.txt \
-H "Cookie: PHPSESSID=42k3613g85ibvdee298cm8vea3" \
-ac -c
```

### 📊 Resultados

Durante el fuzzing se identificaron múltiples valores válidos del parámetro `id`, entre ellos:

<img width="1320" height="631" alt="imagen" src="https://github.com/user-attachments/assets/86627511-8cb6-4eaa-9ad7-1718de731e97" />


```id="idsvalidos"
xya123
xya456
xya789
xyb234
xyb567
xyb890
xyc123
xyc456
xyc724
xyd234
xyd567
xyd890
xye123
xye456
xye789
xyf234
xyf567
xyf890
xyg123
xyg456
xyu597
```

### 🧠 Análisis

* 🔎 Todos los valores válidos retornaban código **200 OK**, por lo que fue necesario identificar diferencias en el contenido.
* ⚠️ Se observó que la mayoría de respuestas tenían el mismo tamaño (**6163 bytes**), excepto un caso particular.

### ⭐ Hallazgo relevante

El ID más interesante fue:

```id="hallazgo"
xyc724
```

Este valor devolvía una respuesta con un tamaño distinto (**5994 bytes**), lo que indicaba contenido diferente.

Al acceder a dicho recurso, se encontraron credenciales en texto plano:

<img width="1322" height="637" alt="imagen" src="https://github.com/user-attachments/assets/b8958e1c-a795-439b-ba0a-dafb09605aa5" />

```id="credenciales"
Usuario: duque
Password: duquelaje81029557!
```

---

Este hallazgo confirma la existencia de una vulnerabilidad de tipo **IDOR (Insecure Direct Object Reference)**, ya que fue posible acceder a información sensible manipulando directamente el parámetro `id`.

## 🔐 Acceso por SSH

Tras obtener las credenciales desde el parámetro vulnerable, se procedió a probarlas en el servicio SSH identificado durante la fase de reconocimiento.

```bash id="sshcmd"
ssh duque@172.17.0.2
```

Se utilizó la siguiente contraseña:

```id="sshpass"
duquelaje81029557!
```

### ✅ Resultado

<img width="1318" height="633" alt="imagen" src="https://github.com/user-attachments/assets/20663769-6ad3-4b24-96d5-3f08e3082ffb" />

El acceso fue exitoso, logrando autenticarse como el usuario:

```id="user"
duque
```

### 🧠 Análisis

* 🔓 Las credenciales obtenidas desde la aplicación web eran válidas también para el servicio SSH.
* ⚠️ Esto evidencia una reutilización de credenciales entre servicios, lo cual representa una mala práctica de seguridad.
* 🔗 Se completa así la cadena de ataque:

  * Enumeración web
  * Acceso con credenciales por defecto
  * Identificación de parámetro vulnerable
  * Exposición de credenciales
  * Acceso al sistema vía SSH

---

Con esto, se logra comprometer completamente el sistema objetivo.
