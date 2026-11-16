# Fazendo root numa TV LG com o SlopBro (antes dos outros docs `webos/*-*.md`)

Esse é o passo anterior a [`root-autostart.md`](root-autostart.md) e
[`magic-mapper-buttons.md`](magic-mapper-buttons.md): os dois assumem que a TV já está com root e com o Homebrew
Channel instalado. Nada disso faz parte do Free My TV em si — é um exploit de terceiros, usado por sua conta e risco,
na sua própria TV. Leia também as orientações de root do próprio
[webosbrew.org](https://webosbrew.org) antes de fazer isso numa TV que importa para você.

## 1. Confira se a TV é de fato explorável

Vá em **[cani.rootmy.tv](https://cani.rootmy.tv)** e digite o modelo exato da TV e a versão do firmware (na TV:
Settings → General → System → About This TV, ou About → check for updates). O site lista todo exploit conhecido para
cada combinação de modelo/firmware do webOS, marcado como corrigido ou não. O SlopBro é o documentado aqui; se o
checador mostrar ele (ou qualquer outro exploit) como não corrigido na sua versão exata, está valendo. Se todos
aparecerem corrigidos, não há como entrar nesse firmware — não tente. Uma atualização de software pode corrigir o
exploit, então confira o firmware atual, não um que você lembra de uma versão antiga.

## 2. Pegue o SlopBro e rode

[throwaway96/slopbro](https://github.com/throwaway96/slopbro) (AGPL-3.0, sem relação com este projeto). Precisa de
Python 3.8+, do IP da TV, e da TV e do seu computador conseguirem se alcançar nos dois sentidos na rede.

```bash
git clone https://github.com/throwaway96/slopbro.git
cd slopbro
python3 slopbro.py <TV_IP>
```

A TV mostra um pedido de pareamento — aceite. O script então serve uma pequena página de exploit para a TV pela sua
rede local; ela baixa um pacote falso de serviço de sistema e o roda como root, o que instala o webOS Homebrew Channel
e deixa o root persistente. O progresso e os erros aparecem no terminal; `--debug` adiciona mais detalhes, escritos
na própria TV em `/tmp/autoroot.log`. O script em si não precisa de um reboot para terminar, mas ele termina
perguntando (como um alerta na tela da TV) se quer reiniciar agora — aceite, já que o Homebrew Channel só "aparece"
de verdade depois de um reboot.

## 3. Antes de reiniciar: desinstale o app Developer Mode da própria LG, se estiver na TV

Isso nos pegou na primeira tentativa: com o app **Developer Mode** da LG ainda instalado, o root não pegava direito
(ou parecia não ter pegado). O próprio script do SlopBro detecta isso e se recusa a terminar de deixar o root
persistente enquanto encontra esse app instalado — remova-o da lista de apps da TV antes de rodar o SlopBro (ou antes
do reboot final, se você já rodou com o Developer Mode ainda ligado), e então reinicie.

## 4. Depois de reiniciar: confirme que funcionou

O **Homebrew Channel** deve aparecer agora na lista de apps da TV. Abra-o uma vez — é assim também que se liga o
Telnet ou o SSH (nas configurações dele), que os comandos dos outros dois docs precisam. **O Telnet não tem senha
nenhuma** (confirmado nesta TV: ele simplesmente te joga direto num shell root) — desligue-o de novo assim que
terminar de usá-lo na sessão. O pessoal do Homebrew Channel diz que o SSH usa por padrão a senha de root `alpine`, a
menos que você tenha adicionado sua própria chave em `/home/root/.ssh/authorized_keys`, mas este projeto nunca chegou
a usar SSH nesta TV (só Telnet), então trate esse detalhe como de segunda mão, não como algo verificado aqui. Veja a
própria documentação do `webosbrew.org` para saber exatamente onde ficam essas opções na tela de configurações do
Homebrew Channel, já que o texto exato muda entre versões.

## Quando terminar de configurar tudo: desligue o Telnet e o SSH de novo

Depois que o `root-autostart.md` e o `magic-mapper-buttons.md` (ou qualquer outra coisa) estiverem no lugar e
funcionando, volte em **Homebrew Channel → Settings** e desligue tanto o **Telnet** quanto o **SSH**. Nenhum dos dois
precisa ficar ligado: o script de autostart e o Magic Mapper já estão salvos no próprio disco da TV e continuam
rodando (e se reiniciando sozinhos) por conta própria — eles não dependem de nada externo, nem precisam de uma porta
aberta na TV para continuar funcionando. O Telnet, em especial, não tem senha nenhuma, então qualquer outra pessoa no
mesmo Wi-Fi poderia conseguir um shell root na TV enquanto ele estiver ligado. Ligue de novo só na próxima vez que
precisar mesmo mudar algo por ele.

## Ressalvas

- Não reinstale o app **Developer Mode** da própria LG depois de ter feito root — os dois não convivem bem juntos
  (veja o passo 3).
- Uma atualização de software da LG, ou um reset de fábrica, remove o root — rodar o SlopBro de novo (depois de
  conferir de novo no cani.rootmy.tv, caso essa atualização também tenha corrigido o exploit) é como recuperá-lo.
- O próprio README do SlopBro diz que não se espera que ele funcione em firmwares webOS 10/11 lançados depois que foi
  corrigido (por volta de outubro de 2026); para webOS 3.5/4.x mais antigos, o autor aponta para outra ferramenta, o
  [dejavuln-autoroot](https://github.com/throwaway96/dejavuln-autoroot).
- Fazer root anula qualquer suporte "normal" que a LG considere válido para a TV, e carrega um risco real, ainda que
  pequeno, de deixar a TV inutilizável se algo der errado no meio do processo (uma queda de energia durante o breve
  passo de instalação do root, por exemplo). Funcionou direitinho na TV contra a qual os docs deste projeto que
  dependem de root foram escritos (uma LG C1, webOS 6.5.3), mas isso é um único caso, não uma garantia para a sua.
