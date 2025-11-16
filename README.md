# Simulando_ataque_de_BruteForce_com_Kali_e_Medusa
O projeto em questão tem o objetivo de mostrar simulações de ataque de força bruta em ambientes vulneráveis como o Metasploitable 2 e DVWA, assim como em outros cenários tendo como exemplos, ataques em FTP e SMB, utilizando Kali Linux e Medusa.

# Configurando o ambiente de simulação: VM Kali Linux e Metasploitable 2
O primeiro objetivo é configurar as duas VMs instaladas no seu virtual Box (Kali e Metasploitable 2) para a (placa de rede exclusiva de hospedeiro -host only-).

E qual a finalidade dessa configuração -host only-?

A primeira finalidade é a COMUNICAÇÂO: para que ambas (Kali e Metas) possam se encontrar na mesma rede, ou no mesmo sub-bloco de rede. Essa configuração é importante, pois permite que o Kali Linux envie e receba pacotes de IP direntamente do Metasploitable 2, fazendo com que o ping, o nmap e o ataque medusa possam funcionar. Sem essa comunicação direta, o ataque via Kali Linux não seria possível. 

A segunda finalidade é o ISOLAMENTO e SEGURANÇA: como o nosso foco são os testes de penetração, essa configuração permite o isolamento das VMs de sua rede doméstica ou corporativa, criando um ambiente seguro e controlado, evitando que dados sejam vazados ou que dispositovos sejam danificados na tentativa de exploração das vulnerabilidades intencionalmente existentes no Metasploitable 2.
