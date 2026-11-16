<div align="center">

# Free My TV — este fork

Um fork pessoal do **[simbasjungles/free-my-tv](https://github.com/simbasjungles/free-my-tv)**, personalizado para uma
TV LG webOS (uma LG OLED55C1PSA). Para saber o que o projeto em si é, como instalar, e tudo o mais sobre ele, veja
[o repositório original](https://github.com/simbasjungles/free-my-tv) e seu próprio README — este fork não muda nada
disso.

![A prateleira do Free My TV nesta TV, com os ícones personalizados](.github/preview-webos.png)

</div>

## O que é diferente aqui

Tudo abaixo é específico deste fork e desta TV; nada disso é comportamento do projeto original.

- **Os ícones da prateleira** de alguns apps (Netflix, YouTube, Crunchyroll, Stremio) foram substituídos ou
  adicionados, já que o ícone que a própria TV informava para alguns deles não carregava. Veja
  [`web/icons/SOURCES.md`](web/icons/SOURCES.md) para saber de onde cada um veio e por quê.
- **Uma correção de performance** (`web/tv.js`): TVs LG que informam 2 GB de memória agora recebem o mesmo modo de
  renderização sem sombras ("flat") que TVs Samsung com essa memória já tinham — sem isso, a prateleira visivelmente
  perdia quadros ao mover o cursor por ela.
- **A configuração desta TV** (`config.properties`, não versionado): ordem dos apps na prateleira, cidade e chave do
  TMDB.

Além disso, esta TV está **com root** (por conta e risco do próprio dono, sem relação com o Free My TV em si) para
algumas personalizações a mais, cada uma documentada em seu próprio arquivo:

- [`webos/rooting-slopbro.md`](webos/rooting-slopbro.md) — como fazer root na TV, com
  [SlopBro](https://github.com/throwaway96/slopbro).
- [`webos/root-autostart.md`](webos/root-autostart.md) — um script que abre o Free My TV sozinho, em todo reboot de
  verdade e toda vez que a TV acorda do modo de espera.
- [`webos/magic-mapper-buttons.md`](webos/magic-mapper-buttons.md) — remapear o botão **Home** do controle para abrir
  o Free My TV diretamente, com o botão **vermelho** como reserva para o launcher da própria LG, usando o
  [andrewfraley/magic_mapper](https://github.com/andrewfraley/magic_mapper).

## Licença

MIT, igual ao projeto original — veja [`LICENSE`](LICENSE).
