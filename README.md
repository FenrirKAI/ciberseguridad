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
| Diccionario de ataque | `rockyou.txt` / `lista_passwords.txt` |

---

## Estructura del repositorio

```
.
├── README.md
└── Imagenes_Informe_Ciberseguridad/
    ├── 01.crear_directorio_de_trabajo.png
    ├── 02.hash_original_archivo.png
    └── ... (30 imágenes en total)
```

---

## Lab 1 — Integridad y Firma Digital

### Objetivo

Comprobar la integridad de un archivo mediante funciones hash (SHA-256) y garantizar su autenticidad mediante firma digital con criptografía asimétrica (RSA). Se simula la alteración del archivo por un intruso para validar la detección automática del cambio.

### 1. Crear el directorio de trabajo del laboratorio

```bash
mkdir lab1
cd lab1
echo "Transferencia: cuenta 12345 monto 1000" > config_bancaria.txt
cat config_bancaria.txt
```

![Crear directorio de trabajo](Imagenes_Informe_Ciberseguridad/01.crear_directorio_de_trabajo.png)

### 2. Generar la huella digital (hash) original del archivo y guardarla

```bash
sha256sum config_bancaria.txt > hash_original.txt
cat hash_original.txt
```

![Hash original del archivo](Imagenes_Informe_Ciberseguridad/02.hash_original_archivo.png)

### 3. Simular el ataque de un intruso que altera el archivo

```bash
sed -i 's/1000/9999/' config_bancaria.txt
cat config_bancaria.txt
```

![Simular ataque del intruso](Imagenes_Informe_Ciberseguridad/03.simular_ataque_intruso.png)

### 4. Generar un nuevo hash del archivo alterado para comparar visualmente

```bash
sha256sum config_bancaria.txt
```

![Nuevo hash del archivo alterado](Imagenes_Informe_Ciberseguridad/04.nuevo_hash_archivo_alterado.png)

### 5. Verificar de manera automática que el archivo fue modificado (resultado esperado: `FAILED`)

```bash
sha256sum -c hash_original.txt
```

![Verificación automática Failed](Imagenes_Informe_Ciberseguridad/05.verificacion_automatica_failed.png)

### 6. Restaurar el archivo a su contenido original

```bash
echo "Transferencia: cuenta 12345 monto 1000" > config_bancaria.txt
sha256sum -c hash_original.txt
```

![Restaurar archivo](Imagenes_Informe_Ciberseguridad/06.generar_llave_privada_rsa.png)

### 7. Generar la llave privada RSA y la llave pública a partir de la privada

```bash
openssl genrsa -out privada.pem 2048
openssl rsa -in privada.pem -pubout -out publica.pem
```

![Generar llaves RSA](Imagenes_Informe_Ciberseguridad/07.generar_llave_publica_rsa.png)

### 8. Firmar el archivo con la llave privada

```bash
openssl dgst -sha256 -sign privada.pem -out firma.bin config_bancaria.txt
```

![Firmar archivo con llave privada](Imagenes_Informe_Ciberseguridad/08.firmar_archivo_llave_privada.png)

### 9. Verificar la firma del archivo usando la llave pública (resultado esperado: `Verified OK`)

```bash
openssl dgst -sha256 -verify publica.pem -signature firma.bin config_bancaria.txt
```

![Verificar firma OK](Imagenes_Informe_Ciberseguridad/09.verificar_firma_ok.png)

### 10. Volver a alterar el archivo y comprobar que la firma ya no es válida (resultado esperado: `Verification Failure`)

```bash
openssl dgst -sha256 -verify publica.pem -signature firma.bin config_bancaria.txt
```

![Alterar archivo - Verification Failure](Imagenes_Informe_Ciberseguridad/10.alterar_archivo_verification_failure.png)

### 11. Restaurar el archivo

```bash
echo "Transferencia: cuenta 12345 monto 1000" > config_bancaria.txt
```

![Restaurar archivo](Imagenes_Informe_Ciberseguridad/11.restaurar_archivo.png)

---

## Lab 2 — Ataque de SSH y defensa con MFA

### Objetivo

Demostrar la vulnerabilidad del servicio SSH ante un ataque de diccionario sin autenticación multifactor utilizando Hydra, y posteriormente aplicar hardening mediante la integración de Google Authenticator (TOTP) con PAM para exigir un segundo factor de autenticación.

### 1. Crear el usuario víctima

```bash
sudo useradd -m -s /bin/bash victima
sudo passwd victima
sudo apt update
```

![Crear usuario víctima](Imagenes_Informe_Ciberseguridad/12.crear_usuario_victima.png)

### 2. Preparar víctima (Ubuntu): instalar el servidor SSH

```bash
sudo apt install -y openssh-server
```

![Instalar servidor SSH en Ubuntu](Imagenes_Informe_Ciberseguridad/13.instalar_servidor_ssh_ubuntu.png)

### 3. Activar el servidor SSH

![Activar servidor SSH](Imagenes_Informe_Ciberseguridad/14.activar_servidor_ssh.png)

### 4. Verificar el estado del servicio SSH

```bash
sudo systemctl enable --now ssh
sudo systemctl status ssh
```

![Verificar estado del SSH](Imagenes_Informe_Ciberseguridad/15.verificar_estado_ssh.png)

### 5. Verificar conectividad con el servidor desde Kali

```bash
ping -c 3 192.168.56.3
```

![Verificar conectividad con el servidor](Imagenes_Informe_Ciberseguridad/16.verificar_conectividad_servidor.png)

### 6. Crear diccionario de contraseñas (o utilizar `rockyou.txt`)

```bash
cat > lista_passwords.txt << EOF
123456
admin
qwerty
password
password123
letmein
EOF
```

![Diccionario rockyou](Imagenes_Informe_Ciberseguridad/17.diccionario_rockyou.png)

### 7. Ejecutar ataque de diccionario contra SSH — Hydra encuentra la contraseña (debilidad sin MFA)

```bash
hydra -l victima -P lista ssh://192.168.56.3 -t 4 -v
```

**Resultado:** `[22][ssh] host: 192.168.56.3 login: victima password: password123`

![Hydra encuentra la contraseña](Imagenes_Informe_Ciberseguridad/18.hydra_encuentra_contrasena.png)

### 8. Hardening con MFA — Instalar el módulo de Google Authenticator

```bash
sudo apt install -y libpam-google-authenticator
```

![Instalar Google Authenticator](Imagenes_Informe_Ciberseguridad/19.instalar_google_authenticator.png)

### 9. Cambiar al usuario víctima y configurar el segundo factor

```bash
su - victima
google-authenticator
```

Responder `y` a la pregunta sobre tokens basados en tiempo y escanear el código QR con la app autenticadora.

![Configurar segundo factor](Imagenes_Informe_Ciberseguridad/20.configurar_segundo_factor.png)

### 10. Confirmar configuración y guardar los códigos de recuperación TOTP

Responder `y` a las preguntas sobre actualización del archivo `.google_authenticator`, no reutilizar tokens, ventana de tiempo y rate-limiting.

![Códigos de recuperación TOTP](Imagenes_Informe_Ciberseguridad/21.codigos_recuperacion_totp.png)

### 11. Configurar PAM para exigir el código TOTP

```bash
sudo nano /etc/pam.d/sshd
```

Agregar la siguiente línea:

```
auth required pam_google_authenticator.so
```

![Configurar PAM con TOTP](Imagenes_Informe_Ciberseguridad/22.configurar_pam_totp.png)

### 12. Configurar el daemon SSH para autenticación interactiva

```bash
sudo nano /etc/ssh/sshd_config
```

Agregar las siguientes líneas:

```
ChallengeResponseAuthentication yes
KbdInteractiveAuthentication yes
UsePAM yes
AuthenticationMethods keyboard-interactive
```

Reiniciar el servicio SSH:

```bash
sudo systemctl restart ssh
```

![Configurar sshd_config](Imagenes_Informe_Ciberseguridad/23.configurar_sshd_config.png)

### 13. Reintentar el ataque desde Kali con Hydra — el ataque falla (eficiencia del MFA)

```bash
hydra -l victima -P lista ssh://192.168.56.3 -t 4 -v
```

**Resultado:** `1 of 1 target completed, 0 valid password found`

![Hydra falla con MFA](Imagenes_Informe_Ciberseguridad/24.hydra_falla_con_mfa.png)

### 14. Comprobar que un usuario legítimo sí puede ingresar con su segundo factor

```bash
ssh victima@192.168.56.3
```

Ingresar contraseña + código de verificación TOTP generado por la app.

![Login usuario legítimo con MFA](Imagenes_Informe_Ciberseguridad/25.login_usuario_legitimo_mfa.png)

---

## Lab 3 — Sniffing de tráfico y túnel SSH

### Objetivo

Evidenciar el riesgo de transmitir credenciales en HTTP plano mediante un análisis de tráfico con Wireshark, y demostrar cómo un túnel SSH cifrado protege la información sensible que viaja por la red.

### 1. Instalar y activar el servidor Apache

```bash
sudo apt install -y apache2
sudo systemctl enable --now apache2
```

![Instalar Apache](Imagenes_Informe_Ciberseguridad/26.instalar_apache.png)

### 2. Crear el formulario de login en HTTP plano

```bash
sudo nano /var/www/html/login.html
```

Contenido del archivo:

```html
<!DOCTYPE html>
<html><body>
<h2> LOGIN (HTTP - INSEGURO) </h2>
<form method="POST" action="/login.html">
Usuario:<input name="user"><br>
Password:<input type="password" name="pass"><br>
<button type="submit">Entrar</button>
</form>
</body></html>
```

![Formulario de login HTTP plano](Imagenes_Informe_Ciberseguridad/27.formulario_login_http_plano.png)

### 3. Acceder a `login.html` desde Kali y filtrar el método `POST` en Wireshark para visualizar las credenciales

En Wireshark aplicar el filtro:

```
http.request.method == "POST"
```

**Resultado:** las credenciales viajan en texto plano y son visibles (`user = hola`, `pass = 123`).

![Wireshark filtro POST y credenciales](Imagenes_Informe_Ciberseguridad/28.wireshark_filtro_post_credenciales.png)

### 4. Crear un túnel SSH cifrado hacia el servidor web y acceder al formulario por el túnel local

```bash
ssh -L 8080:localhost:80 victima@192.168.56.3
```

Acceder en el navegador a:

```
http://localhost:8080/login.html
```

![Túnel SSH cifrado](Imagenes_Informe_Ciberseguridad/29.tunel_ssh_cifrado.png)

### 5. Verificación de encriptación con Wireshark

Aplicar el filtro:

```
tcp.port == 22
```

**Resultado:** todos los paquetes aparecen como `Encrypted packet`, demostrando que el tráfico está cifrado.

![Verificación de encriptación](Imagenes_Informe_Ciberseguridad/30.verificacion_encriptacion.png)

### Opcional

- Implementar VPN para extender el cifrado a toda la comunicación entre cliente y servidor.

---

## Conclusiones generales

- **Lab 1** demostró que las funciones hash detectan cualquier alteración del archivo, y que la firma digital RSA garantiza tanto la integridad como la autenticidad del origen.
- **Lab 2** evidenció que SSH con sólo contraseña es vulnerable a ataques de diccionario, mientras que la incorporación de MFA con TOTP eleva drásticamente la seguridad del acceso remoto.
- **Lab 3** mostró que el tráfico HTTP plano expone credenciales sensibles y que el uso de túneles SSH (o VPN) es indispensable para proteger la información en tránsito.

---

*Repositorio académico — Asignatura de Ciberseguridad*
