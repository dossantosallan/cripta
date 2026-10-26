# Modo Melhorado (v1.7)

Camada opcional de luz, som, sensação de combate, câmera, interface e fluidez. **Não altera regras do jogo**: dano, vida, velocidade, IA, drops, economia e chances continuam os mesmos. Desligado, o jogo se comporta como a v1.6.

Ponto de retorno: commit `61f483d` (v1.6) no `main`. Todo o trabalho está no branch `modo-melhorado`.

## Como ligar e desligar

| Forma | Efeito |
|---|---|
| `F8` | liga/desliga na hora, com aviso na tela |
| endereço terminado em `#melhorado` | abre o jogo já ligado (equivale a `--enhanced`) |
| `Esc` ou ⚙ → Configurações | interruptor geral e um interruptor por módulo |
| `F7` | painel de diagnóstico: qps, tempo do jogo por quadro, módulos um a um |
| `F9` | mede 30 s de jogo e mostra qps médio, 1% mais lento e tempo de CPU |
| `F4` | mostra contornos de colisão (paredes, herói, inimigos) |
| `Alt` (segurar) | mostra o nome de todos os itens no chão |

As escolhas ficam salvas no navegador (`cripta-prefs-v1`), separadas do progresso do altar.

## O que foi aplicado, por módulo

| Módulo | O que faz | Recurso usado |
|---|---|---|
| Luz e brilho | luz das tochas recortada pelas paredes (polígono de visibilidade calculado uma vez por tocha); brilho (bloom) em ⅙ da resolução | Canvas 2D, composição `lighter` e recorte |
| Sombras | sombra projetada e alongada a partir da tocha mais próxima, para herói, inimigos e lacaios | Canvas 2D |
| Distorção | ar tremendo sobre áreas de fogo; onda de choque em explosões, Grito, Nova e morte de chefe | redesenho do próprio quadro em faixas e anel recortado |
| Atmosfera | gotas pingando nas poças com ondulação e som; poeira iluminada perto das tochas; reflexos nas poças | partículas próprias |
| Combate | pausa de impacto (40–75 ms) no corpo a corpo; arco de golpe do Bárbaro; respingos direcionais por tipo de criatura; rastro em todos os projéteis; cores do número de dano por origem; clarão de morte; vinheta de dano na direção do golpe; quadro invertido e câmera lenta na morte de chefe; câmera lenta curta ao limpar a área | tempo de jogo escalado no laço, nunca durante pausa |
| Som | ~45 sons sintetizados (golpes por material, magias, passos por piso, tochas crepitando, moedas, sinos de raros, interface); música procedural por profundidade, abafada no altar; vento ambiente; reverberação por tema de andar; som posicional por distância e lado; abaixa música em explosões | Web Audio API, nenhum arquivo de áudio |
| Câmera | segue com amortecimento e zona morta; antecipa a direção do movimento; enquadra o chefe; recuo por impacto; tremor por ruído suave em vez de salto aleatório | |
| Interface e itens | itens saltam em arco ao cair, com som por material, brilho no chão por raridade, ouro e poções atraídos pelo herói; nomes só por perto (opcional); aviso lateral de coleta que agrupa repetidos; ícones desenhados para cada item na bolsa, borda por raridade, marca "novo"; ordenar bolsa (raridade/tipo/nome); botão "Largar" e clique direito para largar; animação ao equipar e aviso da mudança principal; contador de ouro animado; painéis com entrada suave; brilho do título e da classe escolhida | DOM e CSS sob `#wrap.enh` |

## Mudanças fora do interruptor (permanentes)

- Painel de **Configurações** (`Esc` ou ⚙, só durante a jornada) com volumes, Modo Melhorado, acessibilidade e itens, tela cheia e "Abandonar jornada" com confirmação. Abrir pausa o jogo. `Esc` não é capturado quando bolsa, loja ou bênção estão abertas.
- **Acessibilidade** valem nos dois modos: tremor de tela e clarões podem ser desligados (padrão: ligados, igual à v1.6). Câmera lenta e vibração são do Modo Melhorado.
- "Itens somem após 90 s" é opção, **desligada** por padrão (mudaria a economia se ligada).

## Custo de desempenho

Medido no navegador Chromium sem placa de vídeo (desenho por software), com o mesmo andar, percurso automático de combate de 20 s e duas repetições. Os quadros por segundo absolutos desse ambiente são baixos e não representam um computador ou celular comum; o que importa é a diferença entre os modos.

| Resolução | Modo | qps médio | qps 1% mais lento | CPU do jogo por quadro (média / p99) |
|---|---|---|---|---|
| 1280×720 | v1.6 original | 24,0–24,3 | 14,4 | 3,4 / 9,0 ms |
| 1280×720 | desligado | 24,1–25,7 | 14,4–15,0 | 3,0–3,4 / 8,1–9,0 ms |
| 1280×720 | ligado | 19,4–21,0 | 10,3–11,1 | 4,1–5,3 / 11–13,5 ms |
| 1920×1080 | v1.6 original | 11,8–12,0 | 6,9 | 3,7–4,5 / 9,8–12,1 ms |
| 1920×1080 | desligado | 11,7–11,9 | 5,5–7,5 | 3,9–4,3 / 9,0–9,4 ms |
| 1920×1080 | ligado | 10,1–10,2 | 6,0–6,4 | 5,7–6,0 / 11,2–12,9 ms |

Custo por módulo (1280×720, só aquele módulo ligado; referência ≈ 26–27 qps e ≈ 3,0 ms):

| Módulo | qps | CPU média | Observação |
|---|---|---|---|
| Luz e brilho | 23,2 | 3,8 ms | o mais caro: recorte das luzes e brilho em tela reduzida |
| Distorção | 26,4 | 3,3 ms | só custa durante explosões e áreas de fogo |
| Combate | 25,0 | 2,9 ms | |
| Sombras | 26,7 | 3,2 ms | |
| Som | 26,2 | 3,4 ms | |
| Câmera | 26,3 | 3,2 ms | |
| Atmosfera | 27,1 | 3,0 ms | |
| Interface | 27,0 | 3,0 ms | |

Na primeira versão, a distorção custava cerca de 60 ms por quadro porque relia a tela inteira várias vezes (calor sobre cada tocha). Corrigi assim: cada região é copiada uma vez para uma tela pequena, e o calor das tochas, que ficava sempre ativo, foi removido. Hoje ela só age em explosões e áreas de fogo. Com tudo ligado, o custo caiu de 87 ms para cerca de 4–6 ms. Se algum aparelho sofrer, o primeiro módulo a desligar é **Luz e brilho**.

## Testes

- **Desligado = original:** capturas em tempo controlado (relógio falso, aleatoriedade fixa, movimento reduzido) nas três classes, em quatro cenas: início, combate, saque e bolsa. A diferença entre a v1.6 e o modo desligado ficou dentro da variação entre duas execuções da própria v1.6, e na bolsa a diferença foi de 14 a 43 pixels. O desempenho também foi igual (tabela acima). Nenhum erro no console.
- **Cenários ligados:** combate e saque com Bárbaro, Necromante e Feiticeira; bolsa com itens de todas as raridades (ícones, "novo", ordenar, equipar com aviso, largar pelo botão); Configurações abrindo e fechando por `Esc` sem roubar o `Esc` da bolsa e da bênção; morte de chefe (câmera lenta e quadro invertido); F7, F4 e F8 ida e volta. Desktop 1100×700 e celular 390×800. Nenhum erro.
- **Colisão:** o modo não mexe em movimento nem colisão. O F4 mostra os contornos para conferir; o teste anterior (2.694 faces de parede, 0% de travamento) continua valendo.
- **Capturas A/B:** `docs/AB-modo-melhorado.png` (esquerda original, direita Modo Melhorado; combate e saque das três classes).

## O que ficou de fora e por quê

- **Controle (gamepad)**: o jogo anda por clique e caminho; mapear analógico exigiria mudar o movimento. Fica para quando for pedido.
- **Mapas normais/iluminação por pixel e sombras reais**: exigiriam trocar o desenho para WebGL (mudança no pipeline de renderização, que pede aprovação).
- **Chuva e relâmpagos**: o jogo é todo subterrâneo.
- **Calor sobre as tochas**: foi feito, mas custava caro demais por estar sempre ativo; removido.
- **Sincronia vertical, cache de shaders, AOT**: não se aplicam a Canvas 2D no navegador.
- **Peso de inventário, expiração de itens ligada por padrão**: mudariam regras.
- **Áudio gravado (ex.: pacotes CC0 da Kenney)**: exigiria baixar arquivos externos; o som sintetizado evita isso.

## Licenças de recursos

Nenhum recurso externo foi adicionado. Todo som é sintetizado em tempo real e todo ícone é desenhado por código, dentro do próprio `index.html`. A única dependência externa continua sendo as fontes do Google Fonts (IM Fell English SC, Alegreya Sans), sob a SIL Open Font License 1.1, já usadas desde a v1.0.
