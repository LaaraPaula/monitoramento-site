# Monitoramento de Site 🚀

## Sumário
- [📖 Introdução](#-introdução)
- [📌 Funcionalidades](#-funcionalidades)
- [🔧 Tecnologias Utilizadas](#-tecnologias-utilizadas)
- [⚙️ Configurando a instância](#️-configurando-a-instância)
- [🌐 Conexão e instalação dos pacotes necessários](#-conexão-e-instalação-dos-pacotes-necessários)
- [🤖 Criando o Bot Token do Telegram](#-criando-o-bot-token-do-telegram)
- [🛠️ Criando Script e seus requisitos](#️-criando-script-e-seus-requisitos)
- [🚧 Configuração Cronjob](#-configuração-cronjob)
- [📜 Conclusão](#-conclusão)

## 📖 Introdução
Este projeto tem como objetivo monitorar a disponibilidade de um site hospedado em uma instância EC2 da Aws e registrar possíveis instabilidades.
Envia notificações via Telegram, e se necessário reinicializa o servidor Nginx.

## 📌 Funcionalidades
- ✅ Monitoramento contínuo do site
- 📊 Registro de data das verificações
- ⚠️ Notificações em caso de falhas
- 🔄 Reiniciar o servidor automaticamente em caso de erro no Nginx.


## 🔧 Tecnologias Utilizadas

- **Linguagem:** Bash, HTML, CSS e Linux
- **Ferramentas:** GitHub, Nginx, WSL

## ⚙️ Configurando a instância:
1. Crie uma VPC com 2 sub-redes públicas e 2 sub-redes privadas com um Gateway padrão:
    ![VPC](imagens/criarVPC.png)

2. Crie uma intância EC2 com a AMI Ubuntu:
    ![AMI](imagens/EC2-1.png)

3. Crie um par de chaves para poder se conectar a intância, do tipo .pem(caso esteja usando distribuições Linux):
    ![Chave](imagens/EC2-2.png)

4. Certifique-se sobre a conexão da vpc criada, e se está em uma sub-rede pública, e com o IP público atribuído automaticamente
    ![Configuração-de-Rede](imagens/configRede.png)

5. Crie um grupo de segurança com as seguintes regras:
    ![Grupo-de-Segurança](imagens/EC2-3.png)

    Com a conexão SSH liberado somente para a sua máquina local e o HTTP liberado para ser acessado de qualquer IP.

6. Execute a intância e copie o IP público dela
    ![IP-instancia](imagens/ip.png)


## 🌐 Conexão e instalação dos pacotes necessários

1. Mova sua a chave criada, que se encontra na em C:/Users/seu_Usuario/Downloads, para o seu terminal Linux, neste caso estou usando o Ubuntu através da WSL(Subsistema do Windows para Linux):

    ```bash
    mv /mnt/c/Users/<seu_Usuario>/Downloads/chave.pem /home/<seu_usuario_ubuntu>/projeto
    ```

2. Altere as permições da chave:
    ```bash
    chmod 400 /home/<seu_usuario_ubuntu>/projeto/chave.pem
    ```

3. Conecte a instância através do comando:
    ```bash
    ssh -i /home/<seu_usuario_ubuntu>/projeto/chave.pem ubuntu@IP_DA_INSTANCIA
    ``` 

    > obs: Passe o caminho de onde está a sua chave, e coloque o ip da sua instância

    ![Conexão-EC2](imagens/conexãoEC2.png)

4. Agora já conectado na EC2, atulize os pacotes existentes, baixe os pacotes do Nginx e inicialize o serviço
    ![Instalar-Nginx](imagens/instalaNginx.png)

    ![Iniciar-Nginx](imagens/iniciarNginx.png)

5. No seu navegador, ao acessar `http://IP_DA_INSTANCIA` a página padrão do servidor Nginx deverá estar em exibição.
    ![Pagina-Nginx](imagens/paginaNginx.png)

## 🤖 Criando o Bot Token do Telegram
1. No Telegram, na barra de pesquisa, buscar por @BotFather

2. Configurando o Bot:
    1. Envie o comando:
    ```bash
    /newbot
    ```

    2. Preencha um nome para o usuário bot, ele deve terminar com "bot" (Ex: Teste_bot)
    
    3. Se o nome estiver disponível ele vai responder com o seu TOKEN_BOT, guarde bem ele.

    4. Mande uma mensagem para seu Bot criado, pode ser um "OI" e no seu navegador busque por:
    ```bash
    https://api.telegram.org/botSEU_TOKEN_AQUI/getUpdates
    ```  

    5. Ele retornará um arquivo JSON, e o `"chat": { "id": ... }` é o seu Chat ID, guarde bem ele.


## 🛠️ Criando Scrip e seus requisitos

1. Crie um diretório na pasta home para guardar seu script 
    ```bash
    mkdir ~/script
    ```

2. Entre na pasta e crie um arquivo de monitoramento do tipo shell
    ```bash
    sudo nano ~/script/monitoramento.sh
    ```

3. Crie uma pasta e o arquivo que irá armazenar seus logs e dê permissão de escrever no arquivo
    ```bash
    sudo mkdir /var/log/meus_logs

    sudo chown ubuntu:ubuntu /var/log/meus_logs

    touch /var/log/meus_logs/monitoramento.log
    ```

4. Coloque seu site no servidor movendo o arquivo .html para a pasta padrão do Nginx 
    ```bash
    sudo mv /home/ubuntu/index.html /var/www/html/
    ```

5. Para verificar se o seu site foi hospedado corretamente, em seu navegador digite `http://IP_DA_INSTANCIA`, ele deve exibir o site que vc criou.

6. Escreva o seguinte script:

    ![Script-pt1](imagens/script1.png)

    1. 🔴 Indica o interpretador que irá usar para executar o script
    2. 🔵 Valida se foi passado o ip como parâmetro para execução do script, se sim, armagena na sua respectiva variável
    3. 🟢 Variável que armazena o caminho do arquivo onde irá armazenar as mensagens de monitoramento(log)
    4. 🟡 Variáveis de conexão com o telegram, que é o Token_BOT  e o id do chat desse bot
    5. 🟠 Variável, que após buscar a requisição do site, armazena o status de retorno
    6. 🟣 Variável que captura a data da requisição ao site e formata no padrão (ano/mês/dia)

    ![Script-pt2](imagens/script2.png)

    Usando 'case' para validar o status do site, armagenando mensagens especificas de acordo com os status: 000, 403, 404, 500, 502, 503, 200.

    ![Sprint-pt3](imagens/sprint3.png)
    1. 🔴 Uso o comando 'echo' para guardar a data com a mensagem no log que definimos na variável LOG
    2. 🟢 Manda uma notificação via telegram com a mensagem definida
    3. 🔵 A variável 'REINICIAR' armazena se o erro tem haver com uma queda do servidor, e caso seja uma queda no servidor ele vai tentar reiniciar o serviço
    4. 🟡 Valida se o serviço está ativo novamente e notifica via telegram que o serviço está ou não rodando
    `Salve seu script com Ctrl+o, Enter, Ctrl+x`


## 🚧 Configuração Cronjob

1. Editar o arquivo padrão de execução do cronjob e incluir script para ser executado a cada minuto
    ```bash
    crontab -e
    ```

    `Selecione a opção 1 para o usar o editor de texto nano`
    No final do arquivo inclua a linha `* * * * * /home/ubuntu/monitoramento.sh <ip da instância>`
    `Salve seu script com Ctrl+o, Enter, Ctrl+x`

2. Restartando o cron e habilitando inicialização junto com o sistema
    ```bash
    sudo systemctl restart cron
    sudo systemctl enable cron
    ```

## 📜 Conclusão
Este projeto implementa um servidor web na AWS com Linux e Nginx, focando em alta disponibilidade e monitoramento eficiente. Ele inclui automação para manter o serviço ativo e notificações via Telegram para alertas em caso de falhas. Além disso, utiliza SystemD e logs personalizados para uma administração eficaz.
Agora você tem um scrip de monitoramento de site, prático e rápido para alterar, melhorar e fazer todos os testes e validações que quiser. Use sem moderação!


