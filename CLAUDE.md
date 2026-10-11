# AGENTS.md — Alphractal / Módulo "Fees" em tempo real

Instruções para agentes de código que trabalham neste repositório. Leia antes de escrever qualquer código.

## 1. Contexto do projeto

Projeto do **Inteli Blockchain** (Diretoria de Projetos) para a **Alphractal**, plataforma de inteligência de mercado Web3 de nível institucional desenvolvida pela Nortech Labs.

**Entrega:** sistema de monitoramento em tempo real de custos de taxa (gas) na rede Ethereum, integrado à aba **"Fees"** da plataforma Alphractal.

**Problema:** hoje a aba "Fees" mostra apenas médias históricas estáticas. Isso cria um ponto cego em relação à volatilidade instantânea da mempool, expondo gestores de fundos a risco de execução — operações travadas por estimativas imprecisas ou custos excessivos em picos não previstos.

**Objetivo:** entregar um MVP funcional ponta a ponta que ingere dados on-chain ao vivo, captura a volatilidade das taxas e o lançamento de novos blocos, e converte métricas brutas em estimativas financeiras (gas → USD) exibidas em um painel interativo minimalista.

**Natureza:** protótipo acadêmico e experimental. Prova de conceito, não produto de produção.

## 2. Stack

A stack é livre, mas as recomendações do TAP são o padrão deste repositório. Não troque sem alinhar com o time.

| Camada | Tecnologia |
|---|---|
| Frontend | React + Vite + TypeScript |
| Backend | Node.js + TypeScript |
| Conexão blockchain | WebSockets via provedor RPC (Alchemy / Infura), usando `viem` ou `ethers.js` |
| Entrega ao painel | SSE (Server-Sent Events) para stream em tempo real |

Decisões arquiteturais implícitas nessa stack:

- A ingestão on-chain é **push**, não polling. Assine eventos de novo bloco via WebSocket; não faça `setInterval` batendo em endpoint REST.
- O backend é a única coisa que fala com o RPC. O frontend nunca recebe chave de RPC nem abre WebSocket para o provedor.
- O canal backend → frontend é **SSE** (unidirecional). Não introduza WebSocket bidirecional nesse trecho sem necessidade real — o painel só consome dados.

## 3. Escopo — o que NÃO fazer

Estas restrições vêm do TAP e são vinculantes. Um agente que as violar está produzindo trabalho fora de escopo.

- **Sem deploy em produção.** Nada de integrar no ambiente em nuvem da Alphractal ou substituir sistemas legados. A entrega roda em ambiente isolado.
- **Sem execução de transações.** O sistema é estritamente de **leitura**: dados, análise e telemetria. Não assine, envie nem automatize ordens on-chain. Nada de código que manipule chave privada ou monte transação.
- **Sem deploy em Mainnet.** Nenhum contrato inteligente próprio, nenhum consumo de gas real.
- **Sem auditoria de segurança nem load testing extremo.** Não está no escopo auditoria formal, teste de estresse ou certificação de conformidade financeira.

Se uma tarefa parecer exigir qualquer um desses itens, pare e sinalize em vez de implementar.

## 4. Convenções de código

- **TypeScript em modo estrito.** Sem `any` implícito; sem `@ts-ignore` sem comentário justificando.
- **Valores monetários e de gas:** trate wei/gwei como `bigint`. Nunca use `number` para wei — perde precisão. Converta para `number` só na formatação final da UI.
- **Erros de RPC são esperados, não excepcionais.** Provedor cai, WebSocket desconecta, bloco vem fora de ordem. Trate reconexão e dados faltantes como caminho normal do código, com backoff. O painel deve degradar (mostrar "sem conexão") em vez de quebrar.
- **Segredos em `.env`,** nunca commitados. Mantenha um `.env.example` com as chaves esperadas e valores vazios.
- **Nomes de domínio em inglês** no código (`baseFeePerGas`, `priorityFee`, `blockNumber`) — casam com a terminologia Ethereum e com a API do provedor. Documentação e comentários em português.

## 5. Convenções de commit

- Conventional Commits, mensagem em português.
- Formato usado pelo time: `tipo (#issue): Descrição no imperativo`
  - Ex.: `feat (#7): Adiciona ingestão de novos blocos via WebSocket`
- **Sem atribuição de IA.** Nenhum `Co-Authored-By` de assistente, nenhuma menção a ferramenta de IA em commits, issues ou PRs.
- Não commite nem faça push sem pedido explícito.

## 6. Cronograma (4 semanas)

| Semana | Entrega |
|---|---|
| 1 | Kick-off com o parceiro (18/08/26). Análise de mercado. Protótipo de alta fidelidade do painel. |
| 2 | Setup de backend e frontend. Conexão RPC via WebSockets e cálculo médio das taxas. |
| 3 | Integração frontend ↔ backend. Correção de bugs e polimento. |
| 4 | Demonstração final / Demo Day (03/09/26). |

Reuniões obrigatórias com o parceiro: apenas as duas (kick-off e encerramento). Dúvidas pontuais vão por canal assíncrono, com resposta em até 48h úteis — logo, **não bloqueie o desenvolvimento esperando resposta do parceiro**: registre a suposição, siga em frente e valide depois.

## 7. Definição de pronto (MVP)

O protótipo está pronto quando um usuário consegue:

1. Ver, em tempo real, o custo de execução de transações na Ethereum.
2. Acompanhar a volatilidade da mempool conforme novos blocos chegam.
3. Ler a conversão instantânea de gas para **USD**.
4. Fazer isso num painel interativo minimalista, visualmente coerente com a plataforma Alphractal.

## 8. Licença e visibilidade

Repositório **público**, licença **MIT**. Alunos usam como portfólio; a empresa pode usar o código livremente. Não inclua no repositório dados proprietários, credenciais ou material interno da Alphractal.
