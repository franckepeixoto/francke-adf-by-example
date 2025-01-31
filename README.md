
# Azure Data Factory - Francke Peixoto  
![](https://community.chocolatey.org/content/packageimages/azuredatafactorytools15.0.9.3527.2.png)

Diário de bordo para quem quer entender os principais aspectos do uso da plataforma **Azure Data Factory**

---
O ADF é um serviço nativo da nuvem para gerenciar workloads na integração de dados em lote.

# Termos/Conceitos chave
1.	**Azure tenant**
> É sua identificação como locatário (ou diretório) em instância do Azure Active Directory (ADD).
Ele identifica os usuários do Azure e permite que eles tenham acesso aos serviços do Azure. 
2.	**Resource**
>  Uma instância de um serviço do Azure, permitindo que o serviço seja acessado pelo susuarios de um tenant;
> * O **termo** resource é usado no portal azure para descrever diferentes instãncias de serviços _(ex: Uma conta de Storage ou um Data Factory)_ e é usado no ADF para descrever vários componentes, como pipelines e datasets. A *reutilização* da terminologia pode ser confusa, mas deve ficar claro no contexto se estamos nos referindo aos resources do Azure ou  aos resources do ADF.
3.	**Subscription**
> Especifica um meio de pagamento para serviços do Azure. Um tenant pode conter uma ou mais assinaturas. Cada resource doAzyre está associado a uma assinatura na qual as cobranças de consumo podem ser cobradas.
4.	**Blade**
>  É como são chamadaas as paginas no portal do Azure.
5. **Azure Data Factory (ADF)**
> É um serviço para gerenciar ETL em lote e outras workloads de integração de dados.
6. **Data Factory Instance**
> É uma instância do serviço ADF, muitas vezes referida simplemente como "data factory". A instância é um exemplo de resource.
7. **pipelines**
> Um pipiline é basicamente uma coleção de atividades de movimentação e transformação de dados, agrupadas para alcançar uma tarefa de integração.
8. **Actitivies**
> As actitivies estão disponíveis para movimentação nativa e transformação de dados dentro do ADF, bem como para orquestrar o trabalho realizado por recursos externos, como Databricks e serviços de ML.
> * Os nomes das atividades podem conter caracteres alfanuméricos, hífens,sublinhados e espaços, mas os nomes não podem começar ou terminar com um espaço.
> Seus nomes são validados pela interface do ADF à medida que você digita, portanto, ao adicionar um caractere de espaço, você receberá um erro de validação imediato;  Esse erro desaparece quando voc/çe insere o proximo caractere válido sem espaco.

# ADF & Git
> O ADF possui um forte acoplamento ao Git. Uma sessão do ADF não possui armazenamento permanete próprio, portanto, as alterações são salvas diretamente no seu repositório Git.
**Cada Save** que você **faz** no ADF é um **Git commit** que é imediatamente enviado para o seu repositório hosperado.
---
## Notas
> **Apache Parquet**: È um formato de armazenamento de dados estruturados compactados e orientado a colunas. 
Os aplicativos de Análise que processam um grnade volume de linhas se beneficiam de um formato orientado a colunas, pois dados em colunas fora do escopo da analise não precisam ser lidos.
Colunas individuais geramente contêm alguns valores distintos, permitindo que os dados da coluna sejam fortemente compactados para amazenanamento e recuperação eficientes.
> **Por esse motivo** o Parquet é frequentemente o formato de escolha para **armazenamento** em **data lake** de dados **tabulares**. 

> **Data Integration Units - DIUs**: O poder das atividades de dados de cópias é medido em DIUs, uma única medida que combina **CPU**, **Mémoria** e uso de **Rede**. 
Você pode aumentar esse poder alocando mais DIUS a sua tarefa,na aba de settings.
> * A configurçaão padrão para DIU é "Auto", o que significa que o número de DIUs alocados é determinado automaticamente, com base nas informações sobre  seus armazenamentos de dados de origem.
O Valor de DIUS por padrão é 4, para muitos cenários  reduzir esse valor de unidades mais barato nas execuções; Principalmente  em cenrários de aprendizado e desenvolvimento.
**Limitando o número de DIUs ao valor mínimo de dois.**
> *Mais informações sobre o desempenho das atividades de dados  de DIUs e Copias:* [Copiar recursos de otimização de desempenho de atividade](https://docs.microsoft.com/pt-br/azure/data-factory/copy-activity-performance-features) 

> **Grau de paralelismo de cópia** [_Degree of parallelism_ (DoP)]: Essa opção permite substituir o grau de paralelismo padrão da atividade de cópia de dados.
O grau de paralelismo realmente usado em tempo de execução aparece no campo **usedParallelCopies** do output. 
> * Uma atividade de cópia de dados pode ser executada em paralelo usando vários encademantos para ler arquivos diferentes simultaneamente. 
 O número máximo de **threads** usadas durante a execução de uma atividade é seu grau de paralelismo; o número pode ser definido manualmente para a tividade mas isso não é recomendado.
>> *Esse tipo de comportamento de dimensionamento automático é uma vantagem importante quando usamos serviços do tipo serverless.*

> **Infix Operators**: A expression language do ADF não contém infix ops - todas as operações são implementadas como funções. Da mesma forma que você usa **concat('xp','to')** em vez de **'xp' + 'to'**, todas as operações matemáticas são invocadas via funções.
Ex: *Para calcular 4+4, você deve usar a função add, como add(4,4)*
> **String Interpolation**: Uma string interpolada é um valor de string que contém expressões de espaço reservado que são availiados em tempo de execução. 
Uma expressão de espaço  reservado fica contida entre  chaves **{ }** e precedida por **@**, resultando em **@{ sua_expressao }**
- *No Expression builder: @concat('pipeline: ', @{pipeline().Pipeline}) pode ser substituido por:  pipeline: @{pipeline().Pipeline}}*
>> * Uma expressão é valida somente em Runtime do pipeline para determinar um valor.
>> * O tipo de dados de uma expressão é string,int,float,bool,array ou dictionary

> **Parâmetros Globais**: Geralmente usado para valores de origem constante e que devem ser compartilhados por todos os pipelines. Os parâmetros globais são referidos em expressões, dentro do escopo das atividades de pipeline.
*Ex: pipeline().globalParameters.NomeParametro*

> **Paralelismo**: Por default, as execuções da atividade  ForEach ocorrem em paralelo, exigindo cuidado par agarantir que as atividades de **iterações simultâneas** não inerfiram umas nas outras.
Neste cenário, a alteração de váriaveis deve ser evitada, mas a atividades **Execute Pipeline** fornece uma maneira fácil de isolar iterações.
*O ADF executa iterações  ForEach sequencialmente no modo  DEBUG, isso acaba complicando a detecção de falhas de paralelismo no momento do desenvolvimento.* 
---
### Data Flow
Os data flows fornecem uma rota lowcode para o poder do Azure Databricks. Eles são implementados usando um editor visual e convertidos automaticamente pelo adf em codigo Scala para execução em um cluster **Databricks** gerenciado.
Isso mesmo: *Embora acionado a partir de uma activity de pipeline, a executção dos data flows ocorrem em um cluster*
Um cluster deve ser provisionado antes que um fluxo de dados possa ser executado. 
O provisionamento de um cluster leva alguns minutos portanto, sua primeira tarefa ao desenvolver um data flow é **Ativar um cluster para Debug**

---
### Integration Runtimes - IR
Razões que pode te fazer querer criar seu própprio IR ao inves de usar o IR padrão são:
> * Controle do TTL para a execução de um cluster do Databricks.
> * Você precisa garantir que a movimentção de dados ocorra dentro de uma area geografica especifica.

---
### Deploy 
Em um deploy no ADF, o ideal é que seus resources sejam implantados no branch colaboarivo (dev); Essa abordagem se faz relevante pois ao clicar em Publish em sua instância de desenvolvimento, seu modelo ARM é gerado e armazenado no branch **adf_publish** do repositório.
_(Os modelos ARMs podem ser exportados de forma manual, no entanto é menos produtivo.)_
> * **Azure Resource Manager (ARM)** : Um modelo ARM é um arquivo JSON que define os compornentes de uma solução, por exemplo, o conteúdo de uma instância do Azure Data Factory.
> * **Publish**: ao exevutar um pipelne independetemente da insteface do ADF, ele deve ser implanetado em um ambiente; Os pipilenes publicados são executados usando triggers e suas execuções podem ser acompanhadas na Blade Monitor.
> * **adf_publish**: É o branch nomeado no repositório que contêm os modelos ARM produzido. 

---
O Azure Data Factory é o serviço ETL de nuvem do Azure para integração de dados sem servidor e transformação de dados em expansão. 
Ele oferece uma interface do usuário sem código para criação intuitiva e monitoramento e gerenciamento de painel único. 
Você também pode levantar e mudar os pacotes SSIS existentes para o Azure e executá-los com total compatibilidade no ADF. 
O SSIS Integration Runtime oferece um serviço totalmente gerenciado, para que você não precise se preocupar com o gerenciamento de infraestrutura. 
[Documentação do Azure Data Factory](https://docs.microsoft.com/en-us/azure/data-factory/).

Instâncias em curso: 
- [ADF Peixoto](https--//obfuscated--url-)
- [ADF Fundos Imobiliários & SmallCaps Bovespa](https--//obfuscated--url--)
- [ADF Kaggle](https--//obfuscated--url--)
- [ADF WIPO](https--//obfuscated--url--)
---
**Apress Source Code**
[*Azure Data Factory by Example*](https://www.apress.com/9781484270288) by Richard Swinbank (Apress, 2021).
