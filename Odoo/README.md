# Guía de Migración Odoo: Proxmox → VirtualBox

## 📋 **ÍNDICE**
1. [Contexto y Requisitos](#contexto-y-requisitos)
2. [Preparación del Entorno](#preparación-del-entorno)
3. [Descarga de Archivos](#descarga-de-archivos)
4. [Conversión de Formatos](#conversión-de-formatos)
5. [Configuración VirtualBox](#configuración-virtualbox)
6. [Resolución de Problemas Comunes](#resolución-de-problemas-comunes)
7. [Configuración de Red](#configuración-de-red)
8. [Configuración Odoo](#configuración-odoo)
9. [Verificación Final](#verificación-final)
10. [Checklist de Migración](#checklist-de-migración)

---

## 1. CONTEXTO Y REQUISITOS

### **Escenario**
Migración de VM Odoo desde **Proxmox** (formato qcow2/VMDK) a **VirtualBox** en entorno local para pruebas y migración.

### **Requisitos del Sistema**
- **Host**: Ubuntu/Debian con VirtualBox instalado
- **Espacio**: 100GB mínimo (40GB VM + espacio para conversiones)
- **Red**: Acceso a red local 192.168.1.0/24
- **Herramientas**: 
  - VirtualBox 7.0+
  - qemu-utils
  - zstd (compresión)
  - ssh, curl, wget

### **Estructura de Directorios Recomendada**
```
~/bioaire_migracion/
├── descargas/           # Archivos comprimidos originales
├── vm_actual/          # VM principal (producción)
├── vm_antigua/         # VM backup/antigua
├── backups/            # Copias de seguridad
├── scripts/            # Scripts de automatización
└── documentacion/      # Configuraciones y notas
```

---

## 2. PREPARACIÓN DEL ENTORNO

### **Instalación de Dependencias**
```bash
# Actualizar sistema
sudo apt update && sudo apt upgrade -y

# Instalar herramientas necesarias
sudo apt install -y \
    virtualbox \
    virtualbox-ext-pack \
    qemu-utils \
    zstd \
    wget \
    curl \
    ssh \
    net-tools \
    libguestfs-tools

# Verificar versiones
vboxmanage --version
qemu-img --version
zstd --version
```

### **Configuración de Permisos**
```bash
# Añadir usuario al grupo vboxusers
sudo usermod -a -G vboxusers $USER
newgrp vboxusers

# Crear estructura de directorios
mkdir -p ~/bioaire_migracion/{descargas,vm_actual,vm_antigua,backups,scripts,documentacion}
```

---

## 3. DESCARGA DE ARCHIVOS

### **Descarga desde Servidor Remoto**
```bash
cd ~/bioaire_migracion/descargas

# Script de descarga segura
cat > descargar.sh << 'EOF'
#!/bin/bash
URL="https://servidor.empresa.cat/"
USER="Usuario"
PASS="Contraseña"
FILES="bioaire.conf bioaire.vmdk.zst bioaire-old.conf bioaire-old.vmdk.zst"

for file in $FILES; do
    echo "Descargando: $file"
    wget --user="$USER" --password="$PASS" \
         --no-check-certificate \
         --continue \
         "$URL/$file"
done
EOF

chmod +x descargar.sh
./descargar.sh
```

### **Verificación de Integridad**
```bash
# Verificar checksums SHA256
cat > verificar_checksums.sh << 'EOF'
#!/bin/bash
echo "bd88bc0de755e1e49edd6235019d7d90197095be35e5e93efb6dc64b3c43f9f4  bioaire.conf" > checksums.sha256
echo "c3b2d9ccc0777b2d0e106689f8ea0106ea6b64fecb6f3bd2a625988d4ae353f2  bioaire.vmdk.zst" >> checksums.sha256
echo "25da4a465d06a0256118f2e34e658cc838da1076a6989f202ffee755bd032f65  bioaire-old.conf" >> checksums.sha256
echo "babd07ff76c0d6ab1f34a4aece00834b1c6dc3760dd9c69152de4412ffda1db2  bioaire-old.vmdk.zst" >> checksums.sha256

sha256sum -c checksums.sha256
EOF

chmod +x verificar_checksums.sh
./verificar_checksums.sh
```

---

## 4. CONVERSIÓN DE FORMATOS

### **Descompresión de Archivos**
```bash
# Descomprimir archivos .zst
cd ~/bioaire_migracion

echo "Descomprimiendo bioaire.vmdk.zst..."
unzstd descargas/bioaire.vmdk.zst -o vm_actual/bioaire.vmdk

echo "Descomprimiendo bioaire-old.vmdk.zst..."
unzstd descargas/bioaire-old.vmdk.zst -o vm_antigua/bioaire-old.vmdk

# Verificar tamaños
ls -lh vm_actual/bioaire.vmdk vm_antigua/bioaire-old.vmdk
```

### **Conversión a QCOW2 (Opcional)**
```bash
# Si se prefiere formato QCOW2
qemu-img convert -f vmdk -O qcow2 \
    vm_actual/bioaire.vmdk \
    vm_actual/bioaire.qcow2

qemu-img convert -f vmdk -O qcow2 \
    vm_antigua/bioaire-old.vmdk \
    vm_antigua/bioaire-old.qcow2
```

---

## 5. CONFIGURACIÓN VIRTUALBOX

### **Análisis de Configuración Original**
```bash
# Extraer configuración de hardware desde archivos .conf
cat descargas/bioaire.conf
# Salida esperada:
# memory: 3072
# cores: 2
# mac: BC:24:11:11:79:E7
```

### **Script de Creación de VM**
```bash
cat > ~/bioaire_migracion/scripts/crear_vm_odoo.sh << 'EOF'
#!/bin/bash
# Script para crear VM Odoo en VirtualBox

VM_NAME="BioAire_Nuevo"
DISCO_PATH="$HOME/bioaire_migracion/vm_actual/bioaire.vmdk"
CONF_FILE="$HOME/bioaire_migracion/descargas/bioaire.conf"

# Extraer configuración
MEMORY=$(grep -i memory "$CONF_FILE" | grep -o '[0-9]*')
CPUS=$(grep -i cores "$CONF_FILE" | grep -o '[0-9]*')
MAC=$(grep -i "mac\|virtio" "$CONF_FILE" | grep -o '=[^,]*' | head -1 | tr -d '=:')

# Valores por defecto
MEMORY=${MEMORY:-3072}
CPUS=${CPUS:-2}
MAC=${MAC:-"BC24111179E7"}

echo "=== CREANDO VM: $VM_NAME ==="
echo "Memoria: $MEMORY MB"
echo "CPUs: $CPUS"
echo "MAC: $MAC"

# Eliminar VM existente
VBoxManage unregistervm "$VM_NAME" --delete 2>/dev/null

# Crear nueva VM
VBoxManage createvm --name "$VM_NAME" --ostype "Ubuntu_64" --register

# Configurar hardware
VBoxManage modifyvm "$VM_NAME" \
    --memory "$MEMORY" \
    --cpus "$CPUS" \
    --nic1 bridged \
    --bridgeadapter1 "$(ip route | grep default | awk '{print $5}')" \
    --macaddress1 "$MAC" \
    --graphicscontroller vmsvga \
    --vram 32 \
    --audio none \
    --usb off \
    --chipset ich9

# Configurar almacenamiento
VBoxManage storagectl "$VM_NAME" \
    --name "SATA Controller" \
    --add sata \
    --controller IntelAhci

VBoxManage storageattach "$VM_NAME" \
    --storagectl "SATA Controller" \
    --port 0 \
    --device 0 \
    --type hdd \
    --medium "$DISCO_PATH"

echo ""
echo "✅ VM '$VM_NAME' creada correctamente"
echo "Para arrancar: VBoxManage startvm '$VM_NAME' --type headless"
EOF

chmod +x ~/bioaire_migracion/scripts/crear_vm_odoo.sh
./scripts/crear_vm_odoo.sh
```

---

## 6. RESOLUCIÓN DE PROBLEMAS COMUNES

### **Problema 1: "could not read from the boot medium"**
**Causa**: Disco no conectado correctamente o formato incompatible.

**Solución**:
```bash
# Verificar conexión del disco
VBoxManage showvminfo "BioAire_Nuevo" --details | grep "Storage"

# Reconectar disco
VBoxManage storageattach "BioAire_Nuevo" \
    --storagectl "SATA Controller" \
    --port 0 --device 0 --type hdd --medium none

VBoxManage storageattach "BioAire_Nuevo" \
    --storagectl "SATA Controller" \
    --port 0 --device 0 --type hdd \
    --medium "$HOME/bioaire_migracion/vm_actual/bioaire.vmdk"
```

### **Problema 2: Error de permisos en disco VMDK**
**Causa**: Archivo propiedad de root o permisos incorrectos.

**Solución**:
```bash
sudo chown $USER:$USER ~/bioaire_migracion/vm_actual/bioaire.vmdk
chmod 644 ~/bioaire_migracion/vm_actual/bioaire.vmdk
```

### **Problema 3: VM no obtiene IP**
**Causa**: Configuración de red incorrecta.

**Solución**:
```bash
# Cambiar a adaptador Intel PRO/1000 (más compatible)
VBoxManage modifyvm "BioAire_Nuevo" --nictype1 82540EM

# O cambiar a VirtIO si la VM lo requiere
VBoxManage storagectl "BioAire_Nuevo" --name "SATA" --remove
VBoxManage storagectl "BioAire_Nuevo" --name "VirtIO" --add virtio
VBoxManage storageattach "BioAire_Nuevo" \
    --storagectl "VirtIO" --port 0 --device 0 --type hdd \
    --medium "$HOME/bioaire_migracion/vm_actual/bioaire.vmdk"
```

---

## 7. CONFIGURACIÓN DE RED

### **Arranque y Obtención de IP**
```bash
# Arrancar VM
VBoxManage startvm "BioAire_Nuevo" --type headless

# Esperar arranque (2-3 minutos)
sleep 180

# Buscar IP automáticamente
echo "=== BUSCANDO IP DE LA VM ==="

# Método 1: Propiedades VirtualBox
VBoxManage guestproperty enumerate "BioAire_Nuevo" | grep -i ip

# Método 2: Escaneo ARP por MAC
sudo arp-scan --localnet | grep -i "BC:24:11:11:79:E7"

# Método 3: Desde consola de la VM (VirtualBox GUI)
# Ejecutar: ip a  o  hostname -I
```

### **Configuración Manual de IP (si DHCP falla)**
```bash
# Acceder a consola de la VM y ejecutar:
sudo dhclient enp0s3  # Intentar DHCP

# O configurar IP estática
sudo ip addr add 192.168.1.200/24 dev enp0s3
sudo ip link set enp0s3 up
sudo ip route add default via 192.168.1.1
```

### **Hacer IP estática permanente**
```bash
# Editar /etc/netplan/00-installer-config.yaml
sudo nano /etc/netplan/00-installer-config.yaml

# Añadir configuración:
network:
  ethernets:
    enp0s3:
      dhcp4: no
      addresses: [192.168.1.200/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
  version: 2

# Aplicar cambios
sudo netplan apply
```

---

## 8. CONFIGURACIÓN ODOO

### **Problema Común: Selector de Base de Datos**
**Síntoma**: Odoo muestra `http://IP:8069/web/database/selector` en lugar del login.

**Causa**: Configuración incorrecta en `odoo.conf`:
- `db_name = False` (debería ser nombre de BD)
- `list_db = True` (debería ser False)
- `dbfilter = ^%d` (filtro demasiado amplio)

**Solución Definitiva**:
```bash
# 1. Identificar base de datos correcta
sudo -u postgres psql -c "\l"
# Buscar: gestio, antic, odoo, etc.

# 2. Corregir configuración (ejemplo con BD 'gestio')
CONFIG_FILE="/var/opt/odoo/conf/odoo.conf"

# Backup
sudo cp "$CONFIG_FILE" "${CONFIG_FILE}.backup"

# Aplicar correcciones
sudo sed -i 's/^db_name = .*/db_name = gestio/' "$CONFIG_FILE"
sudo sed -i 's/^list_db = .*/list_db = False/' "$CONFIG_FILE"
sudo sed -i 's/^dbfilter = .*/dbfilter = ^gestio$/' "$CONFIG_FILE"

# 3. Eliminar configuración conflictiva en /etc
sudo mv /etc/odoo/odoo.conf /etc/odoo/odoo.conf.disabled 2>/dev/null

# 4. Reiniciar Odoo
sudo systemctl restart odoo
```

### **Reset de Contraseñas (si es necesario)**
```bash
# Acceder a consola Odoo
sudo -u odoo /var/opt/odoo/pyOdoo/bin/python /var/opt/odoo/OCB/odoo-bin shell -d gestio

# En la consola Python:
# env['res.users'].search([('login','=','admin@bioaire.es')]).password = 'NuevaPass123!'
# env.cr.commit()
```

### **Verificación de Servicios**
```bash
# Dentro de la VM
echo "=== ESTADO DE SERVICIOS ==="
systemctl status odoo --no-pager
systemctl status postgresql --no-pager

echo "=== PUERTOS ABIERTOS ==="
ss -tulpn | grep -E ":8069|:5432|:22"

echo "=== ACCESO WEB ==="
curl -I http://localhost:8069/web
```

---

## 9. VERIFICACIÓN FINAL

### **Checklist de Verificación**
```bash
cat > ~/bioaire_migracion/verificacion_final.sh << 'EOF'
#!/bin/bash
echo "=== CHECKLIST DE VERIFICACIÓN ==="
echo ""

# 1. VM en VirtualBox
echo "1. VM en VirtualBox:"
VBoxManage list vms | grep "BioAire" && echo "✅" || echo "❌"

# 2. VM en ejecución
echo "2. VM ejecutándose:"
VBoxManage list runningvms | grep "BioAire" && echo "✅" || echo "❌"

# 3. IP asignada
echo "3. IP de la VM:"
IP=$(VBoxManage guestproperty get "BioAire_Nuevo" "/VirtualBox/GuestInfo/Net/0/V4/IP" 2>/dev/null | grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}')
[ -n "$IP" ] && echo "✅ $IP" || echo "❌ Sin IP"

# 4. Acceso SSH
echo "4. Acceso SSH:"
if [ -n "$IP" ]; then
    timeout 2 ssh -o ConnectTimeout=1 root@$IP "echo '✅'" 2>/dev/null && echo "✅" || echo "❌"
else
    echo "⚠️  Sin IP para probar"
fi

# 5. Acceso Odoo Web
echo "5. Odoo Web (puerto 8069):"
if [ -n "$IP" ]; then
    timeout 2 curl -s http://$IP:8069/web >/dev/null && echo "✅" || echo "❌"
else
    echo "⚠️  Sin IP para probar"
fi

# 6. Login Odoo (no selector de BD)
echo "6. Login Odoo (no selector BD):"
if [ -n "$IP" ]; then
    curl -s http://$IP:8069/web | grep -qi "database.*manager" && echo "❌ Selector BD" || echo "✅ Login normal"
fi

echo ""
echo "=== RESUMEN ==="
echo "Si todos están en ✅, la migración es EXITOSA"
echo "URL Odoo: http://$IP:8069"
echo "SSH: root@$IP"
EOF

chmod +x ~/bioaire_migracion/verificacion_final.sh
./verificacion_final.sh
```

### **Configuración para Proxy-Reverse**
```nginx
# Ejemplo Nginx para odoo.bioaire.es
server {
    listen 80;
    server_name odoo.bioaire.es;
    
    location / {
        proxy_pass http://192.168.1.41:8069;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_read_timeout 90s;
        
        # WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
    
    location /longpolling {
        proxy_pass http://192.168.1.41:8072;
    }
}
```

---

## 10. CHECKLIST DE MIGRACIÓN

### **Fase 1: Preparación**
- [ ] Espacio en disco verificado (>100GB)
- [ ] VirtualBox y herramientas instaladas
- [ ] Estructura de directorios creada
- [ ] Credenciales de acceso obtenidas

### **Fase 2: Descarga y Verificación**
- [ ] Archivos descargados del servidor remoto
- [ ] Checksums SHA256 verificados
- [ ] Archivos descomprimidos correctamente
- [ ] Permisos de archivos ajustados

### **Fase 3: Configuración VirtualBox**
- [ ] VM creada con parámetros correctos (RAM, CPU)
- [ ] Disco VMDK conectado correctamente
- [ ] Red configurada (bridge al adaptador correcto)
- [ ] VM arranca sin errores de boot

### **Fase 4: Configuración de Red**
- [ ] VM obtiene IP automáticamente (DHCP)
- [ ] Acceso SSH funcionando
- [ ] Ping al gateway y host funcionando
- [ ] IP estática configurada (si es necesario)

### **Fase 5: Configuración Odoo**
- [ ] Odoo servicio activo y corriendo
- [ ] PostgreSQL funcionando
- [ ] Configuración odoo.conf corregida
- [ ] Login Odoo accesible (no selector de BD)
- [ ] Credenciales verificadas/reseteadas

### **Fase 6: Verificación Final**
- [ ] Acceso web desde host funcionando
- [ ] Todos los servicios activos
- [ ] Datos de BD accesibles
- [ ] Documentación actualizada

### **Fase 7: Producción**
- [ ] Proxy-reverse configurado
- [ ] DNS apuntando correctamente
- [ ] Backup de VM realizado
- [ ] Plan de rollback preparado

---

## 📝 **LECCIONES APRENDIDAS**

### **Problemas Comunes y Soluciones**
1. **`db_name = False`** → Cambiar a nombre real de BD
2. **Selector de BD persistente** → `list_db = False` + `dbfilter = ^nombrebd$`
3. **Sin IP en VM** → Cambiar adaptador de red en VirtualBox
4. **Error de boot** → Verificar controlador de disco (SATA/VirtIO)
5. **Permisos de archivo** → Asegurar propiedad de usuario, no root

### **Mejores Prácticas**
- **Siempre verificar checksums** de archivos descargados
- **Probar con IP estática** si DHCP falla
- **Documentar cada paso** para futuras migraciones
- **Crear scripts reutilizables** para automatización
- **Realizar backup** antes de cada cambio importante

---

## 🚨 **CONTACTO Y SOPORTE**

### **Información Técnica Necesaria para Futuras Migraciones**
- Versión exacta de Odoo
- Módulos personalizados instalados
- Configuración de red específica
- Credenciales de acceso
- Tamaño estimado de BD

### **Archivos a Solicitar al Proveedor**
```
1. Archivo de disco VM (.vmdk.zst o .qcow2.zst)
2. Configuración de hardware (.conf)
3. Checksums de verificación (.sha256)
4. Credenciales de acceso (.txt)
5. Documentación de configuración especial
```

---

**✅ MIGRACIÓN COMPLETADA EXITOSAMENTE**

*Esta guía cubre el proceso completo de migración de Odoo desde Proxmox a VirtualBox, incluyendo solución de problemas comunes y checklist para futuras migraciones.*
