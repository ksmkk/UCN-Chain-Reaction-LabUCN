# Decoder — copiar y pegar directamente en Ubuntu

**Academia de Ciberseguridad UCN · Chain Reaction**

Este es el método utilizado durante la resolución del laboratorio. **No necesitas descargar un archivo, crear `decoder.py` ni instalar bibliotecas.**

1. Abre la terminal del Ubuntu de Chain Reaction.
2. Ubícate en la carpeta del proyecto que encontraste durante el reconocimiento (usa `pwd`, `ls` y `cd`).
3. Copia **todo** el siguiente bloque, desde `python3 - <<'PY'` hasta la última línea `PY`, y pégalo en la terminal.
4. Si la terminal muestra `>`, aún está esperando la línea final `PY`; escríbela sola y presiona Enter.
5. Analiza las cadenas: busca una URL, una instrucción con `curl`, los parámetros `-d` y `-o`. Para la URL completa, investiga también cómo se utiliza `_entry`.

> Esta herramienta solo lee `postinst.js` como texto y transforma cadenas; **no ejecuta el JavaScript malicioso ni contacta ningún C2**. No ejecutes los comandos `curl` encontrados en la salida.

```bash
python3 - <<'PY'
"""Analizador estático del stq de Chain Reaction (TryHackMe).
No importa, ejecuta ni contacta el código JS analizado.
Modo de uso: copiar este bloque completo en la terminal de TryHackMe.
"""
import argparse
import ast
import base64
import re
from pathlib import Path


def extract(js: str):
    match_key = re.search(r'\bord\s*=\s*(["\'])(.*?)\1', js)
    match_array = re.search(r'\bstq\s*=\s*(\[.*?\])\s*,\s*ord\s*=', js, re.S)
    if not match_key or not match_array:
        raise ValueError('No se encontraron las variables ord y stq esperadas.')
    key = match_key.group(2)
    strings = ast.literal_eval(match_array.group(1))
    if not isinstance(strings, list) or not all(isinstance(x, str) for x in strings):
        raise ValueError('stq no contiene exclusivamente cadenas.')
    match_entry = re.search(r'_entry\(\s*(["\'])(.*?)\1\s*\)\s*;?\s*$', js)
    entry = match_entry.group(2) if match_entry else None
    return key, strings, entry


def decode(value: str, key: str):
    # JavaScript: Number('O') -> NaN; XOR convierte NaN a entero 0.
    nums = [int(c) if c.isdigit() else 0 for c in key]
    if not nums:
        raise ValueError('La clave está vacía.')
    reversed_b64 = value[::-1].replace('_', '=')
    raw = base64.b64decode(reversed_b64, validate=True).decode('utf-8')
    return ''.join(chr(ord(ch) ^ nums[(7 * i * i) % 10] ^ 333)
                   for i, ch in enumerate(raw))


def main():
    p = argparse.ArgumentParser(description='Decodificador estático para Chain Reaction de TryHackMe')
    p.add_argument('archivo', nargs='?', default='node_modules/typing-coreutils/postinst.js')
    args = p.parse_args()
    source = Path(args.archivo)
    if not source.is_file():
        p.error(f'Archivo no encontrado: {source}. Entra a ~/Desktop/thm-blog o indica la ruta completa.')
    js = source.read_text(encoding='utf-8')
    key, strings, entry = extract(js)
    print(f'Clave (ord): {key}')
    print(f'Cadenas encontradas: {len(strings)}')
    for i, value in enumerate(strings):
        try:
            decoded = decode(value, key)
            if decoded:
                print(f'\n[{i}] {decoded}')
        except (ValueError, UnicodeError) as e:
            print(f'[{i}] No se pudo decodificar: {e}')
    if entry:
        print(f'\nPista: _entry recibe el argumento {entry!r}.')
        print('Pregunta: ¿cómo usa _entry ese argumento junto con las cadenas?')
    print('\nAnálisis estático finalizado; no se ejecutó el JavaScript ni se hicieron conexiones.')


if __name__ == '__main__':
    main()
PY
```

### ¿Qué aprendimos?

- `stq` guarda cadenas transformadas y `ord` es la clave de la transformación.
- El decodificador recupera esas cadenas mediante inversión, Base64 y XOR.
- Los resultados son **evidencias para interpretar**. No copies automáticamente una cadena en TryHackMe sin comprobar qué pregunta responde.

### Si aparece un error

- **Archivo no encontrado:** verifica con `pwd` y `ls` que estás en la carpeta del proyecto.
- **No se encontraron `ord` y `stq`:** inspecciona si estás leyendo el archivo `postinst.js` correcto.
- **Salida demasiado larga:** concéntrate primero en los fragmentos relacionados con Linux; la salida también muestra instrucciones para otros sistemas.

[Volver a la guía de estudiantes](../docs/guia-estudiantes.md).
