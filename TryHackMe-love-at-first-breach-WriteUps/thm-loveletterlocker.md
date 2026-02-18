# 🔐 TryHackMe - LoveLetter Locker (IDOR)

## 📌 Informações do Desafio

- Plataforma: TryHackMe  
- Categoria: Web  
- Dificuldade: Easy  
- Tipo de vulnerabilidade explorada: **IDOR (Insecure Direct Object Reference)**  
- Classificação OWASP: A01 – Broken Access Control  

---

## 🎯 Objetivo

Obter acesso indevido a mensagens privadas armazenadas na aplicação web e encontrar a flag.

---

## 🔎 Reconhecimento Inicial

Após iniciar a máquina, foi realizada enumeração básica:

```bash
nmap -sC -sV -p- <TARGET_IP>
```

Portas abertas identificadas:

- 22/tcp – SSH
- 5000/tcp – Aplicação Web (Flask / Werkzeug)

Header HTTP identificado:

```
Server: Werkzeug/3.1.5 Python/3.12.3
```

A aplicação estava rodando em:

```
http://<TARGET_IP>:5000
```

---

## 🧭 Enumeração de Diretórios

Utilizando Gobuster:

```bash
gobuster dir -u http://<TARGET_IP>:5000 -w /usr/share/wordlists/dirb/common.txt -t 20
```

Endpoints encontrados:

- `/login`
- `/register`
- `/letters`
- `/logout`

---

## 👤 Criação de Usuário

Foi criada uma conta comum através da rota `/register`.

Após realizar login, a aplicação informava que existiam **duas mensagens já cadastradas**, porém não era possível visualizá-las pois pertenciam a outro usuário (provavelmente administrador).

---

## 🧠 Observação Crítica

Após criar uma nova mensagem própria, ao acessá-la foi observado o seguinte padrão na URL:

```
http://<TARGET_IP>:5000/letter/3
```

Isso indicava que o ID da mensagem estava sendo passado diretamente na URL.

Esse comportamento levantou a hipótese de:

🔥 Enumeração de ID (IDOR)

---

## 🚨 Exploração – IDOR

Foi realizado teste manual alterando o ID diretamente na URL:

```
/letter/2
/letter/1
```

A aplicação **não validava se a mensagem pertencia ao usuário autenticado**.

Ao acessar:

```
http://<TARGET_IP>:5000/letter/1
```

Foi possível visualizar o conteúdo da mensagem do administrador.

---

## 🏁 Flag Encontrada

Conteúdo da mensagem:

```
My dearest...

THM{flag omitida}

Forever yours,
Gonz0
```

Mensagem encontrada em `/letter/2`:

```
This city is a neon fever dream, but you\u2019re the only thing that feels real. My heart\u2019s doing 120 in a rented convertible with the check-engine light on, and somehow you\u2019re the steering wheel.
Love you. Terrifyingly. 💘
```

Flag:

```
THM{flag omitida}
```

---

## 🧩 Vulnerabilidade Identificada

### 🔓 IDOR – Insecure Direct Object Reference

A aplicação permitia acessar recursos sensíveis apenas alterando o ID numérico na URL.

Não havia validação de autorização no backend.

Exemplo do que deveria existir:

```python
if letter.user_id != current_user.id:
    abort(403)
```

Provavelmente o sistema apenas buscava o objeto pelo ID:

```python
letter = Letter.query.get(letter_id)
```

Sem verificar o dono da mensagem.

---

## ⚠ Impacto

Essa falha permite:

- Leitura de dados privados de outros usuários
- Exposição de informações sensíveis
- Escalonamento horizontal de privilégio

Classificação:

OWASP Top 10 – A01: Broken Access Control

---

## 🛡 Como Mitigar

- Validar ownership antes de retornar o recurso
- Implementar controle de acesso baseado em usuário
- Nunca confiar apenas em IDs fornecidos na URL
- Utilizar UUID ao invés de IDs sequenciais quando possível

---

## 🧠 Conclusão

Este desafio demonstrou um caso clássico de **Broken Access Control via IDOR**.

A exploração foi possível devido à ausência de validação de autorização no backend.

A enumeração manual de IDs foi suficiente para acessar recursos protegidos e capturar a flag.

---

## 👨‍💻 Autor

Bruno Pereira Braga  
Pentest Jr | Web Security Enthusiast
