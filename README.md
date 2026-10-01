# Vagrant + Jenkins · Infraestrutura e entrega de uma API Node.js

Projeto desenvolvido durante o Bootcamp DevOps da FAP, com o objetivo de provisionar duas máquinas virtuais utilizando Vagrant e implementar uma pipeline CI/CD com Jenkins para uma aplicação Node.js.

O ambiente utiliza uma VM para executar o Jenkins e outra como servidor de produção. A comunicação entre as máquinas é realizada por SSH, enquanto o deploy da aplicação é feito utilizando SCP.

## 1. Tecnologias utilizadas

* Vagrant e VirtualBox
* Ubuntu Server
* Shell Script
* Git e GitHub
* Jenkins
* Node.js e npm
* Express
* Jest e Supertest
* SSH e SCP

## 2. Arquitetura do projeto

O ambiente é composto por duas máquinas virtuais, provisionadas pelo Vagrant.

| Máquina | IP            | Responsabilidade               |
| ------- | ------------- | ------------------------------ |
| Jenkins | 192.168.56.10 | Executar a pipeline CI/CD      |
| Prod    | 192.168.56.20 | Receber e executar a aplicação |

A VM Jenkins é responsável por obter o código do GitHub, instalar as dependências, executar o build, realizar os testes automatizados e enviar a aplicação para a VM Prod.

A VM Prod recebe os arquivos e disponibiliza a aplicação Node.js na porta 3000.

### Caminho da entrega

| Etapa | Onde acontece | Resultado |
| --- | --- | --- |
| Obter o código | Jenkins, a partir do SCM configurado | Workspace com o repositório |
| Install, Build e Test | VM Jenkins | Dependências instaladas, build simulado e testes executados |
| Deploy por SCP | Jenkins → VM Prod | Arquivos copiados para `/home/vagrant/app-prod/` |
| Instalar e iniciar | VM Prod, manualmente | API disponível na porta 3000 |

A transferência é automatizada; a instalação das dependências e a inicialização na Prod ainda exigem ação manual.

## 3. Estrutura do projeto

| Caminho | Finalidade |
| --- | --- |
| `Vagrantfile` | Define as duas VMs e seus provisionadores |
| `vagrant/scripts/setup-jenkins.sh` | Instala Java 21, Node.js 20 e Jenkins |
| `vagrant/scripts/setup-prod.sh` | Instala Node.js 22 na Prod |
| `Jenkinsfile` | Executa Install, Build, Test e transferência por SCP |
| `app/src/` e `app/server.js` | Código e inicialização da API |
| `app/test/` | Testes com Jest e Supertest |
| `app/package.json` e `app/package-lock.json` | Scripts e dependências |

O `Vagrantfile` define a configuração das duas máquinas virtuais.

Os scripts Shell realizam o provisionamento dos ambientes, enquanto o `Jenkinsfile` define as etapas da pipeline.

## 4. Provisionamento com Vagrant

O Vagrant foi utilizado para automatizar a criação e a configuração das máquinas virtuais.

O ambiente utiliza Ubuntu Server, com hostnames e endereços IP distintos para cada VM.

### Inicialização das máquinas

Requisitos: Git, Vagrant, VirtualBox e virtualização habilitada no computador. São duas VMs de 1 GB cada; reserve recursos também para o sistema host.

```bash
git clone https://github.com/ded3v/vagrant-jenkins-cicd.git
cd vagrant-jenkins-cicd
```

Na raiz do projeto, execute:

```bash
vagrant up
```

Para verificar o estado das máquinas:

```bash
vagrant status
```

Para acessar a VM Jenkins:

```bash
vagrant ssh jenkins
```

Para acessar a VM Prod:

```bash
vagrant ssh prod
```

### Provisionamento da VM Jenkins

O script `setup-jenkins.sh` prepara o ambiente necessário para executar o Jenkins e a pipeline.

A máquina utiliza Jenkins, Java, Node.js e npm.

### Provisionamento da VM Prod

O script `setup-prod.sh` prepara o ambiente de produção, instalando o Node.js e o npm.

Durante o desenvolvimento, o provisionamento foi atualizado para utilizar o Node.js 22, garantindo a compatibilidade com as dependências da aplicação.

### Acesso ao Jenkins

Abra [http://localhost:8050](http://localhost:8050) ou [http://192.168.56.10:8080](http://192.168.56.10:8080).

Para obter a senha inicial:

```bash
vagrant ssh jenkins
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Conclua a configuração inicial e instale o plugin **SSH Agent**. Crie um job Pipeline com **Pipeline script from SCM**, a URL deste repositório, branch `*/main` e caminho `Jenkinsfile`.

## 5. Comunicação SSH entre as máquinas

A autenticação SSH foi configurada para permitir que o Jenkins se conecte à VM Prod sem precisar informar a senha durante a execução da pipeline.

Foi utilizado um par de chaves SSH do tipo Ed25519.

A chave pública foi autorizada na VM Prod, enquanto a chave privada foi cadastrada no Jenkins como uma credencial SSH.

O ID da credencial utilizado na pipeline é `app`.

### Preparação necessária para uma nova instalação

O Vagrant instala os programas, mas não configura a credencial `app`, a autorização SSH nem o diretório de destino automaticamente.

1. Crie um par de chaves para esse laboratório e autorize a chave pública em `/home/vagrant/.ssh/authorized_keys` na VM Prod.
2. No Jenkins, cadastre a chave privada como **SSH Username with private key**, com usuário `vagrant` e ID `app`.
3. Na Prod, crie `/home/vagrant/app-prod/` com propriedade do usuário `vagrant`.
4. Para o usuário que executa o agente Jenkins, registre a chave do host Prod em `known_hosts`, conferindo sua impressão digital na própria VM.

O teste SSH precisa usar a mesma identidade e credencial da pipeline. Uma conexão feita pelo usuário `vagrant` na VM Jenkins não comprova que o serviço Jenkins está configurado.

### Validação da conexão

A comunicação foi testada executando o comando `hostname` remotamente:

```bash
ssh vagrant@192.168.56.20 hostname
```

Resultado obtido:

```text
prod
```

Esse teste também foi executado dentro da pipeline utilizando o plugin SSH Agent, confirmando que o Jenkins conseguia se autenticar e executar comandos na VM de produção.

## 6. Pipeline CI/CD com Jenkins

A pipeline foi implementada em um arquivo `Jenkinsfile` na raiz do repositório.

O Jenkins foi configurado utilizando a opção `Pipeline script from SCM`, permitindo buscar o código diretamente do GitHub.

A execução é organizada em quatro stages.

### Install

Instala as dependências da aplicação Node.js.

```bash
cd app && npm install
```

### Build

Executa o script de build definido no `package.json`.

```bash
cd app && npm run build
```

Nesta versão do projeto, o build utiliza um comando `echo` para simular essa etapa.

### Test

Executa os testes automatizados utilizando Jest.

```bash
cd app && npm test
```

Durante a validação da pipeline, os três testes automatizados foram executados com sucesso.

### Deploy

Após a conclusão das etapas anteriores, o Jenkins utiliza a credencial SSH configurada para transferir os arquivos da aplicação para a VM Prod.

```bash
scp -r app/* vagrant@192.168.56.20:/home/vagrant/app-prod/
```

O comando utiliza SCP para realizar a transferência segura dos arquivos.

Nesta implementação, o deploy automatiza a transferência da aplicação. A instalação das dependências e a inicialização do servidor na Prod ainda são realizadas manualmente.

## 7. Execução da aplicação na VM Prod

Após o deploy, os arquivos ficam disponíveis no diretório:

```text
/home/vagrant/app-prod/
```

Para acessar o diretório e instalar as dependências:

```bash
cd /home/vagrant/app-prod
npm ci
```

Para iniciar a aplicação:

```bash
npm start
```

O comando executa o arquivo `server.js`, conforme definido no `package.json`.

Com o servidor em execução, a API pode ser acessada pelo endereço:

http://192.168.56.20:3000

Resposta obtida:

```json
{
  "mensagem": "API funcionando com Jenkins"
}
```

A resposta confirmou que a aplicação foi transferida e estava funcionando na VM Prod.

## 8. Desafios técnicos e soluções implementadas

Durante a validação do deploy, identificamos uma incompatibilidade entre a versão do Node.js instalada na VM de produção e as dependências da aplicação.

A pipeline concluía as etapas de instalação, build, testes e transferência dos arquivos, mas a aplicação apresentava um erro ao iniciar na Prod.

Após verificar os logs e as versões instaladas, identificamos que a VM utilizava o Node.js 12.

Atualizamos o script de provisionamento para utilizar o Node.js 22. Durante a atualização, também foi necessário corrigir um conflito com pacotes antigos do Node.js.

Depois da correção, reinstalamos as dependências utilizando `npm ci` e executamos novamente a aplicação.

Com essas alterações, o servidor iniciou corretamente e a API foi validada pelo navegador.

## 9. Resultados e validações

Ao final da implementação, foram validados:

* Provisionamento das duas máquinas virtuais com Vagrant.
* Configuração da autenticação SSH entre Jenkins e Prod.
* Execução automatizada dos stages Install, Build e Test.
* Execução dos três testes automatizados da aplicação.
* Autenticação SSH utilizando a credencial do Jenkins.
* Transferência dos arquivos para a VM Prod utilizando SCP.
* Instalação das dependências no ambiente de produção.
* Inicialização da aplicação Node.js.
* Acesso à API pelo navegador na porta 3000.

O projeto permitiu colocar em prática conceitos de infraestrutura como código, integração contínua, automação de pipelines e deploy remoto.

## 10. Melhorias futuras

Como evolução do projeto, podemos automatizar as etapas que ainda são realizadas manualmente na VM Prod.

Entre as melhorias previstas estão a instalação das dependências durante o deploy, a inicialização automática da aplicação e a configuração de um serviço para mantê-la em execução após o encerramento da sessão SSH.

Também podemos implementar verificações de saúde da aplicação após o deploy e aprimorar o tratamento de falhas da pipeline.

Essas melhorias permitirão evoluir o projeto para um fluxo de entrega contínua mais completo.

## 11. Autoria e origem

André Chagas Assis Costa e Pedro Delmiro — FAP DevOps, Turma 5.

Projeto desenvolvido a partir de [carlhenriquex/projeto-vagrant-jenkins](https://github.com/carlhenriquex/projeto-vagrant-jenkins).
