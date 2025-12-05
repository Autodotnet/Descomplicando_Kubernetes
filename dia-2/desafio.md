# O Mistério do Pod Faminto

## Contexto

Você é o Engenheiro DevOps responsável por implantar uma nova aplicação de processamento de dados chamada "StressApp". Os desenvolvedores avisaram que essa aplicação pode ser um pouco... "gulosa" com memória RAM.

Seu objetivo é implantar essa aplicação no cluster Kubernetes, mas você precisa garantir que ela não derrube o nó ou afete outros vizinhos. Para isso, você usará seus conhecimentos sobre Pods, Resources (Requests/Limits) e Namespaces.

## Missão

### Parte 1: Preparando o Terreno
Crie um Namespace chamado treinamento-ch2. Toda a sua atividade deve acontecer dentro dele para não bagunçar o cluster.

### Parte 2: O Teste de Estresse (O Erro Esperado)
Crie um manifesto YAML para um Pod com as seguintes especificações:

- **Name:** pod-faminto
- **Image:** polinux/stress
- **Namespace:** treinamento-ch2
- **Command:** O container deve rodar o comando `stress --vm 1 --vm-bytes 250M --vm-hang 1`. (Isso faz o container tentar alocar 250MB de RAM).
- **Resources:**
  - Requests: memory: "100Mi"
  - Limits: memory: "200Mi"

Aplique esse manifesto no cluster.

Observe o status do Pod. O que aconteceu?

Ficou com:
- State: Waiting
- Reason: CrashloopBack
- Last state: Terminated
- Reason: Error
- Exit Code: 1

Use `kubectl get pod -n treinamento-ch2 -w` ou `kubectl describe pod pod-faminto -n treinamento-ch2`.

- Procure pela razão da falha. Você deve ver algo relacionado a OOMKilled.

**R:** O Container está sendo terminado pelo Kernel do Linux por exceder o limite de memória

### Parte 3: A Correção
Agora, crie um novo manifesto (ou edite o anterior) para um Pod chamado pod-comportado.
Mantenha a mesma imagem e comando (`stress --vm 1 --vm-bytes 250M --vm-hang 1`).
Ajuste os Limits de memória para que sejam suficientes para o que a aplicação pede (lembre-se, ela quer 250MB). Dê uma margem de segurança.
Aplique o manifesto e verifique se o Pod fica com status Running.

## Perguntas para Reflexão (Responda mentalmente ou anote)

- Quando o Pod foi morto na "Parte 2", qual mecanismo do Linux (que vimos no Cap 1) foi acionado pelo Kubernetes/Container Runtime para parar o processo?
    
    **O Kernel** Correção - Cgroup

- Qual a diferença prática entre definir um request de 100Mi e um limit de 200Mi? O que o Scheduler usa para decidir onde colocar o Pod?

    O request é o valor reservado para aquele container já o limite é máximo que o container pode usar. 
    O Scheduler leva em consideração os requisitos individuais e coletivos do pod.

**Dica:** Lembre-se que containers são apenas processos isolados. Se um processo tenta usar mais memória do que o cgroup permite, o kernel entra em ação!

## Dúvidas
Eu deveria ver OOMKilled na saída do comando `kubectl logs pod-faminto -n treinamento-ch2`
O problema era no WSL, usando uma VM com Debian consegui ver o OOMKilled