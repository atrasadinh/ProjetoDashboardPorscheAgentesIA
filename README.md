# porsche-sales-dashboard

Dashboard interativo de vendas Porsche em um único arquivo HTML (Tailwind CSS + Chart.js via CDN), com 100 vendas tratadas embutidas no próprio script.

**Dashboard publicada:** https://SEU-USUARIO.github.io/porsche-sales-dashboard/
<!-- Troque SEU-USUARIO pelo seu usuário do GitHub depois de ativar o Pages. -->

<!--
Depois de abrir a dashboard publicada, salve os prints em docs/ e descomente estas linhas:

![Visão geral](docs/dashboard-geral.png)
![Filtro aplicado: 911 + West](docs/dashboard-filtro-911-west.png)
-->

## Arquivos

| Arquivo | Conteúdo |
|---|---|
| `index.html` | A dashboard completa (HTML, CSS, JS e os dados embutidos) |
| `data/vendas_tratadas.json` | A mesma base tratada que está dentro do HTML, para consulta |
| `README.md` | Este documento |

## Perguntas de negócio

1. **Quanto da receita já foi entregue e quanto ainda é pipeline?** KPIs "Receita entregue" e "Receita em pipeline", separados. Só 41 das 100 vendas estão entregues, então somar tudo como "receita" esconderia que mais da metade ainda depende de entrega ou aprovação.
2. **Qual é o ticket típico?** KPI "Ticket mediano", com a média ao lado. A base tem de US$ 58.900 a US$ 286.500, e poucos 911 GT3 e Dakar puxam a média para cima.
3. **Que parcela das vendas cancela e quanto isso custa?** KPI "Taxa de cancelamento" (7 de 100 vendas, US$ 625.250 não realizados) e o gráfico de funil de status.
4. **Qual família sustenta a receita?** KPI "Família líder" e gráfico de receita por família (911, 718, Cayenne, Macan, Panamera, Taycan), com a parte entregue e a parte em pipeline. A agrupação é por família porque os 100 registros têm dezenas de variantes, quase todas com uma única venda.
5. **Onde estão as vendas?** Participação da receita por região dos EUA. O tooltip mostra quantas vendas há por trás de cada fatia, porque a região Northeast tem só 8.
6. **Como a receita evolui no tempo, e a base permite dizer isso?** Gráfico mensal com receita entregue e em pipeline, mais um aviso fixo no topo sobre a qualidade das datas.

Os filtros são Família, Região, Status da entrega, Ano, Condição do veículo (novo, seminovo, rodado) e Qualidade da data. Acima dos filtros há uma frase-resumo que se atualiza sozinha e avisa quando a amostra filtrada tem menos de 20 vendas.

## Prompt e o que mudou até a versão final

**Versão 1 (prompt enviado):**

> Atue como Desenvolvedor Frontend Senior e Designer de UI/UX. Crie um dashboard de vendas interativo em um único arquivo HTML (index.html) para a marca Porsche.
>
> Identidade Visual (Porsche Design)
>
> Requisitos Tecnicos:
> - Utilize Tailwind CSS (via CDN) para estilização e Chart.js (via CDN) para os gráficos.
> - O filtro deve atualizar TODOS os KPIs e os 3 gráficos simultaneamente via JavaScript dinâmico.
> - Insira o array JSON completo com as 100 vendas tratadas diretamente no script JS.
> - Garanta layout 100% responsivo para desktop e dispositivos móveis.

A seção "Identidade Visual" veio sem conteúdo, então a IA escolheu a paleta sozinha (preto, branco e vermelho #D5001C, tipografia condensada em caixa alta). O prompt também não dizia quais filtros criar, e a base ainda não tinha sido enviada. Resultado: 100 vendas **fictícias** em reais, com filtros de modelo, região e trimestre.

**Versão 2:** a planilha real foi enviada. A dashboard foi refeita com ela: valores em dólar (US$), região derivada do estado, família derivada do modelo, filtros de Família, Região, Status e Ano. Nenhum texto novo foi enviado além do arquivo.

**Versão 3:** remoção dos nomes de vendedores do HTML (nenhum filtro ou gráfico os usava) e aviso no gráfico mensal quando há vendas sem data válida.

**Versão 4 (final):** depois de uma revisão crítica pedindo "você aprovaria isso para mostrar à diretoria?", foram feitas estas mudanças:

- Receita separada em **entregue** e **pipeline**, em vez de uma receita única.
- **Taxa de cancelamento** e um gráfico de funil de status. O status deixou de abrir em "Exceto cancelados", porque esse padrão escondia justamente o cancelamento.
- **Ticket mediano** no lugar da média, e filtro de condição do veículo (a quilometragem, que tinha ficado de fora, voltou à base).
- **Aviso fixo** sobre datas inválidas e futuras, e um filtro "Só com data válida".
- Frase-resumo dinâmica e aviso de amostra pequena.
- O requisito original falava em 3 gráficos. A versão final tem **4**, porque o funil de status foi acrescentado nesta revisão. Todos atualizam com os filtros.

## Tratamento da base

A planilha tinha as colunas originais ao lado de colunas `*Sanitized`; a dashboard usa só as `Sanitized`. Problemas que a base bruta tinha:

- **Datas** em formatos misturados (`03/14/2024`, `2024.07.11`, `September 17, 2024`, número serial do Excel) e datas impossíveis (`2024-02-30`, `April 31st`). As impossíveis viraram `INVALID`.
- **Preços** como texto: `$121k`, `USD 112.750`, `$89.750,00`, `eighty two thousand USD`.
- **Ano do modelo e quilometragem** por extenso, truncados ou em unidades mistas (`twenty twenty four`, `20-23`, `KM 18,900`, `zero miles`).
- **Pagamento, estado e status** com variações de grafia e caixa (`wire`, `bank-transfer`, `DELIVERD`, `ca`, `California`).

Passos feitos antes de embutir no HTML:

1. `INVALID` em data virou `null`. São 24 vendas, que continuam nos KPIs e nos gráficos de status, família e região, mas ficam fora do gráfico mensal.
2. Campo `familia` criado a partir da primeira palavra do modelo.
3. Campo `regiao` criado a partir do estado (agrupamento em 5 regiões dos EUA).
4. Condição do veículo calculada pela quilometragem: **novo** até 100 milhas (16 vendas), **seminovo** de 101 a 20.000 (63) e **rodado** acima de 20.000 (21). Esses cortes são uma suposição minha, porque a base não tem uma coluna de condição e as quilometragens são contínuas, sem lacuna natural.
5. Data de corte fixa em 21/10/2026: vendas com data posterior (20 no total) são tratadas como datas futuras.
6. Removidos `customer_name` (nome de cliente), `salesperson` (nome de vendedor) e todas as colunas brutas. A planilha original não está neste repositório.

Definições de negócio usadas nos KPIs: **entregue** é o status `Delivered`; **cancelada** é `Cancelled`; **pipeline** é qualquer outro status (em trânsito, pendente, aguardando etc.); **venda ativa** é entregue mais pipeline.

## Ferramentas de IA

Foi usado o **Claude** (claude.ai), com execução de código para ler e tratar a planilha e a skill de leitura de arquivos para abrir o `.xlsx`. A versão 1 foi pré-visualizada como artefato hospedado do Claude. **Não foi usado ChatGPT com Canvas.**

## Exemplo de filtro aplicado

Sem filtros, a dashboard mostra: receita entregue de US$ 5,1 mi (41 vendas, 41,6% da receita ativa), receita em pipeline de US$ 7,1 mi (52 vendas), 93 vendas ativas, ticket mediano de US$ 119.900 (média de US$ 131.210), cancelamento de 7% e 911 como família líder (34,2% da receita ativa).

Selecionando **Família = 911** e **Região = West**, os KPIs e os quatro gráficos passam a mostrar: 10 vendas registradas, 9 ativas, receita entregue de US$ 503.900 (2 vendas), pipeline de US$ 1.384.000 (7 vendas), ticket mediano de US$ 232.500 (média de US$ 209.767) e 1 cancelamento (10%). A frase-resumo avisa que a amostra é pequena.

Esses números foram conferidos rodando o JavaScript da dashboard em Node, com a interface simulada, contra um cálculo independente em Python. Os cenários testados foram: sem filtros, 911 + West, condição Novo, só com data válida, Taycan + Northeast e o botão de limpar.

## Limitações

- 24 vendas têm data inválida e 20 têm data futura (até out/2027). Foram mantidas como vieram; a dashboard as sinaliza, mas não consegue corrigi-las.
- Com 100 vendas, qualquer recorte por região ou família tem poucas observações. Leia percentuais de recortes pequenos com cautela.
- A quilometragem de uma venda foi convertida na base tratada (`KM 18,900` virou 11.744 milhas), e a conversão não foi auditada linha a linha.
- O layout foi desenhado para desktop e celular, mas ainda precisa de conferência visual em navegador antes de qualquer apresentação.

## Como abrir

Abra `index.html` no navegador. Precisa de internet apenas para carregar Tailwind, Chart.js e a fonte Barlow pelos CDNs. Para publicar: Settings → Pages → Deploy from a branch → `main` / `(root)`.
