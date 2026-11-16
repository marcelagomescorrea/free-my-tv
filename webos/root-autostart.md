# Fazendo o Free My TV abrir sozinho (TVs LG com root)

Essa é uma configuração opcional, do lado da TV, para quem **fez root na própria TV LG** (por exemplo com o
[SlopBro](https://github.com/throwaway96/slopbro), que instala o
[webOS Homebrew Channel](https://github.com/webosbrew/webos-homebrew-channel)). Não faz parte do app, não é
construído nem instalado pelo `webos/build.py` ou `webos/install.py`, e não tem nada a ver com o código do próprio
projeto — é um pequeno script shell colocado direto no sistema de arquivos root da TV, usando um mecanismo que o
próprio Homebrew Channel já oferece. Está documentado aqui só para a configuração sobreviver mesmo que seja
esquecida, e para que outra TV LG com root possa repetir o mesmo processo.

**Fazer root numa TV é decisão e risco seus**, sem relação com este projeto; veja a documentação do próprio Homebrew
Channel e o [cani.rootmy.tv](https://cani.rootmy.tv) para saber se seu modelo e firmware podem ser rooteados.

## O que faz

Com esse script no lugar, a TV abre direto no Free My TV:

- **Em todo reboot de verdade** (uma atualização de software, uma travada, ou *Settings → General → Reboot*).
- **Ao acordar do modo de espera** (desligar e ligar a TV de novo pelo controle) — esse é o caso comum, do dia a dia.
  O webOS nunca reinicia a TV nesse caso, então um script que só roda na inicialização (a abordagem óbvia) nem chega
  a rodar aqui.

Os dois casos batem na mesma disputa de fundo: por um tempo depois que a tela volta (seja de um boot de verdade ou do
modo de espera), o próprio firmware da TV insiste em colocar a **Live TV** na frente, mesmo depois do Free My TV já
ter sido aberto uma vez. O script abaixo continua reabrindo o Free My TV — mas só enquanto outra coisa ainda está em
primeiro plano, então ele nunca briga com o Free My TV depois que ele já venceu de verdade, que foi o que causava a
prateleira piscar numa versão anterior deste script (cada chamada de reabertura reseta o foco da página).

## O script

Fica em `/var/lib/webosbrew/init.d/freemytv-autostart` na TV (não está neste repositório — nada aqui constrói ou
instala ele). O próprio script de inicialização do Homebrew Channel roda todo arquivo executável dessa pasta uma vez,
bem no início de todo boot de verdade (veja o `run-parts /var/lib/webosbrew/init.d` no `startup.sh` dele), que é o
gancho do qual isso depende.

```sh
#!/bin/sh
LOG=/tmp/freemytv-autostart.log

# Garante que o Free My TV é o app em primeiro plano, tentando de novo a cada
# 3s por até 2 minutos. Não faz nada (sem piscar) se ele já está em primeiro
# plano, e para assim que isso acontece.
ensure_foreground() {
  i=0
  while [ "$i" -lt 40 ]; do
    fg=$(luna-send -n 1 -a com.webos.service.applicationmanager luna://com.webos.applicationManager/getForegroundAppInfo '{}' 2>&1)
    case "$fg" in
      *'"appId":"io.freemytv"'*)
        echo "[$(date)] ensure_foreground: already foreground (i=$i)" >> "$LOG"
        return 0
      ;;
    esac
    result=$(luna-send -n 1 -a com.webos.service.applicationmanager luna://com.webos.applicationManager/launch '{"id":"io.freemytv"}' 2>&1)
    echo "[$(date)] ensure_foreground attempt $i: fg=$fg launch=$result" >> "$LOG"
    i=$((i+1))
    sleep 3
  done
}

# Corrige a disputa logo depois do boot: a TV mostra a Live TV por um tempo
# mesmo depois da nossa primeira chamada já ter funcionado.
ensure_foreground

# Acordar do modo de espera (desligar/ligar a TV pelo controle) nunca a
# reinicia, então este script não roda de novo nesse caso -- mas a TV mostra a
# Live TV ali também. Isso observa o estado de energia da TV pelo resto dessa
# sessão de boot e roda a mesma correção de novo toda vez que ela volta a
# "Active".
(
  prevstate="unknown"
  luna-send -i luna://com.webos.service.tvpower/power/getPowerState '{"subscribe":true}' 2>/dev/null | while IFS= read -r line; do
    case "$line" in
      *processing*)
        : # meio da transição; ignora
      ;;
      *'"state":"Active"'*|*'"state": "Active"'*)
        if [ "$prevstate" = "Suspend" ]; then
          echo "[$(date)] woke from standby, re-checking" >> "$LOG"
          ensure_foreground
        fi
        prevstate="Active"
      ;;
      *'"state":"Suspend"'*|*'"state": "Suspend"'*)
        prevstate="Suspend"
      ;;
    esac
  done
) >> "$LOG" 2>&1 &

exit 0
```

A parte em segundo plano (o último `( ... ) &`) é o que cobre o caso do modo de espera: ela continua inscrita no
estado de energia do `com.webos.service.tvpower` por todo o tempo que a TV ficar ligada nesse boot, e só age nas
linhas já estabilizadas, `{"state":"Active"}` / `{"state":"Suspend"}` (não nas várias `"processing"` do meio do
caminho que um ciclo de energia de verdade também manda).

## Instalando

Com o SSH ou o Telnet do Homebrew Channel ligado (nas próprias *Settings* dele), a partir de um computador na mesma
rede:

```bash
ssh root@<IP da TV>          # ou: telnet <IP da TV>  (padrão do Homebrew Channel: sem senha no Telnet)
```

Então, na TV, escreva o script acima em `/var/lib/webosbrew/init.d/freemytv-autostart` e torne-o executável:

```sh
cat > /var/lib/webosbrew/init.d/freemytv-autostart << 'SCRIPTEOF'
# ...o script acima...
SCRIPTEOF
chmod +x /var/lib/webosbrew/init.d/freemytv-autostart
```

Passa a valer no próximo reboot de verdade
(`luna-send -n 1 -a com.webos.service.sleep luna://com.webos.service.sleep/shutdown/machineReboot '{"reason":"remoteKey"}'`,
ou *Settings → General → Reboot*). Para também proteger a sessão *atual* da TV sem reiniciá-la, rode só o bloco do
observador em segundo plano (a parte `( ... ) &`) na mão, pela mesma sessão SSH/Telnet, com `nohup` para sobreviver
ao fechamento da sessão:

```sh
nohup sh -c '... o corpo do bloco ( ... ), sem a indentação ...' >> /tmp/freemytv-autostart.log 2>&1 < /dev/null &
```

## Conferindo

```sh
cat /tmp/freemytv-autostart.log                       # toda tentativa e todo acordar do modo de espera, com hora
ps -ef | grep getPowerState                            # o observador em segundo plano, se estiver rodando
luna-send -n 1 -a com.webos.service.applicationmanager luna://com.webos.applicationManager/getForegroundAppInfo '{}'
```

Quando isso estiver no lugar e funcionando, veja a nota do `rooting-slopbro.md` sobre desligar o Telnet/SSH de novo —
este script não precisa de nenhum dos dois ligado para continuar funcionando.

## Ressalvas

- **Não mexa** em `/media/developer/apps/usr/palm/services/org.webosbrew.hbchannel.service/jumpstart.sh` nem em
  `/var/lib/webosbrew/startup.sh` — só adicione scripts dentro de `init.d/`, como o próprio aviso do Homebrew Channel
  no `jumpstart.sh` diz.
- Uma atualização de software da LG, um reset de fábrica, ou rodar o exploit de root de novo do zero remove isso (e o
  root junto) — reinstale o script do mesmo jeito depois.
- Isso só briga com a Live TV numa janela curta depois do boot/acordar (2 minutos para o loop de inicialização; o
  observador do modo de espera reage dentro de poucos segundos de ter acordado). Se você mesmo trocar
  deliberadamente para a Live TV durante essa janela, nada te impede — o loop só continua reabrindo o Free My TV
  enquanto *outra coisa* já está em primeiro plano, nunca o contrário.
