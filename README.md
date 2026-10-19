# AstroHopper pt-BR

Este repositório é um fork pessoal do [AstroHopper original](https://github.com/artyom-beilis/skyhopper), criado para manter a tradução em português do Brasil e desenvolver melhorias próprias para a busca e a orientação no céu.

O aplicativo ajuda a localizar estrelas e outros objetos celestes usando os sensores do celular, como giroscópio, acelerômetro e bússola. Ele funciona no navegador e pode continuar disponível offline depois que os recursos forem carregados.

## O que esta versão acrescenta

- Interface e manual em português do Brasil. A Wikipédia abre em português quando esse idioma está selecionado.
- Sugestões de objetos enquanto você digita na busca, com seleção por toque, mouse ou teclado.
- Botão `×` para desmarcar o alvo atual.
- Linha e seta de orientação até o alvo, com os deslocamentos angulares atualizados durante o apontamento. A indicação fica ciano enquanto você se move e verde quando o sistema reconhece o alinhamento.
- Cruzeta e rótulo para mostrar a direção atual do telescópio. O rótulo pode ser “Você”, “Seu telescópio”, um texto personalizado ou ficar oculto.
- Leituras de altitude e azimute da direção física do telescópio.
- Arraste para explorar outras regiões do mapa sem alterar a posição real do telescópio. Use `↻` para voltar à visualização dos sensores.

O manual traduzido está em [po/README_pt_BR.md](po/README_pt_BR.md).

## Executar no computador

É necessário ter Python instalado. No PowerShell do Windows, na pasta do projeto:

```powershell
py -3 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe deploy.py
.\.venv\Scripts\python.exe pyserver.py
```

Com o servidor ativo, abra <https://localhost:8443/>. O navegador pode avisar que o certificado HTTPS é local; aceite o aviso para continuar. Mantenha a janela do servidor aberta enquanto usa o aplicativo.

## Testar em um celular na mesma rede Wi-Fi

1. Conecte o computador e o celular à mesma rede Wi-Fi.
2. No Windows, execute `ipconfig` e anote o **Endereço IPv4** do adaptador **Wi-Fi**.
3. No celular, abra `https://IP-DO-COMPUTADOR:8443/`, substituindo o texto pelo endereço encontrado. Por exemplo: `https://192.168.1.2:8443/`.
4. Se o Windows solicitar, permita o Python na rede privada. Se necessário, permita conexões de entrada na porta TCP `8443` para o perfil de rede privada.
5. Aceite o aviso do certificado local no navegador do celular.

Se a página carregar mas os sensores não funcionarem, confira as permissões de orientação e localização do navegador. Para alinhar, aponte o telescópio para uma estrela conhecida, toque em **Alinhar**, selecione essa estrela no mapa e aguarde a contagem terminar. Depois escolha um alvo pela busca ou pelo mapa e acompanhe a seta e os valores angulares.

## Uso do mapa

- Pesquise o nome de uma estrela, planeta ou objeto celeste e selecione uma das sugestões.
- Para remover a seleção, toque em `×` ao lado da busca.
- A cruzeta identifica para onde o telescópio está apontando; o marcador do alvo e a linha tracejada mostram a direção a alcançar.
- Arraste o mapa para explorar o céu. O movimento altera apenas a visualização; `↻` restaura a visão baseada nos sensores.
- Use o botão de configurações para escolher o idioma, personalizar o rótulo de apontamento e ajustar as demais opções.

## Relação com o projeto original

Este fork mantém a atribuição e a licença do AstroHopper. O projeto original foi criado por [Artyom Beilis](https://github.com/artyom-beilis/skyhopper). Consulte [COPYING.md](COPYING.md) para os termos de licença e o repositório original para a documentação e o histórico do projeto.

As alterações deste fork são mantidas no repositório de Pablo Dias Vieira. Elas não são incorporadas automaticamente ao projeto original; propostas de contribuição ao AstroHopper original são feitas separadamente por meio de Pull Requests.
