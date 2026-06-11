# O que realmente acontece dentro de um LLM (e por que você precisa construir um para entender)

Andrej Karpathy tem uma frase emprestada de Feynman que resume bem o problema com a maioria dos tutoriais de IA: se você não consegue construir, você não entende.

Esse guia é uma tentativa de levar esse princípio a sério. As fontes são cinco vídeos e transcrições públicas do Karpathy — desde o tutorial de backpropagation com micrograd até a palestra sobre o que muda na engenharia de software quando IAs viram colaboradores. O objetivo não é resumir o que ele disse. É documentar o que fica quando você tenta montar os blocos na ordem certa.

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

## O que aprendi tentando usar IA para estudar IA

A primeira vez que pedi ao modelo uma explicação de como o ChatGPT funciona, recebi um parágrafo sobre "processamento de linguagem natural" que poderia ter sido escrito em 2018. Genérico, sem pipeline, sem distinção entre base model e assistant.

O que mudou quando adicionei contexto explícito ao prompt — "separe as etapas de pre-training, SFT e RLHF" — foi imediato. A resposta virou um mapa utilizável. O modelo não ficou mais inteligente; eu fiquei mais específico.

A outra coisa que aprendi: pedir ao modelo para estruturar uma trilha de aprendizado baseada nas fontes do Karpathy gerou exatamente a sequência que faz sentido — micrograd primeiro (para entender backpropagation), depois makemore (modelagem de linguagem), depois a implementação do Transformer. A ordem importa porque cada etapa usa o que você construiu antes.

---

## Fontes

Todos os materiais são públicos e gratuitos:

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

> Esse guia é ponto de partida. Os vídeos do Karpathy têm código funcional — ir direto para o repositório [`micrograd`](https://github.com/karpathy/micrograd) depois de ler isso provavelmente vai clarear mais do que qualquer resumo.
