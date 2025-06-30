# Script para Automatizar la Creación de Usuarios en Linux

## 📋 Descripción

Este script de Bash automatiza completamente el proceso de alta (onboarding) de nuevos usuarios en sistemas Linux. Incluye la creación del usuario, grupos, estructura de directorios estándar y generación de contraseña temporal segura.

## ✨ Características

- **Creación automática de usuario** con grupo primario homónimo
- **Gestión de grupos secundarios** (crea el grupo si no existe)
- **Estructura de directorios estándar**: Documentos, Proyectos, Privado
- **Permisos de seguridad** configurados automáticamente
- **Contraseña temporal segura** generada aleatoriamente
- **Archivo de bienvenida** personalizado para cada usuario
- **Interfaz interactiva** con mensajes coloreados
- **Validaciones completas** para evitar errores

## 🛠️ Instalación y Configuración

### Paso 1: Crear el Archivo del Script

```bash
nano crear_usuario.sh
```

Copia y pega el siguiente código completo:

```bash
#!/bin/bash

# ==============================================================================
# Script para Onboarding Automatizado de Usuarios en Linux
#
# Autor: Rafael Chuco
# Fecha: 28/06/2024
#
# Descripción:
# Este script automatiza la creación de un nuevo usuario, su grupo primario,
# la asignación a un grupo secundario, una estructura de directorios estándar
# y la generación de una contraseña inicial.
#
# Temas aplicados:
# - Semana 11: useradd, groupadd, usermod, chpasswd, chage
# - Semana 10: mkdir, chown, chmod
# - Semana 15: Lógica de script, variables, condicionales, funciones
# ==============================================================================

# --- Función para mostrar mensajes de color para mejor legibilidad ---
mostrar_info() {
    echo -e "\e[34m[INFO]\e[0m $1"
}

mostrar_ok() {
    echo -e "\e[32m[OK]\e[0m $1"
}

mostrar_error() {
    echo -e "\e[31m[ERROR]\e[0m $1"
}

# --- 1. Verificación de privilegios de superusuario (root) ---
if [[ $(id -u) -ne 0 ]]; then
   mostrar_error "Este script debe ser ejecutado como root o con sudo."
   exit 1
fi

# --- 2. Solicitar información del nuevo usuario ---
mostrar_info "Iniciando el proceso de creación de nuevo usuario."
read -p "Introduce el nombre de usuario (ej: jlopez): " USERNAME
read -p "Introduce el nombre completo (ej: Juan Lopez): " FULLNAME
read -p "Introduce el grupo secundario (ej: desarrolladores, ventas): " SECONDARY_GROUP

# Validar campos
if [[ -z "$USERNAME" ]] || [[ -z "$FULLNAME" ]] || [[ -z "$SECONDARY_GROUP" ]]; then
    mostrar_error "Todos los campos son obligatorios. Abortando."
    exit 1
fi

# --- 3. Creación de Usuario y Grupos ---
mostrar_info "Creando usuario y grupos..."

# Verificar si el grupo secundario existe
if ! getent group "$SECONDARY_GROUP" > /dev/null; then
    mostrar_info "El grupo secundario '$SECONDARY_GROUP' no existe. Creándolo..."
    groupadd "$SECONDARY_GROUP"
    if [[ $? -eq 0 ]]; then
        mostrar_ok "Grupo secundario '$SECONDARY_GROUP' creado con éxito."
    else
        mostrar_error "No se pudo crear el grupo '$SECONDARY_GROUP'."
        exit 1
    fi
fi

# Verificar si el usuario ya existe
if id "$USERNAME" &>/dev/null; then
    mostrar_error "El usuario '$USERNAME' ya existe. Abortando."
    exit 1
fi

# Verificar si el grupo primario existe
if ! getent group "$USERNAME" > /dev/null; then
    mostrar_info "El grupo primario '$USERNAME' no existe. Creándolo..."
    groupadd "$USERNAME"
    if [[ $? -eq 0 ]]; then
        mostrar_ok "Grupo primario '$USERNAME' creado con éxito."
    else
        mostrar_error "No se pudo crear el grupo primario '$USERNAME'."
        exit 1
    fi
fi

# Crear usuario
useradd -m -c "$FULLNAME" -g "$USERNAME" -G "$SECONDARY_GROUP" "$USERNAME"
if [[ $? -eq 0 ]]; then
    mostrar_ok "Usuario '$USERNAME' creado y añadido al grupo '$SECONDARY_GROUP'."
else
    mostrar_error "No se pudo crear el usuario '$USERNAME'."
    exit 1
fi

# --- 4. Crear estructura de directorios ---
mostrar_info "Creando estructura de directorios en /home/$USERNAME..."
HOME_DIR="/home/$USERNAME"
DIRECTORIES=("Documentos" "Proyectos" "Privado")

for DIR in "${DIRECTORIES[@]}"; do
    mkdir -p "$HOME_DIR/$DIR"
    chown -R "$USERNAME":"$USERNAME" "$HOME_DIR/$DIR"
done

chmod 700 "$HOME_DIR/Privado"
mostrar_ok "Estructura de directorios creada y permisos asignados."

# --- 5. Generar y asignar contraseña ---
mostrar_info "Generando contraseña temporal segura..."
PASSWORD=$(openssl rand -base64 12)

echo "$USERNAME:$PASSWORD" | chpasswd
if [[ $? -eq 0 ]]; then
    mostrar_ok "Contraseña temporal asignada con éxito."
else
    mostrar_error "No se pudo asignar la contraseña."
    exit 1
fi

# Forzar cambio de contraseña
chage -d 0 "$USERNAME"

# --- 6. Mensaje de bienvenida ---
WELCOME_FILE="$HOME_DIR/bienvenido.txt"
mostrar_info "Creando archivo de bienvenida en $WELCOME_FILE"
cat > "$WELCOME_FILE" << EOL
¡Bienvenido/a al sistema, $FULLNAME!

Tu cuenta ha sido creada con éxito.

Aquí están tus credenciales iniciales:
Nombre de Usuario: $USERNAME
Contraseña Temporal: $PASSWORD

IMPORTANTE: Por seguridad, se te pedirá que cambies esta contraseña
en tu primer inicio de sesión.

Tu estructura de directorios:
~/Documentos: Archivos generales.
~/Proyectos: Proyectos de trabajo.
~/Privado: Acceso solo para ti.

¡Que tengas un gran día!
EOL

chown "$USERNAME":"$USERNAME" "$WELCOME_FILE"
mostrar_ok "Archivo de bienvenida creado."

# --- 7. Resumen final ---
echo ""
mostrar_ok "¡PROCESO COMPLETADO!"
echo "--------------------------------------------------"
echo " Resumen de la cuenta creada:"
echo "   Nombre de Usuario: $USERNAME"
echo "   Nombre Completo:   $FULLNAME"
echo "   Directorio Home:   $HOME_DIR"
echo "   Grupos:            $USERNAME, $SECONDARY_GROUP"
echo "   Contraseña Temp:   $PASSWORD"
echo "--------------------------------------------------"
echo "El usuario deberá cambiar esta contraseña en su primer login."
echo ""

```

### Paso 2: Otorgar Permisos de Ejecución

```bash
chmod +x crear_usuario.sh
```

## 🚀 Uso del Script

### Ejecutar el Script

```bash
sudo ./crear_usuario.sh
```

### Ejemplo de Interacción

```
[INFO] Iniciando el proceso de creación de nuevo usuario.
Introduce el nombre de usuario (ej: jlopez): lvaldez
Introduce el nombre completo (ej: Juan Lopez): Lucia Valdez
Introduce el grupo secundario (ej: desarrolladores, ventas): marketing
```

### Ejemplo de Salida Final

```
[OK] ¡PROCESO COMPLETADO!
--------------------------------------------------
 Resumen de la cuenta creada:
   Nombre de Usuario: lvaldez
   Nombre Completo:   Lucia Valdez
   Directorio Home:   /home/lvaldez
   Grupos:            lvaldez, marketing
   Contraseña Temp:   xJkLp/3rQzW+
--------------------------------------------------
El usuario deberá cambiar esta contraseña en su primer login.
```

## 🔍 Verificación (Opcional)

### Verificar datos del usuario y grupos:
```bash
id lvaldez
```

### Listar contenido del directorio personal:
```bash
ls -l /home/lvaldez
```

### Leer el archivo de bienvenida:
```bash
sudo cat /home/lvaldez/bienvenido.txt
```

## 📁 Estructura de Directorios Creada

```
/home/[usuario]/
├── Documentos/     # Para archivos generales
├── Proyectos/      # Para proyectos de trabajo
├── Privado/        # Directorio privado (permisos 700)
└── bienvenido.txt  # Archivo de bienvenida con credenciales
```

## 🔒 Características de Seguridad

- **Contraseña temporal aleatoria** de 12 caracteres base64
- **Cambio obligatorio** de contraseña en el primer login
- **Directorio Privado** con permisos restrictivos (700)
- **Validación de entrada** para prevenir errores
- **Verificación de usuarios existentes** antes de crear

## ⚙️ Requisitos del Sistema

- Sistema Linux con Bash
- Privilegios de superusuario (sudo)
- Comandos estándar: `useradd`, `groupadd`, `chpasswd`, `chage`, `openssl`

## 📝 Conceptos de Linux Aplicados

- **Gestión de usuarios**: `useradd`, `usermod`
- **Gestión de grupos**: `groupadd`
- **Permisos y propiedades**: `chmod`, `chown`
- **Gestión de contraseñas**: `chpasswd`, `chage`
- **Scripting avanzado**: variables, condicionales, funciones

## ⚠️ Notas Importantes

- **Guarda la contraseña temporal** mostrada en el resumen para entregarla al usuario
- El script **verifica si el usuario ya existe** antes de crear uno nuevo
- El **grupo secundario se crea automáticamente** si no existe
- El usuario **debe cambiar la contraseña** en su primer inicio de sesión
