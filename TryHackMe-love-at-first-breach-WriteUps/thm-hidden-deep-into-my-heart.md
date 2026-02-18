# TryHackMe Write-up: Hidden Deep Into my Heart

## 📝 Descrição
Este repositório contém o write-up do desafio **"Hidden Deep Into my Heart"** da plataforma TryHackMe. O foco deste CTF é a exploração web através de enumeração detalhada, análise de arquivos sensíveis e descoberta de diretórios ocultos por meio de buscas recursivas.

## 🛠️ Ferramentas Utilizadas
* **Gobuster**: Enumeração de diretórios e arquivos.
* **Navegador (Inspect Element)**: Análise de código-fonte e tráfego de rede.

## 🚀 Metodologia

### 1. Reconhecimento Inicial
A fase de reconhecimento começou com o mapeamento da estrutura de diretórios do servidor web utilizando o **Gobuster**.
* **Comando:** `gobuster dir -u http://<TARGET_IP>:5000/ -w /usr/share/wordlists/dirb/common.txt`
* **Resultados:** Identificação dos caminhos `/console` (Status 400) e `/robots.txt` (Status 200).

### 2. Coleta de Informações (A Pista no Robots)
A análise do arquivo `robots.txt` foi fundamental, pois revelou duas informações críticas:
1. Um diretório oculto e "proibido": `/cupids_secret_vault/`.
2. Um comentário deixado no arquivo que sugeria uma credencial: `# cupid_arrow_2026!!!`.

### 3. Enumeração Recursiva (O Diferencial)
Ao acessar `/cupids_secret_vault/`, a página parecia ser apenas estática. A chave para avançar foi insistir na enumeração dentro desse diretório específico.
* **Comando:** `gobuster dir -u http://<TARGET_IP>:5000/cupids_secret_vault/ -w <WORDLIST> -x txt,php,html,py`
* **Descoberta:** O scanner localizou um ponto de entrada administrativo em `/cupids_secret_vault/administrator`.

### 4. Exploração e Autenticação
A página descoberta apresentava um formulário de login para o "Cupid's Vault". Utilizando a string encontrada anteriormente no `robots.txt`, o acesso foi obtido com as seguintes credenciais:
* **Usuário:** `admin`
* **Senha:** `<senhadorobots.txt>`

## 🚩 Conclusão
Após a autenticação bem-sucedida, o acesso ao painel "Welcome, Cupid!" foi liberado, revelando a flag do desafio. Este CTF reforça a importância de uma enumeração minuciosa e demonstra como informações sensíveis deixadas em arquivos públicos (como o `robots.txt`) podem comprometer toda a segurança da aplicação.

Escrito por Bruno Braga.
