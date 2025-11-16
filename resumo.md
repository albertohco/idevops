# Resumo do Desenvolvimento da Pipeline de CI/CD

Este documento resume as etapas e correções realizadas para automatizar o processo de integração e entrega contínua (CI/CD) da aplicação "encontros-tech" no Kubernetes da Digital Ocean, utilizando GitHub Actions.

## Objetivo

Automatizar o deploy da aplicação no ambiente Kubernetes, estendendo a pipeline de CI existente para incluir a entrega contínua (CD).

## Trabalho Realizado

### 1. Análise Inicial e Configuração da Pipeline de CD

*   **Problema Identificado:** A pipeline de CD no arquivo `.github/workflows/main.yml` estava com erros, principalmente relacionados à autenticação no Kubernetes e à forma de injetar a tag da imagem no manifesto.
*   **Primeiras Correções:**
    *   Substituição da action `Azure/k8s-deploy@v1` por uma abordagem mais direta e agnóstica à nuvem.
    *   Implementação da action `digitalocean/action-doctl@v2` para configurar o `kubeconfig` do cluster Digital Ocean.
    *   Uso inicial de um comando `sed` para substituir dinamicamente a tag da imagem no `manifesto.yaml` antes do `kubectl apply`.
    *   **Requisito de Secrets:** Orientação para a criação dos secrets `DIGITALOCEAN_ACCESS_TOKEN` (token de acesso da Digital Ocean) e `K8S_CLUSTER` (nome do cluster Kubernetes) no GitHub.

### 2. Resolução de Erros de Autenticação e Permissão

Diversos erros de autenticação e permissão foram encontrados e corrigidos:

*   **Erro `403 You are not authorized to perform this operation` (Token Inicial):** Diagnóstico de que o `DIGITALOCEAN_ACCESS_TOKEN` não possuía as permissões de `write` necessárias. Orientação para gerar um novo token com as permissões corretas.
*   **Erro `no cluster goes by the name "***"`:** Diagnóstico de que o secret `K8S_CLUSTER` estava com um valor placeholder (`***`) ou incorreto. Orientação para preencher com o nome exato do cluster.
*   **Erro `401 Unable to authenticate you`:** Diagnóstico de que o `DIGITALOCEAN_ACCESS_TOKEN` estava inválido ou expirado. Orientação para gerar um novo token e atualizar o secret.
*   **Erro `403 You are not authorized to perform this operation` (kubeconfig save):** Mesmo com permissões aparentemente corretas, a operação de `kubeconfig save` falhava. **Solução:** Orientação para gerar um token de **"Full Access"** na Digital Ocean para garantir todas as permissões necessárias.

### 3. Refinamento da Injeção da Imagem e Configuração do Banco de Dados

*   **Substituição da Tag da Imagem:** Para aderir à preferência por actions em vez de comandos bash, o comando `sed` para a substituição da tag da imagem foi substituído pela action `jacobtomlinson/gha-find-replace@v3`.
*   **Conexão com o Banco de Dados (Secret):** Foi identificado que a etapa de criação do secret do Kubernetes para o `DATABASE_URL` estava faltando. 
    *   **Correção:** Adição de um passo no workflow para criar o namespace `producao` e o secret `encontros-tech-secrets` (utilizando o secret `DATABASE_URL` do GitHub) antes do deploy da aplicação. O comando `kubectl apply` foi ajustado para direcionar o namespace `producao`.

### 4. Correção de Erro de Sintaxe no Manifesto Kubernetes

*   **Erro `strict decoding error: unknown field "spec.template.spec.containers[0].env[0].valueFrom.key"`:** Diagnóstico de um erro de indentação no `manifesto.yaml` na seção `valueFrom.secretKeyRef`. Os campos `name` e `key` estavam com indentação incorreta.
*   **Correção:** Ajuste da indentação no arquivo `02-encontros-tech/k8s/manifesto.yaml` para que `name` e `key` estivessem corretamente aninhados sob `secretKeyRef`.

## Estado Atual da Pipeline

Após todas as correções, a pipeline de CI/CD está configurada para:

1.  Realizar o checkout do código.
2.  Instalar dependências e executar testes Python.
3.  Fazer login no Docker Hub.
4.  Construir e enviar a imagem Docker para o Docker Hub com uma tag versionada (`v${{ github.run_number }}`).
5.  Autenticar no cluster Kubernetes da Digital Ocean usando `doctl`.
6.  Criar o namespace `producao` (se não existir) e o secret `encontros-tech-secrets` com a `DATABASE_URL`.
7.  Atualizar dinamicamente a tag da imagem no `manifesto.yaml` usando a action `find-and-replace`.
8.  Aplicar o manifesto atualizado no namespace `producao` do cluster Kubernetes.

## Próximos Passos

Com a pipeline de deploy funcional, o foco pode se voltar para as recomendações de segurança e performance detalhadas no `diagnostico.md`.
