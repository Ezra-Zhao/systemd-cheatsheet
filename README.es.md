# systemd Cheatsheet · Hoja de referencia de comandos systemd

> 🌍 [中文](README.md) · [Português](README.pt.md)

Una **hoja de referencia de una sola página** con 30 usos reales de systemd: gestión de servicios con `systemctl`, registros con `journalctl`, archivos unit, temporizadores y estado del sistema. Cada entrada responde solo tres preguntas: **qué hace, un comando listo para copiar y pegar, y etiquetas de uso**. Sin parámetros inventados: cada comando y opción está verificado contra la salida de `--help` de systemd 255, las dos unidades de ejemplo se validaron con `systemd-analyze verify` y las expresiones de calendario se ejecutaron con `systemd-analyze calendar`. La mayoría de los comandos de gestión necesitan `sudo`, y hay una condición previa: en contenedores y otros entornos que no arrancan con systemd, `systemctl` simplemente no funciona — comprueba tu entorno antes de empezar.

## Para quién es

Ingenieros de operaciones (SRE/DevOps), desarrolladores backend y cualquiera que «reinicia el servidor entero» cuando un servicio no arranca. Si la diferencia entre `enable` y `start`, cómo filtrar registros por periodo o cómo sustituir cron por temporizadores te obliga a buscarlo cada vez, esta hoja te lo deja claro de una vez.

## Cómo usarla

1. Ve directo a **[CHEATSHEET.md](CHEATSHEET.md)**: el cuerpo de la referencia; los comandos en inglés los entiende cualquier desarrollador.
2. Necesitas una Linux arrancada con systemd (la inmensa mayoría de las distribuciones actuales). En contenedores y en el WSL por defecto casi nada funciona: practica en una máquina virtual o un servidor en la nube.
3. Sigue el orden de la hoja —servicios → registros → unidades → temporizadores → estado del sistema— y prueba cada comando a mano. En una hora los dominas.

## Pruébalo en 30 segundos

```bash
systemctl --version                # imprime la versión (p. ej. systemd 255): la herramienta está
systemctl is-system-running          # estado global; salidas posibles:
#   running   = todo levantado, normal
#   degraded  = levantado, pero con unidades fallidas (mira cuáles con systemctl --failed)
#   starting  = aún arrancando; maintenance = modo de mantenimiento/rescate; offline = el gestor no corre
systemd-analyze                      # tiempo total de arranque — funciona en tu máquina, sin sudo
ps -p 1 -o comm=                     # debe imprimir systemd; en contenedores suele ser otro nombre
```

## Un unit .service listo para usar

```ini
[Unit]
Description=MyApp demo service
After=network.target
Wants=network.target

[Service]
Type=simple
ExecStart=/usr/bin/sleep infinity
Restart=on-failure
User=www-data

[Install]
WantedBy=multi-user.target
```

Cambia `ExecStart` por tu programa, guárdalo como `/etc/systemd/system/myapp.service` y con `sudo systemctl daemon-reload && sudo systemctl enable --now myapp` ya está en marcha. Es el mismo archivo que la hoja disecciona en la sección de unidades, validado con `systemd-analyze verify`.

## Qué incluye

| Categoría | Contenido |
|---|---|
| 🎛️ Servicios (systemctl) | status / start·stop / restart frente a reload / enable solo toca el arranque / --now en un paso / consultas is-* / mask / daemon-reload (8) |
| 🔍 Registros (journalctl) | -u por servicio / -f en vivo / -b por arranque / ventana --since / prioridad -p / uso de disco y limpieza vacuum (6) |
| 📄 Archivos unit | cat con el resultado fusionado / rutas y precedencia / anatomía de un .service / drop-ins con edit / show / list-dependencies / --failed y reset-failed (7) |
| ⏰ Temporizadores | list-timers / anatomía de un .timer / verificar con calendar / el error de habilitar el servicio en vez del timer / disparadores relativos (5) |
| 🧰 Estado del sistema | analyze·blame / set-default / reboot·poweroff e isolate / el trío hostnamectl (4) |

La documentación oficial es solo https://www.freedesktop.org/software/systemd/man/latest/ ; cada comando está verificado contra la salida de ayuda de systemd 255, y las unidades y expresiones de calendario de ejemplo se ejecutaron de verdad.

## Contribuciones bienvenidas

¿Un comando incorrecto, un parámetro desactualizado o una traducción mejorable? Los *pull requests* son bienvenidos. Mantén el formato (descripción / bloque bash / etiquetas) y **solo comandos reales y verificados; ante la duda, mejor no incluirlo**.

## 📚 Más hojas de referencia

Otras hojas del mismo autor, también en 中文 / Español / Português:

- [FFmpeg 命令速查表](https://github.com/Ezra-Zhao/ffmpeg-cheatsheet) — 28 comandos de audio y vídeo
- [Git 实战速查表](https://github.com/Ezra-Zhao/git-cheatsheet) — 30 comandos diarios y de rescate
- [Docker 命令速查表](https://github.com/Ezra-Zhao/docker-cheatsheet) — 28 comandos de contenedores
- [curl 命令速查表](https://github.com/Ezra-Zhao/curl-cheatsheet) — 29 comandos para depurar APIs
- [正则表达式速查表](https://github.com/Ezra-Zhao/regex-cheatsheet) — 30 patrones prácticos de expresiones regulares
- [jq 命令速查表](https://github.com/Ezra-Zhao/jq-cheatsheet) — 30 filtros para JSON
- [终端神器速查表](https://github.com/Ezra-Zhao/terminal-superpowers-cheatsheet) — 14 herramientas de eficiencia para la terminal
- [Adobe 免费替代速查表](https://github.com/Ezra-Zhao/adobe-free-alternatives-cheatsheet) — 16 alternativas gratuitas a Adobe
- [退订省钱速查表](https://github.com/Ezra-Zhao/desubscribe-cheatsheet) — 10 sustituciones de suscripciones, ≈ $1,260/año de ahorro
- [kubectl 命令速查表](https://github.com/Ezra-Zhao/kubectl-cheatsheet) — comandos habituales de Kubernetes
- [SQL 查询速查表](https://github.com/Ezra-Zhao/sql-cheatsheet) — consultas y funciones SQL habituales
- [SSH 命令速查表](https://github.com/Ezra-Zhao/ssh-cheatsheet) — 28 comandos SSH

## Licencia

MIT © 2026 Guangyi Zhao. Ver [LICENSE](LICENSE).
