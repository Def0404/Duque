# Duque
Resolución de la maquina Duque.

Para comenzar, se realizo un escaneo de puertos utilizando Nmap con detección de servicios y versiones, ademas de scripts por defecto:
nmap -p- -sS -sV -Pn --min-rate 5000 172.17.0.2
<img width="766" height="292" alt="imagen" src="https://github.com/user-attachments/assets/fe037f7e-a837-4674-93b6-23f023c2e710" />

El escaneo reveló que el host objetivo se encuentra activo y expone los siguientes servicios:

Puerto 22 (SSH)
Puerto 80 (HTTP)

Enumeracion Web

Tras identificar el servicio HTTP en el puerto 80, se accedió a la aplicación web desde el navegador:

http://172.17.0.2

Interfaz Inicial

La página principal corresponde a un panel llamado NaturGas Solutions, el cual muestra información general de la empresa, como estadísticas y navegación interna.

<img width="1328" height="640" alt="2026-05-04_22-29-24" src="https://github.com/user-attachments/assets/c987c8c0-95d3-4887-9c68-fc46b985f924" />

Se observan las siguientes secciones en el menú principal:

Inicio
Empleados
Normativa
Intranet
Proveedores

Restricciones de accesso

Al intentar acceder a algunas secciones, como Empleados e Intranet, se observó que estas se encuentran restringidas, lo que indica la presencia de algun mecanismo de autenticación o control de acceso.

<img width="1328" height="635" alt="imagen" src="https://github.com/user-attachments/assets/b051a39b-37ef-4b0b-be15-373ba3ff8d18" />

Dado esto, el siguiente paso lógico es realizar enumeración de directorios y archivos ocultos, así como analizar posibles parámetros en la aplicación.
