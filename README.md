# Clean Launcher (versión genérica)

Launcher Android limpio y ligero para radios de auto con Android (5.0 o superior).
Nació para un radio NWD (Allwinner T3); esta versión funciona en **cualquier radio Android**
y activa funciones extra cuando detecta un NWD.

## Funciones en cualquier radio Android

- **Pantalla principal** con hora, clima (Open-Meteo, por GPS o IP), foto de tu carro y música sonando.
- **Música de cualquier app** (Spotify, YouTube Music, etc.) con carátula y controles
  (requiere dar acceso a notificaciones).
- **Atajos** a las apps de radio, Bluetooth, ecualizador y ajustes que ya trae tu radio.
- **Modo pantalla apagada**: pantalla negra al mínimo de brillo; toca para volver.
- **Control de aceite / odómetro por GPS** con recordatorios de mantenimiento.
- **Saludo por voz** al encender (texto a voz del equipo): hora, clima, cinturón, mantenimiento y,
  opcional, pico y placa. En radios genéricos se dispara al encender la pantalla (máx. cada 30 min).
- **Configuración inicial**: nombre, foto del carro, voz y permisos.
- Cajón de aplicaciones.

## Funciones extra en radios NWD

Radio FM propia (dial, presets), Bluetooth integrado (música, marcador, dispositivos), ecualizador
de 10 bandas con modo automático, overlay de volumen, pantalla apagada que respeta el volante,
restauración de la última fuente de audio y luces del auto en pantalla.
Se detecta solo; si falla, activa **Modo compatible** en la configuración (toca la foto del carro).

## Compatibilidad

| Aspecto | Valor |
|---|---|
| Android | 5.0+ (minSdk 21) |
| Radios NWD | funciones completas |
| Otros radios (Android genérico) | funciones genéricas listadas arriba |

Limitaciones en radios no NWD: el launcher abre las apps de radio y Bluetooth que tenga el equipo
(las busca por nombre: "radio", "fm", "tuner", "bluetooth"…). Si tu radio no trae TTS, el saludo
no se oirá. Los botones del volante y el cambio de fuente dependen del fabricante.

## Compilar

No usa Gradle: [`build.sh`](build.sh) llama directamente a `aapt2`, `javac`, `d8`, `zipalign` y
`apksigner`. Funciona en Linux, macOS y Windows (Git Bash).

### Requisitos

| Herramienta | Versión | Notas |
|---|---|---|
| JDK | 17 o superior | Sirve el JBR que trae Android Studio |
| Android SDK Platform | `android-33` | Solo para compilar. El APK corre en Android 7.1+ (minSdk 25) |
| Android SDK Build-Tools | 36.0.0 (probado) | El script usa la más nueva instalada |
| Platform-Tools (`adb`) | cualquiera | Solo para `install` |

### Desde una instalación limpia

1. Instala un JDK 17+ y las [herramientas de línea de comandos del Android SDK](https://developer.android.com/studio#command-tools)
   (o Android Studio, que trae ambos).
2. Instala los paquetes del SDK:
   ```sh
   sdkmanager "platforms;android-33" "build-tools;36.0.0" "platform-tools"
   ```
3. Indica dónde está el SDK si no está en la ruta por defecto de Android Studio:
   ```sh
   export ANDROID_HOME=$HOME/Android/Sdk
   ```
4. Compila:
   ```sh
   sh build.sh                            # genera build/CleanLauncher.apk
   sh build.sh install                    # además lo instala con adb
   RADIO=192.168.1.15:5555 sh build.sh install   # radio por ADB Wi-Fi
   ```

Si falta algo, el script se detiene y dice qué instalar. Detecta el SDK con `ANDROID_HOME` o
`ANDROID_SDK_ROOT` (o las rutas por defecto de Android Studio) y el JDK con `JAVA_HOME`, `javac`
en el `PATH` o el JBR de Android Studio. Variables opcionales: `ANDROID_PLATFORM`, `BUILD_TOOLS`,
`KEYSTORE`, `KEYSTORE_PASS`, `KEY_ALIAS`.
Si no existe `~/.android/debug.keystore`, el script lo crea.

### Integración continua

Cada push a `main` y cada pull request se compila en GitHub Actions
([`.github/workflows/build.yml`](.github/workflows/build.yml)). El APK queda como artefacto
descargable en la pestaña **Actions** del repo. Está firmado con una clave de debug.

## Integración con el radio NWD

| Qué | Cómo |
|---|---|
| Cambio de fuente de audio | broadcast `com.nwd.action.ACTION_REQUEST_CHANGE_SOURCE` (radio=8, BT música=15, USB=2) |
| Radio | Binder crudo a `com.nwd.radio.service` (`RadioClient`) |
| Bluetooth | Binder crudo a `com.bt.bc03` (`BtClient`) |
| Voz sobre la radio | launcher añadido a `/nwdappconfig/app/NaviSound.ini` |
| No ser cerrado en segundo plano | añadido a `BackgroundManager.ini` y `OneClearConfig.xml` |

## Cambios hechos al radio (todos reversibles)

- Deshabilitado bloatware de fábrica (apps de prueba, DiDi, voz, anuncios CaradService, carstore, Aptoide).
- Deshabilitados componentes del launcher de fábrica: FloatBar, WeatherService, SmallPackageUpdater, AMap APSService.
  **No** se deshabilita `com.android.launcher` completo (aloja el aprendizaje de botones del volante y el apagado de pantalla).
- Animaciones a 0.5x.
- Archivos de configuración modificados: los originales están en [`backup/`](backup/).

Para revertir: copiar los `.bak` de `backup/` a `/nwdappconfig/app/` y volver a habilitar los paquetes con
`adb shell pm enable <paquete>`.
