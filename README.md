# SandboxMudBlazor

Projeto criado com o propósito de servir como aplicação de testes do pacote de componentes Mudblazor, bem como aprendizado de técnicas de deploy de aplicações Blazor WASM em diferentes PAAS.

## Heroku

A plataforma Heroku não fornece suporte oficial para deploy de aplicações .NET, entretanto, podemos utilizar **build packs** criados pela comunidade para complementar os serviços oferecidos.

Para esse fim, foram trabalhadas duas abordagens: deploy da aplicação estática (Blazor WASM) por meio de um buildpack oficial ou deploy da aplicação estática em um container docker executando um servidor Nginx interno.

### Instruções para deploy da aplicação no Heroku como aplicação estática com heroku-buildpack-static:

Todos os comandos a seguir devem ser executados no mesmo nível do projeto onde se encontra o arquivo `.csproj` da aplicação que será levantada.

1. Toda nova aplicação no Heroku tem heroku-20 (Ubuntu 20.04) como sua stack padrão, mas caso esta tenha sido alterada previamente, antes de mais nada, devemos redefinir a stack da aplicação para heroku-20.

    `heroku stack:set heroku-20 -a $appName`
    
2. Cadastramos o buildpack `heroku-buildpack-static` na aplicação, o que nos permitirá executar o processo de deploy da aplicação .NET estática. Este processo apenas é possível devido o Blazor WASM ser um framework com renderização client-side, ou seja, não precisamos de execução de código .NET no servidor já que toda a execução é realizada no próprio navegador do usuário.

    `heroku buildpacks:set https://github.com/heroku/heroku-buildpack-static -a $appName`

3. Instalamos localmente o plugin `heroku-cli-static`. Este plugin se trata de uma extensão à gama de comandos disponíveis na heroku-cli, o que permite que trabalhemos com configurações e processos de deploy junto ao `heroku-buildpack-static`.
 
    `heroku plugins:install heroku-cli-static`
    
4. Após instalação do plugin, partimos para a criação do arquivo `static.json`, que contém parâmetros que serão utilizados na hora de realizar o deploy da aplicação para o Heroku.

    `heroku static:init`
    
    Em seguida, será solicitado o preenchimento de alguns parâmetros, que devem ser informados na seguinte forma:

    `{
      "root": "release/wwwroot",
      "clean_urls": true,
      "error_page": "index.html"
    }`

5. Após configurado todo o ambiente para deploy da aplicação estática, devemos buildar/publicar a aplicação por meio da `dotnet-cli`, o parâmetro `-c` especifica o tipo de compilação a ser realizada, já o parâmetro `-o` define a pasta do projeto para onde a publicação deverá ser realizada. Nesse caso, uma pasta chamada `release` será criado no mesmo nível do arquivo `.csproj`, a pasta da publicação contém a pasta `wwwroot`, com o conteúdo estático a ser disponibilizado no Heroku, conforme configurado no passo 4.
    
    - Para ambiente de testes: `dotnet publish -c Debug -o release`
    
    - Para ambiente de produção: `dotnet publish -c Release -o release`
    
6. Em seguida, realizamos o deploy da aplicação publicada no passo 5. Caso duas aplicações tenham sido configuradas no Heroku para diferentes ambientes (testes/proução), a diferenciação pode ser realizada nesse passo.

    `heroku static:deploy -a $appName`
    
**Observação:** Após a configuração inicial, apenas os passos 5 e 6 precisam ser realizados para atualizar a aplicação lançada.

### Instruções para deploy da aplicação no Heroku como container Docker com Nginx:

**ATENÇÃO:** Este seria meu método de deploy preferido, já que é mais prático implantar um fluxo de CI-CD por meio dele, entretanto, devemos estabelecer uma porta no container para ser utilizada pelo NGinx, e o Heroku disponibiliza uma porta aleatória a cada deploy. Como não podemos prever a porta utilizada no momento da configuração do servidor,  esse tipo de lançamento torna-se inviável da forma como está implementado hoje. Mais pesquisas podem trazer uma solução para esse problema.

1. Antes de mais nada, devemos alterar a stack da aplicação para container, dessa forma, o Heroku entenderá que se trata de uma aplicação containerizada em uma imagem Docker.

    `heroku stack:set container -a $appName`

2. Para esse método de deploy, 2 arquivos são essenciais: `dockerfile` e `nginx.conf`, ambos já disponibilizados no repositório, no mesmo nível do arquivo `.csproj` da aplicação Blazor WASM. O `dockerfile` está configurado para buildar a aplicação e disponibilizar os arquivos estáticos diretamente na pasta `wwwroot` do servidor NGinx, e em seguida, alterar o arquivo `default.conf` do container pelo arquivo `nginx.conf`, estabelecendo o acesso da aplicação pela porta configurada.
    
    `docker build --rm -t $containerName .`
    
3. Após buildar e testar o container, o enviamos para a plataforma Heroku, lembrando que o processo de push apenas envia o container para a plataforma, ainda não o disponibiliza no ambiente selecionado.

    `heroku container:push web -a $appName`

4. Concluido o deploy, realizamos o release do container enviado, o que disponibilizará a aplicação do container para acesso dos usuários no ambiente selecionado.

    `heroku container:release web -a $appName`

## Render.com

TODO

## Digital Ocean

TODO
