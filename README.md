# Duque
Resolución de la maquina Duque.

Para comenzar, se realizo un escaneo de puertos utilizando Nmap con detección de servicios y versiones, ademas de scripts por defecto:
nmap -p- -sS -sV -Pn --min-rate 5000 172.17.0.2
<img width="766" height="292" alt="imagen" src="https://github.com/user-attachments/assets/fe037f7e-a837-4674-93b6-23f023c2e710" />

El escaneo reveló que el host objetivo se encuentra activo y expone los siguientes servicios:

Puerto 22 (SSH)

Servicio: OpenSSH 8.9p1
Sistema: Ubuntu Linux
