# 🐶 Love at First Breach – TryHackMe (2026)
When Hearts "Collide" (Ja da a pista aqui)

## 📌 Informações Gerais

- **Plataforma:** TryHackMe  
- **Categoria:** Web  
- **Dificuldade:** Medium  
- **Técnica explorada:** MD5 Collision Attack  
- **Ferramentas utilizadas:**  
  - nmap  
  - gobuster  
  - curl  
  - docker  
  - fastcoll (MD5 collision generator)

---

## 🧠 Visão Geral do Desafio

A aplicação **Matchmaker** compara o **MD5 da imagem enviada pelo usuário** com os hashes das imagens de cachorros armazenadas no sistema.

Se o hash for idêntico ao de um dos dogs, o sistema considera um "match".

A vulnerabilidade está no uso de **MD5 como mecanismo de verificação de identidade**, ignorando que MD5 é vulnerável a colisões criptográficas.

---

## 🔎 Reconhecimento Inicial

### 🔹 Port Scan

```bash
nmap -sV -sC <IP>
```

**Resultados:**

- 22/tcp → OpenSSH  
- 80/tcp → nginx (Matchmaker web app)

Nenhuma outra porta relevante exposta.

---

### 🔹 Enumeração Web

```bash
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt
```

Descobertas:

- `/static`
- `/upload`
- `/view/<uuid>`
- `/admin`

O endpoint `/upload` retornava **405 Method Not Allowed**, indicando que aceitava apenas requisições POST.

---

## 🎯 Entendimento da Lógica da Aplicação

O site afirmava:

> "Matchmaker compares your photo’s MD5 hash to every dog snapshot."

O fluxo identificado foi:

1. Upload da imagem
2. Cálculo do MD5
3. Comparação com hashes armazenados
4. Redirecionamento para `/view/<uuid>`

Conceitualmente, algo similar a:

```python
if md5(uploaded_file) == md5(dog_image):
    return redirect("/view/" + dog.uuid)
```

---

## 💣 Vulnerabilidade Identificada

O sistema confia exclusivamente no **MD5 da imagem** para validar identidade.

MD5 é vulnerável a **colisão criptográfica**, ou seja:

```
Arquivo A ≠ Arquivo B
md5(A) == md5(B)
```

Isso permite enganar o sistema gerando um arquivo diferente com o mesmo hash.

---

## 🛠 Exploração – MD5 Collision Attack

### 1️⃣ Baixar a imagem do cachorro

Identifiquei o `src` da imagem na página `/view/<uuid>`:

```bash
curl -s http://<IP>/view/<uuid> | grep img
```

Depois baixei a imagem:

```bash
curl -o dog.jpg http://<IP>/static/<imagem>.jpg
```

---

### 2️⃣ Gerar colisão usando fastcoll
https://github.com/brimstone/fastcoll

Utilizei o Docker com o projeto `fastcoll`:

```bash
docker pull brimstone/fastcoll

docker run --rm -v $PWD:/work -w /work brimstone/fastcoll \
  --prefixfile dog.jpg -o collision1.jpg collision2.jpg
```

---

### 3️⃣ Verificação da colisão

```bash
md5sum collision1.jpg collision2.jpg
```

Ambos retornaram o **mesmo hash MD5**, confirmando a colisão.

---

### 4️⃣ Upload do arquivo colidente

Ao realizar o upload do arquivo gerado:

- O sistema aceitou a imagem
- O MD5 bateu com o hash armazenado
- O match foi acionado
- A flag foi revelada

---

## 🚨 Impacto da Vulnerabilidade

O uso de MD5 para validação de identidade permite:

- Bypass de lógica de aplicação
- Manipulação de fluxo interno
- Possível acesso a recursos restritos
- Quebra de integridade

---

## 🛡 Recomendações

- Nunca utilizar MD5 para verificação sensível
- Utilizar funções seguras como:
  - SHA-256
  - SHA-3
- Implementar validação adicional além do hash
- Evitar decisões críticas baseadas apenas em checksum

---

## 📚 Conceitos Aplicados

- Cryptographic hash collision
- Weak hash exploitation
- Web application logic abuse
- Insecure design pattern

---

## 🏁 Conclusão

Este desafio demonstra como o uso incorreto de funções criptográficas antigas pode comprometer totalmente a segurança de uma aplicação.

Mesmo sem falhas clássicas como SQL Injection ou RCE, uma decisão de design insegura foi suficiente para comprometer o sistema.

MD5 não deve ser utilizado em contextos de segurança modernos.

---
