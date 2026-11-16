# Remapeando o Home e um botão colorido do controle (TVs LG com root)

Mais uma configuração opcional, do lado da TV, para quem **fez root na própria TV LG** (veja
[`root-autostart.md`](root-autostart.md) para o script de autostart que isso complementa). Não faz parte do app, não
é construído nem instalado pelo `webos/build.py`/`webos/install.py`, e vive inteiramente no sistema de arquivos da
TV através de uma ferramenta de terceiros — documentado aqui só para a configuração sobreviver mesmo que seja
esquecida.

## O que faz

- O botão **Home** do controle abre o **Free My TV** diretamente, em vez do launcher da própria LG.
- O botão **vermelho** do controle abre o launcher da própria LG (`com.webos.app.home`) — um jeito de reserva de
  voltar para ele, já que o Home deixou de fazer isso.
- Todo o resto do controle (setas, OK, Voltar, números, botões de apps, volume etc.) continua funcionando exatamente
  como antes.

## A ferramenta: Magic Mapper

[andrewfraley/magic_mapper](https://github.com/andrewfraley/magic_mapper) (não faz parte deste projeto, licença MIT,
precisa de root). Ela toma o dispositivo de entrada do Magic Remote de forma exclusiva, reenvia todo botão que não
foi mandado remapear para um segundo dispositivo (para a TV continuar vendo eles), e roda uma pequena ação
configurada em JSON para os botões que foram mandados remapear.

**Por que essa e não o [LG Input Hook](https://repo.webosbrew.org/apps/org.webosbrew.inputhook/)**: o LG Input Hook
(o app do Homebrew Channel, documentado como opção geral) foi tentado primeiro — ele *detectava* corretamente todo
aperto de botão (visível nos próprios logs dele), mas as ações "Launch App" e "Execute" nunca chegavam a rodar nada,
para nenhum botão testado (Home, e o botão colorido secundário da própria TV), nesse firmware exato desta TV. Ler
direto o dispositivo de entrada bruto do kernel para o controle (`/dev/input/event*`) foi tentado em seguida e não
capturou nada, em nenhum dos 32 nós de dispositivo, em várias tentativas — o Magic Remote não aparece por uma leitura
simples de `cat`/`od` do jeito que um teclado USB comum apareceria. O próprio código Python do Magic Mapper abre o
mesmo dispositivo (`LGE M-RCU - Builtin [0]`, `/dev/input/event3` nesta TV) usando o tratamento evdev do Python (com
o ioctl de captura exclusiva), que é o que de fato enxerga os apertos de botão; uma leitura simples sem isso não
enxerga.

## Instalando

Com Telnet/SSH de root ligado (veja as instruções do `root-autostart.md` para ligar), na TV:

```sh
curl -sL -o /home/root/magic_mapper.py "https://raw.githubusercontent.com/andrewfraley/magic_mapper/main/magic_mapper.py"
curl -sL -o /var/lib/webosbrew/init.d/start_magic_mapper "https://raw.githubusercontent.com/andrewfraley/magic_mapper/main/start_magic_mapper"
chmod +x /home/root/magic_mapper.py /var/lib/webosbrew/init.d/start_magic_mapper
```

A configuração (na mesma pasta do `/home/root/magic_mapper.py` — ou seja, `/home/root/magic_mapper_config.json`)
precisa ser escrita em **uma linha só**: a sessão Telnet interativa desta TV descartava linhas de um `cat << 'EOF'`
em várias linhas colado de uma vez (provável causa da primeira tentativa não fazer nada, silenciosamente), enquanto
um `echo '...' > arquivo` em uma linha só funcionou de forma confiável:

```sh
echo '{"magic_mapper_settings":{"block_mouse":false,"exclusive_mode":true,"ir_input_enabled":false,"ir_exclusive_mode":true},"home":{"function":"launch_app","inputs":{"app_id":"io.freemytv"}},"red":{"function":"launch_app","inputs":{"app_id":"com.webos.app.home"}}}' > /home/root/magic_mapper_config.json
```

Depois, inicie (`/var/lib/webosbrew/init.d/start_magic_mapper`, já no lugar, também roda isso a cada boot de verdade
— veja a nota do `root-autostart.md` sobre o que conta como boot de verdade versus acordar do modo de espera, que
não se aplica aqui já que o Magic Mapper não precisa capturar nada de novo ao acordar, só quando o processo começa
do zero):

```sh
/var/lib/webosbrew/init.d/start_magic_mapper start    # ou: restart, depois de editar a configuração
```

## Conferindo

```sh
cat /tmp/magic_mapper.log       # quais dispositivos de entrada/saída ele resolveu, e a ação de cada botão
ps -ef | grep magic             # o shell do loop de reinício e o magic_mapper.py rodando
```

Um log saudável termina com `Input loop started, waiting for button presses on 1 device(s)`.

## Mudando o mapeamento

Edite `/home/root/magic_mapper_config.json` (uma linha só, como acima — adicione mais nomes de botões como mais
chaves de nível superior, cada um `{"function": "launch_app", "inputs": {"app_id": "<id do app>"}}` para abrir um
app, ou veja as docstrings do próprio script para suas outras funções), depois `start_magic_mapper restart`. Nomes de
botões que o Magic Mapper conhece (do próprio código-fonte dele): `home`, `back`, `ok`, `up`/`down`/`left`/`right`,
`red`/`green`/`yellow`/`blue`, teclas numéricas `0`-`9`, `guide`, `info`, `settings`, `search`, `voice`, os botões de
apps de serviço (`netflix`, `prime`, `disney`, `alexa`, `lg_channels`, …) e mais alguns — `grep -n '": "'
magic_mapper.py` perto do `KEYCODE_MAP` dele lista todos.

Quando isso estiver no lugar e funcionando, veja a nota do `rooting-slopbro.md` sobre desligar o Telnet/SSH de novo —
o Magic Mapper roda inteiramente por conta própria a partir daqui, e não precisa de nenhum dos dois ligado.

## Ressalvas

- **`exclusive_mode: true`** (o padrão, mantido aqui) significa que o próprio processo do Magic Mapper tem que ser
  quem reenvia todo botão não mapeado; se ele travar ou for parado sem reiniciar, o controle pode parar de responder
  até ser iniciado de novo (`start_magic_mapper start`) ou a TV reiniciar. O loop `start_magic_mapper run` o reinicia
  automaticamente depois de uma travada, com um intervalo crescente depois de travadas repetidas e rápidas — veja os
  próprios comentários do script.
- Uma atualização de software da LG, um reset de fábrica, ou fazer root de novo na TV remove isso — reinstale do
  mesmo jeito depois.
- Sem relação com o script do `root-autostart.md`, que continua, separadamente, garantindo que o Free My TV é o que
  aparece na tela depois de um reboot de verdade ou de acordar do modo de espera; este aqui só muda o que dois
  botões do controle fazem enquanto a TV já está ligada.
