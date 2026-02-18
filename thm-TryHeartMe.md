# TryHeartMe – JWT Privilege Escalation

> Categoria: Web  
> Dificuldade: Easy  
> Plataforma: TryHackMe  
> Foco: Broken Access Control / JWT Manipulation  

---

## 🎯 Objetivo

Acessar e comprar o item oculto **"ValenFlag"** em uma loja temática de Valentine's Day.

---

## 🧠 Reconhecimento Inicial

A aplicação apresentava uma loja online simples com os seguintes endpoints visíveis:

- `/`
- `/login`
- `/register`
- `/product/<slug>`
- `/buy/<slug>`

Nenhum item chamado **ValenFlag** era exibido na página principal.

---

## 🔎 Enumeração de Diretórios

Foi utilizado **Gobuster** para identificar possíveis endpoints ocultos.

```bash
gobuster dir -u http://10.66.173.144:5000 \
  -w /usr/share/wordlists/dirb/common.txt
```

### Resultado relevante:

```
/account              (Status: 302) [--> /login?next=/account]
/admin                (Status: 302) [--> /login?next=/admin]
/login                (Status: 200)
/logout               (Status: 302)
/register             (Status: 200)
```

O endpoint `/admin` indicava possível área restrita por privilégio.

---

## 🔐 Análise do Cookie de Sessão

Após autenticação, foi identificado um cookie chamado:

```
tryheartme_jwt
```

O token foi analisado utilizando a ferramenta online:

👉 https://www.jwt.io/

Ao decodificar o JWT, o payload revelou:

```json
{
  "email": "bruno@bruno",
  "role": "user",
  "credits": 0,
  "iat": 1771382983,
  "theme": "valentine"
}
```

---

## 🚨 Vulnerabilidade Identificada

A aplicação confiava diretamente nos dados do payload do JWT, incluindo:

- `role`
- `credits`

Foi possível modificar o token manualmente para:

```json
{
  "email": "bruno@bruno",
  "role": "admin",
  "credits": 9999,
  "iat": 1771382983,
  "theme": "valentine"
}
```

Após substituir o cookie no navegador (DevTools → Storage → Cookies), a aplicação aceitou o token modificado.

Resultado:

- Role alterada para `admin`
- Créditos alterados para `9999`
- Acesso ao item oculto `/product/valenflag`

---

## 🎯 Exploração

Com privilégios elevados, foi possível acessar:

```
/product/valenflag
```

O item custava:

```
777 credits
```

Com os créditos manipulados, foi possível concluir a compra e obter a flag.

⚠️ Flag omitida neste repositório.

---

## 💥 Impacto

Essa vulnerabilidade permite:

- Escalada de privilégio (User → Admin)
- Manipulação de saldo/créditos
- Acesso a recursos restritos
- Comprometimento completo da aplicação

Classificação:

- Broken Access Control
- Improper JWT Signature Validation
- OWASP Top 10

---

## 🛡️ Mitigação Recomendada

- Validar corretamente a assinatura do JWT
- Utilizar segredo forte
- Não confiar em dados sensíveis armazenados no payload
- Implementar verificação de role no backend baseada em dados do servidor

---

## 📚 Aprendizados

- Funcionamento interno de JWT
- Manipulação de payload
- Uso da ferramenta https://www.jwt.io/
- Enumeração com Gobuster
- Importância da validação de assinatura

---

## 🧩 Conclusão

Este desafio demonstrou como uma implementação incorreta de JWT pode comprometer completamente a segurança de uma aplicação.

Mesmo sendo classificado como "Easy", a falha explorada é comum em aplicações reais mal configuradas.

