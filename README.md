# Progetto per il corso di Linguaggi di Programmazione e Verifica del Software 
Integrare un meccanismo di Usage Control (UCON) nel protocollo MQTT, migliorando la gestione della sicurezza e delle autorizzazioni dinamiche sui dati scambiati tra Publisher e Subscriber.

## Idea di progetto 
Il progetto modella e verifica formalmente un sistema di sottoscrizione dinamico tra utenti (sottoscrittori) e un servizio centrale (broker), il cui accesso è regolato da un modulo di controllo (UCS).  
Ogni sottoscrittore può richiedere l'accesso a dati, che viene concesso o negato in base ad attributi dinamici. L'obiettivo è garantire che solo gli utenti autorizzati possano ricevere informazioni, anche in presenza di cambiamenti nel tempo. Il sistema è modellato in SMV per permettere la verifica automatica di proprietà di sicurezza, come l’assenza di accesso non autorizzato dopo una revoca o un rifiuto ma soprattutto in caso di multi-utenza.

## Scelte Implementative
Le principali scelte implementative sono state:
1. Doppio sottoscrittore e doppio broker
   Ogni sottoscrittore (subscriber1, subscriber2) è gestito da un broker separato (broker1, broker2), che elabora in modo indipendente le richieste.
   Questa struttura consente:
    * Il confronto tra comportamenti distinti;
    * L’estensione a un sistema multi-utente;
    * La simulazione di un'architettura distribuita o multi-dominio.
  
2. Sistema ciclico e tempo simulato
   L'evoluzione del sistema è basata su un contatore (cycle) che simula il tempo e termina dopo 10 cicli.
   Questo permette di osservare lo sviluppo del sistema in più fasi e testare il comportamento nel tempo.

3. Attributi dinamici dei sottoscrittori:
   Ogni sottoscrittore possiede un attributo booleano (attr1, attr2) che varia secondo regole cicliche: attr1 cambia ogni 2 cicli, attr2 ogni 3.
   Gli attributi influenzano le decisioni del modulo UCS, simulando condizioni esterne dinamiche.

4. Decisione UCS e autorizzazione
   Il modulo UCS valuta le richieste in base all'attributo corrente:  
   Se TRUE → PERMIT, se FALSE → DENY.
   La decisione influisce sul comportamento del broker (che consente o rifiuta la sottoscrizione).

5. Ricezione dati condizionata
   I sottoscrittori ricevono dati solo nello stato SUBSCRIBED.
   È esplicitamente vietato ricevere dati se:
   - la sottoscrizione è revocata;
   - la richiesta è stata negata;
   - lo stato è diverso da SUBSCRIBED.

## Analisi del Modello
### Definizione delle variabili
#### Stati dei sottoscrittori (subscriber1_state, subscriber2_state):
* IDLE: inattivo, non ha richiesto ancora nulla;  
* REQUESTING: ha chiesto la sottoscrizione;  
* SUBSCRIBED: sottoscrizione concessa per cui sta ricevendo dati;  
* REVOKED: sottoscrizione revocata.

#### Stati dei broker(broker1_state, broker2_state):
* WAITING_REQUEST: in attesa di una richiesta;
* EVALUATING: valutando la richiesta;
* ALLOW: accettato;
* DENY: rifiutato.

#### Decisioni UCS (ucs_decision_cycle1, ucs_decision_cycle2):
* PENDING: decisione non ancora presa
* PERMIT: permesso a sottoscrivere
* DENY: negazione della sottoscrizione

#### Attributi dinamici (attr1, attr2):
Variabili booleane che rappresentano condizioni esterne 

#### Altre variabili:
cycle: contatore dei cicli, da 0 a 10.  
receiving_data1, receiving_data2: flag booleani che indicano se i sottoscrittori stanno ricevendo dati.

## Dinamiche di evoluzione  
### Sottoscrittori  
In stato IDLE → fanno una richiesta (REQUESTING) se il ciclo è < 10.  
Se il broker consente (ALLOW), diventano SUBSCRIBED.  
Se il broker nega (DENY), tornano IDLE.  
Se sono sottoscritti e l'UCS cambia idea (DENY), passano a REVOKED.  
Se sono REVOKED, dopo un po’ ritentano (IDLE di nuovo).  

### Broker  
Se un sottoscrittore è in REQUESTING, il broker va in EVALUATING.  
Se UCS dice PERMIT, il broker consente (ALLOW).  
Se UCS dice DENY, il broker rifiuta (DENY).  
In tutti gli altri casi, torna in attesa (WAITING_REQUEST).  

### Attributi dinamici  
attr1 cambia ogni ciclo pari (cycle mod 2 = 0).  
attr2 cambia ogni multiplo di 3 (cycle mod 3 = 0).  
Questo simula un ambiente variabile.

### Decisione UCS  
Se l'attributo è TRUE, permette (PERMIT).  
Se è FALSE, nega (DENY).  

### Ricezione dati  
Un sottoscrittore riceve dati (receiving_data = TRUE) solo se è in stato SUBSCRIBED.  
Altrimenti, non riceve dati.  

## Proprietà Specificate (SPEC)  
### Nessun dato durante REVOKED:  
- AG (subscriber1_state = REVOKED -> !receiving_data1)  
- AG (subscriber2_state = REVOKED -> !receiving_data2)  
➔ Se un sottoscrittore è stato revocato, non deve ricevere dati.  

### Nessun dato se il broker nega:  
- AG (broker1_state = DENY -> !receiving_data1)  
- AG (broker2_state = DENY -> !receiving_data2)  
➔ Se un broker ha negato la sottoscrizione, non deve arrivare nessun dato.

### Solo i sottoscritti ricevono dati:  
- AG (receiving_data1 -> subscriber1_state = SUBSCRIBED)  
- AG (receiving_data2 -> subscriber2_state = SUBSCRIBED)  
➔ Se un sottoscrittore sta ricevendo dati, deve essere per forza in stato SUBSCRIBED.























