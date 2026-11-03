# Chain Reaction — Academia de Ciberseguridad UCN

**Investigación guiada de TryHackMe · Grupo avanzado inicial**

En esta actividad investigaremos cómo una dependencia npm puede introducir código ejecutado mediante `postinstall`, ocultar instrucciones y dejar evidencias en Ubuntu. **No reproduce el robo de Binance**: se utiliza para comparar técnicas de compromiso de desarrolladores y confianza en software.

## Comienza por aquí

**1.** [Abrir la sala Chain Reaction en TryHackMe](https://tryhackme.com/room/chainreaction)  
**2.** [Seguir la guía de estudiantes, sin respuestas](docs/guia-estudiantes.md)  
**3.** [Copiar y pegar el decoder completo en la terminal](herramientas/decoder-terminal.md)

### ¿Cómo funciona la actividad?

- **Preguntas 1–4:** explorar el proyecto con `pwd`, `ls`, `cd`, `cat`, `grep` y `npm list`.
- **Preguntas 5–7:** pegar el bloque de Python completo (no crear ningún archivo) e interpretar las cadenas decodificadas.
- **Preguntas 8–10:** investigar procesos, persistencia y datos codificados, utilizando las evidencias encontradas.

El código del decoder se entrega completo, pero **no se entregan las respuestas del CTF**. El trabajo es comprender la evidencia, decidir qué dato corresponde a cada pregunta y justificarlo.

### Seguridad

No ejecutes `node postinst.js`, `npm install`, `npm update` ni instrucciones `curl` extraídas del código sospechoso. El decoder es de lectura y análisis estático para esta sala, no una herramienta genérica de análisis de malware.

**Solo hay una versión del decoder para alumnos:** [decoder-terminal.md](herramientas/decoder-terminal.md), preparada para copiar el bloque directamente en Ubuntu. No necesitas crear ni descargar archivos.

## Referencias

- [TryHackMe — Chain Reaction](https://tryhackme.com/room/chainreaction)
- [MITRE ATT&CK](https://attack.mitre.org/)
**Academia de Ciberseguridad UCN** · Material de uso ético en entornos autorizados.
