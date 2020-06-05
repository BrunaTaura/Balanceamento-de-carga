# Balanceamento-de-carga
Aprenda o passo a passo para configurar um balanceador de carga. Uma eficiente maneira para distribuir o tráfego de dados na rede entre os recursos disponíveis. Garantindo desempenho uniforme e reduzindo o risco de sobrecarga em períodos de alta demanda.

## Pré-requisitos:
Antes de iniciar o projeto, tenha as seguintes ferramentas instaladas.

* Virtual Box
* Vagrant

## Criação das máquinas virtuais
Para exemplo desse projeto, serão instaladas 1 VM Linux Ubuntu e 2 VM's Windows Server
	
### 1º passo:
Acesse o site https://www.vagrantup.com/ > find boxes

### 2º passo:
Após escolher a box desejada, entre em um diretório de sua preferência pelo prompt e digite os comandos:

      vagrant box add nomeDaSuaBox      
      vagrant init nomeDaSuaBox			
      vagrant up	

## Configurando servidor web

Essa configuração deve ser realizada nas 2 VM's Windows Server

### 1º passo:
Instale o IIS

Menu iniciar > Gerenciador do servidor > Adicionar funções e recursos

Clique em próximo até chegar na aba de funções do servidor, marque a opção Servidor Web(IIS) > Adicionar recursos, continue e termine a instalação

### 2º passo:
Para testar se a instalação foi bem sucedida, abra o http://localhost ou 127.0.0.1 em um navegador, deverá aparecer o site inicial do IIS
	
### 3º passo:
Altere o arquivo htm do IIS de uma das VM windows para que seja possível diferenciar qual delas será carregada durante o balaceamento
 
 
## Configurando balanceador de carga
	
### 1º passo:
Altere o ip padrão de suas VM's Windows. Abra o arquivo VagrantFile, que encontra-se no diretório onde as máquinas foram criadas e logo abaixo da linha ( # using a specific IP ) adicione o seguinte script:
      
      config.vm.network "public_network", ip: "SeuPublicIp"
      config.vm.network "private_network", ip: "SeuPrivateIp"

### 2º passo:
Em ambas VM's Windows inclua uma nova regra para liberação da porta 80 no firewall

### 3º passo:
Instale o Nginx em sua máquina Linux, atráves do comando

      sudo apt-get install nginx

### 4º passo:
Utilize seguinte comando para habilitar e iniciar Nginx. Para testar se tudo ocorreu bem abra o http://localhost ou 127.0.0.1 no navegador da máquina Linux, deverá aparecer a página de bem-vindo do nginx

      systemctl enable nginx && systemctl start nginx

### 5º passo:
Para configurar o balanceamento adicione o seguinte script no arquivo nginx.conf, atente-se para adicionar essa configuração dentro do escopo do http já existente no arquivo, o arquivo de configuração nginx.conf encontra-se no caminho /etc/nginx

      server{
      listen 80;
      location / {
      proxy_pass http://app;
      }
      }

      upstream app{
      server SeuPrivateIpVM1 weight = 7;
      server SeuPrivateIpVM2;
      }


>Note que a VM1 tem um peso 7, o que fará com que ela receba mais carga em relação a VM2. Essa divisão do balanceamento é opcional

Após salvar a configuração utilize o comando 

    systemctl restart nginx

Caso seja necessário reinicie sua máquina Linux e execute novamente o comando para iniciar o nginx

### 6º passo:
Para testar sua configuração, abra o navegador e insira o http://localhost ou 127.0.0.1 ao recarregar a página por algumas vezes, deverá aparecer o layout da VM1 e VM2 alternadamente. Para verificar cada uma separadamente basta inserir o SeuPrivateIpVM em qualquer navegador


