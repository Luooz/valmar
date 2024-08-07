"global:
  /# ...
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


vc precisa acrescentar esses campos

História - HIS-2024-1679050 - [Zenite-simulate] Configuração de Alarme para a Aplicação ma-zenite-simulate
História - HIS-2024-1679024 - [Zenite-Hire] Configuração de Alarme para a Aplicação ma-zenite-hire
História - HIS-2024-1679045 - [Zenite-Balance] Configuração de Alarme para a Aplicação ma-zenite-balance

eu sugiro vc comecar com a balanbce

global:
  application:
    name: ma-zenite
    critical: true
    spec:
      # -- Estratégia de rollout que será usada para deploy de nova versão da aplicação (rolling update, canary, bluegreen, etc)
      strategy:
        # Especificação -> https://argo-rollouts.readthedocs.io/en/stable/features/canary/#other-configurable-features
        # Documentação Techstack -> http://rede-commons.pages.gitlab.prd.useredecloud/techstacks/documentation/infrastructure/deploy/?h=deploy#canary
        # O parametro stableService não deve ser utilizado
        # O parametro canaryService é um boolean, que pode ser "true" para aplicações que possuem um endpoint (ex: API ou frontEnd)
        # https://argo-rollouts.antecipacao-recebiveis.aws.dev.useredecloud/rollouts/
        # rota de preview http://ma-zenite-preview.ma.antecipacao-recebiveis.aws.prd.useredecloud
        canary:
          # Habilitar o endpoint para acessar exclusivamente a nova versão ("preview")
          canaryService: true
          # Definir como será cada passo do rollout (percentuais de virada, pausas por tempo ou indefinidas até ação manual, validações automatizadas, etc)
          # Abaixo é apenas um exemplo de "steps", cada aplicação deve ter seus os passos de rollout adequados
          steps:
            # Subir 10% dos pods com a nova versão da aplicação (10% arredondado para cima). Ex: Se replicaCount são 3 pods, 20% vai resultar em 1 pod
            - setWeight: 10
            # Pausar o rollout por 30m. Unidades suportadas: s, m, h
            - pause:              
                duration: 30m
            - setWeight: 40
            # Pausar indefinidamente até ação manual para seguir ao próximo passo
            - pause: {}
            # Aumentar para 100% os pods com a nova versão da aplicação
            - setWeight: 100
      autoscaling:
        enabled: true
        maxReplicas: 5
        minReplicas: 3
        targetCPU: 70
        targetMemory: 70
      container:
        env:
        - name: ANTICIPATION_HOST
          value: http://ma-parametrizacao-antecipacao.ma.antecipacao-recebiveis.aws.prd.useredecloud
        - name: DR_CANAIS_CORPORATIVOS_HOST
          value: http://dr-canais-corporativos-api.dr.contas-receber.aws.prd.useredecloud
        - name: KOSMO_HOST
          value: http://ma-calculo-antecipacao.ma:8080
        - name: V3_HOST
          value: https://vpce-0cb1f388a26631821-b9ryc0ou.execute-api.us-east-1.vpce.amazonaws.com/prd
        - name: BALANCE_BLOCKED_HOST  
          value: http://ma-saldos-bloqueados.ma.antecipacao-recebiveis.aws.prd.useredecloud
        - name: RAV_AUTO_HOST
          value: https://vpce-0cb1f388a26631821-b9ryc0ou.execute-api.us-east-1.vpce.amazonaws.com/prd
        - name: KOSMO_ORQUESTRADOR
          value: http://ma-antecipacao-orquestrador.ma:8080
        - name: PARAMETROS_CRUD_HOST
          value: http://parametros-crud.ma:3000
        - name: REGION
          value: sa-east-1
        - name: SECRET_MANAGER
          value: ma-16036-secret-sa-east-1-prd
        - name: SECRET_MANAGER_RAV_AUTO
          value: ma-api-rav-auto-prd
        - name: API_ID_RAV_AUTO
          value: ztvcw3s03h
        - name: ENVIRONMENT
          value: prd
        port: 3000
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /
            port: 3000
            scheme: HTTP
          initialDelaySeconds: 60
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /
            port: 3000
            scheme: HTTP
          initialDelaySeconds: 60
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources:
          limits:
            cpu: 500m
            memory: 512Mi
          requests:
            cpu: 125m
            memory: 256Mi
  serviceAccount:
    create: true
  stackParts:
    secret:
      enabled: true
    api:
      enabled: true 
      spec:
        ingress:
          customRules:
           - - host: ma-zenite.ma.antecipacao-recebiveis.aws.prd.useredecloud
               http:
                paths:
                  - path: /v3/balance
                    pathType: Prefix
                    backend: 
                      service:
                        name: ma-zenite-balance
                        port:
                          number: 8080
                  - path: /v3/saldo
                    pathType: Prefix
                    backend: 
                      service:
                        name: ma-zenite-balance
                        port:
                          number: 8080
                  - path: /v3/bandeiras
                    pathType: Prefix
                    backend: 
                      service:
                        name: ma-zenite-balance
                        port:
                          number: 8080
                  - path: /v3/simular-pv
                    pathType: Prefix
                    backend: 
                      service:
                        name: ma-zenite-simulate
                        port:
                          number: 8080
                  - path: /v3/simular
                    pathType: Prefix
                    backend: 
                      service:
                        name: ma-zenite-simulate
                        port:
                          number: 8080
                  - path: /v3/contratar-pv
                    pathType: Prefix
                    backend: 
                      service:
                        name: ma-zenite-hire
                        port:
                          number: 8080
                  - path: /v3/contratar
                    pathType: Prefix
                    backend: 
                      service:
                        name: ma-zenite-hire
                        port:
                          number: 8080
                  - path: /v3/consulta
                    pathType: Prefix
                    backend: 
                      service:
                        name: ma-zenite-operation
                        port:
                          number: 8080
    observability:
      enabled: true
      alerts:
        serviceNowGroup: "ANTECIPACAO INTERNA (S002968)"
        statusCode5xx:
          enabled: true
          threshold: 1
          waitingTimeToFire: "1m"
          contactCmr: true
          designatedGroup: "ANTECIPACAO INTERNA (S002968)"
          businessImpact: "Indisponibilidade do serviço na plataforma Zenite"
          description: "Aplicação ma-zenite obteve erro 500, necessário intermediação para resolução do problema, esse erro afeta diretamente a orquestração dos canais de antecipação. Deve acionar o plantonista da sigla MA1."
        availability:
          enabled: true
          contactCmr: true
          designatedGroup: "ANTECIPACAO INTERNA (S002968)"
          businessImpact: "Indisponibilidade do serviço na plataforma Zenite"
          description: "Aplicação ma-zenite obteve erro 500, necessário intermediação para resolução do problema, esse erro afeta diretamente a orquestração dos canais de antecipação. Deve acionar o plantonista da sigla MA1."

É zenite antiga, mas pra você ter como base. vc precisa acrescentar esses campos, referente ao alarme na zenite nova.


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

            vc precisa acrescentar esses campos, referente ao alarme na zenite nova
