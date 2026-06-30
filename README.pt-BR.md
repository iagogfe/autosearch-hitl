<div align="center">

[English](README.md) · **Português**

# autosearch-hitl

**Otimização autônoma com humano no centro (human-in-the-loop):**
você define o que é "melhor", a skill itera sozinha — *muda → mede → mantém/descarta* — com segurança e bom senso pra parar.

![license](https://img.shields.io/github/license/iagogfe/autosearch-hitl)
![release](https://img.shields.io/github/v/release/iagogfe/autosearch-hitl)
![CI](https://img.shields.io/github/actions/workflow/status/iagogfe/autosearch-hitl/ci.yml?branch=master&label=CI)
![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen)
[![skills.sh](https://img.shields.io/badge/skills.sh-install-111111)](https://www.skills.sh/iagogfe/autosearch-hitl)

</div>

`autosearch-hitl` é uma **Agent Skill** que generaliza a ideia do
[`autoresearch`](https://github.com/karpathy/autoresearch) do **Andrej Karpathy**
para o dia a dia de engenharia. Em vez de você ajustar código no braço e medir na
mão, a skill conduz o ciclo de experimentação por você — em **qualquer** coisa que
tenha uma métrica objetiva (latência, custo, tempo de CI, cobertura de testes,
qualidade de busca, etc.), não só treino de modelos.

## Por que existe

Otimizar é repetitivo: muda um pouco → roda → vê se melhorou → repete, dezenas de
vezes. É justamente o tipo de trabalho que um agente faz bem — **desde que a decisão
de "ficou melhor?" seja um número, não um achismo.** A `autosearch-hitl` automatiza
esse laço mantendo **você no comando das decisões que importam**.

## Human-in-the-loop, de propósito

Ninguém quer um agente 100% automático mexendo no código de produção às cegas. A
skill foi feita ao redor do **humano no centro**:

- Ela **pergunta o que você quer** e te orienta — não exige decorar comandos.
- Tem um **"porteiro" de pré-condições**: antes de começar, checa se o problema é
  otimizável e, se não for, **recusa e explica o porquê** (em vez de gastar tempo
  otimizando a coisa errada).
- Quando falta a forma de medir, ela **te ajuda a montar** e pede sua confirmação de
  que a medição mede a coisa certa.
- Trabalha numa **cópia isolada** e te entrega o resultado pra você revisar e decidir
  o merge.

## Como funciona

1. **Pré-condições (o porteiro).** A skill só roda se houver:
   1. *Artefato controlável* — algo concreto pra alterar.
   2. *Métrica objetiva* — um número que diz melhor/pior, com direção.
   3. *Medição repetível* — um comando que produz esse número (ou que dá pra criar).
   4. *Mudança reversível* — dá pra desfazer uma tentativa ruim.
   Faltou algo insanável? Ela recusa e explica.

2. **Isolamento seguro.** Para código, trabalha num **git worktree** dedicado
   (`autoopt/<tag>`) — nunca toca no seu branch. Para outros domínios, usa snapshot.

3. **Loop medido.** Aplica uma ideia → mede → **mantém se melhorou, reverte se
   piorou** → registra. Cada melhoria vira um commit; cada piora é descartada.

4. **Para na hora certa.** Encerra sozinha quando **estagna** (várias tentativas sem
   ganho) e entrega um relatório + o resultado vencedor.

## Instalação

Mais fácil — via CLI do [skills](https://www.skills.sh):

```bash
npx skills add iagogfe/autosearch-hitl
```

Ou manualmente (skills são descobertas em `~/.claude/skills/`):

```bash
git clone https://github.com/iagogfe/autosearch-hitl
cp -r autosearch-hitl/skills/autosearch-hitl ~/.claude/skills/autosearch-hitl
# (ou crie um symlink pra manter versionado)
```

Reinicie seu agente. No Claude Code aparece como `/autosearch-hitl`. Funciona também
em outros agentes compatíveis com skills (Codex etc.) — a skill é só markdown, sem
dependência de runtime.

## Uso

Abra seu agente num projeto e diga o que quer melhorar, em linguagem natural:

```
use autosearch-hitl pra reduzir a latência do endpoint POST /order
use autosearch-hitl pra aumentar a cobertura de testes do módulo de pagamentos
use autosearch-hitl pra melhorar a qualidade da busca contra esse conjunto de avaliação
```

A skill cuida do resto: triagem → medição → isolamento → loop → relatório.

### Não sabe como pedir? Pode falar simples

Não precisa ser técnico. Pedidos casuais (até "burros") funcionam:

```
como posso melhorar isso aqui?
isso tá lento, dá pra deixar mais rápido?
faz meus testes cobrirem mais
dá pra esse código custar menos?
melhora os resultados dessa busca
```

Se não estiver claro como medir o "melhor", a skill **te pergunta e ajuda a montar**
antes de mexer em qualquer coisa — então dá pra começar mesmo sem saber a métrica.

## Resultados (casos reais)

Aplicada em código de produção de uma **API de pagamentos**, com ganhos medidos e
**zero regressão**:

- **Latência de um endpoint crítico** (`create_order`): ~46% mais rápido,
  reduzindo consultas redundantes ao banco (de ~22 para ~7 por request).
- **Tempo da suíte de testes**: **−57%** (identificou um teste que travava ~10s numa
  resolução de DNS real e o isolou, sem tocar em código de produção).
- **Qualidade de busca** (serviço de documentação): **+162% de MRR** — a resposta
  certa passou a vir em 1º lugar na maioria das consultas (via BM25 + sinônimos de
  domínio; 4 de 8 ideias foram descartadas pela métrica).

> A skill é tão boa quanto a métrica que você define. Com poucos dados, é fácil
> "overfittar" o número — defina uma medição que represente o uso real.

## Créditos

A ideia central — o loop autônomo *muda → mede → mantém/descarta* — é do
**[Andrej Karpathy](https://github.com/karpathy)**, no projeto
**[`autoresearch`](https://github.com/karpathy/autoresearch)**. Esta skill apenas
generaliza esse conceito para qualquer domínio mensurável, com foco em
human-in-the-loop. Todo o crédito da ideia original é dele. 🙏

## Licença

[MIT](./LICENSE).
