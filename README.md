# Análise de Impacto Populacional — Piauí

Ferramenta web (HTML + JS, sem backend) para estimar a população atingida em uma área de análise no Piauí (IBGE, Censo 2022). Permite escolher a base geográfica (setor censitário, bairro ou município) e o modo de seleção: por raio/buffer (ponto, coordenadas ou KML/KMZ importado) ou clicando diretamente nas unidades que você quer somar.

## Arquivos

- `index.html` — aplicação completa (mapa, interface e lógica).
- `piaui_data.js` — setores censitários (GeoJSON), com população, área e situação urbana/rural (`SIT`). **Precisa estar na mesma pasta que o `index.html`.**
- `piaui_bairros.js` — bairros (GeoJSON). **Idem.**
- `piaui_municipios.js` — municípios (GeoJSON). **Idem.**
- `piaui_domicilios_pontos.js` — coordenadas exatas de cada domicílio do Censo 2022 (IBGE, malha de coordenadas de endereços). **Idem.**
- `piaui_domicilios_setor.js` — contagem pré-calculada de domicílios particulares por setor censitário inteiro (join espacial feito uma vez, offline). Usado como denominador no método "Por domicílios", evitando reescanear 1,4 milhão de pontos a cada análise. **Idem.**
- `piaui_domicilios_municipio.js` — domicílios particulares e coletivos por município (Censo 2022, agregado direto do código do município). **Idem.**
- `piaui_demografia_setor.js` — população exata por sexo e 11 faixas etárias, por setor censitário (Censo 2022, Agregados por Setores - Demografia). **Idem.**
- `piaui_demografia_bairro.js` — o mesmo, agregado por bairro (soma dos setores que compõem cada bairro). **Idem.**
- `piaui_demografia_municipio_exata.js` — o mesmo, agregado por município (soma de todos os setores do município). **Idem.**
- `logo_governo_pi.png` — logo oficial do Governo do Estado do Piauí/SEPLAN, usada no cabeçalho. **Idem.**

## Como usar localmente

Baixe os cinco arquivos, mantenha-os na mesma pasta e abra o `index.html` no navegador (com internet, já que os mapas e bibliotecas são carregados via CDN).

## Como publicar com GitHub Pages

1. Suba estes arquivos na raiz do repositório.
2. Em **Settings → Pages**, selecione a branch e a pasta `/ (root)`.
3. O GitHub Pages publica automaticamente o `index.html` como página inicial.

## Bases disponíveis

| Base | Unidades | População total | Cobertura |
|---|---|---|---|
| Setor censitário | 7.340 | 3.271.199 | Estado inteiro |
| Bairro | 479 | 1.382.558 | Só os 25 municípios com divisão por bairro |
| Município | 224 | 3.271.199 | Estado inteiro |

## Ordem do painel lateral

Os controles seguem a ordem natural de preenchimento: **1)** escolha a base de dados (Setor/Bairro/Município) → **2)** escolha o modo de seleção → **3)** preencha os controles daquele modo (raio, KML, seleção manual ou município) → **4)** refine com os filtros de sexo/idade, que ficam por último porque atuam sobre o resultado já calculado.

## Modos de seleção

- **Raio / buffer**: clique no mapa, digite latitude/longitude, ou importe um KML/KMZ.
- **Seleção manual (clique)**: clique diretamente nas unidades para somá-las — útil para casos como "quero estes três municípios". Clique de novo para remover.
- **Por município**: escolha um município num menu suspenso (224 opções) e veja direto a população, os domicílios (particulares e coletivos) e o perfil demográfico daquele município inteiro, sem precisar desenhar raio nem clicar no mapa.

## Filtros (sexo e faixa etária)

O painel **"Filtros de sexo e idade"** (na parte de baixo da configuração, depois de definir a área de análise) usa **caixas de seleção (checkboxes)** em vez de menus — dá pra marcar quantas combinações quiser:

- **Sexo**: Homens / Mulheres (marque um, outro ou os dois — os dois marcados = total).
- **Faixa etária**: 11 faixas do Censo (0–4, 5–9, 10–14, 15–19, 20–24, 25–29, 30–39, 40–49, 50–59, 60–69, 70+). Botões "marcar todas" / "limpar" facilitam selecionar em bloco. Pelo menos uma caixa de cada grupo fica sempre marcada.

Os valores no card de resultado, na tabela de unidades e no PDF exportado refletem as caixas marcadas. **O dado agora é exato em qualquer base** (Setor, Bairro ou Município) — vem direto do Censo 2022 (IBGE, Agregados por Setores - Demografia), não é mais estimado por percentual de município. Setor é a fonte original; Bairro e Município são a soma exata dos setores que os compõem.

## Filtro urbano / rural (modo raio/buffer e Por município)

Um problema comum: um setor urbano grande é só "tocado de raspão" pela borda do raio, mas entra na conta inteiro (principalmente no método "Unidade tocada"), inflando o número final. O painel **"Área urbana / rural"** (visível nos modos "Raio/buffer" e "Por município") resolve isso:

- Duas caixas de seleção: **Urbana** e **Rural** (ambas marcadas = sem filtro, padrão).
- Desmarque **Urbana** pra descontar completamente os setores urbanos da soma — sobra só a população dos setores rurais, e vice-versa.
- Baseado no campo oficial `SITUACAO` do IBGE (malha de setores censitários — `PI_setores_CD2022.shp`), já embutido no `piaui_data.js` como `SIT` ('U'/'R'). Setores sem essa classificação (poucos casos residuais da malha) nunca são excluídos, por segurança.
- **No modo raio/buffer**: funciona na base Setor censitário e no método "Por domicílios" (que sempre opera em granularidade de setor). Não tem efeito nas bases Bairro/Município nesse modo.
- **No modo "Por município"**: quando o filtro está ativo, o app soma só os setores do município escolhido que batem com a situação marcada (população, área, perfil demográfico e domicílios recalculados na hora a partir dos setores) — em vez do dado oficial agregado do município inteiro. Sem filtro, continua usando o dado oficial exato do IBGE.
- Ao contrário dos filtros de sexo/idade, mudar esse filtro **recalcula a análise inteira** (não é só um re-corte do resultado em cache), porque muda quais setores entram na soma.
- O resumo da análise e o PDF exportado mostram o filtro aplicado, quantos setores foram descartados, e a área correspondente só aos setores incluídos.

## Métodos de contagem (modo raio/buffer)

- **Total da unidade tocada** (padrão): conta a população inteira de qualquer unidade tocada — mais rápido, mas pode sobrestimar.
- **Proporcional à área**: calcula a interseção geométrica real e conta só a fração proporcional — mais preciso, porém mais pesado.
- **Por domicílios**: para cada setor censitário tocado, conta quantos domicílios particulares (pontos exatos) caem dentro da zona de análise e quantos existem no setor inteiro, e **rateia a população oficial do setor (Censo 2022) nessa mesma proporção**. Os pontos só indicam onde a população está mais concentrada dentro do setor — não são multiplicados por nenhuma média, então domicílios vagos ou de uso ocasional não criam população fictícia, e nenhum setor pode ultrapassar sua população oficial (mesmo quando o buffer cobre 100% do setor, o resultado bate exato com o Censo). Mais preciso que "proporcional à área" porque usa a distribuição real das casas, não uma fração geométrica. Sempre opera em granularidade de setor censitário, independente da base selecionada no painel. Setores tocados sem nenhum domicílio geocodificado ficam de fora da soma, e isso é sinalizado no resumo e no PDF. Quando esse método está ativo, o card de resultado mostra uma linha extra com a **média efetiva de moradores/domicílio** do resultado (também aparece no modo "Por município").

> **Correção de metodologia (30/07/2026):** versões anteriores multiplicavam a contagem de domicílios diretamente pela V0005 (média oficial de moradores em domicílios *ocupados*). Como a malha de coordenadas do Censo geocodifica endereços sem indicar ocupação, ela inclui domicílios vagos e de uso ocasional — multiplicar esse total pela V0005 (calibrada só para ocupados) inflava a população estimada, na proporção da taxa de vacância de cada setor (chegou a superestimar em ~23% num teste com dados reais). A correção acima elimina esse viés por construção.

> **Otimização de performance (31/07/2026):** a correção acima, sem cuidado, deixava a análise lenta (15+ segundos) em rotas KML com muitos trechos, porque recalculava "quais setores a rota toca" do zero a cada vez, testando os 7.340 setores contra cada trecho sem nenhum filtro barato antes. Agora a bbox de cada setor/bairro/município é pré-calculada uma vez no carregamento da página (~20ms) e usada como filtro rápido antes do teste geométrico real — e a contagem de domicílios do setor inteiro vem de um arquivo pré-calculado (`piaui_domicilios_setor.js`) em vez de escanear 1,4 milhão de pontos toda vez. Resultado: mesmo cálculo, ~150x mais rápido (rota de 75 trechos caiu de ~16s para ~100ms num teste controlado).

## Informação de área

O card de resultado exibe a **área do buffer** (zona de influência) e a **área coberta**:

- Modo **proporcional**: área do buffer (zona de análise).
- Modo **total (unidade tocada)**: soma da área de todas as unidades tocadas — normalmente maior que o buffer, pois conta as unidades inteiras.

Ambas as informações também aparecem nos parâmetros do relatório PDF.

## Domicílios na área (modo raio/buffer)

Além da população, o resultado do modo **raio/buffer** também mostra o total de **domicílios** dentro da área analisada, usando a **malha de coordenadas de endereços do Censo 2022** (IBGE) — ou seja, a localização exata de cada domicílio, não uma estimativa por polígono de setor.

- **Contagem exata**: cada domicílio é testado individualmente contra a geometria da área (ponto dentro do buffer/polígono), não há rateio nem estimativa por área.
- **Funciona em qualquer base** (Setor, Bairro ou Município) e **independe do método de contagem** escolhido para a população (total/proporcional) — domicílio é sempre contado de forma exata.
- Mostra separadamente **domicílios particulares** (residências) e **domicílios coletivos** (quartéis, asilos, presídios, alojamentos etc.).
- O dado é o **endereço geocodificado**, não o status de ocupação — ou seja, inclui domicílios vagos e de uso ocasional (o Censo não publica esse status junto da coordenada).
- 97,5% das coordenadas são o endereço original exato do Censo; o restante usa aproximações do próprio IBGE (apartamentos no mesmo número, face de quadra, localidade ou, raramente, centroide do setor) quando o endereço original não pôde ser geocodificado com precisão.

## Resumo da análise

Todo resultado gera automaticamente um **parágrafo de resumo** em linguagem natural — tanto na tela (painel lateral) quanto no PDF — sintetizando: população atingida, percentual, unidades, área, perfil de sexo e idade e o efeito de qualquer filtro ativo.

## Exportar relatório em PDF

O botão **"Exportar relatório (PDF)"** gera, direto no navegador, um PDF com:

1. Parâmetros da análise (base, modo, raio, metodologia, filtros, área)
2. Resumo narrativo da análise
3. Caixa com o resultado principal (refletindo filtros ativos, com nota do total sem filtro)
4. Tabela completa de unidades com valores filtrados
5. Perfil demográfico completo (21 faixas de idade × sexo) com nota quando há filtros ativos
6. Notas metodológicas e rodapé com numeração de páginas

## Observações técnicas

- **Simplificação geométrica**: polígonos de setor simplificados (tolerância ≈ 22 m), coordenadas arredondadas para 5 casas decimais (≈ 1 m).
- **Sistema de coordenadas**: SIRGAS2000 tratado como WGS84 — diferença < 1 m, sem impacto prático.
- **Cálculo por interseção geométrica real** (turf.js), não por distância de centróide.
- **`piaui_domicilios_pontos.js` (~15 MB)**: mais de 1,4 milhão de coordenadas, codificadas em binário (Float32) + base64 para reduzir o tamanho — é o maior arquivo do projeto e deixa o carregamento inicial um pouco mais lento (poucos segundos a mais, dependendo da conexão). A decodificação só acontece na primeira análise em modo raio/buffer (lazy loading), não trava o carregamento da página.
