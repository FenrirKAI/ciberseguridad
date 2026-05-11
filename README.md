# Laboratorios de Ciberseguridad

**Autor:** Iker Alessandro Navarro Deumacan  
**Carrera:** Ingeniería en Informática  
**Asignatura:** Ciberseguridad  

---

## Descripción

Este repositorio documenta los laboratorios prácticos realizados durante la asignatura de Ciberseguridad. Cada laboratorio aborda un escenario distinto de ataque y defensa, mostrando paso a paso los comandos ejecutados, los resultados obtenidos y las medidas de mitigación aplicadas.

El propósito del repositorio es servir como evidencia técnica y guía de referencia sobre los principales mecanismos de seguridad estudiados: integridad de archivos, autenticación robusta y cifrado de tráfico de red.

---

## Entorno de trabajo

| Componente | Detalle |
|---|---|
| Virtualización | VirtualBox (red interna `192.168.56.0/24`) |
| Máquina víctima | Ubuntu Server (`192.168.56.3`) |
| Máquina atacante | Kali Linux |
| Herramientas utilizadas | OpenSSL, SHA-256, Hydra, Google Authenticator, PAM, OpenSSH, Apache2, Wireshark |
| Diccionario de ataque | `rockyou.txt` |

---

## Estructura del repositorio

```
.
├── README.md
└── imagenes/
    ├── 01.crear_directorio_de_trabajo.png
    ├── 02.hash_original_archivo.png
    └── ... (30 imágenes en total)
```

---

## Lab 1 — Integridad y Firma Digital

### Objetivo

Comprobar la integridad de un archivo mediante funciones hash (SHA-256) y garantizar su autenticidad mediante firma digital con criptografía asimétrica (RSA). Se simula la alteración del archivo por un intruso para validar la detección automática del cambio.

### 1. Crear el directorio de trabajo del laboratorio

![Crear directorio de trabajo](imagenes/01.crear_directorio_de_trabajo.png)

### 2. Generar huella digital (hash) original, simular ataque del intruso y generar nuevo hash

![Hash original del archivo](imagenes/02.hash_original_archivo.png)
![Simular ataque del intruso](imagenes/03.simular_ataque_intruso.png)
![Nuevo hash del archivo alterado](imagenes/04.nuevo_hash_archivo_alterado.png)

### 3. Verificar de manera automática que el archivo fue modificado (resultado esperado: `Failed`)

![Verificación automática Failed](imagenes/05.verificacion_automatica_failed.png)

### 4. Generar llave privada y pública RSA, firmar archivo y verificar la firma

![Generar llave privada RSA](imagenes/06.generar_llave_privada_rsa.png)
![Generar llave pública RSA](imagenes/07.generar_llave_publica_rsa.png)
![Firmar archivo con llave privada](imagenes/08.firmar_archivo_llave_privada.png)
![Verificar firma OK](imagenes/09.verificar_firma_ok.png)

### 5. Volver a alterar el archivo (firma inválida) y restaurar el archivo

![Alterar archivo - Verification Failure](imagenes/10.alterar_archivo_verification_failure.png)
![Restaurar archivo](imagenes/11.restaurar_archivo.png)

---

## Lab 2 — Ataque de SSH y defensa con MFA

### Objetivo

Demostrar la vulnerabilidad del servicio SSH ante un ataque de diccionario sin autenticación multifactor utilizando Hydra, y posteriormente aplicar hardening mediante la integración de Google Authenticator (TOTP) con PAM para exigir un segundo factor de autenticación.

### 1. Crear el usuario víctima

![Crear usuario víctima](imagenes/12.crear_usuario_victima.png)

### 2. Preparar víctima (Ubuntu): instalar y activar el servidor SSH

![Instalar servidor SSH en Ubuntu](imagenes/13.instalar_servidor_ssh_ubuntu.png)
![Activar servidor SSH](imagenes/14.activar_servidor_ssh.png)
![Verificar estado del SSH](imagenes/15.verificar_estado_ssh.png)

### 3. Verificar conectividad con el servidor desde Kali

![Verificar conectividad con el servidor](imagenes/16.verificar_conectividad_servidor.png)

### 4. Crear diccionario de contraseñas o utilizar `rockyou.txt`

![Diccionario rockyou](imagenes/17.diccionario_rockyou.png)

### 5. Ejecutar ataque de diccionario contra SSH — Hydra encuentra la contraseña (debilidad sin MFA)

![Hydra encuentra la contraseña](imagenes/18.hydra_encuentra_contrasena.png)

### 6. Hardening con MFA — Instalar el módulo de Google Authenticator

![Instalar Google Authenticator](imagenes/19.instalar_google_authenticator.png)

### 7. Cambiar al usuario víctima y configurar el segundo factor

![Configurar segundo factor](imagenes/20.configurar_segundo_factor.png)
![Códigos de recuperación TOTP](imagenes/21.codigos_recuperacion_totp.png)

### 8. Configurar PAM para exigir el código TOTP

![Configurar PAM con TOTP](imagenes/22.configurar_pam_totp.png)

### 9. Configurar el daemon SSH (`sshd_config`) y reiniciar el servicio

```
ChallengeResponseAuthentication yes
KbdInteractiveAuthentication yes
UsePAM yes
AuthenticationMethods keyboard-interactive
```

![Configurar sshd_config](imagenes/23.configurar_sshd_config.png)

### 10. Reintentar el ataque desde Kali con Hydra — el ataque falla (eficiencia del MFA)

![Hydra falla con MFA](imagenes/24.hydra_falla_con_mfa.png)

### 11. Comprobar que un usuario legítimo sí puede ingresar con su segundo factor

![Login usuario legítimo con MFA](imagenes/25.login_usuario_legitimo_mfa.png)

---

## Lab 3 — Sniffing de tráfico y túnel SSH

### Objetivo

Evidenciar el riesgo de transmitir credenciales en HTTP plano mediante un análisis de tráfico con Wireshark, y demostrar cómo un túnel SSH cifrado protege la información sensible que viaja por la red.

### 1. Instalar y activar el servidor Apache

![Instalar Apache](imagenes/26.instalar_apache.png)

### 2. Crear el formulario de login en HTTP plano

![Formulario de login HTTP plano](imagenes/27.formulario_login_http_plano.png)

### 3. Acceder a `login.html` desde Kali y filtrar el método `POST` en Wireshark para visualizar las credenciales

![Wireshark filtro POST y credenciales](imagenes/28.wireshark_filtro_post_credenciales.png)

### 4. Crear un túnel SSH cifrado hacia el servidor web y acceder al formulario por el túnel local

```bash
ssh -L 8080:localhost:80 victima@192.168.56.3
```

![Túnel SSH cifrado](imagenes/29.tunel_ssh_cifrado.png)

### 5. Verificación de encriptación

![Verificación de encriptación](imagenes/30.verificacion_encriptacion.png)

### Opcional

- Implementar VPN para extender el cifrado a toda la comunicación entre cliente y servidor.

---

## Conclusiones generales

- **Lab 1** demostró que las funciones hash detectan cualquier alteración del archivo, y que la firma digital RSA garantiza tanto la integridad como la autenticidad del origen.
- **Lab 2** evidenció que SSH con sólo contraseña es vulnerable a ataques de diccionario, mientras que la incorporación de MFA con TOTP eleva drásticamente la seguridad del acceso remoto.
- **Lab 3** mostró que el tráfico HTTP plano expone credenciales sensibles y que el uso de túneles SSH (o VPN) es indispensable para proteger la información en tránsito.

---

*Repositorio académico — Asignatura de Ciberseguridad*
| Herramientas utilizadas | OpenSSL, SHA-256, Hydra, Google Authenticator, PAM, OpenSSH, Apache2, Wireshark |
| Diccionario de ataque | `rockyou.txt` |

---

## Estr
