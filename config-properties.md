# O que tem no `config.properties`

Esse arquivo é onde ficam as configurações de cada instalação: a cidade, a chave do TMDB, a ordem da prateleira,
coisas assim. Ele não vai para o Git (está no `.gitignore`) porque carrega dados que são só seus — principalmente a
chave do TMDB, que é pessoal e gratuita. `build.sh` (Android/Google TV) e `webos/build.py` (esta TV, LG) leem o
mesmo arquivo; `tizen/build.py` (Samsung) também. Ele é criado automaticamente, vazio, na primeira vez que você roda
qualquer um desses builds — é só abrir e preencher.

A maioria das opções abaixo também pode ser trocada depois, direto na TV, em *Home Settings*: o que você escolhe lá
sempre tem prioridade sobre o que está aqui. `config.properties` é só o ponto de partida.

Nem toda opção vale para toda TV — cada uma diz entre parênteses onde funciona.

## As que usamos nesta TV

- **`tmdb_key`** — a chave gratuita do TMDB (themoviedb.org/settings/api, o "API Read Access Token" ou a "API Key").
  Sem ela, a tela inicial funciona normalmente, só sem as sugestões de conteúdo em alta. É a única informação
  realmente secreta deste arquivo — nunca compartilhe a sua.
- **`tmdb_region`** — seu país, em duas letras (`BR`, `US`, `PT`...). Decide quais títulos aparecem em qual app de
  streaming.
- **`latitude`** / **`longitude`** — coordenadas decimais da sua cidade, para a previsão do tempo e para saber a
  hora do pôr do sol (que deixa a tela inicial escura à noite, se o tema estiver em "auto").
- **`city`** — o nome mostrado no card do tempo (opcional; vazio mostra só "Weather"/"Clima").
- **`shelf_apps`** *(LG e Samsung)* — a ordem dos apps na prateleira, separados por vírgula, pelo nome ou parte dele
  (sem diferenciar maiúsculas/minúsculas). Foi assim que fixamos a ordem Crunchyroll, YouTube, Netflix, Prime Video,
  Max, Apple TV, Disney+, Stremio nesta TV. Muda uma vez só, aqui; depois disso, o que for escolhido na TV (Home
  Settings) é o que vale.
- **`theme`** — o visual da tela inicial: `auto` (escuro do pôr ao nascer do sol), `light` ou `dark`.
- **`show_crypto`** — `true` mostra o preço de criptomoedas (via Binance) na tela inicial e na proteção de tela.
  Desligamos essa opção aqui.
- **`currency`** — em qual moeda mostrar os preços de cripto (`BRL`, `USD`...; só importa com `show_crypto=true`).

## As outras opções (não usamos, mas existem)

- **`hide_animation`** — `true` tira desenhos/anime (gênero "Animation" do TMDB) das sugestões.
- **`open_in_stremio`** / **`open_in_nuvio`** — `true` faz uma sugestão abrir direto na página do título dentro do
  Stremio ou do Nuvio, em vez de abrir o app de streaming.
- **`language`** — `auto` (segue o idioma da TV) ou um código da pasta `app/assets/i18n` (`en`, `pt`, `es`...).
- **`temperature`** — `C`, `F` ou `auto` (Fahrenheit quando a TV está configurada para os EUA).
- **`clock_format`** *(LG e Samsung)* — `24` ou `12` força o formato do relógio; `auto` segue o idioma. *Home
  Settings* > *Clock*, na TV, tem prioridade.
- **`disable_assistant`** *(Android/Google TV)* — `true` também desliga o Google Assistant; o microfone do controle
  para de funcionar.
- **`screensaver`** *(Android/Google TV)* — `false` mantém a proteção de tela original da TV (o Free My TV muda só a
  tela inicial e os apps de promoção).
- **`hide_services`** — serviços de streaming para tirar das sugestões mesmo com o app instalado (`netflix`, `prime`,
  `disney`...), separados por vírgula. Serve para apps que a TV já vem com instalado mas você não assina.
- **`hide_apps`** *(LG e Samsung)* — apps para tirar da prateleira, do "Todos os apps" e das sugestões, por id ou
  nome, separados por vírgula.
- **`open_picks_in`** *(LG)* — onde uma sugestão abre: `apps` (o próprio app de streaming) ou `nuvio` (a página do
  título dentro do Nuvio).
- **`watchmode_key`** — chave gratuita do Watchmode (watchmode.com); faz as sugestões abrirem a página do próprio
  título dentro do app (quando o app suporta), em vez de só abrir o app. Google TV, Android TV e, no Samsung, Prime
  Video.
- **`drawer_first`** *(LG e Samsung)* — `true` abre a tela inicial já com a gaveta de apps/entradas aberta.
- **`samsung_home_button`** *(Samsung, Tizen 3+)* — `true` faz o Free My TV voltar quando o botão Home é apertado
  dentro de um app aberto por ele. Só no seu próprio build (muda o manifesto do pacote).
- **`plex_url`** / **`plex_token`** *(Google TV)* — endereço e token do seu servidor Plex, para os lançamentos dele
  entrarem nas sugestões. O token abre sua conta Plex inteira — prefira o de um usuário gerenciado do Plex Home, e
  nunca compartilhe o seu.
- **`trakt_client_id`** *(Google TV)* — o Client ID do seu próprio app no Trakt (trakt.tv), para assinar e sincronizar
  o que você já assistiu e sua watchlist. Só o Client ID vai no build; nenhum segredo.
- **`immich_server`** — endereço do seu servidor Immich, para "Minhas Fotos" (tela de proteção e fundo da tela
  inicial) usar fotos de verdade. A chave **não** entra aqui — ela vai direto para a TV com
  `python3 tools/send_key.py <IP da TV> immich_api_key`, nunca pelo `config.properties` (que vira parte do app, que
  qualquer outro app na TV pode ler).
- **`home_assistant_url`** / **`home_assistant_entities`** *(Google TV)* — endereço do seu Home Assistant e até 8
  entidades (luzes, interruptores, cenas...) para botões na gaveta inferior. O token também não entra aqui — mesma
  lógica do Immich, via `python3 tools/send_key.py <IP da TV> home_assistant_token`.

## O padrão para chaves que abrem seus próprios servidores

Toda chave que dá acesso a algo seu — Immich, Home Assistant — segue a mesma regra: nunca vai em `config.properties`,
porque esse arquivo vira parte do pacote instalado, que qualquer app na própria TV pode ler. Em vez disso, ela é
enviada direto para a TV já instalada, por `adb`/SSH, com `python3 tools/send_key.py <IP> <nome_da_chave>` — pedida
na hora, nunca digitada em uma conversa ou guardada em texto.
