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
    <a href="https://plot.ly/~andreferraribr/41/?share_key=9PtTkxbKNjr7l8Om2F2b2T" target="_blank" title="dru x recursos ordinário e outras fontes fiscais" style="display: block; text-align: center;"><img src="https://plot.ly/~andreferraribr/41.png?share_key=9PtTkxbKNjr7l8Om2F2b2T" alt="dru x recursos ordinário e outras fontes fiscais" style="max-width: 100%;width: 900px;"  width="1200" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="andreferraribr:41" sharekey-plotly="9PtTkxbKNjr7l8Om2F2b2T" src="https://plot.ly/embed.js" async></script>
</div>


***   


### Arrecadação

Vamos utilizar os dados do ano de 2018.

Abaixo apresentamos a arrecadação por espécie, fonte e esfera orçamentária.

Em vermelho temos a fonte de recursos ordinários, também conhecida como fonte 100.

Inúmeras espécies irrigam os recursos ordinários: impostos, taxas, contribuições econômicas, contribuições sociais...

Os recursos ordinários, por sua vez, irrigam o orçamento fiscal.


```{r, arrecadação por fonte (R$ Bi)}

# Governo Federal 2018: arrecadação por fonte (R$ Bi)

influx_s<- nat_rec%>%
  # filtra pelo ano de 2018 e valore maiores que R$ 0
  filter(ano == 2018,influx_reais>0 )%>%
# utilizar group_by para incluir as variáveis specie_name_cod, fonte_name_cod, esfera_in no gráfico sankey
    group_by( especie_name_cod, fonte_name_cod, esfera_in)%>%
 # transformar em bilhões de reais
   summarise(total = sum(influx_reais/1000000000))%>%
  arrange(desc(total ))
  # cria variávies para identificar espécies e fontes menores que determinado valor e reclassificá-las como "demais"
influx_s<- influx_s%>%
    mutate(fonte_demais = ifelse(total > 0, fonte_name_cod, "demais fontes") )%>%
  mutate(especie_demais = ifelse(total > 0, especie_name_cod, "demais espécies") )%>%
  group_by( especie_demais, fonte_demais, esfera_in)%>%
  summarise(value = sum(total))

# arredondar valor para melhorar visualização da tabela
influx_s$value<-as.numeric(myround(influx_s$value, 1))
names<- vector()
nodes<- vector()

# loop para extrair nomes dos nodes a partir da df influx_s
for(i in 1:ncol(influx_s)-1)
{
for(j in 1:NROW(influx_s[,i]))
{
names <- c(names,influx_s[j,i]) ;
}
}

# obeter apenas os nomes únicos.
name<- unique(as.character(names))

# criar nodes
nodes<- data.frame(cbind(node = c((0:(length(name)-1))), name))

nodes$node <- as.numeric(as.character(nodes$node))

## criar links
links <- influx_s%>%
  group_by_at(1:2)%>%
  summarise(value = sum(value))
colnames(links)<-c("source","target","value")
##
links<-as.data.frame(links)

## criar links adcionais caso a df influx_s tenha mais de duas variáveis
for(i in 1:(ncol(influx_s)-3))
{
loop_link <- influx_s%>%
  group_by_at((1+i):(2+i))%>%
  summarise(teste = sum(value))
loop_link<-as.data.frame(loop_link)
colnames(loop_link)<-c("source","target","value")
# juntar os links
links<-rbind(links,loop_link)
}

# mesclar nodes aos valores
links <- merge(links, nodes, by.x = "target", by.y = "name")
links <- merge(links, nodes, by.x = "source", by.y = "name")

## remover duplicados 
links <- distinct(links, target,source, .keep_all= TRUE)

# preparar os dados finais para o sankey
links_final <- links[ , c("node.y", "node.x", "value")]
colnames(links_final) <- c("source", "target", "value")


p <- plot_ly(
    type = "sankey",
    width = 1200, height = 900,
    orientation = "h",
    node = list(
      label = nodes$name,
      # quantidade de cores == lenght nodes
      color = c(rep("lightgray",57),"red", rep("lightgray",73)),
      pad = 15,
      thickness = 20,
      line = list(
        color = "black",
        width = 0.5
      )
    ),

    link = list(
      source = links_final$source,
      target = links_final$target,
      value =  links_final$value,
      # quantidade de cores == lenght links
      color = "lightgray"
    )
  ) %>% 
  layout( title = "Governo Federal 2018: arrecadação por fonte (R$ Bi)",
          font = list(size = 10
                      )
)


# Create a shareable link to your chart
#chart_link = api_create(p, filename="emaranhado")
#chart_link
# exibir gráfico e tabela

p
```

***   


### A Faixa da discórdia   

#### DRU x Orçamento da Seguridade Social

Criamos um filtro para melhorar a compreensão do fluxo da arrecadação.

Filtramos espécies e fontes com valores superiores a R$ 20 bilhões.

Em azul temos a faixa da discórdia: a desvinculação da arrecadação de contribuições sociais.

Parte da arrecadação das contribuições sociais é desvinculada via Desvinculação de Receitas da União (DRU).

Devido à DRU, a seguridade social perde a garantia de ter R$ 120 bilhões gastos exclusivamente em despesas da seguridade social. O governo redireciona os R$ 120 bi para a fonte recursos ordinários e agora o governo pode utilizar esses recursos para pagar qualquer tipo de despesa, inclusive despesas da seguridade social.

Ao desvincular o orçamento, o governo ganha flexibilidade para alocar os recursos.


Vídeo sobre a DRU [https://www.youtube.com/watch?v=nb9LkOjUZyg]

```{r, Desvinculação de Recursos da União (R$ Bi)}

# Governo Federal 2018: Desvinculação de Recursos da União (R$ Bi)

influx_s<- nat_rec%>%
  # filtra pelo ano de 2018 e valore maiores que R$ 0
  filter(ano == 2018,influx_reais>0 )%>%
# utilizar group_by para incluir as variáveis specie_name_cod, fonte_name_cod, esfera_in no gráfico sankey
    group_by( especie_name_cod, fonte_name_cod, esfera_in)%>%
 # transformar em bilhões de reais
   summarise(total = sum(influx_reais/1000000000))%>%
  arrange(desc(total ))
  # cria variávies para identificar espécies e fontes menores que determinado valor (R$ 20 bi) e reclassificá-las como "demais"
influx_s<- influx_s%>%
    mutate(fonte_demais = ifelse(total > 20, fonte_name_cod, "demais fontes") )%>%
  mutate(especie_demais = ifelse(total > 20, especie_name_cod, "demais espécies") )%>%
  group_by( especie_demais, fonte_demais, esfera_in)%>%
  summarise(value = sum(total))

# arredondar valor para melhorar visualização da tabela
influx_s$value<-as.numeric(myround(influx_s$value, 1))
names<- vector()
nodes<- vector()

# loop para extrair nomes dos nodes a partir da df influx_s
for(i in 1:ncol(influx_s)-1)
{
for(j in 1:NROW(influx_s[,i]))
{
names <- c(names,influx_s[j,i]) ;
}
}

# obeter apenas os nomes únicos.
name<- unique(as.character(names))

# criar nodes
nodes<- data.frame(cbind(node = c((0:(length(name)-1))), name))

nodes$node <- as.numeric(as.character(nodes$node))

## criar links
links <- influx_s%>%
  group_by_at(1:2)%>%
  summarise(value = sum(value))
colnames(links)<-c("source","target","value")
##
links<-as.data.frame(links)

## criar links adcionais caso a df influx_s tenha mais de duas variáveis
for(i in 1:(ncol(influx_s)-3))
{
loop_link <- influx_s%>%
  group_by_at((1+i):(2+i))%>%
  summarise(teste = sum(value))
loop_link<-as.data.frame(loop_link)
colnames(loop_link)<-c("source","target","value")
# juntar os links
links<-rbind(links,loop_link)
}

# mesclar nodes aos valores
links <- merge(links, nodes, by.x = "target", by.y = "name")
links <- merge(links, nodes, by.x = "source", by.y = "name")

## remover duplicados 
links <- distinct(links, target,source, .keep_all= TRUE)

# preparar os dados finais para o sankey
links_final <- links[ , c("node.y", "node.x", "value")]
colnames(links_final) <- c("source", "target", "value")


p <- plot_ly(
    type = "sankey",
    width = 1200, height = 900,
    orientation = "h",
    node = list(
      label = nodes$name,
      # quantidade de cores == lenght nodes
      color = c("lightgray","blue", rep("lightgray",6), "red", rep("lightgray",13),"red", "blue"),
      pad = 15,
      thickness = 20,
      line = list(
        color = "black",
        width = 0.5
      )
    ),

    link = list(
      source = links_final$source,
      target = links_final$target,
      value =  links_final$value,
      # quantidade de cores == lenght links
      color = c("red" ,rep("lightgray",4),"blue", rep("lightgray",24))
    )
  ) %>% 
  layout( title = "Governo Federal 2018: Desvinculação de Recursos da União (R$ Bi)",
          font = list(size = 10
                      )
)



# Create a shareable link to your chart
#chart_link = api_create(p, filename="DRU")
#chart_link
# exibir gráfico e tabela

p
```

***   


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

```{r, arrecadação CIDE (repetição do chunk anterior agora filtrado por natureza da receita "12200821" - CIDE) }
# utilizar group_by para incluir as variáveis origem e esfera no gráfico sankey
nat_rec_s<- nat_rec%>%
  # filtra pelo ano de 2018 e CIDE COMBUSTÍVEIS COMERCIALIZAÇÃO PRINCIPAL (Natureza da Receita 12200821)

  filter(ano == "2018",nat_cod == "12200821" , influx_reais>1 )%>%
  group_by( nat, esfera_in, fonte_name_cod)%>%
  summarise(value = sum(influx_reais/1000000000))%>%
  arrange(desc(value ))

# arredondar valor para melhorar visualização da tabela
nat_rec_s$value<-as.numeric(myround(nat_rec_s$value, 1) )
names<- vector()
nodes<- vector()

# loop para extrair nomes dos nodes a partir da df influx_s
for(i in 1:ncol(nat_rec_s)-1)
{
for(j in 1:NROW(nat_rec_s[,i]))
{
names <- c(names,nat_rec_s[j,i]) ;
}
}

# obeter apenas os nomes únicos.
name<- unique(as.character(names))

## reverter a ordem de nodes 

# criar nodes
nodes<- data.frame(cbind(node = c((0:(length(name)-1))), name))

nodes$node <- as.numeric(as.character(nodes$node))

## criar links

links <- nat_rec_s%>%
  group_by_at(1:2)%>%
  summarise(value = sum(value))
colnames(links)<-c("source","target","value")
##
links<-as.data.frame(links)

## criar links adcionais caso a df influx_s tenha mais de duas variáveis

for(i in 1:(ncol(nat_rec_s)-3))
{
loop_link <- nat_rec_s%>%
  group_by_at((1+i):(2+i))%>%
  summarise(teste = sum(value))
loop_link<-as.data.frame(loop_link)
colnames(loop_link)<-c("source","target","value")

# juntar os links
links<-rbind(links,loop_link)
}

# mesclar nodes aos valores
links <- merge(links, nodes, by.x = "target", by.y = "name")
links <- merge(links, nodes, by.x = "source", by.y = "name")

## remover duplicados 
links <- distinct(links, target,source, .keep_all= TRUE)

links_final <- links[ , c("node.y", "node.x", "value")]
colnames(links_final) <- c("source", "target", "value")

p <- plot_ly(
    type = "sankey",
    orientation = "h",
    width = 1200, height = 300,
    node = list(
      label = nodes$name,
      # quantidade de cores == lenght nodes
      color = c("lightgray", "red", "pink","red"),
      pad = 15,
      thickness = 20,
      line = list(
        color = "black",
        width = 0.5
      )
    ),

    link = list(
      source = links_final$source,
      target = links_final$target,
      value =  links_final$value,
      # quantidade de cores == lenght links
      color = c("lightgray", "red", "lightgray")
    )
  ) %>% 
  layout(
    title = "Governo Federal 2018: CIDE COMBUSTÍVEIS COMERCIALIZAÇÃO PRINCIPAL x fonte de recurso (R$ Bi)",
    font = list(
      size = 10
    )
)



# Create a shareable link to your chart
# Set up API credentials: https://plot.ly/r/cide
 #chart_link = api_create(p, filename="cide")
 #chart_link
# exibir gráfico e tabela


p
```

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

```{r, destinação da CIDE (semelhante ao chunk anterior, mas com dados dos pagamentos/outflux)}
# filtra nodes objeto do estudo
# Governo Federal 2018: Pagamentos realizados com a fonte CIDE (R$ Bi)
outflux_s<- outflux%>%
  filter(ano == "2018", outflux_reais>1, fonte_cod == "111" )%>%
  group_by(fonte, elemento)%>%
  summarise(value = sum(outflux_reais/1000000000))%>%
  arrange(desc(value ))




# arredondar valor para melhorar visualização da tabela

outflux_s$value<-as.numeric(myround(outflux_s$value, 1))


nodes<- vector()
names<- c()


# loop para extrair nomes dos nodes a partir da df outflux_s
for(i in 1:ncol(outflux_s)-1)
{
for(j in 1:NROW(outflux_s[,i]))
{
names <- c(names,outflux_s[j,i]) ;
}
}


# obeter apenas os nomes únicos.
name<- unique(as.character(names))

## reverter a ordem de nodes 

# criar nodes
nodes<- data.frame(cbind(node = c((0:(length(name)-1))), name))

nodes$node <- as.numeric(as.character(nodes$node))





## criar links

links <- outflux_s%>%
  group_by_at(1:2)%>%
  summarise(value = sum(value))
colnames(links)<-c("source","target","value")
##
links<-as.data.frame(links)


## criar links adcionais caso a df outflux_s tenha mais de duas variáveis
loop_link<-c("")

for(i in 1:(ncol(outflux_s)-3))
{
loop_link <- outflux_s%>%
  group_by_at((1+i):(2+i))%>%
  summarise(teste = sum(value))
loop_link<-as.data.frame(loop_link)
colnames(loop_link)<-c("source","target","value")

# juntar os links
links<-rbind(links,loop_link)
}

# mesclar nodes aos valores
links <- merge(links, nodes, by.x = "target", by.y = "name")
links <- merge(links, nodes, by.x = "source", by.y = "name")

## remover duplicados 
links <- distinct(links, target,source, .keep_all= TRUE)


links_final <- links[ , c("node.y", "node.x", "value")]
colnames(links_final) <- c("source", "target", "value")


p <- plot_ly(
    type = "sankey",
    orientation = "h",
    width = 1200, height = 300,
    node = list(
      label = nodes$name,
      color = c("pink", rep("lightgray",3)),
      pad = 15,
      thickness = 20,
      line = list(
        color = "black",
        width = 0.5
      )
    ),

    link = list(
      source = links_final$source,
      target = links_final$target,
      value =  links_final$value
    )
  ) %>% 
  layout(
    title = "Governo Federal 2018: Pagamentos realizados com a fonte CIDE (R$ Bi)",
    font = list(
      size = 10
    )
)

# Create a shareable link to your chart

#chart_link = api_create(p, filename="Pagamentos realizados com a fonte CIDE")
#chart_link
p

```

***   


### Todos os pagamento de 2018   

#### Detalhamento por esfera, indicador de resultado primário, fonte e elemento da despesa


Vamos ampliar o escopo da nossa análise. O gráfico abaixo traz todos os pagamentos efetuados em 2018. A informação é detalhada por esfera, indicador de resultado primário, fonte e elemento da despesa.

Destacamos, em vermelho, os recursos ordinários.   

Os recursos ordinários foram utilizados para pagar 51 tipos diferentes de elementos da despesa.


```{r, Governo Feredal 2018: pagamentos por esfera, indicador de resultado primário, fonte de recurso e elemento da despesa (R$ bi)}
outflux_s<- outflux%>%
  filter( outflux_reais>1, ano == "2018" )%>%
  group_by(esfera_out, res_primario, fonte, elemento)%>%
  summarise(value = sum(outflux_reais/1000000000))%>%
  arrange(desc(value ))

# arredondar valor para melhorar visualização da tabela

outflux_s$value<-round(outflux_s$value, 1)

names<- c()

# loop para extrair nomes dos nodes a partir da df outflux_s
for(i in 1:ncol(outflux_s)-1)
{
for(j in 1:NROW(outflux_s[,i]))
{
names <- c(names,outflux_s[j,i]) ;
}
}


# obeter apenas os nomes únicos.
name<- unique(as.character(names))

## reverter a ordem de nodes 

# criar nodes
nodes<- data.frame(cbind(node = c((0:(length(name)-1))), name))

nodes$node <- as.numeric(as.character(nodes$node))

## criar links

links <- outflux_s%>%
  group_by_at(1:2)%>%
  summarise(value = sum(value))
colnames(links)<-c("source","target","value")
##
links<-as.data.frame(links)

## criar links adcionais caso a df outflux_s tenha mais de duas variáveis
loop_link<-c("")

for(i in 1:(ncol(outflux_s)-3))
{
loop_link <- outflux_s%>%
  group_by_at((1+i):(2+i))%>%
  summarise(teste = sum(value))
loop_link<-as.data.frame(loop_link)
colnames(loop_link)<-c("source","target","value")

# juntar os links
links<-rbind(links,loop_link)
}

# mesclar nodes aos valores
links <- merge(links, nodes, by.x = "target", by.y = "name")
links <- merge(links, nodes, by.x = "source", by.y = "name")


links_final <- links[ , c("node.y", "node.x", "value")]
colnames(links_final) <- c("source", "target", "value")

p <- plot_ly(
    type = "sankey",
    orientation = "h",
    width = 1200, height = 1000,
    node = list(
      label = nodes$name,
      color = c(rep("lightgray",13),"red",rep("lightgray",130)),
      pad = 15,
      thickness = 20,
      line = list(
        color = "black",
        width = 0.5
      )
    ),

    link = list(
      source = links_final$source,
      target = links_final$target,
      value =  links_final$value
    )
  ) %>% 
  layout(
    title = "Governo Feredal 2018: pagamentos por esfera, indicador de resultado primário, fonte de recurso e elemento da despesa (R$ bi)",
    font = list(
      size = 7
    )
)

# Create a shareable link to your chart
# Set up API credentials: https://plot.ly/r/outflux
#chart_link = api_create(p, filename="despesa 2018")
#chart_link
p


```

***   


### Fonte e pagamentos de pensão e aposentadoria

O controle por fonte permite visualizar quais fontes o governo utilizou para pagar despesas com pensões e aposentadorias.

Destacamos, em vermelho, quatro pontos interessantes. Em termos nominais, no período de 2005 a 2018, o governo destinou para a seguridade social:

1) R$ 539 bi de recursos ordinários
2) R$ 197 bi de recursos provenientes da Remuneração das disponibilidades do Tesouro Nacional
3) R$ 163 bi de recursos provenientes de Títulos de responsabilidade do Tesouro Nacional
4) R$ 185 bi de recursos do orçamento fiscal para cobrir despesas com aposentadorias e pensões


```{r, Governo Federal 2005 a 2018: fontes de recurso utilizadas para pagamentos de pensão e aposentadoria (R$ Bi nominais)}

# Governo Federal 2005 a 2018: fontes de recurso utilizadas para pagamentos de pensão e aposentadoria (R$ Bi nominais)

outflux_s<- outflux%>%
  # filtrar pensao e aponsentadoria
  filter( outflux_reais>1, str_detect(elemento, "PENS|APOSEN" ))%>%
  group_by(fonte,esfera_out, elemento)%>%
  summarise(value = sum(outflux_reais/1000000000))%>%
  arrange(desc(value ))


# arredondar valor para melhorar visualização da tabela

outflux_s$value<-round(outflux_s$value, 1)

names<- c()

# loop para extrair nomes dos nodes a partir da df outflux_s
for(i in 1:ncol(outflux_s)-1)
{
for(j in 1:NROW(outflux_s[,i]))
{
names <- c(names,outflux_s[j,i]) ;
}
}


# obeter apenas os nomes únicos.
name<- unique(as.character(names))

## reverter a ordem de nodes 

# criar nodes
nodes<- data.frame(cbind(node = c((0:(length(name)-1))), name))

nodes$node <- as.numeric(as.character(nodes$node))


## criar links

links <- outflux_s%>%
  group_by_at(1:2)%>%
  summarise(value = sum(value))
colnames(links)<-c("source","target","value")
##
links<-as.data.frame(links)


## criar links adcionais caso a df outflux_s tenha mais de duas variáveis
loop_link<-c("")

for(i in 1:(ncol(outflux_s)-3))
{
loop_link <- outflux_s%>%
  group_by_at((1+i):(2+i))%>%
  summarise(teste = sum(value))
loop_link<-as.data.frame(loop_link)
colnames(loop_link)<-c("source","target","value")

# juntar os links
links<-rbind(links,loop_link)
}

# mesclar nodes aos valores
links <- merge(links, nodes, by.x = "target", by.y = "name")
links <- merge(links, nodes, by.x = "source", by.y = "name")


links_final <- links[ , c("node.y", "node.x", "value")]
colnames(links_final) <- c("source", "target", "value")


p <- plot_ly(
    type = "sankey",
    orientation = "h",
    width = 1200, height = 1000,
    node = list(
      label = nodes$name,
       # quantidade de cores == lenght nodes
      color = c( rep("lightgray",2), "red", "lightgray", "red", "red",rep("lightgray",35),"red",rep("lightgray",8)),
      pad = 15,
      thickness = 20,
      line = list(
        color = "black",
        width = 0.5
      )
    ),

    link = list(
      source = links_final$source,
      target = links_final$target,
      value =  links_final$value
    )
  ) %>% 
  layout(
    title = "Governo Federal 2005 a 2018: fontes de recurso utilizadas para pagamentos de pensão e aposentadoria (R$ Bi nominais)",
    font = list(
      size = 7
    )
)

# Create a shareable link to your chart
# Set up API credentials: https://plot.ly/r/outflux
#chart_link = api_create(p, filename="pensão aposentadoria")
#chart_link
p


```

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


```{r, saldo DRU x Fonte 100, fig.width = 12,  fig.height= 9}
#Recursos de contribuições sociais carimbados como fonte 100 (recursos ordinários)
dru<-  nat_rec%>%
  filter(especie_cod == "121", fonte_cod == "100" )

# recursos ordinários utilizados para despesas com o orçamento da seguridade social
ord<-outflux%>%
  filter(fonte_cod == "100", esfera_out_cod == "S" )


dru_ano<-dru%>%
  group_by( ano)%>%
  summarise(value = sum(influx_reais/1000000000))

#dru_ano<-dru_ano%>%
 # mutate(tipo = "dru")


ord_ano<-ord%>%
  group_by( ano)%>%
  summarise(value = sum(outflux_reais/-1000000000))
#comparar DRU x recursos do orçamento fiscal destinados à seguridade social

#ord_ano<-ord_ano%>%
 # mutate(tipo = "fonte_100")


dru_ord<- merge(ord_ano, dru_ano, by.x = "ano", by.y = "ano")

dru_ord<- dru_ord %>%
  mutate(value=value.x+value.y)

fluxo<- rbind(ord_ano, dru_ano)

fluxo<- fluxo%>%
  mutate(tipo = ifelse(value>0, "DRU", "Recursos Ordinários"))%>%
  arrange((tipo))

fluxo$value<-round(fluxo$value,1)

p <- plot_ly(dru_ord)%>%
  ggplot( aes(x = ano, y = value)) +
  geom_bar(data = fluxo, aes(fill = tipo), stat = "identity") +
  geom_point(color = "white", size =0.5)+
  geom_line(group=1, color = "gray")+
  theme_bw()+
  ggtitle("DRU (azul) x recursos ordinários destinados à seguridade social (vermelho)", "Governo Federal 2005 a 2018 (R$ bi nominais)")+
  xlab("")+
  ylab("")+
  theme(
    panel.grid.major.x = element_blank(),
    panel.grid.minor.x = element_blank()
  )+
     scale_fill_manual(
    values = c("blue",  "#ff0000"))+
  geom_text(aes(label = sprintf("%0.1f", round(value, digits = 1))), size= 3.5, color = "white"  ,vjust=-0.25, position = position_dodge(0.1))+
  theme(axis.text.x = element_text(size = 9, hjust = 0, vjust = 0), legend.position="top")

p



#chart_link = api_create(p, filename="dru x fontes 100")
#chart_link


```

***   


### DRU x Fontes fiscais

Para a nossa análise ficar mais completa, devemos incluir outras fontes que irrigam o orçamento da seguridade social.

Abaixo exibimos o gráfico após a inclusão de duas fontes de recursos:
1) Remuneração das disponibilidades do Tesouro Nacional
2) Títulos de responsabilidade do Tesouro Nacional.

Com a inclusão das duas fontes a seguridade social passa a apresentar um déficit crescente.

```{r, saldo DRU x fontes fiscais, fig.width = 12,  fig.height= 9}
#Recursos de contribuições sociais carimbados como fonte 100 (recursos ordinários)
dru_1<-  nat_rec%>%
  filter(especie_cod == "121", fonte_cod == "100" )

# recursos ordinários utilizados, remuneração conta única e títulos para despesas com o orçamento da seguridade social
ord_1<-outflux%>%
  filter(fonte_cod %in% c("100","188","144"), esfera_out_cod == "S" )


dru_ano_1<-dru_1%>%
  group_by( ano,fonte)%>%
  summarise(value = sum(influx_reais/1000000000))

#dru_ano<-dru_ano%>%
 # mutate(tipo = "dru")


ord_ano_1<-ord_1%>%
  group_by( ano,fonte)%>%
  summarise(value = sum(outflux_reais/-1000000000))
#comparar DRU x recursos do orçamento fiscal destinados à seguridade social

#ord_ano<-ord_ano%>%
 # mutate(tipo = "fontes_fiscais")
ord_ano_2<-ord_1%>%
  group_by( ano)%>%
  summarise(value = sum(outflux_reais/-1000000000))

ord_ano_2<- ord_ano_2%>%
  arrange(value)

dru_ord_1<- merge(ord_ano_2, dru_ano_1, by.x = "ano", by.y = "ano")

dru_ord_1<- dru_ord_1 %>%
  mutate(value=value.x+value.y)

fluxo_1<- rbind(ord_ano_1, dru_ano_1)

fluxo_1<- fluxo_1%>%
  mutate(tipo = ifelse(value>0, "DRU", fonte))%>%
  arrange((tipo))


fluxo_1$value<-round(fluxo_1$value,1)

p <- plot_ly(dru_ord_1)%>%
  ggplot( aes(x = ano, y = value)) +
  geom_bar(data = fluxo_1, aes(fill = tipo), stat = "identity") +
  geom_point(color = "white", size =0.5)+
  geom_line(group=1, color = "gray")+
  theme_bw()+
  ggtitle("DRU (azul) x recursos fiscais destinados à seguridade social", "Governo Federal 2005 a 2018 (R$ bi nominais)")+
  xlab("")+
  ylab("")+
  theme(
    panel.grid.major.x = element_blank(),
    panel.grid.minor.x = element_blank(),
    legend.text = element_text(size = 8)
  )+
     scale_fill_manual(
    values = c("blue", "#ff0000", "#4d4d4d", "#b3b3b3"  ),
    breaks = c("DRU", "RECURSOS ORDINARIOS", "REMUNERACAO DAS DISPONIB. DO TESOURO NACIONAL", "TITULOS DE RESPONSABILID.DO TESOURO NACIONAL"),
    labels = c("DRU", "Recursos Ordinários", "Remuneração das Disponibilidades", "Títulos do Tesouro")
  )+
  geom_text(aes(label = sprintf("%0.1f", round(value, digits = 1))), size= 3.5, color = "white"  ,vjust=-0.25 , position = position_dodge(0.1))+
  theme(axis.text.x = element_text(size = 9, hjust = 0, vjust = 0),legend.position="top")


p


#chart_link = api_create(p, filename="dru x fontes fiscais")
#chart_link


```

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





