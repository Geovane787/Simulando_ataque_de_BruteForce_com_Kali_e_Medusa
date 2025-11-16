# Simulando_ataque_de_BruteForce_com_Kali_e_Medusa
O projeto em questão tem o objetivo de mostrar simulações de ataque de força bruta em ambientes vulneráveis como o Metasploitable 2 e DVWA, assim como em outros cenários tendo como exemplos, ataques em FTP e SMB, utilizando Kali Linux e Medusa.

# Configurando o ambiente de simulação: VM Kali Linux e Metasploitable 2
O primeiro objetivo é configurar as duas VMs instaladas no seu virtual Box (Kali e Metasploitable 2) para a placa de rede exclusiva de hospedeiro *host only*.

E qual a finalidade dessa configuração *host only*?

A primeira finalidade é a COMUNICAÇÂO: para que ambas (Kali e Metas) possam se encontrar na mesma rede, ou no mesmo sub-bloco de rede. Essa configuração é importante, pois permite que o Kali Linux envie e receba pacotes de IP direntamente do Metasploitable 2, fazendo com que o ping, o nmap e o ataque medusa possam funcionar. Sem essa comunicação direta, o ataque via Kali Linux não seria possível. 

A segunda finalidade é o ISOLAMENTO e SEGURANÇA: como o nosso foco são os testes de penetração, essa configuração permite o isolamento das VMs de sua rede doméstica ou corporativa, criando um ambiente seguro e controlado, evitando que dados sejam vazados ou que dispositovos sejam danificados na tentativa de exploração das vulnerabilidades intencionalmente existentes no Metasploitable 2.

PASSO A PASSO: após a instalação do virtual box e posteriormente do Kali Linux e Metasploitable 2, clique em uma das duas VMs instaladas, clique em configurações, depois va na aba rede e escolha a opção placa de rede exclusiva de hospedeiro  *host only*, em seguida configure a outra máquina da mesma forma. Pronto, seu ambiente se encontra pronto para a simulaçaão do ataque de força bruta.

# Simulando ataque de Brute Force via FTP usando o Kali Linux e Metasploitable 2

O primeiro passo consiste em verificar qual é o endereço IP da máquina Metasploitable 2. Para isso deveremos abrir o Metasploitable 2 e usar os comandos ifconfig ou ip a, e em seguida todas as informações necessárias no que diz respeito ao IP da máquina será listada logo abaixo, conforme a imagem a seguir:

<img width="720" height="400" alt="VirtualBox_Ubuntu_04_11_2025_20_37_35ifconfig" src="https://github.com/user-attachments/assets/94f9e757-e530-4991-88a1-0b9f876a203c" />


1. 🔍 Primeiro Ping na Rede Metasploitable 2.
   
A primeira parte da primeira imagem mostra o resultado do comando ping na máquina Kali Linux para o endereço IP 192.168.56.101, que é o endereço da máquina Metasploitable 2.

Comando Executado:

Bash

ping 192.168.56.101
Resultado do Ping:

O comando enviou 3 pacotes (mostrado em "3 packets transmitted").

Recebeu 3 respostas ("3 received").

A estatística final é: "0% packet loss" (0% de perda de pacotes).

O tempo de ida e volta (time) para as respostas foi muito baixo (em milissegundos, ex: 13.7 ms, 6.27 ms).

Significado:

Este resultado confirma que há conectividade de rede básica (Camada 3 - IP) e funcionalidade ICMP (Protocolo de Mensagens de Controle da Internet) entre a máquina Kali (atacante) e a máquina Metasploitable 2 (alvo).

O ping bem-sucedido é o primeiro passo de reconhecimento, garantindo que o alvo está ativo e acessível antes de prosseguir com varreduras de porta (nmap) ou ataques.

2. 🛡️ Varredura de Portas (Nmap)
A primeira imagem também mostra uma varredura de portas com o Nmap na mesma máquina alvo:

Comando Executado:

Bash

nmap -sV -p 21,22,80,445,139 192.168.56.101
A opção -sV tenta determinar a versão do serviço em execução nas portas abertas.

A opção -p especifica as portas a serem varridas. A porta 21/tcp é a porta padrão para o protocolo FTP (File Transfer Protocol).

Resultado Relevante:

Porta: 21/tcp

Serviço: ftp

Versão: vsftpd 2.3.4

Isso confirma que o serviço FTP está ativo e vulnerável (vsftpd 2.3.4 é conhecido por vulnerabilidades, mas aqui o foco é o login).


<img width="1920" height="923" alt="VirtualBox_Kali_04_11_2025_20_52_06pingnokalimetas" src="https://github.com/user-attachments/assets/e4ecc225-56e6-4ec7-892a-4bb1c5d7332c" />


3. 💥 Ataque de Força Bruta ao FTP com Medusa
   
A segunda imagem documenta o processo e o resultado do ataque de força bruta contra o serviço FTP na porta 21/tcp do Metasploitable 2, usando a ferramenta Medusa a partir do Kali Linux.

A. Preparação do Ataque (Criação de Listas)
Antes de executar o Medusa, foram criados dois arquivos de texto: users.txt (lista de usuários) e pass.txt (lista de senhas) com nomes e senhas comuns, como pode ser visto pelos comandos echo:

Criação de Usuários:

Bash

echo -e "user\nmsfadmin\nadmin\nroot" > users.txt
(Este comando cria uma lista de usuários para testar.)

Criação de Senhas:

Bash

echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt
(Este comando cria uma lista de senhas para testar.)

B. Execução do Ataque com Medusa
O comando Medusa combina os usuários e senhas das listas para tentar fazer o login no servidor FTP.

Comando Executado:

Bash

medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6

-h 192.168.56.101: Especifica o alvo (host).

-U users.txt: Fornece o arquivo com a lista de usuários.

-P pass.txt: Fornece o arquivo com a lista de senhas.

-M ftp: Indica o módulo a ser usado, neste caso, o File Transfer Protocol (FTP).

-t (de Threads) define o número de threads (ou conexões) paralelas que o Medusa deve usar para tentar as combinações de login. Um valor de 6 significa que 6 tentativas de login serão realizadas simultaneamente, acelerando o processo.

C. Resultados e Sucesso do Ataque
O Medusa executa a combinação de credenciais até encontrar um par válido. Os resultados mostram várias tentativas (ACCOUNT CHECK) até que o Medusa encontra credenciais bem-sucedidas.

Credenciais Encontradas:

Usuário 1: msfadmin / msfadmin

Usuário 2: user / user

Usuário 3: root / root (e outras como password ou 123456)

Confirmação (Login Manual): O atacante, em seguida, usa o comando ftp 192.168.56.101 e insere o par de credenciais msfadmin/msfadmin, que foi encontrado pelo Medusa. O terminal exibe a mensagem: "230 Login successful."

Significado: O ataque de força bruta foi bem-sucedido. O Medusa identificou senhas fracas (padrões ou senhas que correspondiam ao nome de usuário) no servidor FTP do Metasploitable 2, concedendo ao atacante acesso ao sistema de arquivos do servidor através do protocolo FTP, o que permite transferência de arquivos e, dependendo da configuração, pode ser um ponto de entrada para escalonamento de privilégios ou mais reconhecimento.


<img width="1920" height="923" alt="VirtualBox_Kali_05_11_2025_07_06_08criandoataquebruteforce" src="https://github.com/user-attachments/assets/f1ec6ed1-8b6a-4719-92ef-cbbcd58b52f1" />
