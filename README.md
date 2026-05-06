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
<img width="1323" height="634" alt="imagen" src="https://github.com/user-attachments/assets/744b75b9-700d-4ff8-8f8b-f35c30059fd0" />

### 🧠 Análisis

* ⚠️ El uso de credenciales por defecto representa una **grave vulnerabilidad de seguridad**.
* 🔓 Esto permitió el acceso directo al sistema de facturación sin necesidad de realizar ataques más complejos.
* 🧪 Este tipo de fallo es común en entornos mal configurados o en aplicaciones en desarrollo.

### 🚨 Impacto

* Acceso no autorizado a información sensible
* Posible manipulación de datos internos
* Escalada hacia otras vulnerabilidades dentro del sistema

---

Una vez autenticado, se obtuvo acceso al panel interno, lo que permitió continuar con la enumeración y análisis de nuevas funcionalidades.
