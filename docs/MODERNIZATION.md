# Modernização experimental do designer

Este ramo atualiza somente o fork do iReport. O ERP, seus relatórios e o perfil habitual do NetBeans não fazem parte da migração. O código mantém a licença AGPL e os avisos do upstream. O fork ainda é local, sem publicação de release.

## Alvo e alterações

- Apache NetBeans 31 (`RELEASE310`), execução do IDE em JDK 21 ou 25.
- JasperReports e chart themes 6.21.4; POI 5.3.0; Groovy 4.0.28 no designer.
- Adaptação das APIs removidas de fontes, alinhamento de texto/imagem, gráficos, exportadores e datasets.
- Preservação de tamanho de fonte fracionário e do comportamento de desfazer/refazer.
- Registro de MIME por anotação e correção de atalhos inválidos no layer.
- Processamento de anotações explicitamente habilitado para builds com JDK 25.
- Remoção da alteração global de provedores XML feita pelo instalador antigo.
- Gravação mantém uma referência forte ao documento e atualiza seu conteúdo de forma síncrona antes de escrever o arquivo, conforme o ciclo de vida atual do editor NetBeans.
- Arquivos `.form` sincronizados nas propriedades visuais alteradas.

O JDK 17 serve para testar o motor de relatórios que executa no ERP. As APIs do NetBeans 31 exigem um JDK mais novo; executar este IDE no JDK 17 não é um alvo suportado.

## Construção e validação

Use Maven 3.9.9 e JDK 21 ou 25. Execute a preparação das bibliotecas legadas conforme `BASELINE.md`. O build fica restrito ao módulo designer e suas dependências:

```powershell
./scripts/Build-Designer.ps1 -JavaHome 'C:/Program Files/Java/jdk-21.0.10'
```

Para uma construção limpa e teste do corpus, aponte exclusivamente para cópias de laboratório:

```powershell
mvn -B -ntp -s build-settings.xml -pl ireport-designer -am clean package '-Direport.test.corpus=D:/Desenvolvimento/AtheneSistemas/ireport-modernizacao/laboratorio/fixtures/athene-snapshot'
./scripts/Test-EngineCompatibility.ps1 -JavaHome 'C:/Program Files/Java/jdk-25' -SaveGroovy
./scripts/Test-EngineCompatibility.ps1 -JavaHome 'C:/Program Files/Java/jdk-17' -ErpGroovy
```

O teste do motor deriva o classpath do relatório Surefire e exclui as APIs NetBeans. A opção `ErpGroovy` substitui Groovy 4 por 3.0.20, já presente no repositório Maven local. Ela não reproduz todas as dependências ou a execução completa do ERP.

## Perfil isolado

```powershell
./scripts/New-LabProfile.ps1 -Name validacao-nb31
./scripts/Start-LabNetBeans.ps1 -Name validacao-nb31 -NetBeansHome 'D:/Desenvolvimento/netbeans-31/netbeans' -JavaHome 'C:/Program Files/Java/jdk-25'
```

O staging cria um perfil novo no diretório irmão `laboratorio`, não altera o perfil habitual e rejeita nomes reutilizados. Não use o arquivo NBM diretamente no perfil de produção enquanto os critérios pendentes não estiverem concluídos. O rollback do laboratório consiste em fechar seu IDE e voltar a usar o perfil habitual, que não foi atualizado.

Para compatibilidade dos relatórios Groovy com Java 17/Groovy 3, o lançador aplica `-J-Dgroovy.target.bytecode=17` e `-J-Dgroovy.target.indy=false`. Sem essas opções, um relatório compilado no JDK 25 produziu bytecode 69 e falhou no Java 17; com elas, a transferência e o preenchimento passaram. Abrir o plugin por outro lançador requer as mesmas opções. Isso ainda precisa virar configuração própria do compilador antes de uma distribuição geral.

## Evidências obtidas em 06/10/2026

| Validação | Resultado e alcance |
|---|---|
| Package JDK 21 | Passou; cinco testes de regressão, corpus opcional separado |
| Clean package JDK 25 | Passou; pacote candidato posterior passou em sete testes, incluindo corpus e conversão de compilado moderno |
| Leitura/regravação do corpus | 215 cópias JRXML passaram em load/write/load/write canônico; não prova equivalência visual ou de dados |
| JRXML moderno | Preservou `textAdjust`, `ContainerHeight`, fonte 10.5, alinhamento e `ignorePagination` |
| Compilar/preencher/exportar | Java e Groovy sintéticos; PDF e XLS verificados automaticamente |
| Transferência para Java 17 | Java e Groovy compilados no laboratório preenchidos e exportados em Java 17; Groovy 3.0.20 no destino |
| Abertura e edição visual | Exemplo moderno abriu nos JDKs 21 e 25; fonte 11,5 gravada no JDK 25, reabertura após reinício em JDK 21 e preview aprovados |
| Conversão de compilado moderno | Compilar/serializar/carregar/converter `.jasper` 6.21.4 para JRXML preservou os atributos verificados |
| Proteção do ERP | SHA-256 dos 3.102 arquivos monitorados: nenhuma diferença |

Os logs completos ficam no diretório pai do fork. O corpus registra resultados em `ireport-designer/target/corpus-results/results.tsv`. Saídas de regressão e compilados ficam em `target/compatibility-results`; nunca são escritos sobre os originais.

## Critérios ainda pendentes para release

- Ampliar os testes de edição e de desfazer/refazer pela interface nos dois JDKs. O exemplo básico foi salvo, fechado, reaberto após reinício e pré-visualizado. O upstream usa `PERSISTENCE_NEVER` para as abas; foi observado aviso sobre descrição MultiView não serializável. Não foi implementada restauração automática de abas.
- Comparar visualmente relatórios representativos, subrelatórios, imagens, fontes, gráficos, crosstabs e formulários complexos, usando dados sintéticos.
- Verificar conversão de `.jasper` de versões suportadas e mensagens para arquivos incompatíveis.
- Validar impressão física em 58/80 mm e A4, margens, corte, paginação e cancelamento. O erro de espaço em branco e rodapé do cupom não foi corrigido nesta migração, pois os relatórios existentes estão fora do escopo.
- Testar instalação/atualização/desinstalação do NBM em perfil descartável, além do staging direto utilizado até aqui.
- Auditar dependências e licença, reduzir bibliotecas legadas, configurar compilação Groovy sem exigir flags do lançador, definir versão e política de distribuição.
- Validar módulos opcionais (JasperServer, Hive e aplicação standalone), que não foram incluídos neste build.

Este NBM é um candidato experimental. Build e regressões automatizadas aprovados não equivalem a homologação para o perfil de produção.
