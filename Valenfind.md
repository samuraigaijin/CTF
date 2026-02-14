# Write-up: ValenFind (TryHackMe)

## 1. Introdução
Este documento detalha o processo de exploração da máquina **ValenFind** no TryHackMe. O desafio simula uma aplicação de encontros (Secure Dating) que contém falhas críticas de segurança, permitindo a transição de um usuário comum para o vazamento total de dados sensíveis.

## 2. Reconhecimento (Recon)
A fase inicial consistiu no mapeamento de portas e serviços utilizando ferramentas de varredura.

* **Portas Abertas**: 
    * `22/tcp`: OpenSSH 9.6p1.
    * `5000/tcp`: Werkzeug 3.0.1 (Python 3.12.3).
* **Enumeração de Diretórios**: O uso do **Gobuster** revelou rotas importantes da aplicação Flask:
    * `/dashboard`
    * `/login`
    * `/register`

## 3. Exploração de Vulnerabilidade (LFI)
Ao analisar o código-fonte da página de perfil, foi identificado um comportamento inseguro na função de carregamento dinâmico de temas.

* **Falha**: O parâmetro `layout` na rota `/api/fetch_layout` era concatenado diretamente ao caminho do sistema de arquivos sem sanitização.
* **Vetor de Ataque**: **Local File Inclusion (LFI)** através de *Directory Traversal*.
* **Prova de Conceito (PoC)**: 
    * Leitura do arquivo `/etc/passwd` para identificação de usuários do sistema (usuário `ubuntu` localizado).
    * Exposição do código-fonte da aplicação (`app.py`) através de erros detalhados que revelaram o caminho absoluto `/opt/Valenfind/`.
    * Descoberta de uma chave de API estática no código: `ADMIN_API_KEY = "<chave>"`.

## 4. Exfiltração de Dados
Com a chave de administrador exposta, foi possível interagir com uma rota administrativa de exportação do banco de dados.

* **Metodologia**: Uso de `curl` com injeção de cabeçalho personalizado (`X-Valentine-Token`).
* **Comando**: 
  ```bash
  curl -H "X-Valentine-Token: <chave>" http://<IP>:5000/api/admin/export_db --output backup.db


Análise do Banco de Dados: O banco de dados SQLite revelou o armazenamento de senhas em texto puro (cleartext), resultando na captura da flag administrativa no registro do usuário cupid.

## 5. Conclusão e Mitigação
O desafio demonstrou como falhas de configuração e desenvolvimento — como o vazamento de caminhos em logs de erro e o uso de chaves estáticas — podem comprometer a infraestrutura.

## Recomendações de Segurança:

Implementação de Allowlist: Restringir o carregamento de arquivos a uma lista pré-definida de nomes permitidos.

Sanitização de Entradas: Bloquear caracteres de navegação de diretório (../).

Gestão de Segredos: Utilizar variáveis de ambiente para chaves de API em vez de hardcoding no código-fonte.

Hashing de Senhas: Nunca armazenar credenciais em texto puro; utilizar algoritmos como Argon2 ou BCrypt.

Escrito por Bruno Braga - Analista de Segurança.
