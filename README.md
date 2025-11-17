# Simulando_ataque_de_BruteForce_com_Kali_e_Medusa
O projeto em questão tem o objetivo de mostrar simulações de ataque de força bruta em ambientes vulneráveis como o Metasploitable 2 e DVWA, assim como em outros cenários tendo como exemplos, ataques em FTP e SMB, utilizando Kali Linux e Medusa.

# Configurando o ambiente de simulação: VM Kali Linux e Metasploitable 2
O primeiro objetivo é configurar as duas VMs instaladas no seu virtual Box (Kali e Metasploitable 2) para a placa de rede exclusiva de hospedeiro *host only*.

E qual a finalidade dessa configuração *host only*?

A primeira finalidade é a COMUNICAÇÂO: para que ambas (Kali e Metas) possam se encontrar na mesma rede, ou no mesmo sub-bloco de rede. Essa configuração é importante, pois permite que o Kali Linux envie e receba pacotes de IP direntamente do Metasploitable 2, fazendo com que o ping, o nmap e o ataque medusa possam funcionar. Sem essa comunicação direta, o ataque via Kali Linux não seria possível. 

A segunda finalidade é o ISOLAMENTO e SEGURANÇA: como o nosso foco são os testes de penetração, essa configuração permite o isolamento das VMs de sua rede doméstica ou corporativa, criando um ambiente seguro e controlado, evitando que dados sejam vazados ou que dispositovos sejam danificados na tentativa de exploração das vulnerabilidades intencionalmente existentes no Metasploitable 2.

PASSO A PASSO: após a instalação do virtual box e posteriormente do Kali Linux e Metasploitable 2, clique em uma das duas VMs instaladas, clique em configurações, depois va na aba rede e escolha a opção placa de rede exclusiva de hospedeiro  *host only*, em seguida configure a outra máquina da mesma forma. Pronto, seu ambiente se encontra pronto para a simulaçaão do ataque de força bruta.

# 1. Simulando ataque de Brute Force via FTP usando o Kali Linux e Metasploitable 2

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


# 2. Simulando ataque de Brute Force usando o Kali e o Medusa via formulário web (DVWA).


💻 1. Preparação e Análise da Requisição (Imagem 1)
A primeira imagem () mostra a fase inicial de reconhecimento e análise do formulário de login do DVWA.

Alvo: O alvo é a aplicação web de teste de vulnerabilidades DVWA, acessada no endereço IP local 192.168.56.101 na página login.php.

Tentativa Manual e Análise: É feita uma tentativa de login manual (username: "geo", password: "123456") que falha ("Login failed").

Captura de Dados: O painel de Ferramentas do Desenvolvedor do navegador (aba Network / Rede) é crucial. Ele mostra a requisição POST enviada para login.php com o status 302 (Redirecionamento, pois a tentativa de login falhou).

Identificação do Payload: No painel Request (Requisição) à direita, o Form Data (Dados do Formulário) revela exatamente como o servidor espera receber as credenciais: username=geo, password=123456, e o botão Login=Login. Essa estrutura é essencial para configurar a ferramenta de ataque.


<img width="1920" height="923" alt="VirtualBox_Kali_05_11_2025_19_11_20testantoataqueloginesenhaSOweb" src="https://github.com/user-attachments/assets/d6aba9ce-0d41-47e8-81e8-90df70f92720" />


2. Execução do Ataque de Força Bruta (Imagem 2)
A segunda imagem () mostra o uso da ferramenta Medusa para executar o ataque de força bruta contra o formulário de login, aproveitando a informação coletada na etapa anterior.

Ferramenta: O ataque foi realizado com o Medusa, um scanner de força bruta rápido e modular.

Comando: O comando utilizado direciona o Medusa para:

-h 192.168.56.101: O host alvo.

-u users.txt: Um arquivo contendo uma lista de nomes de usuário para tentar com o comando echo -e "user\nmsfadmin\nadmin\nroot" > users.txt

-P pass.txt: Um arquivo contendo uma lista de senhas a serem testadas para cada usuário com o comando echo -e "123456\npassword\qwerty\nmsfadmin" > pass.txt

-M ftp

-M http: Especifica o módulo de protocolo HTTP.

-M PAGE '/dvwa/login.php: O caminho para a página de login.

-M 'FORM: username=^USER^&password=^PASS^&Login=Login": O payload exato da requisição POST (o formato FORM foi visto na Imagem 1).

-M 'FAIL=Login failed": Especifica uma string de resposta do servidor que indica uma falha de login, permitindo ao Medusa identificar quando essa string está ausente (o que significa sucesso).

-t 6 (de Threads) define o número de threads (ou conexões) paralelas que o Medusa deve usar para tentar as combinações de login. Um valor de 6 significa que 6 tentativas de login serão realizadas simultaneamente, acelerando o processo.


Resultados de Sucesso (SUCCESS): O Medusa testa as combinações e relata vários logins bem-sucedidos (ACCOUNT FOUND com [SUCCESS]), incluindo:

Usuário admin com a senha password.

Usuário admin com a senha 123456.

Usuário msfadmin com a senha 123456.

Usuário user com a senha 123456.

...e outras combinações.


<img width="1920" height="923" alt="VirtualBox_Kali_06_11_2025_18_49_06ataqueformularioweb" src="https://github.com/user-attachments/assets/c4cce559-6e23-4c85-95d7-bfaa4f5287f2" />



<img width="1920" height="923" alt="VirtualBox_Kali_06_11_2025_19_03_51segundalistadesucessosataqueweb" src="https://github.com/user-attachments/assets/809c7ca2-6461-4fd0-82c3-2ba7d613ad45" />


✅ 3. Confirmação do Acesso (Imagem 3)
A terceira imagem () valida o sucesso do ataque usando uma das credenciais encontradas.

Acesso Confirmado: O atacante usa o par admin / password descoberto pelo Medusa e obtém acesso ao dashboard principal do DVWA, index.php.

URL: A URL muda de login.php para index.php, confirmando que a sessão foi estabelecida.

Prova da Exploração: O painel Ferramentas do Desenvolvedor (aba Headers / Cabeçalhos) agora mostra os dados de login enviados (username: "admin", password: "password") que resultaram em um acesso bem-sucedido.

Conclusão: O ambiente de teste DVWA permitiu o ataque de força bruta, validando que, com as configurações de segurança padrão (ou de baixa segurança), um formulário de login pode ser facilmente comprometido por meio de listas de senhas e usernames comuns.


<img width="1920" height="923" alt="VirtualBox_Kali_06_11_2025_19_04_41conseguindologarweb" src="https://github.com/user-attachments/assets/869ae031-dde1-453e-ad75-b4704c31ae30" />


# Simulando ataque em cadeia, enumeração SMB + password spraying #


🛡️ Análise de Enumeração SMB para PentestO primeiro código que você executou é um comando de enumeração usando a ferramenta enum4linux contra o endereço IP 192.168.56.101, com o objetivo de coletar informações sobre o serviço SMB (Server Message Block). O SMB é um protocolo de rede usado para compartilhar arquivos, impressoras e comunicações entre processos.

💻 O Comando e Seu Significado: O comando que você executou é: enum4linux -a 192.168.56.101 | tee enum_output.txt

enum4linux: É uma ferramenta de enumeração para sistemas Windows/Samba no Linux, geralmente usada em pentesting para extrair informações sobre usuários, grupos, compartilhamentos, e políticas de segurança de um alvo que executa o SMB.

-a: É o argumento para realizar uma enumeração "all" (completa), que tenta executar todas as verificações disponíveis na ferramenta, incluindo NetBIOS, usuários, grupos, compartilhamentos, SID (Security Identifier), etc.

192.168.56.101: É o endereço IP do alvo da enumeração (a máquina servidora SMB).
| tee enum_output.txt: Este é um mecanismo de redirecionamento do shell:

| (Pipe): Redireciona a saída padrão do comando enum4linux (o que aparece no terminal) para o próximo comando.

tee: Este comando faz duas coisas simultaneamente:Exibe a saída na tela (no terminal). Salva a mesma saída no arquivo enum_output.txt para documentação. Objetivo: Obter o máximo de informações possível sobre o serviço SMB no IP 192.168.56.101 e registrar todo o processo e resultado no arquivo enum_output.txt.

🎯 Retorno da Lista de Possíveis Alvos SMB: O resultado da execução do enum4linux (visível nas duas imagens, especialmente a segunda, que mostra o conteúdo do enum_output.txt) fornece uma lista robusta de potenciais alvos de ataque, essenciais para a documentação de pentest. 

**1. Informações de Identificação Geral (Imagem 1):**

   **Workgroup/Domain Name:**
   
   WORKGROUP. Isso indica que o alvo faz parte de um grupo de trabalho, não de um domínio Active Directory, embora o enum4linux tente buscar SIDs de domínio.
   
   **Netstat Information (NBTSTAT):**
   
 METASPLOITABLE <00> - B <ACTIVE> Workstation Service: Confirma que o alvo é provavelmente uma máquina Metasploitable (frequentemente usada para testes), com o serviço de estação de trabalho ativo.
 
 METASPLOITABLE <20> - B <ACTIVE> File Server Service: Confirma a existência de um serviço de Servidor de Arquivos (File Server) ativo (via porta TCP 445/139), um vetor de ataque direto via SMB.
 
 WORKGROUP <1D> - B <ACTIVE> Master Browser: Indica que o host está atuando como o "navegador mestre" na rede, responsável por manter a lista de computadores disponíveis.
 
 Session Check: [+] Server 192.168.56.101 allows sessions using username "", password "". Essa é uma descoberta crítica! Indica que é possível acessar o servidor SMB anonimamente (sessão nula) sem credenciais, o  que é uma falha de configuração grave e que permite a enumeração de recursos como usuários, grupos e compartilhamentos (que é o que o enum4linux faz).

**3. Lista de Usuários e Grupos Enumerados (Imagem 2):**

   A parte mais importante para a lista de alvos são as contas enumeradas. A listagem (exibida na segunda imagem) contém vários SIDs (Security Identifiers), nomes de contas (usuários e grupos) e seus tipos.


   <img width="1920" height="923" alt="VirtualBox_Kali_16_11_2025_21_11_16listandotecnicassmb" src="https://github.com/user-attachments/assets/3f377740-dce1-4f64-94ff-aa0160ed1d8e" />


   <img width="1920" height="923" alt="VirtualBox_Kali_16_11_2025_21_12_04acessandoarquivoenum4" src="https://github.com/user-attachments/assets/b0f95b1f-0e5f-4af0-b7db-8aae1ae6c7ff" />

   
**O código exibido na imagem é uma sequência de comandos executados em um terminal Linux (Kali Linux), provavelmente para realizar um ataque de força bruta contra um serviço SMB (Server Message Block) em uma rede.**

💻 **Comandos e Explicação:**

Os comandos mostram o uso da ferramenta Medusa para tentar adivinhar nomes de usuário e senhas válidas em um serviço SMB.

**1. Preparação dos Arquivos**

echo -e "user\nmsfadmin\nservice" > smb.users.txt

Função: Cria um arquivo chamado smb.users.txt que contém uma lista de nomes de usuário a serem testados.

Conteúdo: A lista contém user e nmsfadmin e nservice. O uso de -e e \n garante que cada nome de usuário esteja em uma nova linha.

echo -e "P@$$w0rd\nwelcome123\nmsfadmin" > smb.pass.txt

Função: Cria um arquivo chamado smb.pass.txt que contém uma lista de senhas a serem testadas.

Conteúdo: A lista contém P@$$w0rd, welcome123 e msfadmin.

2. Ataque de Força Bruta com Medusa
medusa -H 192.168.56.101 -U smb.users.txt -P smb.pass.txt -e nsrht -f -Z -T 50

Ferramenta: Medusa, um brute-force password cracker (quebrador de senhas por força bruta) modular, rápido e agressivo.

Opções:

-H 192.168.56.101: Define o host alvo (endereço IP).

-U smb.users.txt: Especifica o arquivo de nomes de usuário a serem testados.

-P smb.pass.txt: Especifica o arquivo de senhas a serem testadas.

-e nsrht: Define opções de verificação adicionais (por exemplo, n para no password, s para same username as password, etc.).

-f: Interrompe (para) a verificação do host alvo após encontrar uma combinação válida (sucesso).

-Z: Define o módulo de ataque. Neste caso, está implícito que é para o serviço SMB (o protocolo é deduzido pela saída, mas o módulo específico seria -M smb).

-T 50: Define o número de threads (conexões paralelas) a serem usadas (aumenta a velocidade do ataque).

3. Resultados do Ataque (Log do Medusa)
O output do Medusa mostra os resultados das tentativas:

Falhas:

ACCOUNT CHECK: [SMBNT] Host: 192.168.56.101 (1 of 1, 0 complete) user: user (1 of 3, 0 complete) Password: P@$$w0rd (1 of 3 complete)

...e outras tentativas que resultaram em FAILURE (falha) ou ACCOUNT CHECK (verificação de conta, indicando falha na senha).

Sucesso:

ACCOUNT CHECK: [SMBNT] Host: 192.168.56.101 (1 of 1, 0 complete) user: msfadmin (2 of 3, 1 complete) Password: msfadmin (3 of 3 complete)

Esta linha indica que a combinação de usuário: msfadmin e senha: msfadmin foi válida (SUCCESS - ACCESS ALLOWED).

🔑 Pós-Ataque e Verificação
Após o sucesso, a imagem mostra duas tentativas de login usando a ferramenta smbclient, que é o utilitário de cliente SMB no Linux, usado para acessar compartilhamentos de rede:

smbclient //192.168.56.101/ -U msfadmin

Tentativa: Conectar-se ao host sem especificar um compartilhamento (/), solicitando o nome de usuário msfadmin.

Resultado: session setup failed: NT_STATUS_LOGON_FAILURE. O login falhou, provavelmente porque a senha não foi fornecida corretamente na linha de comando ou por algum erro de sintaxe/ambiente.

smbclient //192.168.56.101/msfadmin -U WORKGROUP/msfadmin

Tentativa: Conectar-se a um compartilhamento chamado msfadmin no host, usando o usuário msfadmin e especificando o WORKGROUP (WORKGROUP/msfadmin).

Resultado: Login bem-sucedido! A senha (implícita pela tentativa anterior de sucesso do Medusa) foi aceita.

A saída final lista os recursos compartilhados (shares) disponíveis, como print$, smbtest, IPC$, ADMIN$, e o compartilhamento msfadmin.

📝 Conclusão para Documentação
Esta sequência demonstra o processo de teste de penetração ou hacking ético para identificar credenciais fracas e enumerar recursos compartilhados em um servidor SMB. O sucesso é alcançado através de um ataque de força bruta usando o Medusa, que descobre a credencial msfadmin:msfadmin, e a subsequente verificação de acesso usando o smbclient.
