# Instalar WSL + Docker + entorno VNC con xschem (Windows 10)

Guía rápida para dejar andando el entorno del curso de diseño analógico / CMOS (IIC-OSIC-TOOLS con escritorio VNC y xschem).

## Requisitos

- Windows 10 versión 21H2 o superior (verificalo con `winver`).
- Virtualización activada en la BIOS (ver paso 1).
- Mucho espacio libre: la imagen de Docker pesa varios GB.

---

## 1. Activar la virtualización en la BIOS

Si el Administrador de tareas (Ctrl+Shift+Esc → Rendimiento → CPU) dice **Virtualización: Deshabilitada**, hay que activarla:

1. Reiniciá y entrá a la BIOS (suele ser **Supr**, **F2**, **F10** o **F12** al arrancar).
2. Pasá a **Advanced Mode** (normalmente **F7**).
3. Buscá la opción según tu procesador:
   - **AMD:** `SVM Mode` → **Enabled** (suele estar en *Advanced → CPU Configuration* o *AMD CBS → CPU Common Options*).
   - **Intel:** `Intel Virtualization Technology` / `VT-x` → **Enabled**.
4. Guardá y salí (**F10 → Yes**).

> No confundir con las opciones de TPM / fTPM: no tienen que ver con esto. Y no actives "Erase fTPM NV", porque borra el TPM.

Volvé a mirar el Administrador de tareas: tiene que decir **Habilitada**.

## 2. Instalar WSL con Ubuntu

Abrí **PowerShell como administrador** y corré:

```powershell
wsl --install -d Ubuntu
```

- Si pide reiniciar, **reiniciá** y volvé a correr el comando.
- Si da error de virtualización, revisá el paso 1 y probá:
  ```powershell
  bcdedit /set hypervisorlaunchtype auto
  wsl --update
  wsl --install -d Ubuntu
  ```
- Se abre una ventana de Ubuntu: creá un **usuario y contraseña** de Linux (al escribir la contraseña no se ve nada, es normal). Si pregunta por métricas, es opcional (`y` o `n`).

Verificá con:

```powershell
wsl --list --verbose
```

Debe aparecer Ubuntu con `VERSION 2`.

Dentro de Ubuntu, actualizá los paquetes:

```bash
sudo apt update && sudo apt upgrade -y
```

Después, en PowerShell:

```powershell
wsl --shutdown
```

## 3. Instalar Docker Desktop

1. Descargalo de [docker.com](https://www.docker.com/products/docker-desktop/) e instalalo (dejá marcada la opción **Use WSL 2 instead of Hyper-V**).
2. Abrí Docker Desktop → **Settings → Resources → WSL Integration** → activá **Ubuntu** → **Apply & Restart**.
3. Probá en Ubuntu:
   ```bash
   docker run hello-world
   ```
   Si aparece "Hello from Docker!", está todo bien.

## 4. Preparar y ejecutar `start_vnc.bat`

El script (del repositorio que indique el profe) lanza un contenedor con las herramientas y vincula una carpeta de tu PC para guardar los diseños.

1. Creá una carpeta en otro disco, sin espacios ni tildes. Ejemplo: `D:\opensource`.
2. Abrí `start_vnc.bat` **como texto**, sin ejecutarlo: clic derecho → *Abrir con* → *Bloc de notas*.
   - Ojo: no lo confundas con `start_shell.bat`, que es otro script.
   - Leelo antes de ejecutarlo: tiene que tener básicamente comandos `docker` y variables.
3. Cambiá la línea de la carpeta de trabajo (por ejemplo `set DESIGNS=...`) por tu ruta, respetando el estilo de barras que ya tenga:
   ```bat
   set DESIGNS=D:/opensource
   ```
4. Guardá (Ctrl+S) y verificá que quede como `.bat` y no `.bat.txt`.
5. Con **Docker Desktop abierto** ("Engine running" abajo a la izquierda), clic derecho sobre `start_vnc.bat` → **Ejecutar como administrador**.
6. Si aparece "editor desconocido" (SmartScreen/UAC), es normal en scripts sin firma digital: *Más información → Ejecutar de todas formas* (o *Sí* en el UAC). Si pide permiso de Firewall, aceptá para redes privadas.

La primera vez descarga una imagen muy grande, así que tarda. Puede que la ventana se cierre sola al terminar; comprobá en Docker Desktop → **Containers** que haya uno `iic-osic-tools_xvnc...` en verde.

## 5. Conectarse al escritorio

**Opción A: navegador**

```
http://localhost/?password=abc123
```

**Opción B: cliente VNC** (como el profe)

1. Descargá **TigerVNC Viewer** (tigervnc.org, es un `.exe` suelto).
2. En "VNC server" poné `localhost:5901` → Connect → contraseña `abc123`.

> `abc123` es la clave por defecto del proyecto. Si no anda, buscá `VNC_PW` en `start_vnc.bat`.

## 6. Abrir xschem

Dentro del escritorio, abrí una terminal y escribí:

```bash
cd /foss/designs
xschem
```

Para abrir un esquemático existente: `xschem nombre.sch`.

## Uso diario

**Guardá siempre en `/foss/designs`**: esa carpeta es la que se sincroniza con `D:\opensource`. Lo que guardes en otro lado se pierde si se borra el contenedor.

**Para cerrar todo:**

1. Guardá tu trabajo (Ctrl+S) y cerrá xschem y el navegador/TigerVNC.
2. En Docker Desktop → Containers, apretá ⏹ (stop). **No uses el tacho**, porque borra el contenedor.
3. Cerrá Docker Desktop (clic derecho en la ballena → Quit).
4. Opcional, para liberar RAM: `wsl --shutdown` en PowerShell.

**Para abrir todo de nuevo:**

1. Abrí Docker Desktop y esperá "Engine running".
2. Apretá ▶ en el contenedor (o ejecutá `start_vnc.bat` como administrador).
3. Esperá medio minuto y entrá por el navegador o TigerVNC.
4. Abrí una terminal y escribí `xschem`.

## Problemas comunes

| Problema | Solución |
|---|---|
| `WSL_E_WSL_OPTIONAL_COMPONENT_REQUIRED` | Reiniciar la PC y correr `wsl --install --no-distribution`. |
| `HCS_E_HYPERV_NOT_INSTALLED` / "virtualización no habilitada" | Activar SVM/VT-x en la BIOS (paso 1). |
| "No hay distribuciones instaladas" | Volver a correr `wsl --install -d Ubuntu`. |
| El navegador no carga el escritorio | Esperar un poco; revisar que el contenedor esté en verde. |
| Connection refused en TigerVNC | El contenedor no está corriendo: darle ▶ en Docker Desktop. |
