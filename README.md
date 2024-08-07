[StackPart] Observability¶
Situação da solução: Disponível
Propósito¶
O StackPart Observability simplifica a gestão de alertas das aplicações, possibilitando habilitar alertas padrões e criar alertas customizados.

Pré-Requisitos¶
AWS: Existir a conta de Plataforma.
Kubernetes: Existir o cluster do TechStack na conta.
Git e Pipeline: Ter um repositório da aplicação com pipeline EKS configurada.
Getting-Started: Ter entendimento sobre as configurações básicas do TechStack.
Como Utilizar¶
Por padrão o StackPart Observability já está habilitado, portanto não será necessário habilitá-lo no arquivo de configuração do TechStack.

_techstack/values_*.yml

global:
  # ...
  stackParts:
    # ...
    observability:
      enabled: true
Alertas¶
Configurações de integração com o CMR para acionamentos¶
Ao habilitar os alertas, eles automaticamente serão criados no Grafana. Quando forem disparados, será enviado um e-mail para a caixa de alertas cadastrada na conta de plataforma. Entretanto, para ambientes produtivos é recomendado a configuração do alerta com a integração do CMR.

O acionamento via CMR segue o seguinte fluxo:

Image title

É possível integrar automaticamente os alertas emitidos pelas suas aplicaçoes criadas vai Techstacks com o CMR. Para tal, é necessário habilitar a opção contactCMR e fornecer alguns campos extras que ajudarão o time do CMR a identificar o seu problema no Monitoramento e assim, acionar o time responsável pela peça de software.

Os campos abaixo são obrigatórios:

businessImpact: Deve ser colocado de forma objetiva, quais os possíveis impactos o mesmo pode causar e quem pode ser impactado. Se possível descrever qual o Serviço de Negócio esta ou pode estar sendo impactado;

description: Deve ser colocado do que se trata aquele desvio (métrica e sua explicação) com a maior quantidade de detalhes possível;

designatedGroup: Qual grupo de suporte o incidente gerado será enviado no ServiceNow (utilize este link para consultar o seu grupo)

Abaixo segue um exemplo de um alerta com metadados do CMR bem configurados:

_techstack/values_*.yml

global:
  # ...
  stackParts:
    # ...
    observability:
      enabled: true
      alerts:
        availability:
          enabled: true
          contactCmr: true
          designatedGroup: "PLATAFORMA REDEFLEX (S000620)"
          businessImpact: "Indisponibilidade do serviço de parametrização da Antecipação na plataforma Kosmos"
          description: "Aplicação Ma-BFF obteve erro 500, necessário intermediação para resolução do problema, esse erro afeta diretamente a parametrização da plataforma Kosmos. Deve acionar o plantonista da sigla MA1."
Atenção no parâmetro contactCmr em Prod

É imprescindível informar o time do CMR via e-mail (cmr@userede.com.br e grupo/distribution list LD_G_CMR) sempre que um novo alerta for criado neste modelo em Produção.

Alertas padrões¶
Com o StackPart Observability habilitado, é possível também habilitar e configurar os alertas padrões, conforme exemplos abaixo:


 Disponibilidade
 Tempo de resposta
 Response StatusCode 5xx
Por padrão o alerta de disponibilidade já está habilitado e dispara caso uma das instâncias da aplicação fique indisponível.

_techstack/values_*.yml

global:
# ...
  stackParts:
    # ...
    observability:
      enabled: true
      alerts:
        # ...
        availability:
          enabled: true
          contactCmr: true
          designatedGroup: CmrGrupoA
          businessImpact: Autorização de Pagamento [Rede, Boletos]
          description: >-
            Verifique se o sistema antifraudes está operacional em afirmativo
            entre em contato com a Squad AutorizaPagot.

global:
# ...
  stackParts:
    # ...
    observability:
      enabled: true
      alerts:
        # ...
        responseTime:
          enabled: true
          timeInMs: 100

global:
# ...
  stackParts:
    # ...
    observability:
      enabled: true
      alerts:
        # ...
        statusCode5xx:
          enabled: true
          contactCmr: true
          threshold: 1
          designatedGroup: CmrGrupoB
          businessImpact: Autorização de Pagamento [Rede, Boletos]
          description: >-
            Verifique se o SGBD Marte está com lock na tabela `transactions`,
            o mesmo acontece devido a problemas no sistema transacional, caso não.
            Entre em contato com a Squad AutorizaPagot.


Em casos de quebras de linhas para os campos designatedGroup, businessImpact e ou description utilize a quebra do bloco com a notação >-

Auto Instrumentação¶
A instrumentação de aplicativos elimina automaticamente a necessidade de adicionar linhas de código manualmente para enviar dados de rastreamento, através da medição remota automatizada e coleta de dados sobre seu aplicativo.

_techstack/values_*.yml

global:
  # ...
  stackParts:
    # ...
    observability:
      enabled: true
      autoInstrumentation: true
Atenção no parâmetro autoInstrumentation

Quando autoInstrumentation: true, deve-se considerar um novo tempo de setup da aplicação e a modificação do atributo initialDelaySeconds em readinessProbe e livenessProbe. Para mais informações, acessar HealthCheck.

Dicas¶
Detalhes de configuração
Chave	Tipo	Valor padrão	Obrigatório	Descrição
global.stackParts.observability.enabled	boolean	false	Não	Habilita o stackpart observability
global.stackParts.observability.alerts.contactCmr	boolean	false	Não	Habilita o envio do alerta para o CMR
global.stackParts.observability.alerts.serviceNowGroup	string	""	Não	Nome do grupo do Service Now para abertura de incidente
global.stackParts.observability.alerts.availability.enabled	boolean	true	Não	Habilita o alerta de disponibilidade. Dispara caso uma das instâncias da aplicação fique indisponível.
global.stackParts.observability.alerts.availability.checkInterval	string	"1m"	Não	Intervalo entre cada checagem
global.stackParts.observability.alerts.availability.waitingTimeToFire	string	"5m"	Não	Quanto tempo o Grafana irá aguardar para disparar o alerta
global.stackParts.observability.alerts.availability.contactCmr	boolean	"false"	Não	Habilita o envio do alerta para o CMR
global.stackParts.observability.alerts.availability.designatedGroup	string	""	*Requerido	Grupo designado do CMR
global.stackParts.observability.alerts.availability.businessImpact	string	""	*Requerido	Descrição do impacto de negócio
global.stackParts.observability.alerts.availability.description	string	""	*Requerido	Descrição detalhada para acionamento e resolução do problema ao CMR
global.stackParts.observability.alerts.responseTime.enabled	boolean	false	Não	Habilita o alerta de tempo de resposta. Dispara caso uma das instâncias da aplicação fique indisponível.
global.stackParts.observability.alerts.responseTime.checkInterval	string	"1m"	Não	Intervalo entre cada checagem
global.stackParts.observability.alerts.responseTime.waitingTimeToFire	string	"5m"	Não	Quanto tempo o Grafana irá aguardar para disparar o alerta
global.stackParts.observability.alerts.responseTime.percentile	float	0.99	Não	Percentil das requisições que será considerado para alerta (Max.: 1, que é igual a 100%)
global.stackParts.observability.alerts.responseTime.timeInMs	int	100	Sim	Tempo de resposta que será considerado para alerta
global.stackParts.observability.alerts.responseTime.contactCmr	boolean	"false"	Não	Habilita o envio do alerta para o CMR
global.stackParts.observability.alerts.responseTime.designatedGroup	string	""	*Requerido	Grupo designado do CMR
global.stackParts.observability.alerts.responseTime.businessImpact	string	""	*Requerido	Descrição do impacto de negócio
global.stackParts.observability.alerts.responseTime.description	string	""	*Requerido	Descrição detalhada para acionamento e resolução do problema ao CMR
global.stackParts.observability.alerts.statusCode5xx.enabled	boolean	false	Não	Habilita o alerta de response status code 5xx. Dispara caso uma das instâncias da aplicação fique indisponível.
global.stackParts.observability.alerts.statusCode5xx.checkInterval	string	"1m"	Não	Intervalo entre cada checagem
global.stackParts.observability.alerts.statusCode5xx.waitingTimeToFire	string	"1m"	Não	Quanto tempo o Grafana irá aguardar para disparar o alerta
global.stackParts.observability.alerts.statusCode5xx.threshold	int	1	Não	Qual o número de ocorrências será considerado para alertar
global.stackParts.observability.alerts.statusCode5xx.contactCmr	boolean	"false"	Não	Habilita o envio do alerta para o CMR
global.stackParts.observability.alerts.statusCode5xx.designatedGroup	string	""	*Requerido	Grupo designado do CMR
global.stackParts.observability.alerts.statusCode5xx.businessImpact	string	""	*Requerido	Descrição do impacto de negócio
global.stackParts.observability.alerts.statusCode5xx.description	string	""	*Requerido	Descrição detalhada para acionamento e resolução do problema ao CMR
Requerido, somente se alerts.<alertType>.contactCmr contiver o valor true.
Especificação¶
Solução Técnica
Os alertas permitem que você aprenda sobre problemas em suas aplicações momentos após eles ocorrerem. Crie, gerencie e execute ações em seus alertas em uma visão única e consolidada e melhore a capacidade de sua equipe de identificar e resolver problemas rapidamente.

Solução Técnica¶
O diagrama a seguir fornece uma visão geral de como o Grafana Alerting funciona e apresenta alguns dos principais conceitos que funcionam juntos e formam o núcleo de nosso mecanismo de alerta flexível e poderoso.

Image title

Visão geral do Grafana Alerting¶
Alert rules

Defina os critérios de avaliação que determinam se uma instância de alerta será disparada. Uma regra de alerta consiste em uma ou mais consultas e expressões, uma condição, a frequência de avaliação e, opcionalmente, a duração durante a qual a condição é atendida.

Os alertas gerenciados do Grafana oferecem suporte a alertas multidimensionais, o que significa que cada regra de alerta pode criar várias instâncias de alerta. Isso é excepcionalmente poderoso se você estiver observando várias séries em uma única expressão.

Depois que uma regra de alerta é criada, ela passa por vários estados e transições. O estado e a integridade das regras de alerta ajudam você a entender vários indicadores de status importantes sobre seus alertas.

Labels

Corresponda uma regra de alerta e suas instâncias a políticas de notificação e silêncios. Eles também podem ser usados ​​para agrupar seus alertas por gravidade.

Notification policies

Defina onde, quando e como os alertas são roteados. Cada política de notificação especifica um conjunto de correspondências de rótulo para indicar por quais alertas elas são responsáveis. Uma política de notificação tem um ponto de contato atribuído a ela que consiste em um ou mais notificadores.

Contact points

Defina como seus contatos são notificados quando um alerta é disparado. Oferecemos suporte a uma infinidade de ferramentas ChatOps para garantir que os alertas cheguem à sua equipe.

Features¶
One page for all alerts¶
Uma única página Grafana Alerting consolida os alertas gerenciados pelo Grafana e os alertas que residem em sua fonte de dados compatível com o Prometheus em um único local.

Multi-dimensional alerts¶
As regras de alerta podem criar várias instâncias de alerta individuais por regra de alerta, conhecidas como alertas multidimensionais, dando a você o poder e a flexibilidade para obter visibilidade de todo o sistema com apenas um único alerta.

Routing alerts¶
Encaminhe cada instância de alerta para um ponto de contato específico com base nos rótulos que você definir. As políticas de notificação são o conjunto de regras para onde, quando e como os alertas são roteados para os pontos de contato.

Silencing alerts¶
Os silêncios permitem que você pare de receber notificações persistentes de uma ou mais regras de alerta. Você também pode pausar parcialmente um alerta com base em determinados critérios. Os silêncios têm sua própria seção dedicada para melhor organização e visibilidade, para que você possa verificar suas regras de alerta em pausa sem sobrecarregar a exibição de alerta principal.

Mute timings¶
Com tempos mudos, você pode especificar um intervalo de tempo quando não deseja que novas notificações sejam geradas ou enviadas. Você também pode congelar as notificações de alerta por períodos recorrentes, como durante um período de manutenção.


Lucas, assuma essas 3 historias
História - HIS-2024-1679050 - [Zenite-simulate] Configuração de Alarme para a Aplicação ma-zenite-simulate
História - HIS-2024-1679024 - [Zenite-Hire] Configuração de Alarme para a Aplicação ma-zenite-hire
História - HIS-2024-1679045 - [Zenite-Balance] Configuração de Alarme para a Aplicação ma-zenite-balance
 
Basicamente vamos fazer os alarmes nesses módulos acima. Eu sugiro vc comecar com a Balance.

global:
  ...
  stackParts:
    observability:
      enabled: true
      alerts:
        availability:
          enabled: true
          contactCmr: true
          designatedGroup: "PLATAFORMA REDEFLEX (S000620)"
          businessImpact: "Serviço fora do ar"
          description: "Erro 500 na aplicação ma-zenite-balance. Precisamos de ajuda para resolver. Afeta a plataforma Kosmos. Chamar MA1."
        responseTime:
          enabled: true
          timeInMs: 100
        statusCode5xx:
          enabled: true
          contactCmr: true
          threshold: 1
          designatedGroup: "PLATAFORMA REDEFLEX (S000620)"
          businessImpact: "Erros 5xx na aplicação"
          description: "Erros 5xx na aplicação ma-zenite-balance. Verificar logs. Chamar time de suporte."
