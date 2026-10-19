# Especificação de melhorias do AstroHopper

**Estado:** primeiro ciclo implementado localmente; pendente de revisão em dispositivos reais  
**Versão:** 0.2  
**Idioma:** português do Brasil  
**Repositório de referência:** [artyom-beilis/skyhopper](https://github.com/artyom-beilis/skyhopper)

Este documento registra as ideias iniciais para uma versão derivada do AstroHopper. O objetivo é transformar as sugestões em requisitos verificáveis antes de implementar cada funcionalidade. As decisões marcadas como **a definir** devem ser refinadas em conjunto.

## Estado da implementação

- **Implementado:** sugestões de busca por prefixo, seleção por toque/mouse/teclado, rótulo local do apontamento, arrasto manual por toque e mouse, seta com ponta para o alvo, direções localizadas e Wikipédia em português para `pt_BR`.
- **Preservado:** busca direta no campo de configurações, alinhamento existente, seleção de objetos no mapa, pinça para zoom e retorno ao modo de bússola.
- **Pendente de validação manual:** sensores de orientação em aparelhos reais, ergonomia em tela estreita e comportamento do teclado virtual.
- **Fora deste incremento:** os itens priorizados na seção 6, como favoritos, filtros, ficha detalhada e planejamento de sessão.

## 1. Objetivo do produto

Facilitar a busca e o apontamento de estrelas e objetos celestes, principalmente em celulares usados junto a um telescópio. A pessoa deve conseguir encontrar um alvo pelo nome, entender para que lado mover o telescópio e reconhecer qual direção do céu corresponde ao apontamento atual.

As melhorias devem preservar o uso simples em campo, funcionar em telas pequenas e respeitar a interface de observação noturna do projeto.

## 2. Situação atual observada no código

- A busca principal aceita texto e chama `findTargetByName()`. Ela resolve nomes exatos e prefixos quando há uma única correspondência. Prefixos ambíguos não apresentam uma lista para escolha.
- A função `plotBearing()` desenha uma linha de orientação e escreve os valores `Up`, `Down`, `Left` e `Right` diretamente no canvas. Esses textos não passam por `_tr()` nem fazem parte do catálogo gettext, por isso aparecem em inglês mesmo com o idioma pt-BR selecionado.
- O movimento manual passa por `manualMove()` e `moveSky()`. No caminho atual ele depende de toque, só altera o deslocamento horizontal e é ignorado quando a bússola está ativa.
- A tela já mostra o alvo selecionado no mapa. A orientação atual e o alvo são conceitos diferentes e devem continuar visualmente identificáveis.

## 3. Princípios e limites

1. **Uso em campo:** ações principais acessíveis com uma mão e sem exigir digitação longa.
2. **Leitura noturna:** manter fundo escuro, contraste suficiente e evitar flashes ou grandes áreas brancas.
3. **Escolha explícita:** sugestões de busca não devem trocar o alvo sem a confirmação da pessoa.
4. **Compatibilidade:** busca, alinhamento por sensores, modo manual e zoom existentes devem continuar funcionando.
5. **Tolerância a sensores:** informar quando bússola ou orientação não estiverem disponíveis; não apresentar uma direção como precisa se o app não puder calculá-la.
6. **Privacidade:** nomes personalizados, favoritos e preferências ficam no dispositivo, salvo decisão futura explícita de sincronização.
7. **Entrega incremental:** implementar e revisar cada melhoria em uma alteração pequena, com critérios de aceite próprios.

## 4. Escopo inicial

### Incluído no primeiro ciclo

1. Sugestões selecionáveis enquanto a pessoa digita o nome de uma estrela ou objeto.
2. Nome configurável para o indicador do apontamento atual, com opções iniciais “Você” e “Seu telescópio”.
3. Movimento manual do mapa por arrastar e soltar, tanto em tela touch como com mouse.
4. Seta que indique a direção do alvo selecionado em relação ao apontamento atual.
5. Corrigir a tradução pt-BR de cima/baixo/esquerda/direita que hoje é desenhada diretamente no canvas.
6. Abrir a Wikipédia em português quando o idioma selecionado no app for português.

### Fora do escopo inicial

- Contas, login, sincronização com nuvem ou compartilhamento de dados pessoais.
- Alterar os modelos astronômicos, catálogos ou cálculos de posição do projeto.
- Navegação por voz ou instruções de áudio.
- Transformar o mapa em um planetário com recursos além da busca e orientação existentes.

## 5. Requisitos funcionais

### RF-01 — Busca com sugestões

**História:** como pessoa observadora, quero começar a digitar o nome de uma estrela ou objeto e escolher uma sugestão compatível, para não precisar saber ou digitar o nome exato.

**Comportamento proposto**

- A busca principal apresenta uma lista de sugestões conforme o texto é digitado.
- A lista considera os nomes disponíveis no catálogo. Deve permitir ampliar depois para nomes alternativos e identificadores como Messier, NGC e IC.
- Cada opção mostra o nome reconhecido e, quando existir no catálogo, o tipo do objeto e um identificador complementar.
- A pessoa precisa escolher uma opção para confirmar o alvo. Digitar um prefixo ambíguo não seleciona automaticamente o primeiro resultado.
- A busca não diferencia maiúsculas de minúsculas e deve preservar a normalização já usada pelo projeto para nomes e números.
- Resultados duplicados que apontam para o mesmo objeto devem aparecer uma única vez.
- O componente deve funcionar por toque, mouse e teclado. Teclado: `↑`/`↓` percorrem opções, `Enter` confirma a opção destacada e `Esc` fecha a lista sem alterar o alvo confirmado.
- A lista deve caber em telas estreitas, permitir rolagem quando necessário e não ficar escondida atrás do teclado virtual ou dos controles da tela.
- Ao confirmar um resultado, o app seleciona o objeto e fecha as sugestões. Ao apagar o texto, o alvo só é limpo conforme o comportamento atual ou uma decisão explícita da pessoa.

**Estados da busca**

| Estado | Comportamento |
| --- | --- |
| Campo vazio | Não mostra sugestões; mantém o texto de orientação localizado. |
| Há correspondências | Mostra opções ordenadas de forma estável, começando pelas correspondências mais diretas. |
| Nenhuma correspondência | Informa “Nenhum objeto encontrado” em uma mensagem localizada. |
| Catálogo ainda não carregado | Não exibe resultado incorreto; mostra estado de carregamento se a busca já estiver disponível. |
| Opção selecionada | Confirma o alvo, atualiza o mapa e fecha a lista. |

**Critérios de aceite**

- Ao digitar um prefixo com mais de uma correspondência, as opções são exibidas e nenhuma é selecionada por conta própria.
- Ao tocar ou clicar em uma sugestão, o alvo correto é selecionado.
- Com teclado, é possível abrir, percorrer, confirmar e fechar as sugestões sem usar o mouse.
- Uma busca sem correspondências apresenta uma mensagem em português quando o idioma pt-BR está ativo.
- A busca existente por nome completo e a seleção de objetos no mapa continuam funcionando.

**A definir:** limite inicial de sugestões visíveis; ordem precisa dos resultados; inclusão de aliases e nomes populares já no primeiro incremento; aplicação também ao campo de busca das configurações.

### RF-02 — Nome do indicador do apontamento atual

**História:** como pessoa observadora, quero identificar no mapa a direção para a qual o telefone ou telescópio está apontando por um nome familiar, para distinguir essa referência do objeto que estou procurando.

**Interpretação inicial:** o nome se refere ao indicador da direção atual de apontamento no céu. Ele não representa o local geográfico de observação nem renomeia uma estrela do catálogo.

**Comportamento proposto**

- O indicador acompanha a direção atual calculada pelos sensores quando eles estão disponíveis e alinhados.
- A pessoa pode escolher “Você”, “Seu telescópio” ou informar um nome próprio.
- O nome aparece ao lado do indicador, com tamanho e posição que não cubram o alvo ou as instruções de movimento.
- A configuração pode ser alterada a qualquer momento e fica salva localmente no dispositivo.
- Se a orientação não estiver disponível, o app informa que o indicador não pode acompanhar o apontamento atual. Não deve deixar uma posição antiga parecendo atual.
- Deve haver uma forma simples de ocultar o rótulo, sem desativar alinhamento ou busca.

**Critérios de aceite**

- A pessoa consegue selecionar “Você” ou “Seu telescópio” sem digitar.
- A pessoa consegue salvar, alterar e limpar um nome personalizado.
- O rótulo acompanha a direção atual quando o telefone é movido com sensores ativos.
- Em modo manual ou sem sensores, o estado do indicador fica claro e não é apresentado como posição ao vivo.
- O texto não encobre botões, sugestões de busca ou o marcador do alvo em telas pequenas.

**A definir:** se o rótulo acompanha o centro da tela, um marcador próprio no mapa ou os dois; onde fica a configuração; valor inicial; limite de caracteres; persistência ao trocar de idioma ou limpar dados.

### RF-03 — Arrastar e soltar o mapa

**História:** como pessoa observadora, quero arrastar o mapa para explorar o céu manualmente, para ajustar a visualização sem depender dos sensores do telefone.

**Comportamento proposto**

- Permitir deslocamento horizontal e vertical do mapa quando o modo manual estiver ativo ou quando o app oferecer uma ação explícita para entrar nele.
- Funcionar com toque e mouse. O comportamento deve usar o mesmo sentido de arrasto em todos os dispositivos.
- Ao iniciar o arrasto, não selecionar acidentalmente uma estrela ou objeto no fim do gesto.
- Um toque curto sem deslocamento continua podendo selecionar um objeto, conforme o comportamento atual.
- Preservar o gesto de pinça para zoom e evitar que um gesto de zoom também mova o mapa de forma inesperada.
- Não rolar a página enquanto a pessoa arrasta sobre o mapa; os controles fora do mapa continuam utilizáveis normalmente.
- Disponibilizar uma ação visível para voltar ao apontamento dos sensores ou recentralizar o mapa.
- Indicar claramente se o mapa está em orientação por sensores ou em movimento manual.

**Critérios de aceite**

- Arrastar o mapa nos dois eixos muda a região do céu exibida no modo manual.
- O mesmo gesto funciona em celular touch e computador com mouse.
- Um toque curto seleciona o objeto sob o toque; um arrasto não seleciona o objeto final por engano.
- Pinça continua ajustando o campo de visão sem movimentos bruscos ou saltos.
- A ação de recentralização retorna à orientação atual quando os sensores estiverem disponíveis.
- Ativar ou encerrar o movimento manual não apaga o alvo selecionado nem desfaz o alinhamento já calculado.

**A definir:** se arrastar ativa o modo manual automaticamente; direção do arrasto em relação às estrelas; ação e rótulo do botão de recentralizar; suporte a trackpad.

### RF-04 — Seta orientativa para o alvo

**História:** como pessoa observadora, quero ver uma seta que aponte para o objeto selecionado, para saber em que direção movimentar o telescópio.

**Comportamento proposto**

- A seta deve representar a direção do alvo em relação ao apontamento atual, usando a posição projetada no mapa e o estado de alinhamento disponível.
- Quando o alvo estiver visível na área do mapa, a indicação aponta diretamente para ele ou para seu marcador.
- Quando o alvo estiver fora da área visível, uma seta posicionada junto à borda indica para que lado está o alvo. A seta não deve ser confundida com o marcador do apontamento atual.
- A orientação pode apresentar também os graus restantes nos eixos horizontal e vertical, mantendo os valores existentes quando forem úteis.
- A seta e os textos usam as cores e o contraste da interface noturna e não encobrem controles.
- Sem alvo selecionado, a seta fica oculta. Sem dados suficientes de alinhamento ou orientação, o app explica o estado e evita sugerir uma direção possivelmente errada.
- A indicação se atualiza enquanto o telefone ou o mapa se move, sem piscar ou saltar entre posições próximas.

**Critérios de aceite**

- Selecionar um alvo visível apresenta indicação coerente entre o centro/apontamento e o alvo.
- Selecionar um alvo fora da tela mantém a direção visível por uma seta na borda.
- Mover o telefone ou o mapa atualiza a indicação em tempo real de acordo com o modo ativo.
- Remover o alvo selecionado remove a indicação.
- A indicação nunca cobre os controles de busca, alinhamento e configurações em telas pequenas.
- A direção não é mostrada como confiável quando o app ainda não tem alinhamento suficiente.

**A definir:** seta sobreposta ao mapa ou em uma bússola dedicada; comportamento quando o alvo está atrás do observador; inclusão de distância angular; regra exata para requerer alinhamento.

### RF-05 — Localizar direções no idioma selecionado

**História:** como pessoa que usa o app em português, quero ler as direções de movimento no idioma selecionado, para não precisar traduzir “Up”, “Down”, “Left” e “Right” durante a observação.

**Comportamento proposto**

- Substituir os textos fixos desenhados no canvas por mensagens que passem pelo sistema de tradução.
- Incluir as mensagens correspondentes no catálogo pt-BR e manter o fallback dos demais idiomas.
- Revisar os textos próximos de orientação para identificar outras mensagens dinâmicas não localizadas.

**Critérios de aceite**

- Com pt-BR ativo, as indicações exibem “Cima”, “Baixo”, “Esquerda” e “Direita”.
- Com outro idioma ativo, as direções usam a tradução disponível ou o fallback existente.
- Os valores angulares e símbolos de grau permanecem corretos.
- A alteração não afeta a matemática nem a posição do desenho no canvas.

### RF-06 — Abrir a Wikipédia no idioma selecionado

**História:** como pessoa que usa o app em português, quero abrir informações do alvo na Wikipédia em português, para continuar a consulta no idioma que escolhi no aplicativo.

**Comportamento proposto**

- Ao acionar a informação/Wikipédia de um alvo com a interface em português (incluindo `pt_BR`), abrir a busca correspondente em `pt.wikipedia.org`.
- Preservar o nome ou identificador usado atualmente para pesquisar o objeto, incluindo a conversão existente de identificadores Messier, NGC e IC quando aplicável.
- Codificar o texto de busca como parâmetro de URL para que espaços, acentos e caracteres especiais não quebrem o endereço.
- Para outros idiomas, manter o comportamento existente até que haja uma regra de idioma correspondente.
- A escolha do site deve acompanhar o idioma selecionado no app, não apenas o idioma preferido do navegador ou do sistema operacional.

**Critérios de aceite**

- Com pt-BR selecionado, abrir a informação de uma estrela ou objeto direciona para uma busca na Wikipédia em português.
- O termo pesquisado corresponde ao alvo selecionado e permanece corretamente codificado na URL.
- Trocar o app para inglês mantém a busca na Wikipédia em inglês, conforme o comportamento atual.
- A busca funciona com nomes contendo espaços, acentos ou identificadores de catálogo.

**A definir:** aplicar a mesma lógica de seleção de idioma aos demais idiomas disponíveis no AstroHopper ou limitar inicialmente a mudança a pt-BR.

## 6. Melhorias adicionais para priorizar

Estas opções podem ser consideradas depois do primeiro ciclo ou incorporadas após refinamento:

| Prioridade sugerida | Melhoria | Benefício |
| --- | --- | --- |
| P1 | Favoritos e lista de observação | Salvar alvos para uma sessão e avançar entre eles rapidamente. |
| P1 | Ficha rápida do objeto | Mostrar tipo, magnitude, constelação, altitude e azimute do alvo selecionado. |
| P1 | Filtros de busca | Filtrar estrelas, planetas, nebulosas, galáxias, aglomerados e objetos acima do horizonte. |
| P1 | Estado dos sensores | Mostrar de forma simples se GPS, bússola e orientação estão disponíveis, alinhados ou precisam de calibração. |
| P2 | Histórico de alvos recentes | Retornar a objetos pesquisados sem repetir a busca. |
| P2 | Preferências de sessão | Lembrar idioma, limite de magnitude, campo de visão e nome do apontamento localmente. |
| P2 | Ajuda contextual curta | Explicar alinhamento, seta, modo manual e permissões no momento em que forem necessários. |
| P2 | Acessibilidade e ergonomia | Melhorar alvos de toque, escala de texto, foco de teclado e leitura por tecnologia assistiva. |
| P2 | Modo de alto contraste noturno | Oferecer paleta ajustável, preservando baixa emissão de luz e legibilidade. |
| P3 | Compartilhar uma lista de observação | Exportar ou importar alvos em formato simples, sem exigir conta. |
| P3 | Planejamento de sessão | Ordenar alvos por visibilidade e horário para uma noite de observação. |
| P3 | Feedback de apontamento | Exibir um estado de proximidade ao alvo quando a direção estimada estiver dentro de uma tolerância configurável. |

As prioridades são sugestões. O escopo de cada item precisa considerar a precisão dos sensores, o catálogo disponível e a experiência do projeto original.

## 7. Fluxos principais propostos

### Encontrar um objeto

1. A pessoa toca no campo de busca.
2. Digita parte do nome ou identificador.
3. O app mostra opções compatíveis.
4. A pessoa escolhe uma opção.
5. O mapa marca o alvo e a seta indica como alcançá-lo.
6. A pessoa move o telescópio e acompanha a indicação até o alvo.

### Explorar manualmente

1. A pessoa ativa o modo manual ou inicia o arrasto conforme a decisão final de interação.
2. Arrasta o mapa para a região desejada.
3. Toca em um objeto ou faz uma busca para selecioná-lo.
4. A indicação de alvo se atualiza relativamente ao mapa manual.
5. A pessoa escolhe recentralizar para voltar à direção dos sensores.

### Identificar o apontamento atual

1. A pessoa abre a configuração do rótulo de apontamento.
2. Escolhe “Você”, “Seu telescópio” ou informa um nome.
3. O mapa apresenta o rótulo ao lado da referência de apontamento.
4. Se os sensores deixarem de fornecer orientação, o app informa que a referência não está atualizada.

## 8. Requisitos de interface e acessibilidade

- Usar textos curtos e consistentes em português brasileiro.
- Traduzir todos os estados, mensagens de erro e instruções introduzidos pelas funcionalidades.
- Preservar a aparência escura e a paleta adequada à observação noturna.
- Garantir áreas de toque adequadas em celular e não depender apenas de cor para diferenciar alvo, apontamento e seta.
- Manter foco visível e ordem de teclado previsível nos campos, listas e botões.
- Não bloquear o zoom do navegador fora da área de interação do mapa.
- Conferir rótulos longos e nomes astronômicos em telas estreitas e em orientação paisagem.

## 9. Requisitos de qualidade

- A busca deve permanecer responsiva com o catálogo completo carregado.
- Atualizações da seta não devem exigir redesenho mais caro do que o ciclo atual do mapa.
- Gestos de toque devem evitar seleção acidental, rolagem da página e conflito entre arrasto e pinça.
- A experiência principal deve continuar disponível offline depois que os arquivos do app e o catálogo forem carregados, conforme o funcionamento atual do projeto.
- Preferências pessoais devem ser armazenadas localmente e removíveis pelo usuário.
- Novas mensagens precisam entrar no fluxo de tradução do projeto para que outros idiomas possam traduzi-las.

## 10. Estratégia de entrega

1. **Ajuste pt-BR:** localizar as quatro direções do canvas e revisar textos de orientação não catalogados.
2. **Busca:** definir aliases, ordenação e comportamento de seleção; implementar e revisar isoladamente.
3. **Seta:** validar primeiro a geometria visual e estados sem alinhamento, com alvo dentro e fora da tela.
4. **Arrasto manual:** integrar mouse e toque, verificando seleção, pinça e retorno aos sensores.
5. **Rótulo de apontamento:** finalizar a semântica do marcador e as regras de persistência.
6. **Revisão em dispositivos:** testar tela estreita, orientação paisagem, teclado virtual, mouse e sensores reais.
7. **Pull requests pequenos:** separar funcionalidades quando isso reduzir conflitos e facilitar a revisão do projeto original.

## 11. Plano de verificação

Para cada funcionalidade, registrar navegador e dispositivo usados e conferir:

- fluxo normal e estado sem resultados;
- modo pt-BR e pelo menos um idioma existente;
- tela estreita e orientação paisagem;
- uso com toque e, quando aplicável, teclado e mouse;
- interação com alinhamento, zoom, seleção no mapa e modo manual;
- ausência de sobreposição entre seta, rótulos, lista de sugestões e controles;
- operação sem rede após os recursos locais estarem carregados.

## 12. Decisões pendentes para a próxima revisão

1. A busca deve sugerir apenas nomes oficiais do catálogo ou também nomes populares e aliases desde a primeira versão?
2. O que exatamente deve receber o nome “Você” ou “Seu telescópio”: o centro da tela, um marcador da direção atual ou o local geográfico de observação?
3. O movimento manual deve iniciar ao arrastar ou exigir primeiro a ativação de um botão de modo manual?
4. A seta deve ficar sobre o mapa ou em um componente separado, como uma bússola?
5. O indicador de apontamento, a seta do alvo e a linha de direção atual devem coexistir? Como diferenciá-los visualmente?
6. Quais melhorias adicionais entram no primeiro ciclo: favoritos, ficha do objeto, filtros ou estado dos sensores?
7. Quais navegadores e aparelhos devem ser considerados mínimos para a primeira entrega?

## 13. Glossário

- **Alvo:** estrela ou objeto celeste selecionado para encontrar.
- **Apontamento atual:** direção para a qual o telefone/telescópio está orientado segundo os sensores ou o modo manual.
- **Alinhamento:** referência usada pelo app para relacionar a orientação do aparelho com a posição no mapa celeste.
- **Modo manual:** navegação do mapa sem seguir diretamente a orientação dos sensores.
- **Campo de visão:** extensão angular do céu exibida na área do mapa.
