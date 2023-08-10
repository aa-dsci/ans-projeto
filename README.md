![](https://github.com/aa-dsci/ans-projeto/blob/main/ans-title.png)

# Registro de Reclamações - Agência Nacional de Saúde Suplementar (ANS)

- __Descrição__: Projeto criado para retratar uma situação hipotética,mas, factível de uma análise de negócios utilizando o registro de reclamações dos usuários junto à ANS.
- __Objetivos__: Fornecer insights para atuação de um escritório de advocacia que busca adentrar no ramo hospitalar. As perguntas a serem respondidas são:
    - Considerando as reclamações como proxy de potenciais casos; há alguma tendência ao longo dos anos que justifique investimento no setor hospitalar?
    - Há indicativo de alguma categoria de empresa no setor que precise de maior investimento?
    - Há alguma sazonalidade no ano em que o número de reclamações é maior ou menor?

----
## 1. Introdução

Este projeto será apresentado por diferentes seções. Estas seções separam as principais etapas de um projeto de Ciência de dados e Análise de Negócios que devem ocorrer sequencialmente para que o resultado seja efetivo ao auxiliar a tomada de decisão por parte de gestores. Além disso, a ordem cronológica deste texto seguirá os passos que foram dados para obter cada resultado, de modo que o leitor possua uma experiência imersiva e seja capaz de reproduzir, caso seja de sua vontade, os processos executados.

__Etapas do projeto:__ 
- Justicativa do projeto
- Extração e limpeza de dados
- Análise exploratória
- Análise estatística
- Produtos
- Discussão crítica e conclusão

----
## 2. Justificativa do projeto
Em um mundo que cada vez mais há a necessidade de tomar as decisões baseadas em evidências, é importante que não haja disperdício de dois recursos valiosos para qualquer organização: tempo e dinheiro. No Brasil há o Sistema Único de Saúde, que integra tanto a rede pública quanto a particular na cobertura de serviços prestados à população. Uma parcela significativa da população opta pela rede privada por questões como agilidade e ampla cobertura de serviços,mas, por muitas vezes, se encontra insatisfeita com os serviços prestados pelas redes particulares. 

Esta insatisfação é acentuada porque é o próprio dinheiro do consumidor que está comprometido quando um serviço não é bem prestado, e sua queixa, a depender do caso, pode gerar processos judiciais contra o estabelecimento de serviços médicos em questão. Considerando este contexto, este projeto irá proporcionar uma primeira abordagem do cenário brasileiro de reclamações contra instituições privadas de saúde em que sejam identificados as melhores oportunidades de negócio para escritórios de advocacia captarem clientes e redes de hospital que busquem melhorar seu atendimento de modo a evitar novas chances de processo judicial.

Todas as etapas de codificação foram realizadas no __R Studio__ e sua posterior visualização de dados com __Office 365__ e o __Microsoft Power BI__. 

## 3. Extração e limpeza de dados

Os dados foram obtidos no seguinte [link](https://dados.gov.br/dados/conjuntos-dados/indice-geral-de-reclamacoes---igr-metodologia-ate-2022). Foi selecionado o intervalo de cinco anos entre 2015 e 2020 para esta avaliação. Os dados correspondem ao Índice Geral de Reclamação; que correspondem ao nível de insatisfação dos clientes com a empresa em determinado mês, onde o valor 0 (zero) representa "nenhuma insatisfação".

```
##Os dados originais apresentam a coluna Competência que refere-se ao mês/ano em uma única linha.
##Desta forma, o resultado para cada coluna de mês repete-se 12 vezes (12 meses do ano).
##Para limpar estes dados, realizei uma subamostragem apenas com a mesma competência: 20XX12 onde 'X' refere-se aos últimos dois dígitos do ano da tabela.

 
IGR_2015 <- read_delim("alex/IGR_2015.csv",
                        delim = ";", escape_double = FALSE, trim_ws = TRUE)
IGR_2016 <- read_delim("alex/IGR_2016.csv",
                        delim = ";", escape_double = FALSE, trim_ws = TRUE)
IGR_2017 <- read_delim("alex/IGR_2017.csv",
                       delim = ";", escape_double = FALSE, trim_ws = TRUE)
IGR_2018 <- read_delim("alex/IGR_2018.csv",
                       delim = ";", escape_double = FALSE, trim_ws = TRUE)
IGR_2019 <- read_delim("alex/IGR_2019.csv",
                       delim = ";", escape_double = FALSE, trim_ws = TRUE)
IGR_2020 <- read_delim("alex/IGR_2020.csv",
                       delim = ";", escape_double = FALSE, trim_ws = TRUE)
subset_IGR_2015<-IGR_2015[which(IGR_2015$Competencia==201512),]
subset_IGR_2016<-IGR_2016[which(IGR_2016$Competencia==201612),]
subset_IGR_2017<-IGR_2017[which(IGR_2017$Competencia==201712),]
subset_IGR_2018<-IGR_2018[which(IGR_2018$Competencia==201812),]
subset_IGR_2019<-IGR_2019[which(IGR_2019$Competencia==201912),]
subset_IGR_2020<-IGR_2020[which(IGR_2020$Competencia==202012),]
```

Em seguida foi realizada a união de todas as tabelas anuais em um único _dataframe_ :

```
IGR_temporario = merge (subset_IGR_2015,subset_IGR_2016, by="Razao Social (Registro ANS)")
IGR_temporario = merge (IGR_temporario,subset_IGR_2017, by="Razao Social (Registro ANS)")
IGR_temporario = merge (IGR_temporario,subset_IGR_2018, by="Razao Social (Registro ANS)")
IGR_temporario = merge (IGR_temporario,subset_IGR_2019, by="Razao Social (Registro ANS)")
IGR_temporario = merge (IGR_temporario,subset_IGR_2020, by="Razao Social (Registro ANS)")
```

Ao aplicar a função "colnames(IGR_temporario)" [percebemos que há colunas que se repetem](https://github.com/aa-dsci/ans-projeto/blob/main/ans-ss1.png) por conta da fusão entre os cinco _dataframes_ anteriores.
Há a necessidade, portanto, de se manter apenas as colunas de interesse, de acordo com índice delas:
```
#Filtrando apenas as colunas de interesse

IGR<-IGR_temporario[,-c(4,5,18,19,20,21,22,23,36,37,38,39,40,41,54,55,56,57,58,59,72,73,74,75,76,77,90,91,92,93,94,95,108,109)]
```

É comum que em uma base de dados bruta haja valores não correspondentes ao tipo apropriado de dado. Problemas como caracteres de texto, espaços em branco ou com símbolos que não correspondem a sinais matemáticos em uma coluna numérica, por exemplo, causam problemas em posteriores análises estatísticas. Esses termos não-aplicáveis são conhecidos pela abreviação "NA".
```
#Identificando valores NA
#Nesta primeira etapa o comando is.na converte a tabela em uma matriz do tipo booleano com valores 'TRUE' ou 'FALSO' para o comando que afirma que a célula é N.A

is.na(IGR)
#Para se ter uma noção se são erros pontuais ou alguma coluna inteira possui valores inválidos, cabe somar para cada coluna a quantidade de NA como verdadeiro identificados

colSums(is.na(IGR))
#Além disso, caso seja do interesse da equipe, pode-se identificar o índice das colunas que possuem valores NA como verdadeiro.

which(colSums(is.na(IGR))>0)

#Como a quantidade de NA's variou, em seus maiores valores, entre 25 e 39 em um universo de 891 observações, avalio que não há necessidade de excluir nenhum pool de dados ou série anual deste projeto 

#Por fim, pode-se substituir os valores NA por zero.
IGR_0<-replace(IGR,is.na(IGR),0)

#Uma boa prática consiste em repassar os comandos anteriores relacionados à identificação de valores NA no objeto "IGR_0" para confirmar que o procedimento foi executado corretamente.

#Exportando para CSV
write.csv(IGR,"alex/IGR_editado.csv", row.names = FALSE)

```
O arquivo "IGR_editado.csv" foi preparado para gerar uma versão com informações condensadas que servirão de apoio para a etapa da análise exploratória. Esta preparação envolveu a reordenação das colunas por meses, seguido de posterior exportação, conforme se vê abaixo:

```
#ordendando os meses para inserção no BI de dados sumarizados


meses_ordenado <-subset(IGR_editado, select = c("jan.15","jan.16","jan.17","jan.18","jan.19","jan.20",
                                                "fev.15","fev.16","fev.17","fev.18","fev.19","fev.20",
                                                "mar.15","mar.16","mar.17","mar.18","mar.19","mar.20",
                                                "abr.15","abr.16","abr.17","abr.18","abr.19","abr.20",
                                                "mai.15","mai.16","mai.17","mai.18","mai.19","mai.20",
                                                "jun.15","jun.16","jun.17","jun.18","jun.19","jun.20",
                                                "jul.15","jul.16","jul.17","jul.18","jul.19","jul.20",
                                                "ago.15","ago.16","ago.17","ago.18","ago.19","ago.20",
                                                "set.15","set.16","set.17","set.18","set.19","set.20",
                                                "out.15","out.16","out.17","out.18","out.19","out.20",
                                                "nov.15","nov.16","nov.17","nov.18","nov.19","nov.20",
                                                "dez.15","dez.16","dez.17","dez.18","dez.19","dez.20"))

 write.csv(IGR,"alex/meses_ordenado.csv", row.names = FALSE)
```

Em seguida, foi criado um arquivo XLSX (Medidas_ANS.xlsx) a partir do arquivo "meses_ordenado.csv" contendo medidas gerais em função do tempo para inserir como consulta no Power BI.

----
![](https://github.com/aa-dsci/ans-projeto/blob/main/ans-ss2.png)


## 4. Análise exploratória

Nesta etapa foi utilizado o aplicativo Microsoft Power BI para visualização dos dados e observar quais perguntas os dados nos incitam a fazer. Com a criação de um novo projeto na plataforma, optei por utilizar dois arquivos que serviram para alimentar as consultas no Power Query: __IGR_editado.csv__ e __Medidas_ANS.xlsx__. 

Para a consulta "IGR_editado", foram aplicadas as etapas de alteração de fonte, adequação dos tipos de dados para cada coluna, renomeação de colunas, separação da coluna 'Razão Social (Registro ANS)' em duas separadas e criação de somatório do índce de reclamação por empresa. A codificação M correspondente à essas alterações encontra-se a seguir:
```
let
    Fonte = Csv.Document(File.Contents("D:\Documentos\alex\IGR_editado.csv"),[Delimiter=";", Columns=75, Encoding=1252, QuoteStyle=QuoteStyle.None]),
    #"Tipo Alterado" = Table.TransformColumnTypes(Fonte,{{"Column1", type text}, {"Column2", type text}, {"Column3", type text}, {"Column4", type text}, {"Column5", type text}, {"Column6", type text}, {"Column7", type text}, {"Column8", type text}, {"Column9", type text}, {"Column10", type text}, {"Column11", type text}, {"Column12", type text}, {"Column13", type text}, {"Column14", type text}, {"Column15", type text}, {"Column16", type text}, {"Column17", type text}, {"Column18", type text}, {"Column19", type text}, {"Column20", type text}, {"Column21", type text}, {"Column22", type text}, {"Column23", type text}, {"Column24", type text}, {"Column25", type text}, {"Column26", type text}, {"Column27", type text}, {"Column28", type text}, {"Column29", type text}, {"Column30", type text}, {"Column31", type text}, {"Column32", type text}, {"Column33", type text}, {"Column34", type text}, {"Column35", type text}, {"Column36", type text}, {"Column37", type text}, {"Column38", type text}, {"Column39", type text}, {"Column40", type text}, {"Column41", type text}, {"Column42", type text}, {"Column43", type text}, {"Column44", type text}, {"Column45", type text}, {"Column46", type text}, {"Column47", type text}, {"Column48", type text}, {"Column49", type text}, {"Column50", type text}, {"Column51", type text}, {"Column52", type text}, {"Column53", type text}, {"Column54", type text}, {"Column55", type text}, {"Column56", type text}, {"Column57", type text}, {"Column58", type text}, {"Column59", type text}, {"Column60", type text}, {"Column61", type text}, {"Column62", type text}, {"Column63", type text}, {"Column64", type text}, {"Column65", type text}, {"Column66", type text}, {"Column67", type text}, {"Column68", type text}, {"Column69", type text}, {"Column70", type text}, {"Column71", type text}, {"Column72", type text}, {"Column73", type text}, {"Column74", type text}, {"Column75", type text}}),
    #"Cabeçalhos Promovidos" = Table.PromoteHeaders(#"Tipo Alterado", [PromoteAllScalars=true]),
    #"Tipo Alterado1" = Table.TransformColumnTypes(#"Cabeçalhos Promovidos",{{"Razao Social (Registro ANS)", type text}, {"Cobertura.x", type text}, {"Porte.x", type text}, {"dez/15", type number}, {"nov/15", type number}, {"out/15", type number}, {"set/15", type number}, {"ago/15", type number}, {"jul/15", type number}, {"jun/15", type number}, {"mai/15", type number}, {"abr/15", type number}, {"mar/15", type number}, {"fev/15", type number}, {"jan/15", type number}, {"dez/16", type number}, {"nov/16", type number}, {"out/16", type number}, {"set/16", type number}, {"ago/16", type number}, {"jul/16", type number}, {"jun/16", type number}, {"mai/16", type number}, {"abr/16", type number}, {"mar/16", type number}, {"fev/16", type number}, {"jan/16", type number}, {"dez/17", type number}, {"nov/17", type number}, {"out/17", type number}, {"set/17", type number}, {"ago/17", type number}, {"jul/17", type number}, {"jun/17", type number}, {"mai/17", type number}, {"abr/17", type number}, {"mar/17", type number}, {"fev/17", type number}, {"jan/17", type number}, {"dez/18", type number}, {"nov/18", type number}, {"out/18", type number}, {"set/18", type number}, {"ago/18", type number}, {"jul/18", type number}, {"jun/18", type number}, {"mai/18", type number}, {"abr/18", type number}, {"mar/18", type number}, {"fev/18", type number}, {"jan/18", type number}, {"dez/19", type number}, {"nov/19", type number}, {"out/19", type number}, {"set/19", type number}, {"ago/19", type number}, {"jul/19", type number}, {"jun/19", type number}, {"mai/19", type number}, {"abr/19", type number}, {"mar/19", type number}, {"fev/19", type number}, {"jan/19", type number}, {"dez/20", type number}, {"nov/20", type number}, {"out/20", type number}, {"set/20", type number}, {"ago/20", type number}, {"jul/20", type number}, {"jun/20", type number}, {"mai/20", type number}, {"abr/20", type number}, {"mar/20", type number}, {"fev/20", type number}, {"jan/20", type number}}),
    #"Colunas Renomeadas" = Table.RenameColumns(#"Tipo Alterado1",{{"Cobertura.x", "Cobertura"}, {"Porte.x", "Porte"}}),
    #"Dividir Coluna por Delimitador" = Table.SplitColumn(#"Colunas Renomeadas", "Razao Social (Registro ANS)", Splitter.SplitTextByDelimiter("(", QuoteStyle.Csv), {"Razao Social (Registro ANS).1", "Razao Social (Registro ANS).2", "Razao Social (Registro ANS).3"}),
    #"Tipo Alterado2" = Table.TransformColumnTypes(#"Dividir Coluna por Delimitador",{{"Razao Social (Registro ANS).1", type text}, {"Razao Social (Registro ANS).2", type text}, {"Razao Social (Registro ANS).3", type text}}),
    #"Colunas Removidas" = Table.RemoveColumns(#"Tipo Alterado2",{"Razao Social (Registro ANS).3"}),
    #"Colunas Renomeadas1" = Table.RenameColumns(#"Colunas Removidas",{{"Razao Social (Registro ANS).1", "Razão Social"}, {"Razao Social (Registro ANS).2", "Registro ANS"}}),
    #"Valor Substituído" = Table.ReplaceValue(#"Colunas Renomeadas1",")","",Replacer.ReplaceText,{"Registro ANS"}),
    #"Erros Removidos" = Table.RemoveRowsWithErrors(#"Valor Substituído", {"out/19"}),
    #"Soma Inserida" = Table.AddColumn(#"Erros Removidos", "Adição", each List.Sum({[#"dez/15"], [#"nov/15"], [#"out/15"], [#"set/15"], [#"ago/15"], [#"jul/15"], [#"jun/15"], [#"mai/15"], [#"abr/15"], [#"mar/15"], [#"fev/15"], [#"jan/15"], [#"dez/16"], [#"nov/16"], [#"out/16"], [#"set/16"], [#"ago/16"], [#"jul/16"], [#"jun/16"], [#"mai/16"], [#"abr/16"], [#"mar/16"], [#"fev/16"], [#"jan/16"], [#"dez/17"], [#"nov/17"], [#"out/17"], [#"set/17"], [#"ago/17"], [#"jul/17"], [#"jun/17"], [#"mai/17"], [#"abr/17"], [#"mar/17"], [#"fev/17"], [#"jan/17"], [#"dez/18"], [#"nov/18"], [#"out/18"], [#"set/18"], [#"ago/18"], [#"jul/18"], [#"jun/18"], [#"mai/18"], [#"abr/18"], [#"mar/18"], [#"fev/18"], [#"jan/18"], [#"dez/19"], [#"nov/19"], [#"out/19"], [#"set/19"], [#"ago/19"], [#"jul/19"], [#"jun/19"], [#"mai/19"], [#"abr/19"], [#"mar/19"], [#"fev/19"], [#"jan/19"], [#"dez/20"], [#"nov/20"], [#"out/20"], [#"set/20"], [#"ago/20"], [#"jul/20"], [#"jun/20"], [#"mai/20"], [#"abr/20"], [#"mar/20"], [#"fev/20"], [#"jan/20"]}), type number),
    #"Colunas Renomeadas2" = Table.RenameColumns(#"Soma Inserida",{{"Adição", "Total por Empresa"}})
in
    #"Colunas Renomeadas2"

```

Importando o arquivo Medidas_ANS.xlsx; foram criadas duas consultas, a partir de suas: "Mês" e "Ano". Nestas consultas foram realizadas como tratamento apenas a adequação do tipo de dado e a promoção de cabeçalhos.

```

//Editando a consulta Mês
let
    Fonte = Excel.Workbook(File.Contents("D:\Documentos\alex\Medidas_ANS.xlsx"), null, true),
    Plan1_Sheet = Fonte{[Item="Plan1",Kind="Sheet"]}[Data],
    #"Cabeçalhos Promovidos" = Table.PromoteHeaders(Plan1_Sheet, [PromoteAllScalars=true]),
    #"Tipo Alterado" = Table.TransformColumnTypes(#"Cabeçalhos Promovidos",{{"Soma", type number}, {"Mês", type text}, {"Porte - Grande", type number}, {"Porte - Médio", type number}, {"Porte - Pequeno", type number}})
in
    #"Tipo Alterado"

//Editando a consulta Ano
let
    Fonte = Excel.Workbook(File.Contents("D:\Documentos\alex\Medidas_ANS.xlsx"), null, true),
    Plan2_Sheet = Fonte{[Item="Plan2",Kind="Sheet"]}[Data],
    #"Cabeçalhos Promovidos" = Table.PromoteHeaders(Plan2_Sheet, [PromoteAllScalars=true]),
    #"Tipo Alterado" = Table.TransformColumnTypes(#"Cabeçalhos Promovidos",{{"Ano", Int64.Type}, {"Média", type number}, {"Soma", type number}, {"Porte - Grande", type number}, {"Porte - Médio", type number}, {"Porte - Pequeno", type number}})
in
    #"Tipo Alterado"

```

Após as primeiras visualizações, os seguintes Dashboards foram elaborados:
