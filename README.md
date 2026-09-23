# 🗺️ Gestor de Rotas — Obras

Ferramenta web de arquivo único para planejar, organizar e acompanhar rotas de visitas a obras em campo. Feita para uso no celular, com foco em quem trabalha em regiões de sinal fraco.

O usuário cola os links do Google Maps enviados pelos clientes, a ferramenta extrai as coordenadas, plota tudo no mapa e monta a rota do dia, com filtro por semana e por dia.

> **Sem backend, sem build, sem dependências para instalar.** É um único `index.html`: abre no navegador e funciona.

<!-- Adicione aqui um print ou GIF: ![Demonstração](docs/demo.gif) -->

---

## O problema

Em serviços de campo (instalações, vistorias, manutenção), as obras chegam por WhatsApp, cada uma com um texto de identificação e um link de localização. Com dezenas de paradas por semana, isso vira:

- uma lista longa e difícil de ordenar;
- rotas montadas "de cabeça", com deslocamentos desnecessários;
- dificuldade de saber o que ficou pendente em semanas anteriores;
- dependência de internet em locais onde ela não existe.

## A solução

| Necessidade | Como a ferramenta resolve |
|---|---|
| Entrada rápida | Cola a identificação + link de vários clientes de uma vez; o parser separa e extrai as coordenadas |
| Visualização | Mapa interativo (Leaflet) com marcadores numerados por dia de serviço |
| Rota eficiente | Otimização por distância real de rua (OSRM), com fallback para linha reta |
| Acompanhamento | Filtro por semana ISO e por dia; semanas passadas continuam acessíveis para achar pendências |
| Obras longas | Data inicial e final; a parada aparece em todas as semanas que atravessa |
| Campo sem sinal | Navegação por deep link no Organic Maps / Maps.me com mapa offline |
| Privacidade | Dados ficam só no navegador do usuário (`localStorage`) |

---

## Funcionalidades

### 1. Importação de paradas
- Colagem em lote: linhas de identificação seguidas do link de cada obra.
- Extração automática de latitude/longitude a partir de URLs do Google Maps e de coordenadas em texto.
- Paradas cujo link não pôde ser lido (ex.: links curtos) são marcadas com aviso e permitem **correção manual das coordenadas**.

### 2. Lista de paradas
- Datas de **início** e **fim** (para obras de vários dias).
- Crachá de numeração por dia de serviço, ex.: `1-14/09`, `2-14/09`, `1-15/09`. Obras de vários dias mostram o intervalo, ex.: `1-14→18/09`.
- Reordenação manual (↑ ↓) e remoção individual com confirmação.

### 3. Filtro por semana e dia
- Seletor de semana (padrão **ISO 8601**, semana começando na segunda-feira) com navegação ◀ ▶ e opção "Todas as semanas".
- Botões de dia (Seg a Sex) com contagem de obras em cada um.
- **O mesmo filtro controla lista e mapa**, então os dois ficam sempre em sincronia.
- Contador filtrado: `Paradas (12 de 42)`.
- Ações como *otimizar rota*, *limpar tudo*, *abrir no Google Maps* e *ver pontos offline* valem apenas para a seleção atual. O "Limpar tudo" informa quantas paradas serão apagadas e quantas permanecem.
- A semana escolhida é lembrada entre sessões.

### 4. Otimização de rota
- Usa o endpoint `trip` do [OSRM](https://project-osrm.org/) para ordenar as paradas pelo trajeto real de rua, mantendo a primeira parada como ponto de partida.
- Se o serviço estiver indisponível ou sem internet, cai automaticamente para **vizinho mais próximo** com distância de Haversine (linha reta) e avisa o usuário.
- Reordena somente as paradas da seleção atual, sem alterar as demais semanas.

### 5. Navegação offline
- Botões "🧭 Navegar até aqui" abrem, no **Organic Maps** ou **Maps.me**, a rota da parada anterior até a atual.
- A escolha do app é salva.

---

## Como usar

1. Abra o `index.html` no navegador (celular ou desktop). Também pode ser hospedado via [GitHub Pages](https://pages.github.com/).
2. Em **1. Adicionar paradas**, cole a identificação e o link de cada obra e toque em **Adicionar à lista**.
3. Informe a **data de início** (e o fim, se durar vários dias) de cada parada.
4. Escolha a **semana** (e o dia) em **2. Paradas**.
5. Toque em **Otimizar rota por distância** ou reordene manualmente.
6. Use **Google Maps** (com internet) ou **Navegar (offline)** na lista.

### Exemplo de entrada

```text
14/09- NOTA 000000001_ CLIENTE EXEMPLO A
https://www.google.com/maps/place/...
15/09- NOTA 000000002_ CLIENTE EXEMPLO B
https://www.google.com/maps/place/...
```

---

## Decisões técnicas

- **Arquivo único, zero build.** HTML, CSS e JavaScript puro (vanilla). A biblioteca Leaflet 1.9.4 vai embutida no próprio arquivo, então a ferramenta carrega mesmo sem CDN. Ideal para enviar por WhatsApp e abrir direto no celular.
- **Persistência local.** Paradas, semana selecionada e app de navegação ficam em `localStorage`. Não há servidor nem conta, e nenhum dado de cliente sai do aparelho (exceto as coordenadas enviadas ao OSRM ao otimizar).
- **Degradação graciosa.** Sem internet, lista, ordenação manual, otimização em linha reta e navegação offline continuam funcionando; só o fundo do mapa e o OSRM dependem de rede.
- **Semanas ISO 8601.** Cálculo próprio de semana ISO, incluindo viradas de ano, com limite de 60 dias para intervalos de obras.
- **Filtro como fonte única de verdade.** Uma variável de semana e uma de dia alimentam lista, mapa, contadores e ações, evitando divergência entre as telas.
- **Reordenação segura em lista filtrada.** Ao otimizar ou mover paradas dentro de uma semana, a nova ordem é aplicada nas mesmas posições que elas ocupavam na lista global, sem afetar as outras semanas.

## Stack

| Camada | Tecnologia |
|---|---|
| Interface | HTML5 + CSS3 (responsivo, mobile-first) |
| Lógica | JavaScript (ES6+), sem frameworks |
| Mapa | [Leaflet 1.9.4](https://leafletjs.com/) + tiles do OpenStreetMap |
| Roteirização | [OSRM](https://project-osrm.org/) (API pública) |
| Navegação offline | Deep links do Organic Maps / Maps.me |
| Armazenamento | `localStorage` |

## Limitações conhecidas

- O servidor público do OSRM não tem garantia de disponibilidade e limita o número de pontos; para uso intenso, o ideal é uma instância própria.
- Links curtos do Google Maps precisam ser abertos e copiados por completo (ou ter as coordenadas informadas à mão), pois o navegador não os resolve sozinho.
- Os dados são por navegador/aparelho: não há sincronização entre dispositivos.
- O Organic Maps / Maps.me não faz rota com vários pontos de uma vez, então a navegação offline é uma etapa por vez.

## Ideias para o futuro

- Exportar/importar a lista em JSON ou CSV (backup e troca de aparelho).
- Marcar paradas como concluídas e listar pendências de semanas anteriores.
- Agrupar a lista por semana em seções recolhíveis.
- Instalação como PWA com cache dos tiles.

---

## Privacidade

A ferramenta não envia a lista de paradas para nenhum servidor próprio. Se for usar dados reais, **não versione arquivos com nomes, notas ou localizações de clientes** neste repositório. Os exemplos acima são fictícios.

## Licença

Defina a licença do projeto (por exemplo, [MIT](https://choosealicense.com/licenses/mit/)) e adicione o arquivo `LICENSE`.

## Autor

**Abne Melo dos Santos**
Engenheiro Eletricista, pós-graduado em Engenharia de Software. Desenvolvimento Web e automação de processos de campo. Teresina, PI, Brasil.

- GitHub: [abnemeloeng-dotcom](https://github.com/abnemeloeng-dotcom)
- LinkedIn: [in/abnemelo](https://www.linkedin.com/in/abnemelo)
- E-mail: abnemelo.eng@gmail.com
- Outro projeto: [abnemeloeng-relatorio](https://abnemeloeng-dotcom.github.io/abnemeloeng-relatorio/)
