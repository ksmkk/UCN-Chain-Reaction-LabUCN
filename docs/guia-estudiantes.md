# Chain Reaction — Investigación guiada

**Academia de Ciberseguridad UCN · Grupo avanzado inicial**  
**Plataforma:** [TryHackMe — Chain Reaction](https://tryhackme.com/room/chainreaction-bt)  
**Modalidad:** investigación de evidencias en Ubuntu, con apoyo de un decodificador estático.

> **Desafío:** un desarrollador instaló una dependencia aparentemente normal. ¿Podemos demostrar qué se instaló, qué código se ejecutaba y qué rastros dejó? **No ejecutaremos el malware ni contactaremos su servidor.**

## Antes de comenzar

- Abre la máquina de TryHackMe en el navegador o mediante el acceso que indique la sala.
- Trabaja únicamente en el entorno autorizado de la actividad.
- Mantén abiertas estas instrucciones y una terminal Ubuntu.
- No ejecutes `npm install`, `npm update`, `node postinst.js` ni comandos `curl` que aparezcan dentro del malware. Son **evidencia para leer**, no comandos de práctica.

## Parte A — Detectar el problema (preguntas 1–4)

### 1. Encontrar el proyecto

En la terminal:

```bash
pwd
ls
ls ~/Desktop
```

**Piensa:** ¿en qué directorio estás? ¿Ves alguna carpeta de un proyecto del desarrollador?

Entra al directorio del proyecto que hayas encontrado y revisa su contenido:

```bash
cd ~/Desktop/REEMPLAZA_POR_LA_CARPETA_DEL_PROYECTO
ls -la
```

Si encuentras un archivo `package.json`, lee qué dependencias declara:

```bash
cat package.json
```

**Pregunta:** ¿para qué sirve este archivo? Si `cat` dice «No such file or directory», comprueba `pwd`: probablemente no estás en el directorio correcto.

### 2. Versión declarada versus versión instalada — Q1

```bash
grep axios package.json
npm list axios
```

**Piensa:** ¿ambos comandos entregan lo mismo? ¿Qué podría significar el signo `^` en una versión de npm? Anota la versión **instalada**, no solamente la declarada.

### 3. Investigar dependencias de Axios — Q2

```bash
grep -A 10 '"dependencies"' node_modules/axios/package.json
```

`-A 10` muestra diez líneas después de la coincidencia. Busca una dependencia que merezca mayor investigación. Abre su propio `package.json` y comprueba su versión:

```bash
cat node_modules/NOMBRE_DEL_PAQUETE/package.json
```

**Pregunta:** ¿cómo puede una biblioteca instalar código de otra empresa o persona?

### 4. Encontrar la ejecución automática — Q3

En ese `package.json`, busca el apartado `scripts` y en particular `postinstall`.

**Pregunta:** ¿qué programa podría ejecutarse durante la instalación sin que el usuario lo invoque manualmente?

### 5. La clave de la ofuscación — Q4

En la carpeta del paquete sospechoso, localiza el archivo JavaScript de instalación. **No lo ejecutes**. Puedes examinarlo con `less` y salir con `q`:

```bash
less node_modules/NOMBRE_DEL_PAQUETE/postinst.js
```

Como el archivo está escrito prácticamente en una sola línea, usa una búsqueda que muestre solo el fragmento que nos interesa:

```bash
grep -o 'ord = "[^"]*"' node_modules/NOMBRE_DEL_PAQUETE/postinst.js
```

**Pregunta:** ¿por qué un atacante ocultaría direcciones e instrucciones dentro de cadenas transformadas?

## Parte B — Usar el decoder (preguntas 5–7)

La Academia entrega el **[bloque completo para pegar en Ubuntu](../herramientas/decoder-terminal.md)**. No hace falta crear archivos ni escribir código Python manualmente. Es el mismo procedimiento utilizado durante la preparación de esta clase.

1. Abre el enlace del decoder en GitHub y copia el bloque completo, incluyendo `python3 - <<'PY'` y la última línea `PY`.
2. Regresa a la terminal de la máquina Ubuntu de TryHackMe y comprueba que estás en la carpeta del proyecto con `pwd`.
3. Pega el bloque completo y espera los resultados.
4. Interpreta las cadenas; el programa **no da por sí solo las respuestas completas**.

La herramienta lee texto de `postinst.js`; no ejecuta JavaScript ni realiza conexiones a servidores externos. **No ejecutes los comandos que descubras en las cadenas.**

### Q5. ¿Cuál es la URL C2 completa?

Busca una URL en la salida y verifica si el código agrega un identificador al final. **¿Cómo se construye la dirección completa?**

### Q6. ¿Qué mensaje solicita el payload?

En la cadena de Linux, identifica `curl` y revisa qué aparece después de `-d` (datos que enviaría en una solicitud).

### Q7. ¿Dónde quedaría guardado el archivo?

En la misma cadena, encuentra `-o` (ruta donde guardaría la respuesta). **No ejecutes el `curl` encontrado.**

## Parte C — Encontrar los rastros (preguntas 8–10)

### Q8. Investigar procesos

```bash
ps auxww | grep -E 'python|unattended|apt.conf'
```

**Piensa:** ¿qué proceso parece una actualización normal de Ubuntu, pero ejecuta un archivo desde el directorio personal? Distingue el proceso legítimo del sospechoso. Copia solo la **línea de comandos** que pide la sala, no el PID.

### Q9. Identificar persistencia

Usa la ruta del archivo sospechoso encontrada antes:

```bash
grep -n -A 20 'init_location' RUTA_DEL_ARCHIVO_SOSPECHOSO
```

`-n` muestra números de línea y `-A 20` veinte líneas posteriores. Busca modificaciones en archivos de inicio de sesión como `~/.profile`. **¿Qué podría pasar al cargar ese perfil?** Busca después la técnica correspondiente en [MITRE ATT&CK](https://attack.mitre.org/).

### Q10. Interpretar el último mensaje

```bash
grep -n 'LastInfo' RUTA_DEL_ARCHIVO_SOSPECHOSO
```

Copia el contenido codificado y prueba localmente:

```bash
printf '%s' 'CADENA_ENCONTRADA' | rev | base64 -d
```

**Piensa:** ¿por qué un analista debe diferenciar codificar, cifrar y ofuscar información?

## Cierre: explica el incidente con tus palabras

Sin mirar los comandos, completa esta secuencia: **proyecto → biblioteca → dependencia → script → cadenas ocultas → servidor C2 → archivo → proceso → persistencia**.

**Preguntas de reflexión:** ¿qué decisión facilitó el ataque? ¿Qué evidencias demostraron la infección? ¿Qué controles aplicarías: versiones bloqueadas, revisión de dependencias, limitación de scripts, mínimo privilegio y monitoreo?

**Relación con Binance:** comparación de técnicas y riesgos de confianza en software; este laboratorio no reproduce ese incidente.
