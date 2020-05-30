# Full Cycle - Desafio 03 - Processo de CI

1) Você deverá pegar sua aplicação Laravel das fases anteriores e  adicioná-la em um pipeline de integração contínua utilizando o Google Cloud Build, para isso terá que:

- Gerar a imagem do docker-compose e fazer o push no seu Container Registry do GCP. 
- Criar uma trigger para ser disparada todas as vezes que um commit entrar no repositório do Github.
- Os steps do Google Cloud Build deverão ser: 
  - Executar o docker-compose
  - Executar o composer
  - Copiar o arquivo .env.example para .env
  - Rodar um artisan key:generate
  - Executar as migrações
  - Executar os testes utilizando o PHPUnit



Solução:

- Instalar o docker-compose dentro do Registry do google

  - Criar novo Dockerfile e cloudbuild.yaml para gerar imagem com o docker-compose

    - Copiar os arquivos Dockerfile e cloudbuild.yaml do repositório [cloud-builders-community](https://github.com/GoogleCloudPlatform/cloud-builders-community/tree/master/docker-compose) 

    - Gerar um novo [repositório no Github](https://github.com/phmelo/fullcycle-desafio-ci-gcp-docker-compose), contendo os arquivos. 

    - Entrar no Google Cloud Platform

      - Entrar na sua aplicação.

      - Entrar no Google Cloud Build (Clicando no menu lateral no topo a esquerda. Em Tools)

        - Depois clique em Triggers ("Acionadores") e siga os passos da interface
          - Connect Repository (Github)
            - Ele vai pedir para autenticar e instalar o Google Cloud Build no Github.
          - Selecione o repositório do github
            - Criar push trigger (Depois edite)
            - Em build configuration, troque para o arquivo cloudbuild.yaml
            - Após finalizar a criação da Trigger, clique em "Run Trigger"

      - Depois de terminar o processo de build ela estará em Container Registry, pronta para ser usada.

        

- Criar nova Trigger para gerar novas imagens da aplicação Laravel com a imagem docker-compose.

  - Siga os passos no GCP da mesma forma que a anterior, só que dessa vez escolhendo este repositório e escolhendo o arquivo gcpbuild.yaml.
    - Primeiro ele vai usar o docker compose para executar o nosso arquivo yaml "docker-compose-gcp.yaml"
    - Depois ele instala as dependências do php composer
    - Depois ele copia o arquivo de configuração .env.example, criando um novo arquivo .env. 
      - Atenção nesse passopois, como não estamos mais usado variáveis de ambiente do Dockerize, precisamos alterar o .env.example e adicionando as credenciais do banco de dados e os hostnames dos container corretamente. (Certamente não é uma boa prática)
    - Depois geramos a chave e migrations do Laravel
    - E por fim, executamos os testes unitários de exemplo, usando phpunit.





2) Você deverá instalar a App do Google Cloud Build disponível no  Market Place do Github. Crie um branch develop em seu repositório. Todas as vezes que uma pull request for criada, imediatamente o Google Cloud  Build deverá iniciar o processo de CI.

Solução:

- Entrar no market place do Github e instalar o Google Cloud Build, vinculando a conta e o projeto criado no GCP.
- Alterar a Trigger responsável pelo pipeline do Laravel para só ser chamada caso seja criado um pull request no GitHub, na branch "develop"
  - Event: Pull request (GitHub App only)
- Criar um novo branch no projeto Laravel, chamado Develop, fazer uma alteração e jogar no repositório.
  - git checkout -b develop
  - Alterar qualquer arquivo
  - git add .
  - git commit -m 'Commit para gerar um pull request'
  - git push origin develop
- Entrar no GitHub e criar um novo pull request
  - Clicar no botão "Compare & pull request"
  - Criar o Pull request
  - Observar os testes sendo executados
    - Pode entrar no GCP, e olhar a build history que foi iniciado.
  - Fazer merge com a master



Bonus:

https://cloud.google.com/sdk/install

