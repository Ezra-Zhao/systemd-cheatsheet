# systemd Cheatsheet · Cola de referência de comandos systemd

> 🌍 [中文](README.md) · [Español](README.es.md)

Uma **cola de uma página só** com 30 usos reais do systemd: gestão de serviços com `systemctl`, logs com `journalctl`, arquivos unit, temporizadores e estado do sistema. Cada entrada responde a apenas três perguntas: **o que ela faz, um comando pronto para copiar e colar, e etiquetas de uso**. Sem parâmetros inventados: cada comando e opção foi conferido na saída de `--help` do systemd 255, as duas unidades de exemplo foram validadas com `systemd-analyze verify` e as expressões de calendário foram executadas com `systemd-analyze calendar`. A maioria dos comandos de gestão precisa de `sudo`, e há uma condição prévia: em contêineres e outros ambientes que não inicializam com systemd, o `systemctl` simplesmente não funciona — confira seu ambiente antes de começar.

## Para quem é

Engenheiros de operações (SRE/DevOps), desenvolvedores backend e qualquer pessoa que «reinicia o servidor inteiro» quando um serviço não sobe. Se a diferença entre `enable` e `start`, como filtrar logs por período ou como trocar o cron por temporizadores te trava, esta cola resolve isso de uma vez.

## Como usar

1. Vá direto ao **[CHEATSHEET.md](CHEATSHEET.md)**: o corpo da referência; os comandos em inglês qualquer desenvolvedor entende.
2. Você precisa de um Linux inicializado com systemd (a imensa maioria das distribuições atuais). Em contêineres e no WSL padrão quase nada funciona: pratique numa máquina virtual ou num servidor na nuvem.
3. Siga a ordem da cola —serviços → logs → unidades → temporizadores → estado do sistema— e teste cada comando na mão. Em uma hora você domina tudo.

## Teste em 30 segundos

```bash
systemctl --version                # imprime a versão (ex.: systemd 255): a ferramenta está lá
systemctl is-system-running          # estado global; saídas possíveis:
#   running   = tudo no ar, normal
#   degraded  = no ar, mas com unidades falhas (veja quais com systemctl --failed)
#   starting  = ainda inicializando; maintenance = modo de manutenção/resgate; offline = o gerenciador não está rodando
systemd-analyze                      # tempo total de inicialização — funciona na sua máquina, sem sudo
ps -p 1 -o comm=                     # deve imprimir systemd; em contêineres costuma ser outro nome
```

## Um unit .service pronto para usar

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

Troque `ExecStart` pelo seu programa, salve como `/etc/systemd/system/myapp.service` e com `sudo systemctl daemon-reload && sudo systemctl enable --now myapp` ele já está rodando. É o mesmo arquivo que a cola disseca na seção de unidades, validado com `systemd-analyze verify`.

## O que está incluído

| Categoria | Conteúdo |
|---|---|
| 🎛️ Serviços (systemctl) | status / start·stop / restart vs. reload / enable só mexe na inicialização / --now em um passo / consultas is-* / mask / daemon-reload (8) |
| 🔍 Logs (journalctl) | -u por serviço / -f ao vivo / -b por inicialização / janela --since / prioridade -p / uso de disco e limpeza vacuum (6) |
| 📄 Arquivos unit | cat com o resultado mesclado / caminhos e precedência / anatomia de um .service / drop-ins com edit / show / list-dependencies / --failed e reset-failed (7) |
| ⏰ Temporizadores | list-timers / anatomia de um .timer / verificar com calendar / o erro de habilitar o serviço em vez do timer / gatilhos relativos (5) |
| 🧰 Estado do sistema | analyze·blame / set-default / reboot·poweroff e isolate / o trio hostnamectl (4) |

A documentação oficial é só https://www.freedesktop.org/software/systemd/man/latest/ ; cada comando foi verificado contra a saída de ajuda do systemd 255, e as unidades e expressões de calendário de exemplo foram executadas de verdade.

## Contribuições são bem-vindas

Achou um comando errado, um parâmetro desatualizado ou uma tradução a melhorar? *Pull requests* são bem-vindos. Mantenha o formato (descrição / bloco bash / etiquetas) e **só comandos reais e verificados; na dúvida, melhor não incluir**.

## 📚 Mais folhas de referência

Outras colas do mesmo autor, também em 中文 / Español / Português:

- [FFmpeg 命令速查表](https://github.com/Ezra-Zhao/ffmpeg-cheatsheet) — 28 comandos de áudio e vídeo
- [Git 实战速查表](https://github.com/Ezra-Zhao/git-cheatsheet) — 30 comandos do dia a dia e de resgate
- [Docker 命令速查表](https://github.com/Ezra-Zhao/docker-cheatsheet) — 28 comandos de contêineres
- [curl 命令速查表](https://github.com/Ezra-Zhao/curl-cheatsheet) — 29 comandos para depurar APIs
- [正则表达式速查表](https://github.com/Ezra-Zhao/regex-cheatsheet) — 30 padrões práticos de expressões regulares
- [jq 命令速查表](https://github.com/Ezra-Zhao/jq-cheatsheet) — 30 filtros para JSON
- [终端神器速查表](https://github.com/Ezra-Zhao/terminal-superpowers-cheatsheet) — 14 ferramentas de eficiência para o terminal
- [Adobe 免费替代速查表](https://github.com/Ezra-Zhao/adobe-free-alternatives-cheatsheet) — 16 alternativas gratuitas ao Adobe
- [退订省钱速查表](https://github.com/Ezra-Zhao/desubscribe-cheatsheet) — 10 trocas de assinaturas, ≈ $1,260/ano de economia
- [kubectl 命令速查表](https://github.com/Ezra-Zhao/kubectl-cheatsheet) — comandos habituais do Kubernetes
- [SQL 查询速查表](https://github.com/Ezra-Zhao/sql-cheatsheet) — consultas e funções SQL habituais
- [SSH 命令速查表](https://github.com/Ezra-Zhao/ssh-cheatsheet) — 28 comandos SSH

## Licença

MIT © 2026 Guangyi Zhao. Veja [LICENSE](LICENSE).
