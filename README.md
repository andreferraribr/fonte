---
title: "Plano Mais Brasil:"
subtitle: "PEC DDD (desvincular, desindexar e desobrigar)"
output:
  html_document:
    code_folding: hide
---
***   

### Fonte de recursos:
#### A chave para entender a desvinculação do orçamento federal

No dia 05/11/2019 o governo federal anunciou medidas econômicas para equilibrar as contas públicas [https://www.youtube.com/watch?v=v9PENV_E7vM].

Uma das medidas pretende desvincular recursos orçamentários. Mas o que é a desvinculação de recursos?

Vamos utilizar dados abertos para explicar:

1) o fluxo orçamentário
2) a desvinculação de recursos
3) o conceito de fonte de recursos

import chart_studio.tools as tls
tls.get_embed('https://plot.ly/~elizabethts/9/')

***

#### Para programadores e curiosos

Coletei os dados no portal Siga Brasil do Senado Federal [https://www12.senado.leg.br/orcamento/sigabrasil] e depois preparei o código para gerar os gráficos.

Disponibilizei o código e arquivos no GitHub: https://github.com/andreferraribr/fonte






#### Preparar os dados

A partir do Siga Brasil baixamos as planilhas:

1) "nat_rec", com detalhes sobre os valores arrecadados.   
2) "outflux", com detalhes sobre os pagamentos totais (pagamentos da LOA + pagamentos de restos a pagar).

Vídeo sobre LOA e restos a pagar [https://www.youtube.com/watch?v=ZcqgaEjJ7Aw]



<div>
    <a href="https://plot.ly/~andreferraribr/39/?share_key=oMkE7R8a7arSoJZPpFYWcZ" target="_blank" title="dru x recursos ordinários" style="display: block; text-align: center;"><img src="https://plot.ly/~andreferraribr/39.png?share_key=oMkE7R8a7arSoJZPpFYWcZ" alt="dru x recursos ordinários" style="max-width: 100%;width: 900px;"  width="900" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="andreferraribr:39" sharekey-plotly="oMkE7R8a7arSoJZPpFYWcZ" src="https://plot.ly/embed.js" async></script>
</div>



***   


### Arrecadação

Vamos utilizar os dados do ano de 2018.

Abaixo apresentamos a arrecadação por espécie, fonte e esfera orçamentária.

Em vermelho temos a fonte de recursos ordinários, também conhecida como fonte 100.

Inúmeras espécies irrigam os recursos ordinários: impostos, taxas, contribuições econômicas, contribuições sociais...

Os recursos ordinários, por sua vez, irrigam o orçamento fiscal.


#### DRU x Orçamento da Seguridade Social

Criamos um filtro para melhorar a compreensão do fluxo da arrecadação.

Filtramos espécies e fontes com valores superiores a R$ 20 bilhões.

Em azul temos a faixa da discórdia: a desvinculação da arrecadação de contribuições sociais.

Parte da arrecadação das contribuições sociais é desvinculada via Desvinculação de Receitas da União (DRU).

Devido à DRU, a seguridade social perde a garantia de ter R$ 120 bilhões gastos exclusivamente em despesas da seguridade social. O governo redireciona os R$ 120 bi para a fonte recursos ordinários e agora o governo pode utilizar esses recursos para pagar qualquer tipo de despesa, inclusive despesas da seguridade social.

Ao desvincular o orçamento, o governo ganha flexibilidade para alocar os recursos.


Vídeo sobre a DRU [https://www.youtube.com/watch?v=nb9LkOjUZyg]


### Classificação por fonte/destinação de recurso

A seguir, texto extraído de documento pulicado no site do Tesouro Nacional:

A classificação orçamentária por fontes/destinações de recursos tem como objetivo identificar
as fontes de financiamento dos gastos públicos. As fontes/destinações de recursos reúnem
recursos oriundos de determinadas Naturezas de Receita, conforme regras previamente
estabelecidas. Por meio do orçamento público, essas fontes/destinações são associadas a
determinadas despesas de forma a evidenciar os meios para atingir os objetivos públicos.   

Como mecanismo integrador entre a receita e a despesa, o código de fonte/destinação de
recursos exerce um duplo papel no processo orçamentário. Para a receita orçamentária, esse
código tem a finalidade de indicar a destinação de recursos para a realização de determinadas
despesas orçamentárias. Para a despesa orçamentária, identifica a origem dos recursos que
estão sendo utilizados.

[http://www.tesouro.fazenda.gov.br/documents/10180/676688/Item+11+-+Classifica%C3%A7%C3%A3o+por+Fonte-Destina%C3%A7%C3%A3o+de+Recursos.pdf/988a562d-cd16-4b00-ad16-90d8b7f6bb8d]

***   

#### Arrecadação e fonte CIDE

Utilizamos a CIDE Combustíveis para ilustrar a classificação orçamentária por fontes/destinações de recursos.

Em 2018, parte da arrecadação da CIDE Combustíveis foi carimbada como fonte 111 (CIDE) e parte como fonte 100 (recursos ordinários).


***   


### Pagamentos

#### Pagamentos e fonte CIDE 

Até aqui o nosso foco foi o ingresso de recursos, a arrecadação.

Agora, vamos nos concentrar na destinação dos recursos, a despesa.

No caso específico da CIDE, a lei 10.336/2001 estabelece as seguintes destinações para os recursos arrecadados:

I - pagamento de subsídios a preços ou transporte de álcool combustível, de gás natural e seus derivados e de derivados de petróleo;

II - financiamento de projetos ambientais relacionados com a indústria do petróleo e do gás; e

III - financiamento de programas de infra-estrutura de transportes.

Nosso objetivo é apenas mostrar a destinação da CIDE por elemento da despesa.

O gráfico abaixo aponta a destinação de recursos da CIDE para:

1) obras e instalações   

2) distribuição constitucional ou legal de receitas   

3) outros serviços de terceiros - pessoa jurídica   

***

Não entramos nos detalhes da destinação dos recursos a ponto de verificar se a lei foi obedecida. De qualquer forma, segue link de planilha com as ações governamentais pagas com recursos da fonte CIDE. [https://github.com/andreferraribr/fonte/blob/master/cide.xlsx?raw=true]

O Manual Técnico do Orçamento traz os conceitos de elemento da despesa: [http://www.planejamento.gov.br/assuntos/orcamento-1/informacoes-orcamentarias/arquivos/MTOs/mto_atual.pdf]


### Todos os pagamento de 2018   

#### Detalhamento por esfera, indicador de resultado primário, fonte e elemento da despesa


Vamos ampliar o escopo da nossa análise. O gráfico abaixo traz todos os pagamentos efetuados em 2018. A informação é detalhada por esfera, indicador de resultado primário, fonte e elemento da despesa.

Destacamos, em vermelho, os recursos ordinários.   

Os recursos ordinários foram utilizados para pagar 51 tipos diferentes de elementos da despesa.



***   


### Fonte e pagamentos de pensão e aposentadoria

O controle por fonte permite visualizar quais fontes o governo utilizou para pagar despesas com pensões e aposentadorias.

Destacamos, em vermelho, quatro pontos interessantes. Em termos nominais, no período de 2005 a 2018, o governo destinou para a seguridade social:

1) R$ 539 bi de recursos ordinários
2) R$ 197 bi de recursos provenientes da Remuneração das disponibilidades do Tesouro Nacional
3) R$ 163 bi de recursos provenientes de Títulos de responsabilidade do Tesouro Nacional
4) R$ 185 bi de recursos do orçamento fiscal para cobrir despesas com aposentadorias e pensões
***   


### DRU x Fonte Recursos Ordinários

Vimos anteriormente que parte da arrecadação das contribuições sociais percorre o seguinte fluxo:

1) parte da arrecadação é desvinculada via DRU,
2) A DRU irriga a fonte de recursos ordinários,
3) Os recursos ordinários são direcionados para o orçamento fiscal.

Agora vamos apurar o saldo das transferências líquidas entre o orçamento da seguridade social e o orçamento fiscal.

O saldo é positivo quando a seguridade social transfere mais recursos do que recebe. Ou seja, a desvinculação via DRU foi maior que o retorno via recursos ordinários. Por exemplo, em 2005 a DRU foi de  R$ 32,4 bi, enquanto R$ 8,5 bi retornaram para a seguridade social via recursos ordinários. Logo, o orçamento da seguridade social foi superavitário em R$ 23,9 bi.

É importante ressaltar que apuramos apensas os recursos ordinários destinados para a seguridade social. Contudo, recursos ordinários da esfera fiscal também podem ser utilizados para pagar despesas típicas da seguridade social, reduzindo assim o superávit da seguridade social.

Sob esse prisma, a seguridade fiscal foi superavitária em todo o período, com exceção de 2014, 2016 e 2017. Contudo, o déficit de apenas dois anos é praticamente igual a todo o superávitdo período.

***   


### DRU x Fontes fiscais

Para a nossa análise ficar mais completa, devemos incluir outras fontes que irrigam o orçamento da seguridade social.

Abaixo exibimos o gráfico após a inclusão de duas fontes de recursos:
1) Remuneração das disponibilidades do Tesouro Nacional
2) Títulos de responsabilidade do Tesouro Nacional.

Com a inclusão das duas fontes a seguridade social passa a apresentar um déficit crescente.

***
### Comentários Finais

A desvinculação de receitas aumenta a flexibilidade da gestão de caixa do governo federal.

Caso a PEC DDD seja aprovada, o governo terá mais liberdade para alocar os recursos públicos.



***

#### Disponibilizei os gráficos no plot.ly:

1) arrecadação por fonte https://chart-studio.plot.ly/~andreferraribr/21.embed

2) DRU  https://chart-studio.plot.ly/~andreferraribr/23.embed

3) CIDE arrecadação https://chart-studio.plot.ly/~andreferraribr/25.embed

4) CIDE pagamento https://chart-studio.plot.ly/~andreferraribr/35.embed

5) pagamento por fonte: https://chart-studio.plot.ly/~andreferraribr/27.embed

6) pagamentos de aposentadorias e pensões https://chart-studio.plot.ly/~andreferraribr/29.embed

7) DRU x fonte 100 https://chart-studio.plot.ly/~andreferraribr/33.embed

8) DRU x fontes fiscais https://chart-studio.plot.ly/~andreferraribr/31.embed


***
#### Para maiores informações sobre fontes de recursos, acessar:

1) Fontes de Recursos : [https://www12.senado.leg.br/orcamento/glossario/fonte-de-recursos] 

2) Orçamento Fácil: vinculação de receitas: [https://www.youtube.com/watch?v=OPdGF5QGXyE]


***  
Obtive ajuda importante nos seguintes endereços:


1) definir cores:    
  [https://www.w3.org/TR/css-color-3/#svg-color]   
  [https://www.w3schools.com/colors/colors_picker.asp?colorhex=FF0000]   

2) formatar gráficos e R markdown:   
  [file:///C:/Users/03092181794/Desktop/R/sankey/final.html]   
  [https://bookdown.org/lyzhang10/lzhang_r_tips_book/how-to-plot-data.html]   
  [https://aledemogr.com/2017/05/29/plots-based-on-un-data-in-r/]   
  [https://plot.ly/r/graphing-multiple-chart-types/]   
  [https://rstudio.com/wp-content/uploads/2015/02/rmarkdown-cheatsheet.pdf]





