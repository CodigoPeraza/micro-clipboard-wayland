# Copiar y pegar en Micro bajo Wayland usando Kitty

> Integra el portapapeles del sistema con el editor **Micro** en un entorno Wayland, aprovechando el soporte *OSC 52* del terminal **Kitty**.

---

## 1 · Requisitos

Terminal: Kitty
No se necesita `wl-clipboard`, `xclip` ni *xwayland*.

---

## 2 · Configuración rápida

1. **Habilitar el acceso directo al portapapeles en Kitty**

   Edita (o crea) `~/.config/kitty/kitty.conf` y añade:

   ```bash
   # Permitir leer y escribir sin diálogos
   clipboard_control write-clipboard read-clipboard

   # Opcional: manejar también la selección primaria X11/Wayland
   # clipboard_control write-primary read-primary
   ```

2. **Indica a Micro que use la vía *terminal***

   Crea/edita `~/.config/micro/settings.json` 
   añadiendo o ajustando:

   ```jsonc
   {
     "clipboard": "terminal"
   }
   ```

   > El valor *terminal* fuerza a Micro a enviar secuencias OSC 52 en lugar de invocar herramientas externas.

3. (Re)inicia Kitty y Micro para que lean sus nuevos ajustes.

---

## 3 · Pruebas de funcionamiento

```bash
# Copiar texto desde la shell y pegar en otra aplicación
$ echo "Hola Micro" | kitty +kitten clipboard

# Comprobar que quedó copiado
$ kitty +kitten clipboard --get-clipboard
Hola Micro

# Dentro de Micro
- Selecciona texto y pulsa Ctrl+C → debería estar en el portapapeles global.
- Pulsa Ctrl+V → se inserta lo que tengas copiado desde fuera.
```

Si los pasos anteriores funcionan, el flujo OSC 52 → Kitty → Portapapeles está operativo.

---

## 4 · ¿Cómo funciona?

| Acción | Micro emite              | Kitty hace                                       |
| ------ | ------------------------ | ------------------------------------------------ |
| Copiar | `ESC ]52;c;<BASE64> BEL` | Decodifica y guarda en el portapapeles           |
| Pegar  | `ESC ]52;?;? BEL`        | Responde con otra secuencia OSC 52, Micro la lee |

El estándar OSC 52 es texto puro; viaja incluso a través de SSH. Solo el terminal local (Kitty) debe entenderlo.

---

## 5 · Alternativa: script externo + *clipboard kitten*

Si prefieres la vía *external* (o usas otro terminal compatible con el kitten), crea:

```bash
# ~/.local/bin/kittyclip
#!/usr/bin/env bash
case "$1" in
  copy)  kitty +kitten clipboard ;;
  paste) kitty +kitten clipboard --get-clipboard ;;
  *)     echo "Uso: $0 {copy|paste}" >&2 ; exit 1 ;;
esac
```

Hazlo ejecutable: `chmod +x ~/.local/bin/kittyclip`.

Luego ajusta `settings.json`:

```jsonc
{
  "clipboard": "external",
  "clipboardcmd": "~/.local/bin/kittyclip"
}
```

---

## 6 · Seguridad

* `read-clipboard` expone el portapapeles a **todos** los procesos dentro de Kitty.
  *Si compartes sesión o ejecutas scripts no confiables, reemplaza por `read-clipboard-ask`.*
* OSC 52 limita el tamaño (\~100 kB). Para pegar grandes bloques usa el *clipboard kitten* con archivos (`kitty +kitten clipboard archivo.txt`).

---

## 7 · Troubleshooting

1. **Versión antigüa de Kitty** → actualiza (`kitty --version`).
2. `clipboard=internal` aún activo → cámbialo por `terminal`.
3. Otro programa captura OSC 52 → prueba en una sesión limpia.
4. *Wayland/xwayland* mezclados → asegúrate de no tener *xclip* en PATH si no lo deseas.

---

## 8 · Créditos y enlaces

* Foro de Arch Linux sobre *clipboard kitten*     ---> https://wiki.archlinux.org/title/Kitty
* Manual oficial de Kitty ‒ *section clipboard*   ---> https://sw.kovidgoyal.net/kitty/kittens/clipboard/
* Foros de Micro ‒ *Clipboard integration*        ---> https://github.com/zyedidia/micro/issues/2238?utm_source=chatgpt.com

> Adaptado para Fedora + Hyprland pero debería funcionar igual en cualquier distribución Wayland.
