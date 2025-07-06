# VNC - Configurando um servidor em um Ubuntu Desktop X11/Xorg

## Contexto

Eu tenho um Ubuntu Desktop (com ambiente gráfico rodando sobre o **Xorg**) que desejo acessar remotamente via VNC.

Para resolver isso irei instalar e configurar um serviço de vnc server neste ubuntu

## Pré-requisitos

### Firewall

Caso o servidor use um firewall, você deve liberar a porta 5900 (que é a padrão do vnc)

```bash
# CASO VC USE O FIREWALLD

# Adicione a regra no firewalld
sudo firewall-cmd --permanent --add-port=5900/tcp

# Recarregue o firewalld
sudo firewall-cmd --reload

# Teste se funcionou
sudo firewall-cmd --list-ports
```

### Possuir o pacote netstat instalado

```bash
sudo apt -y update
sudo apt -y install net-tools
```

## Instalando o servidor VNC

```bash
# Instala o pacote
sudo apt install x11vnc -y

# Define a senha
x11vnc -storepasswd ~/.vnc/passwd
chmod 600 ~/.vnc/passwd
```

## Criando um serviço para o servidor

**Importante:** O arquivo abaixo, parte do pressuposto que o usuário se chama *anderson*. Se o nome do usuário for diferente, deve-se alterar a string `/home/anderson` nas linhas do "ExecStart" e "Environment"

**Path do Arquivo:**
`/etc/systemd/system/x11vnc.service`

**Conteúdo do arquivo:**

```txt
[Unit]
Description=Start x11vnc at startup.
After=multi-user.target

[Service]
Type=simple
User=anderson
ExecStart=/usr/bin/x11vnc -auth guess -forever -loop -noxdamage -repeat -rfbauth /home/anderson/.vnc/passwd -rfbport 5900 -shared -ncache 10
Restart=always
Environment="DISPLAY=:0"
Environment="XAUTHORITY=/home/anderson/.Xauthority"

[Install]
WantedBy=multi-user.target
```

**Opções:**

```txt
-auth guess - Tenta adivinhar o local do .Xauthority
-forever- Mantém o servidor rodando mesmo após desconexão
-loop - Reconecta automaticamente
-noxdamage - Melhora desempenho em ambientes gráficos simples
-rfbport 5900 - Porta padrão do VNC
-shared - Permite múltiplas conexões simultâneas
-ncache - Fator n vezes de cache(framebuffer) do lado cliente
DISPLAY=:0 - Conecta-se à sessão gráfica primária
XAUTHORITY=... - Local do arquivo de autorização do Xorg
```

**Ativando e Reiniciando o serviço:**

```bash
# Recarregue os serviços do systemd
sudo systemctl daemon-reexec
sudo systemctl daemon-reload

# Ative o serviço para iniciar na inicialização:
sudo systemctl enable x11vnc

# Inicie o serviço imediatamente:
sudo systemctl start x11vnc

# Ver o status do serviço
sudo systemctl status x11vnc
```

## Acessando o servidor através do computador cliente

**Comando:**

```bash
vncviewer <ip_do_servidor>:5900
```

**Opcional:** Para maior segurança, você pode se conectar usando SSH Tunneling, encapsulando a conexão VNC via SSH:

Em um terminal do computador cliente, abra o túnel ssh:

```bash
ssh -L 5900:localhost:5900 seu_usuario@ip_do_servidor
```

ainda no computador cliente, abra um segundo terminal, e se conecte localmente:

```bash
vncviewer localhost:5900
```

## Debugando

Veja o log do serviço através do comando

```bash
sudo systemctl status x11vnc
```

**Curiosidade:** Além de erros e warning o log pode mostrar dicas. Por exemplo: Eu descobri sobre o parâmetro **ncache** porque apareceu uma mensagem no log sugerindo usá-lo

```txt
jul 04 17:00:38 andersontj x11vnc[496104]: 04/07/2025 17:00:38
jul 04 17:00:38 andersontj x11vnc[496104]: The VNC desktop is:      andersontj:0
jul 04 17:00:38 andersontj x11vnc[496104]: PORT=5900
jul 04 17:00:38 andersontj x11vnc[496104]: ************************************>
jul 04 17:00:38 andersontj x11vnc[496104]: Have you tried the x11vnc '-ncache' >
jul 04 17:00:38 andersontj x11vnc[496104]: The scheme stores pixel data offscre>
jul 04 17:00:38 andersontj x11vnc[496104]: retrieval.  It should work with any >
jul 04 17:00:38 andersontj x11vnc[496104]:     x11vnc -ncache 10 ...
jul 04 17:00:38 andersontj x11vnc[496104]: One can also add -ncache_cr for smoo>
jul 04 17:00:38 andersontj x11vnc[496104]: More info: http://www.karlrunge.com/>
lines 1-22/22 (END)...skipping...
```
