# 🕵️‍♂️ Samba & FTP Enumeration CTF

**Autor:** Córdoba

**Fecha:** 26/02/2026

**Categoría:** Network Enumeration / eJPT

**Objetivo:** 192.17.162.3


## 1. Reconocimiento Inicial y Escaneo de Puertos

El primer paso de la auditoría consistió en verificar la conectividad con la máquina objetivo y realizar un escaneo exhaustivo de puertos para identificar la superficie de ataque.

* **Comprobación de red:** Confirmé la conectividad mediante trazas ICMP.

  *Evidencia:*
  <img width="766" height="192" alt="ping" src="https://github.com/user-attachments/assets/17f445f1-ea28-44fb-9d50-65408e02b678" />

* **Escaneo de Puertos:** Utilicé Nmap buscando en los 65.535 puertos (-p-). Esto fue crucial, ya que reveló los puertos estándar (22 SSH, 139/445 SMB) y descubrió un servicio oculto en un puerto no estándar (5554).

  *Evidencia:*
  <img width="1108" height="302" alt="nmap" src="https://github.com/user-attachments/assets/9f51dc37-8122-4ce1-95da-61c0d39b707a" />

* **Versionado de Servicios:** Para profundizar en el servicio Samba detectado en el puerto 445, empleé el escáner de Metasploit, confirmando la versión exacta y el sistema operativo subyacente (Ubuntu).

  *Evidencia:*
  <img width="1460" height="663" alt="smb_version" src="https://github.com/user-attachments/assets/5d20fc3e-bb62-455c-96e8-f457db80bcad" />

## 2. Enumeración y Explotación de SMB (Flag 2)

Con la confirmación del servicio SMB, procedí a la fase de enumeración de usuarios y recursos compartidos.

* **Enumeración de Usuarios:** Utilicé herramientas automatizadas para extraer la lista de usuarios válidos del sistema (Josh, Bob, Nancy, etc.).

  *Evidencias:*
  <img width="1199" height="503" alt="enum4linux" src="https://github.com/user-attachments/assets/13f1181a-39b2-40f3-b29a-992f3debb784" />
  
  <img width="1186" height="772" alt="smb_enumusers" src="https://github.com/user-attachments/assets/16c1430e-0d77-4fa6-9ea1-d7b5a8940e22" />

* **Ataque de Fuerza Bruta:** Sabiendo que los usuarios existían, lancé un ataque de diccionario contra el servicio SMB usando Metasploit, obteniendo credenciales válidas para el usuario josh.

  *Evidencia:*
  <img width="1101" height="282" alt="smblogin" src="https://github.com/user-attachments/assets/2da03eed-72c2-44ff-8630-2d9e1cff0182" />

* **Acceso a Recurso Oculto:** Al autenticarme con las credenciales de josh, accedí a su directorio personal oculto (que compartía nombre con su usuario). Dentro de este recurso, encontré y extraje el archivo que contenía la segunda bandera y una pista sobre un servicio FTP.

  *Evidencia:*
  <img width="819" height="370" alt="flag2" src="https://github.com/user-attachments/assets/7c840023-df0f-4afe-9365-4b6ea57f363e" />

## 3. Banner Grabbing y Movimiento Lateral (Flag 4)

Aplicando la metodología de reutilización de credenciales, intenté autenticarme en el servicio SSH (puerto 22) utilizando la contraseña descubierta de josh.

* **Hallazgo:** Aunque la conexión fue rechazada por falta de permisos de terminal, el servidor reveló un Message of the Day (MOTD) o Banner configurado por el administrador, el cual contenía directamente la cuarta bandera.

  *Evidencia:*
  <img width="699" height="495" alt="flag4" src="https://github.com/user-attachments/assets/7b73c692-648d-4e1f-b087-9c42aaa85879" />

## 4. Evasión de Restricciones en SMB (Flag 1)

El objetivo requería encontrar un recurso compartido con acceso anónimo. Sin embargo, el servidor contaba con medidas de hardening (fortificación) que bloqueaban la enumeración estándar de carpetas.

* **Fallo de Enumeración Automática:** Herramientas como Metasploit no lograron listar los recursos ocultos debido a restricciones del protocolo.

  *Evidencia:*
  <img width="1480" height="703" alt="enumshares" src="https://github.com/user-attachments/assets/b30ea38a-f758-42a5-8d35-67bb0ee0d859" />

* **Identificación de Diccionarios:** En mi máquina atacante, localicé un directorio con diccionarios de fuerza bruta específicos para este entorno.

  *Evidencia:*
  <img width="421" height="196" alt="carpeta-diccionarios" src="https://github.com/user-attachments/assets/8cf9690b-b0b2-49dc-80fd-7deada90acb5" />

* **Fuerza Bruta de Directorios (Scripting):** Desarrollé un bucle en Bash utilizando el diccionario shares.txt y la herramienta smbclient para probar nombres de carpetas a ciegas. Logré descubrir el recurso oculto anónimo llamado pubshares.

  *Evidencia:*
  <img width="1335" height="596" alt="pubfiles" src="https://github.com/user-attachments/assets/12fab7e2-633e-4d05-ac52-d3f2a31955b6" />

* **Extracción:** Accedí a pubshares sin contraseña y descargué la primera bandera.

  *Evidencia:*
  <img width="943" height="430" alt="flag1" src="https://github.com/user-attachments/assets/88df043b-0289-4826-8d25-8922fb95a82c" />

## 5. Explotación de FTP Oculto (Flag 3)

La pista de la Flag 2 y el escaneo inicial de Nmap apuntaban a un servicio FTP ejecutándose en el puerto no estándar 5554.

* **Banner Grabbing:** Al conectarme al puerto 5554, el banner de bienvenida del FTP reveló inadvertidamente tres nombres de usuario (Ashley, Alice, Amanda) y confirmó que usaban contraseñas débiles.

* **Preparación del Ataque:** Utilicé el segundo diccionario encontrado en mi máquina local para preparar el ataque de fuerza bruta.

  *Evidencia:*
  <img width="1203" height="397" alt="segunda_wordlist_ftp" src="https://github.com/user-attachments/assets/76a7a04b-d8b8-4952-9d36-71790379b34a" />

* **Ejecución de Hydra:** Configuré y lancé la herramienta Hydra contra el puerto 5554 apuntando a las tres usuarias descubiertas.

  *Evidencia:*
  <img width="903" height="272" alt="comando_hydra" src="https://github.com/user-attachments/assets/ffdbb941-10ee-4390-9fbf-75baaf0d7ef7" />

* **Obtención de Credenciales y Acceso:** Hydra logró descifrar exitosamente una de las contraseñas en cuestión de segundos.

  *Evidencia:*
  <img width="854" height="230" alt="solucion_hydra" src="https://github.com/user-attachments/assets/11eff747-b239-429f-88d5-d7d08cea803a" />

* **Extracción Final:** Con las credenciales válidas, inicié sesión en el FTP, listé los archivos y descargué la tercera y última bandera.

  *Evidencia:*
  <img width="1475" height="513" alt="flag3" src="https://github.com/user-attachments/assets/af8bfb85-0e5a-44d8-87d7-7fa9f7aeb00b" />

## ✅ Conclusiones

Se han identificado y explotado con éxito 4 vectores de vulnerabilidad distintos en este host, demostrando graves deficiencias en la configuración de seguridad:

* **Seguridad por Oscuridad y Evasión de Enumeración:** Ocultar recursos SMB (pubshares) no es una medida efectiva si no se restringen los permisos de acceso anónimo. Un simple ataque de diccionario reveló la ruta (Flag 1).
* **Contraseñas Débiles en SMB:** La falta de políticas de complejidad de contraseñas permitió comprometer la cuenta del usuario josh mediante fuerza bruta, comprometiendo sus archivos personales (Flag 2).
* **Information Disclosure (Fuga de Información):** Exposición de datos confidenciales a través de banners de bienvenida en servicios expuestos (SSH revelando la Flag 4 y FTP revelando usuarios válidos).
* **Servicios en Puertos No Estándar y Contraseñas Débiles:** Intentar ocultar un servicio FTP en el puerto 5554 no evitó su descubrimiento. Esto, sumado al uso de contraseñas de diccionario por parte de los usuarios, permitió el compromiso total del servicio (Flag 3).
