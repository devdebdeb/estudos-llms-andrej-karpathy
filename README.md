# O que realmente acontece dentro de um LLM (e por que você precisa construir um para entender)

Andrej Karpathy tem uma frase emprestada de Feynman que resume bem o problema com a maioria dos tutoriais de IA: se você não consegue construir, você não entende.

Esse guia é uma tentativa de levar esse princípio a sério. As fontes são cinco vídeos e transcrições públicas do Karpathy — desde o tutorial de backpropagation com micrograd até a palestra sobre o que muda na engenharia de software quando IAs viram colaboradores. O objetivo não é resumir o que ele disse. É documentar o que fica quando você tenta montar os blocos na ordem certa.

---

## Contexto e Objetivos

**Assunto escolhido:** Funcionamento interno dos Modelos de Linguagem de Grande Escala (LLMs) — da matemática das redes neurais até o pipeline completo que transforma um modelo bruto em um assistente de IA.

A motivação foi direta: ferramentas como ChatGPT e Claude fazem parte do dia a dia, mas a maioria das explicações disponíveis trata o funcionamento interno como caixa preta. Esse caderno temático é uma tentativa de abrir essa caixa com as fontes do Andrej Karpathy, ex-diretor de IA da Tesla e pesquisador da OpenAI — que são, de longe, as mais didáticas disponíveis em inglês.

Os quatro objetivos que guiaram o estudo: entender o pipeline completo de treinamento (Pre-training, SFT e RLHF); compreender a arquitetura Transformer partindo da implementação, não da teoria; entender por que a tokenização BPE causa falhas específicas e previsíveis; e explorar o que muda na engenharia de software quando modelos de linguagem viram colaboradores ativos.

---

## O pipeline de treinamento, sem o hype

A maioria das explicações sobre como o ChatGPT funciona pula direto para o resultado e ignora as três fases que produzem esse resultado.

**Pre-training** é quando o modelo lê a internet inteira. Não "aprende" no sentido humano — comprime padrões estatísticos de como palavras se conectam. O produto dessa fase é um "Base Model": um simulador de documentos. Custa milhões de dólares e milhares de GPUs. O resultado bruto não sabe responder perguntas — só sabe continuar texto.

**Supervised Fine-Tuning (SFT)** é onde o modelo estuda exemplos de perguntas com respostas escritas por humanos. Aqui ele aprende o formato de um assistente — como deve responder, qual tom usar, o que não fazer.

**RLHF (Reinforcement Learning from Human Feedback)** é a fase mais contraintuitiva. O modelo tenta resolver problemas, recebe feedback sobre quais tentativas foram melhores, e ajusta. É aqui que surgem comportamentos como raciocínio em cadeia — o modelo aprende que "pensar em voz alta" antes de responder gera respostas melhores.

A analogia que Karpathy usa é escolar: pre-training é leitura extensiva, SFT é estudar com gabarito, RLHF é praticar com correção. O que ela ilumina bem é a sequência — cada fase depende da anterior.

---

## Por que LLMs erram ao soletrar "strawberry"

O modelo não lê letras. Lê tokens.

O algoritmo Byte Pair Encoding (BPE) divide texto em pedaços que podem ser palavras inteiras, sílabas ou fragmentos arbitrários — dependendo do que aparece com frequência suficiente no corpus de treinamento. "strawberry" pode virar três tokens: `straw`, `ber`, `ry`. Contar letras "r" dentro disso exige que o modelo faça aritmética sobre sua própria representação interna, o que não é para o que ele foi treinado.

Isso também explica por que modelos performam pior em idiomas menos representados no corpus (como coreano, ou português em alguns contextos), e por que espaços em branco causam bugs inesperados em certas tarefas de código.

---

## O que Karpathy chama de "LLM OS"

A ideia é simples e estranha ao mesmo tempo: o modelo de linguagem não é um chatbot. É o núcleo de um sistema operacional novo.

O context window funciona como RAM — o que está ali é o que o modelo "sabe" naquele momento. Ferramentas externas (interpretadores de código, buscadores, APIs) funcionam como periféricos. A diferença é que, em vez de programar regras explícitas para cada situação, você descreve o que quer e o modelo decide como usar as ferramentas disponíveis.

Isso não é ficção científica — é o que acontece quando você usa o Claude para pesquisar e escrever código ao mesmo tempo, ou quando um agente verifica seu calendário antes de responder sobre disponibilidade.

---

## Engenharia de Prompts e Cicatrizes

Três tentativas. Duas falharam de formas diferentes.

---

### Tentativa 1 — O prompt ingênuo

**Prompt:**
```
O que é o ChatGPT?
```

**Resultado:** O modelo retornou uma explicação genérica sobre "processamento de linguagem natural" e "modelos de linguagem treinados em grandes volumes de texto". Poderia ter sido escrita em 2018. Sem pipeline técnico, sem distinção entre base model e assistant, sem nenhuma referência às fontes carregadas.

**Problema identificado:** O prompt não deu contexto suficiente para o modelo saber o nível de profundidade esperado, nem sinalizou que havia fontes técnicas disponíveis para consulta. A pergunta aberta demais gerou uma resposta Wikipedia.

---

### Tentativa 2 — Adicionando estrutura técnica

**Prompt:**
```
Explique o pipeline de treinamento de um LLM como o ChatGPT,
separando as etapas de Pre-training, SFT e RLHF.
```

**Resultado:** O modelo explicou como o pre-training cria um "simulador de documentos da internet" e como SFT e RLHF transformam esse simulador em um assistente com formato e comportamento específicos. As fontes do Karpathy foram acionadas. A distinção entre base model e assistant apareceu naturalmente.

**O que mudou:** Nomear as etapas no prompt forçou o modelo a seguir uma estrutura em vez de improvisar. O modelo não ficou mais inteligente — o prompt ficou mais específico.

---

### Tentativa 3 — Pedindo trilha de aprendizado

**Prompt:**
```
Qual é a trilha de aprendizado ideal para entender profundamente
o funcionamento de LLMs, de acordo com as fontes?
```

**Resultado:** O modelo estruturou a sequência correta: micrograd (para entender backpropagation na prática), makemore (modelagem de linguagem bigrama), implementação do Transformer do zero, e finalmente o tokenizer. A ordem faz sentido porque cada repositório pressupõe o que foi construído antes.

**Lição:** Ancorar o prompt em "de acordo com as fontes" muda o comportamento do modelo — em vez de gerar uma resposta genérica de internet, ele passa a trabalhar com o material específico carregado no NotebookLM.

---

### Dificuldades encontradas

- **Respostas sem referência às fontes:** Nas primeiras tentativas, o modelo respondia a partir do conhecimento geral, ignorando as transcrições do Karpathy. Solução: adicionar "com base nas fontes carregadas" ou citar o nome do autor diretamente no prompt.
- **Nível de abstração errado:** Prompts curtos geravam respostas introdutórias demais; prompts muito técnicos geravam respostas que assumiam conhecimento prévio que eu ainda não tinha. Calibrar o nível exigiu duas ou três iterações por tema.
- **Resumos genéricos demais:** Pedir "resuma a fonte X" gerava parágrafos sem hierarquia de importância. O padrão que funcionou: pedir "quais são os três conceitos que o Karpathy considera mais mal compreendidos nessa fonte?".

---

## Fontes

| # | Título | O que cobre |
|---|--------|-------------|
| 1 | [Intro to Large Language Models](https://www.youtube.com/watch?v=zjkBMFhNj_g) | Visão geral do treinamento; diferença entre Base Model e Assistant |
| 2 | [Let's build GPT: from scratch, in code, spelled out](https://www.youtube.com/watch?v=kCc8FmEb1nY) | Arquitetura Transformer e mecanismo de Self-Attention |
| 3 | [The spelled-out intro to neural networks and backpropagation: building micrograd](https://www.youtube.com/watch?v=VMj-3S1tku0) | Backpropagation e otimização de redes neurais |
| 4 | [Let's build the GPT Tokenizer](https://www.youtube.com/watch?v=zduSFxRajkE) | Tokenização BPE e seus efeitos colaterais |
| 5 | [Software Is Changing (Again)](https://www.youtube.com/watch?v=LCEmiRjPEtQ) | Engenharia agêntica e o futuro do software |

---

## Glossário

**Backpropagation** — o algoritmo que calcula como cada peso da rede contribuiu para o erro e ajusta todos eles de uma vez. Sem isso, a rede não tem como saber o que mudar.

**Context Window** — a memória de trabalho do modelo. Tudo que está dentro dele é visível; tudo fora não existe para aquela geração.

**Token** — a unidade atômica que o modelo processa. Pode ser uma palavra, parte de uma palavra, ou um fragmento de código, dependendo do tokenizer.

**Transformer** — a arquitetura que usa self-attention para decidir quais partes de uma sequência são relevantes para cada posição. Descrita no paper "Attention is All You Need" (2017) e ainda é a base de praticamente todos os LLMs relevantes.

**Agentic Engineering** — coordenar modelos com acesso a ferramentas do mundo real (código, busca, APIs) para que tomem decisões em sequência antes de devolver controle ao humano.

---

## Prompts Reutilizáveis

**Tokenização:**
```
Aja como Andrej Karpathy. Explique em termos simples por que a tokenização BPE
faz com que um modelo falhe ao tentar contar as letras 'r' na palavra 'strawberry'.
```

**Pipeline:**
```
Compare o papel do Pre-training e do RLHF no treinamento de um LLM.
Qual analogia do mundo real melhor representa cada etapa e por quê elas
não podem ser invertidas de ordem?
```

**LLM OS:**
```
Explique o conceito de 'LLM OS' descrito pelo Karpathy. Como o Context Window
se compara à RAM de um sistema operacional convencional? Quais são os limites
dessa analogia?
```

**Trilha prática:**
```
Quais são os repositórios do Karpathy que devo estudar na ordem correta para
entender Deep Learning de verdade? Explique o que cada um ensina e por que
a sequência importa.
```

**Software 3.0:**
```
O que Karpathy quer dizer com 'Software 3.0'? Como a engenharia agêntica
muda o trabalho de quem desenvolve software hoje?
```

---

> Esse guia é ponto de partida. Os vídeos do Karpathy têm código funcional — ir direto para o repositório [`micrograd`](https://github.com/karpathy/micrograd) depois de ler isso provavelmente vai clarear mais do que qualquer resumo.
