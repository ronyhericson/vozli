# VozLi

Treino de inglês em sessões de 10 minutos, do vocabulário isolado até a conversa livre.

VozLi é um aplicativo de página única para quem fala português do Brasil e quer sair da tradução de palavras soltas e chegar a uma conversa real em inglês. A sessão dura dez minutos cronometrados e avança por cinco etapas, cada uma só liberada depois de um número de acertos. Tem ditado por microfone, leitura em voz alta das falas do professor e um relatório final baseado nos erros que você de fato cometeu.

Tudo roda em um único arquivo HTML, sem build, sem dependências instaladas e sem backend.

---

## Sumário

- [Funcionalidades](#funcionalidades)
- [O plano de estudo](#o-plano-de-estudo)
- [Como funciona por dentro](#como-funciona-por-dentro)
- [Base de conhecimento e fontes](#base-de-conhecimento-e-fontes)
- [Executando localmente](#executando-localmente)
- [Publicando no GitHub Pages](#publicando-no-github-pages)
- [Motor de IA: Google Gemini](#motor-de-ia-google-gemini)
- [Compatibilidade](#compatibilidade)
- [Personalizando o conteúdo](#personalizando-o-conteúdo)
- [Acessibilidade](#acessibilidade)
- [Estrutura do arquivo](#estrutura-do-arquivo)
- [Roadmap](#roadmap)
- [Licença](#licença)
- [Autor](#autor)

---

## Funcionalidades

**Progressão em cinco etapas.** Palavras soltas em A1, vocabulário do dia a dia em A2–B1 no sentido inverso, frases curtas, frases completas com tempos verbais e, por fim, bate-papo livre. Cada etapa exige um número de acertos para liberar a próxima, mas destrava sozinha após algumas tentativas: quem manda é o cronômetro, não o desempenho.

**Cronômetro de 10 minutos.** Com pausa, retomada, reinício do relógio sem perder o progresso e parada que devolve à tela inicial. Com o cronômetro pausado, o campo de resposta trava — não dá para estudar com calma e fingir que foi contra o relógio.

**Correção instantânea com julgamento inteligente.** A verificação principal é local e responde na hora. Quando a resposta não bate com o gabarito, ela vai para um modelo de linguagem decidir se é sinônimo válido, registro diferente ou erro de digitação, com explicação curta em português.

**Todo erro devolve o par completo.** A forma correta em inglês e a tradução em português, lado a lado, nos dois sentidos de tradução.

**Ditado por microfone.** Botão estilo WhatsApp: a barra de gravação mostra a transcrição em tempo real, o tempo decorrido, descarte e envio. O idioma reconhecido muda conforme a etapa — português na etapa que pede tradução para o português, inglês nas demais.

**Leitura em voz alta.** Cada fala do professor tem um botão de ouvir, e há um interruptor de leitura automática. O sintetizador lê apenas o conteúdo em inglês e ignora as explicações em português, para não misturar as pronúncias.

**Relatório da sessão.** Pontos fortes, pontos a melhorar, exercícios concretos para o dia seguinte e a lista crua dos itens errados com a forma correta. Gerado a partir do histórico real da sessão, com versão calculada localmente caso a IA não responda.

**Responsivo.** No celular o plano de estudo vira um painel sob demanda e o chat ocupa a tela inteira, com altura em `dvh`, campo de 16px para evitar o zoom automático do iOS e respeito à área segura.

---

## O plano de estudo

| # | Etapa | Nível CEFR | Formato | Acertos para avançar |
|---|-------|-----------|---------|----------------------|
| 1 | Palavras essenciais | A1 | português → inglês | 6 |
| 2 | Palavras do dia a dia | A2–B1 | inglês → português | 6 |
| 3 | Frases curtas | A2 | frase inteira, PT → EN | 5 |
| 4 | Frases completas | B1 | tempos verbais e conectivos | 4 |
| 5 | Bate-papo livre | B1+ | conversa com o professor | — |

A inversão de sentido na etapa 2 é intencional: traduzir do português para o inglês exercita a produção, traduzir do inglês para o português exercita o reconhecimento. As duas habilidades são diferentes e as duas precisam de treino.

---

## Como funciona por dentro

### Correção em duas camadas

A primeira camada é local e determinística. A resposta passa por uma normalização que baixa para minúsculas, remove acentos, descarta pontuação e apara artigos e o `to` do infinitivo. Para palavras, a comparação é contra a lista de respostas aceitas. Para frases, é calculada a sobreposição de tokens contra cada variante aceita, e acima de 92% a resposta é dada como certa.

A segunda camada só entra quando a primeira não reconhece a resposta. Aí a resposta vai para o modelo `claude-sonnet-4-6` com um prompt que pede JSON puro:

```json
{ "ok": true, "note": "explicação curta em português", "fix": "melhor forma correta" }
```

Esse desenho mantém o acerto instantâneo — nenhuma espera de rede quando você acerta — e reserva a inteligência para o caso em que ela realmente importa, que é decidir se `I don't like coffee` e `I do not like coffee` valem o mesmo. Se a chamada falhar, cai de volta para uma sobreposição de tokens com limiar de 70%.

### O professor da etapa 5

A conversa livre mantém o histórico das últimas doze mensagens e usa um prompt de sistema que fixa o nível em B1, limita a resposta a duas ou quatro frases, exige uma pergunta aberta ao final para manter o diálogo e pede a correção em uma linha separada no formato `Fix: <frase corrigida>`, para não interromper a conversa a cada erro.

### Voz

Duas APIs nativas do navegador, sem biblioteca externa:

- **`SpeechRecognition`** para o ditado, com resultados intermediários ligados e corte automático em 45 segundos.
- **`speechSynthesis`** para a leitura, com seleção da melhor voz disponível para o idioma e velocidade levemente reduzida no inglês.

O texto lido é extraído dos elementos marcados como conteúdo em inglês dentro do balão, e não do balão inteiro.

### Persistência

Configuração do Gemini e resultado da última sessão são gravados no `localStorage`, sob as chaves `vozli:ai` e `vozli:last`. Quando a API `window.storage` existe — caso do ambiente de artefatos do Claude — os mesmos dados são espelhados nela, e o que estiver lá é migrado para o `localStorage` na primeira carga. Toda leitura e escrita é protegida por `try/catch`, porque navegação anônima e janelas embutidas podem bloquear o armazenamento; nesse caso a configuração vale só para a sessão aberta e o aplicativo avisa ao salvar.

Nenhum dado sai do dispositivo. A chave da API fica apenas neste navegador.

---

## Base de conhecimento e fontes

O banco embutido tem **110 palavras** e **25 frases**, classificadas por nível CEFR e compiladas das referências oficiais abaixo. O aplicativo exibe essa lista no painel "Ver fontes".

| Fonte | O que foi usado |
|-------|-----------------|
| [Oxford 3000 e Oxford 5000](https://www.oxfordlearnersdictionaries.com/wordlists/oxford3000-5000) — Oxford Learner's Dictionaries | Núcleo de vocabulário, escolhido por frequência no Oxford English Corpus, com cada palavra alinhada de A1 a B2 |
| [A2 Key Vocabulary List](https://www.cambridgeenglish.org/Images/506886-a2-key-2020-vocabulary-list.pdf) — Cambridge English | Vocabulário de nível A2 das etapas 2 e 3 |
| [B1 Preliminary Vocabulary List](https://www.cambridgeenglish.org/pl/Images/506887-b1-preliminary-vocabulary-list.pdf) — Cambridge English | Vocabulário produtivo das etapas 4 e 5 |
| [English Vocabulary Profile](https://www.englishprofile.org) — English Profile | Atribuição de nível CEFR por verbete, expressão e phrasal verb |
| [Quadro Comum Europeu (CEFR)](https://www.coe.int/en/web/common-european-framework-reference-languages) — Conselho da Europa | Os seis níveis A1–C2 que organizam a progressão |

As listas pertencem às respectivas instituições e foram usadas como referência de nível e seleção. As traduções para o português são deste projeto.

---

## Executando localmente

Não há instalação, build nem gerenciador de pacotes. Basta abrir o arquivo:

```bash
git clone https://github.com/seu-usuario/vozli.git
cd vozli
```

Depois abra o `index.html` no navegador, ou sirva por HTTP, o que é recomendado porque o microfone exige contexto seguro:

```bash
python3 -m http.server 8000
# acesse http://localhost:8000
```

`localhost` conta como contexto seguro, então o microfone funciona sem certificado.

A única dependência externa é a família tipográfica IBM Plex, carregada do Google Fonts. Sem internet, o navegador cai para as fontes do sistema e o layout continua íntegro.

---

## Publicando no GitHub Pages

1. Renomeie o arquivo para `index.html`, tudo em minúsculas.
2. Crie um repositório **público** no GitHub, sem inicializar com README.
3. Use **Add file → Upload files**, arraste o `index.html` e confirme em **Commit changes**.
4. Vá em **Settings → Pages**, escolha **Deploy from a branch**, selecione a branch `main` e a pasta `/ (root)`, e salve.
5. Em um ou dois minutos o endereço aparece no formato `https://seu-usuario.github.io/vozli/`.

O GitHub Pages serve por HTTPS, o que é exatamente o que o navegador exige para liberar o microfone.

---

## Motor de IA: Google Gemini

O professor usa o Google Gemini para julgar sinônimos, explicar erros, conduzir o bate-papo da etapa 5 e escrever o relatório. É o único provedor, configurado pelo painel **Gemini** na barra lateral.

### Configuração

1. Gere uma chave gratuita em [aistudio.google.com/apikey](https://aistudio.google.com/apikey).
2. Abra **Configurar** no painel Gemini, cole a chave e clique em **Ver modelos da minha chave** — a lista passa a mostrar apenas os modelos que aquela chave realmente pode usar, o que evita o erro 404 de modelo inexistente.
3. Escolha um modelo, clique em **Testar conexão** e depois em **Salvar**. Chave, modelo e lista de modelos ficam gravados no `localStorage` do navegador, então não é preciso preencher de novo nas próximas visitas. O botão **Apagar chave** remove tudo.

### Como a chamada é montada

Endpoint `generativelanguage.googleapis.com/v1beta/models/{modelo}:generateContent`, com a chave no cabeçalho `x-goog-api-key`. As mensagens são convertidas para o formato nativo: o papel `assistant` vira `model` e o prompt de sistema vai em `system_instruction`.

O adaptador se adapta ao modelo escolhido em vez de assumir um formato fixo:

- **Controle de raciocínio por versão.** A família 2.5 recebe `thinkingConfig.thinkingBudget = 0` e a 3 em diante recebe `thinkingLevel = "low"`. Sem isso, os modelos com raciocínio gastam todo o orçamento pensando e devolvem texto vazio.
- **Remoção automática de campos desconhecidos.** Se a API responder `Unknown name "<campo>"`, o adaptador lê o nome na própria mensagem de erro, retira aquele campo e repete a chamada, até quatro vezes. Isso cobre parâmetros que variam de modelo para modelo sem precisar mapear cada um.
- **Instrução de sistema com alternativa.** Se o modelo não suportar `system_instruction`, o prompt passa a ser o início da conversa em vez de ser descartado.
- **Limite por requisição.** Cada chamada é abortada em 25 segundos, com uma nova tentativa depois de 2 segundos quando o erro é passageiro. O orçamento total é de 70 segundos.
- **Erros traduzidos.** Chave inválida, acesso negado, modelo inexistente, limite atingido, modelo congestionado, resposta cortada por tokens e bloqueio do filtro de conteúdo chegam como mensagens específicas, não como falha genérica.

### Limites e segurança

O nível gratuito tem cota por minuto e por dia. Isso pesa pouco aqui, porque a IA só é chamada quando a verificação local não reconhece a resposta — em uma sessão inteira costumam ser poucas chamadas.

> **Em página pública, a chave é enviada a partir do navegador do visitante.** Restrinja a chave por domínio no Google Cloud ou coloque um proxy na frente que guarde a chave no servidor.

### Sem chave configurada

O aplicativo continua utilizável, com degradação controlada, e avisa isso no início da sessão:

- as cinco etapas, o cronômetro, o ditado e a leitura em voz alta seguem funcionando
- a correção de palavras e frases usa a verificação local por normalização e sobreposição de tokens
- o relatório passa a ser o calculado por estatística
- sinônimos fora do gabarito são marcados como erro e a etapa 5 fica indisponível

---

## Compatibilidade

| Recurso | Chrome | Edge | Safari | Firefox |
|---------|:------:|:----:|:------:|:-------:|
| Interface, etapas, cronômetro | ✅ | ✅ | ✅ | ✅ |
| Leitura em voz alta (`speechSynthesis`) | ✅ | ✅ | ✅ | ✅ |
| Ditado por microfone (`SpeechRecognition`) | ✅ | ✅ | ✅ | ❌ |

O reconhecimento de fala não é implementado no Firefox. O aplicativo detecta a ausência e avisa no chat em vez de falhar em silêncio. Dentro de janelas incorporadas o navegador costuma negar o microfone — nesse caso, abra a página em uma aba própria.

---

## Personalizando o conteúdo

Os bancos ficam no topo do bloco `<script>`, em quatro constantes. Palavras seguem o formato `["inglês", ["tradução", "sinônimo aceito"]]`:

```javascript
const WORDS_A1 = [
  ["water",["agua"]],
  ["house",["casa"]],
  ["job",["emprego","trabalho"]]
];
```

Frases seguem `["frase em português", ["variante aceita 1", "variante aceita 2"]]`:

```javascript
const SENT_B1 = [
  ["Eu ainda não terminei o projeto.",
   ["i havent finished the project yet","i have not finished the project yet"]]
];
```

Duas regras importantes ao adicionar itens:

- Escreva as respostas aceitas **sem acentos, sem pontuação e em minúsculas**, porque a comparação acontece depois da normalização. `água` deve ser cadastrado como `agua`.
- Inclua as variantes contraídas e expandidas (`havent` e `have not`), já que o apóstrofo é removido na normalização.

Para mudar a exigência de cada etapa, ajuste o campo `need` na constante `STAGES`. Para mudar a duração da sessão, altere os dois lugares onde aparece `600` (em segundos): o valor inicial de `S.secs` e o divisor no cálculo da barra de progresso em `paintClock()`.

---

## Acessibilidade

- Contraste do texto principal e dos estados de erro verificado contra o fundo pastel.
- Foco visível com contorno de 2px em todos os controles.
- `aria-label` nos botões que só têm ícone: microfone, envio, descarte de gravação e leitura em voz alta.
- `prefers-reduced-motion` desliga a animação de entrada dos balões e o equalizador da gravação.
- Alvos de toque de 44px ou mais nos botões do compositor em telas estreitas.
- Tecla `Esc` fecha modais e o painel de plano; `Enter` envia e `Shift+Enter` quebra linha.

---

## Estrutura do arquivo

Arquivo único, cerca de 1.500 linhas e 65 KB, organizado em blocos comentados:

```
index.html
├── <style>            tema, layout, responsividade
├── <header>           marca, cronômetro, controles da sessão
├── <aside>            plano de estudo, placar, fontes
├── <section.chat>     fluxo de mensagens e compositor
├── <footer>           marca registrada e crédito
└── <script>
    ├── BANCO DE CONTEÚDO        palavras e frases por nível
    ├── ETAPAS                   definição e limiares
    ├── ESTADO                   sessão, placar, histórico, log
    ├── RENDER                   balões, placar, plano
    ├── CRONÔMETRO               contagem, pausa, reinício
    ├── FLUXO DO TREINO          perguntas, respostas, avanço
    ├── MOTOR DE IA              adaptador do Gemini e fallback local
    ├── PROFESSOR                julgamento e conversa
    ├── RELATÓRIO                geração e fallback local
    ├── VOZ                      síntese e reconhecimento
    └── CONTROLES                eventos da interface
```

---

## Roadmap

- [ ] Avaliação de pronúncia comparando o áudio transcrito com o texto esperado
- [ ] Histórico entre sessões, com curva de acertos ao longo dos dias
- [ ] Exportação do relatório em PDF
- [ ] Repetição espaçada: reintroduzir palavras erradas nas sessões seguintes
- [ ] Durações alternativas de 5 e 20 minutos
- [ ] Proxy serverless de referência pronto para copiar
- [ ] Streaming das respostas do professor na etapa 5

---

## Licença

Adicione um arquivo `LICENSE` ao repositório com a licença de sua escolha. Se não tiver preferência, a MIT é a escolha usual para projetos assim.

O código e as traduções para o português são deste projeto. As listas de vocabulário e os descritores de nível pertencem à Oxford University Press, ao Cambridge University Press & Assessment, ao English Profile e ao Conselho da Europa, e foram usados como referência, com atribuição dentro do próprio aplicativo.

VozLi® é marca registrada.

---

## Autor

Desenvolvido por **Rony Hericson dos Santos**.
