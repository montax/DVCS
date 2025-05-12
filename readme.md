# Distributed Version Control Systems (DVCS)

I **Distributed Version Control Systems (DVCS)**, in italiano *sistemi di controllo versione distribuiti*, sono strumenti di versionamento in cui **ogni sviluppatore possiede una copia completa del repository (codice e cronologia)** sul proprio computer. Ciò rappresenta un approccio **peer-to-peer** (paritario) invece del classico modello **client-server** dei VCS centralizzati. In pratica non esiste un singolo server obbligatorio: ogni clone funge da repository completo e le sincronizzazioni avvengono scambiandosi *patch* (insiemi di modifiche) tra pari. Sebbene non sia tecnicamente richiesto un server centrale, **spesso si adotta per convenzione un repository “principale”** considerato fonte ufficiale della verità, attorno a cui i collaboratori si coordinano.

Esempi popolari di DVCS includono **Git, Mercurial** e Bazaar - tra questi **Git è attualmente il più diffuso a livello mondiale**. Git fu inizialmente sviluppato da Linus Torvalds (creatore del kernel Linux) proprio per gestire il codice del kernel in modo distribuito. I DVCS sono considerati una delle innovazioni più significative nello sviluppo software degli ultimi decenni (Joel Spolsky nel 2010 li definì “forse il più grande avanzamento degli ultimi dieci anni”).

**Vantaggi rispetto ai VCS centralizzati:**

* **Lavorare offline:** ogni sviluppatore può effettuare commit, visualizzare la cronologia e creare branch senza essere connesso alla rete. Si può quindi lavorare in viaggio o senza VPN, e sincronizzare le modifiche in un secondo momento (*push/pull*).
* **Prestazioni locali elevate:** le operazioni comuni (commit, diff, log) sono molto rapide perché avvengono in locale, senza dover comunicare con un server remoto ad ogni azione. La rete serve solo per condividere le modifiche tra repository (push/pull), riducendo la latenza.
* **Nessun punto unico di failure:** ogni clone contiene l'intera storia del progetto, fungendo di fatto da backup completo. Se il server principale cade o un clone si corrompe, ce ne sono altre copie integrali disponibili.
* **Commit locali e workflow flessibili:** si possono effettuare commit “privati” nel proprio repository locale senza renderli subito pubblici. Questo consente di salvare *checkpoint* del lavoro o sperimentare liberamente. In seguito, si possono condividere in blocco più commit mediante push. Inoltre, i DVCS supportano nativamente modelli di sviluppo avanzati: branching e merging frequenti, flussi di lavoro non-lineari, fork di progetti, integrazione di contributi da molte fonti, ecc..
* **Collaborazione decentralizzata:** facilita contributi esterni (es. progetti open-source) perché chiunque può clonare un repository, apportare modifiche e poi proporre le modifiche indietro senza necessitare di accesso in scrittura al repository centrale. In progetti FOSS è anche più semplice creare un *fork* indipendente se necessario.
* **Modelli gerarchici opzionali:** un DVCS permette sia approcci completamente distribuiti (ognuno ha il suo repo) sia approcci semi-centralizzati. Ad esempio, in team strutturati si può adottare un modello *“committer/maintainer”* (chiamato anche *lieutenant model*), in cui esiste comunque un repository centrale ufficiale, ma sviluppatori secondari possono scambiare patch tra di loro prima di inviarle al centralizzatore.

**Svantaggi principali:** i DVCS presentano pochi svantaggi intrinseci rispetto ai sistemi centralizzati. Uno è che **il clone iniziale di un repository è più lento e pesante**, dovendo copiare l'intera cronologia del progetto (tutte le revisioni) invece che solo l'istantanea corrente. Inoltre, l'assenza di un meccanismo di *locking* centralizzato può creare difficoltà con **file binari non unibili** (ad esempio asset grafici o documenti), dato che in un DVCS più persone potrebbero modificarli in parallelo senza blocchi. Infine, ogni utente archivia l'intera storia in locale, richiedendo **maggiore spazio disco** (in genere poco problematico al giorno d'oggi) e potenzialmente esponendo il codice a più copie non protette. In generale comunque i benefici superano di gran lunga questi inconvenienti, specialmente considerando strumenti ausiliari come Git LFS per i file grandi.

**Casi d'uso principali:** oggi i DVCS sono utilizzati praticamente in ogni contesto di sviluppo software, dai piccoli progetti personali ai grandi sistemi enterprise. Il loro **punto di forza emerge nei progetti con team distribuiti o numerosi collaboratori indipendenti** (ad esempio comunità open-source): ciascuno può lavorare in parallelo sul proprio clone e proporre le modifiche quando pronte, evitando colli di bottiglia. Un caso emblematico è il **kernel Linux**, mantenuto tramite Git: centinaia di sviluppatori sparsi per il mondo inviano patch ai maintainer, grazie al modello distribuito che permette di integrare contributi in modo scalabile. Anche in team aziendali più piccoli, però, un DVCS offre flessibilità nel ramificare lo sviluppo (feature branch, bugfix branch, ecc.) e sicurezza nel sapere che ogni membro ha una copia completa del repository. In sintesi, i DVCS con Git in testa sono diventati lo **standard de-facto** per il versionamento moderno, rimpiazzando in molti casi strumenti centralizzati come Subversion o CVS.

## Parte Pratica

Di seguito verrà illustrato come utilizzare concretamente un DVCS (nello specifico Git) in vari scenari: **locale**, con repository **remoto ospitato (GitHub)** e con server **self-hosted (Forgejo/Gitea)**. L'ambientazione consigliata è una distribuzione **Ubuntu Server LTS** (es. 20.04, 22.04), ma i comandi indicati (soprattutto quelli `apt` e di shell) sono analoghi per qualsiasi sistema Debian-based.

### Git in locale (repository locale)

In questo scenario configuriamo Git su un server locale e creiamo un repository standalone accessibile solo in locale (nessun server remoto). I passi fondamentali sono:

1. **Installazione di Git su Ubuntu:** assicurarsi che Git sia installato sul sistema. Sui sistemi Ubuntu/Debian l'installazione avviene tramite il package manager APT. Eseguire i comandi seguenti con privilegi di root (o anteponendo `sudo`):

   ```bash
   sudo apt update
   sudo apt install git
   ```

   Questo installerà Git (se non presente) dai repository di Ubuntu. È possibile verificare la versione installata con `git --version`. (Ad esempio, l'output potrebbe essere `git version 2.xx.x`).

2. **Configurazione iniziale di Git (utente e email):** una volta installato, va configurata l'identità dell'utente che effettuerà i commit. Git associa infatti ad ogni commit un nome e una email. Configuriamo questi parametri a livello globale (valgono per tutti i repository dell'utente corrente) usando i comandi:

   ```bash
   git config --global user.name "Nome Cognome"
   git config --global user.email "tuo.email@esempio.com"
   ```

   Sostituire con il proprio nome ed email. Queste informazioni compariranno nei log dei commit. È possibile controllare la configurazione con `git config --list` (che elencherà tra le altre cose `user.name` e `user.email` appena impostati). Queste impostazioni vengono salvate nel file `~/.gitconfig` dell'utente. *(Nota: è possibile anche configurare altri parametri, ad es. l'editor predefinito per i messaggi di commit, ma per ora ci limitiamo ai requisiti di base.)*

3. **Creazione di un nuovo repository locale:** scegliamo o creiamo una directory che diventerà il nostro repository Git. Ad esempio:

   ```bash
   mkdir progetto-demo
   cd progetto-demo
   git init
   ```

   Il comando `git init` inizializza un repository Git vuoto nella cartella corrente. Verrà creata una sottodirectory nascosta `.git/` contenente l'intera struttura del repository (database degli oggetti, riferimenti, configurazioni locali). Dopo questo comando, la directory `progetto-demo` è sotto controllo di versione Git (anche se inizialmente non contiene ancora alcun commit).

   *Esempio:* supponendo di trovarsi in `/home/utente/progetto-demo`, `git init` risponderà con un messaggio tipo: *“Initialized empty Git repository in /home/utente/progetto-demo/.git/”*. Da questo punto in poi possiamo cominciare a registrare modifiche tramite commit.

4. **Eseguire operazioni di commit:** aggiungiamo dei file al repository e creiamo la prima revisione (commit). Ad esempio, creiamo un semplice file di testo e salviamolo:

   ```bash
   echo "Questo è un file di esempio" > README.txt
   ```

   Abbiamo creato un file `README.txt`. Ora informiamo Git che vogliamo tracciare questo nuovo file e includerlo nel prossimo commit, usando `git add`:

   ```bash
   git add README.txt
   ```

   Il comando `git add` sposta il file nell'*area di staging* (o *index*), preparando i contenuti per il commit. Possiamo anche usare `git add .` per aggiungere *tutte* le modifiche correnti (nuovi file, modifiche a file esistenti, cancellazioni) in un colpo solo.

   Adesso creiamo un **commit** effettivo con `git commit`. Ogni commit richiede un messaggio descrittivo; lo passeremo inline con l'opzione `-m`:

   ```bash
   git commit -m "Aggiunge file di esempio"
   ```

   Questo comando registra in modo permanente lo stato attuale (il contenuto di `README.txt`) nella cronologia del repository, associandogli l'ID univoco di commit (SHA-1) e il messaggio *"Aggiunge file di esempio"* come descrizione. Git risponderà mostrando il messaggio di commit, il numero di file interessati, righe aggiunte/rimosse, ecc. (Esempio: *“\[main (root-commit) e4f3b0c] Aggiunge file di esempio 1 file changed, 1 insertion(+), 0 deletions(-) create mode 100644 README.txt”*).

   *Nota:* Il branch predefinito creato da `git init` potrebbe chiamarsi `master` oppure `main` a seconda della versione di Git e configurazione. Su Git recenti (>=2.28) il default è `main`. In questa guida useremo `main` come branch principale, ma tenere presente la differenza se sul vostro sistema il branch di default si chiama `master`. È possibile rinominare il branch di default oppure specificarlo nei comandi di push (vedi sezione Git + GitHub) in base al caso.

5. **Visualizzare lo storico delle revisioni:** ora il repository ha almeno un commit. Possiamo controllare la cronologia con il comando:

   ```bash
   git log
   ```

   Questo mostrerà l'elenco dei commit (dal più recente al più vecchio) con dettagli quali SHA-1 (identificatore) del commit, autore, data e messaggio di commit. Nel nostro esempio vedremo un singolo commit con il messaggio "Aggiunge file di esempio". Per una visuale più compatta, si può usare `git log --oneline --graph --all`, che mostra ogni commit su una riga (SHA abbreviato e messaggio) ed eventualmente un grafico ASCII dei branch. Al momento, con un solo commit e un solo branch, l'output di `git log --oneline` potrebbe essere ad esempio:

   ```
   e4f3b0c Aggiunge file di esempio
   ```

   Un altro comando utile è `git status`, che mostra lo stato delle modifiche rispetto all'ultimo commit (file modificati non ancora committati, nuovi file non tracciati, ecc.). Se lo eseguiamo ora, dovrebbe indicare *“nothing to commit, working tree clean”* a conferma che non ci sono modifiche pendenti (abbiamo appena commitato tutto). In caso contrario, `git status` ci aiuterebbe a capire cosa non è ancora versionato o salvato.

A questo punto abbiamo un repository Git locale funzionante. Possiamo continuare a lavorare creando/modificando file, usando `git add` per metterli in staging e `git commit` per registrare snapshot di versione. Ogni commit aggiornerà la cronologia visibile con `git log`. Con solo un repository locale, però, le modifiche restano sulla singola macchina. Nei passi successivi vedremo come **condividere il repository** in remoto tramite servizi esterni (GitHub) o server self-hosted (Forgejo/Gitea).

### Git + GitHub (Hosted - repository remoto ospitato)

In questo scenario configuriamo un repository Git remoto su **GitHub** - una piattaforma online popolare per l'hosting di repository Git - e colleghiamo il nostro repository locale ad esso. In questo modo potremo **pubblicare (push)** le nostre modifiche online e **sincronizzare (pull)** il lavoro da/verso GitHub, abilitando la collaborazione con altri o semplicemente un backup remoto.

**Passo 1: Creare un account e un repository su GitHub.** Se non lo si possiede già, registrare un account su GitHub (andando su github.com e seguendo la procedura di signup). Una volta autenticati, creare un nuovo repository tramite l'interfaccia web (pulsante “New” o “New repository”). Bisogna scegliere un nome per il progetto (ad esempio *progetto-demo*), opzionalmente una descrizione, e impostare se il repo sarà **pubblico** (visibile a tutti) o **privato**. Per questa prova, può essere pubblico.

> ⚠️ **Importante:** *non* selezionare l'opzione di inizializzare il repo con un **README**, file di licenza o .gitignore durante la creazione su GitHub. Creiamo infatti un repository **vuoto** su GitHub, dato che abbiamo già un repository locale con commit. Eviteremo così conflitti iniziali. (In alternativa, se si volesse inizializzare su GitHub con un README, bisognerebbe poi fare un pull in locale con `--allow-unrelated-histories` per unire le storie, ma complicherebbe inutilmente questo primo approccio.)

Una volta creato, GitHub mostrerà la pagina vuota del nuovo repository. In alto sarà indicato l'URL per clonare o collegare il repo remoto, in due forme: **HTTPS** o **SSH**. Ad esempio:

* URL HTTPS: `https://github.com/tuoUsername/progetto-demo.git`
* URL SSH: `git@github.com:tuoUsername/progetto-demo.git`

**Passo 2: Collegare il repository locale con il repository GitHub remoto.** Torniamo al terminale sul nostro server Ubuntu, all'interno della cartella del repository locale (`progetto-demo`). Dobbiamo aggiungere il repository GitHub come *remote* in Git. Usiamo il comando `git remote add`:

```bash
git remote add origin https://github.com/<tuoUsername>/progetto-demo.git
```

Questo comando associa il nome remoto `origin` all'URL del nostro repository su GitHub. (È convenzione chiamare *origin* il repository remoto principale.) Possiamo verificare che sia stato aggiunto correttamente con `git remote -v`, che elencherà i remoti configurati (fetch/push).

> 💡 **Autenticazione:** se usiamo l'URL HTTPS, al primo tentativo di comunicazione con GitHub verranno richieste le credenziali. Dal 2021 GitHub **non accetta più password in chiaro** per l'autenticazione Git su HTTPS, ma richiede token personali (Personal Access Token) in sostituzione della password, oppure l'uso di altri metodi. In pratica, quando verrà chiesto username e password, bisognerà fornire il proprio username GitHub e come “password” un **token PAT** generato dalle impostazioni GitHub (o utilizzare un helper di credenziali per memorizzarlo). In alternativa, usando l'URL SSH (del tipo `git@github.com:...`), l'autenticazione avverrà tramite **chiave SSH**. Quest'ultimo metodo richiede di aver creato una coppia di chiavi SSH sul proprio server (con `ssh-keygen`) e aver aggiunto la **chiave pubblica** nell'account GitHub (sezione *SSH Keys*). In questa guida procediamo con HTTPS per semplicità, ma tenere presente questi aspetti di autenticazione. (Maggiori dettagli sui problemi comuni di autenticazione sono discussi in *Difficoltà comuni* a fine guida.)

**Passo 3: Eseguire il push iniziale verso GitHub.** Ora abbiamo un remote denominato `origin`. Possiamo inviarvi i nostri commit con:

```bash
git push -u origin main

# nel caso non funzionasse

git push --set-upstream origin master
```
In Git, upstream indica il ramo remoto che il tuo ramo locale “segue” di default per operazioni di fetch, pull e push senza dover specificare ogni volta remoto e ramo di destinazione. Impostare un upstream significa definire una relazione di tracking tra un ramo locale e uno remoto, semplificando così il flusso di lavoro quotidiano

Spieghiamo questo comando: `push` trasferisce i commit dal nostro repository locale al remoto (`origin`). `-u` serve a definire il branch remoto di upstream (collega localmente il branch corrente a quello remoto, così i comandi futuri `git pull` o `git push` sapranno a quale branch fare riferimento di default). `origin main` indica di pushare il branch locale corrente (dovrebbe essere `main`, come creato da `git init`) al branch `main` sul remote. Se il branch remoto `main` non esiste, verrà creato. Dopo aver inviato, Git stamperà qualcosa come:

```
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 227 bytes | 227.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://github.com/tuoUsername/progetto-demo.git
 * [new branch]      main -> main
branch 'main' set up to track remote branch 'main' from 'origin'.
```

Se l'operazione va a buon fine, il repository su GitHub non è più vuoto: contiene ora i commit e i file che avevamo in locale (nell'esempio, README.txt e il commit "Aggiunge file di esempio"). Possiamo verificare accedendo alla pagina web del repo GitHub: dovremmo vedere il file e i dettagli del commit.

> Se durante `git push` si è verificato un errore di autenticazione, significa che la procedura di autenticazione non è stata configurata correttamente (ad esempio PAT mancante o chiave SSH non configurata). In tal caso, Git mostra un messaggio *“Permission denied (publickey)”* nel caso di SSH, oppure un prompt ripetuto di credenziali nel caso HTTPS. Bisogna risolvere impostando correttamente token/SSH (vedi *Difficoltà comuni*). Se invece l'errore è *“remote: Repository not found”* o simile, verificare l'URL remoto e i permessi (ad es. push su un repository privato di un altro utente senza accesso).

**Passo 4: Eseguire pull (sincronizzazione) da GitHub.** Ora che il repository remoto è configurato, è importante saper anche ricevere le modifiche. Il comando base è `git pull` (equivalente a fare prima `git fetch` e poi `git merge`). Se sei l'unico a lavorare sul repository, potresti non aver nulla da scaricare. Per testare la funzionalità, prova a fare una modifica tramite l'interfaccia web di GitHub: ad esempio, modifica il contenuto di README.txt dal browser (GitHub permette di editare file online) e fai Commit direttamente sul sito con un nuovo messaggio (es: “Modifica README via web”). A questo punto il repository remoto ha un commit in più che il tuo locale non conosce. Sul server Ubuntu, lancia:

```bash
git pull origin main
```

Git si connetterà a GitHub, scaricherà il nuovo commit e proverà a fonderlo nel tuo branch locale. Se non ci sono conflitti, la modifica comparirà nel tuo file README.txt locale. Il log (`git log`) mostrerà anche il commit fatto via web. In assenza di modifiche concorrenti, GitHub potrebbe permetterti anche di fare `git pull` semplice (senza specificare `origin main`, perché grazie a `-u` il branch locale *main* traccia ormai quello remoto). **In un contesto multiutente**, la sequenza tipica di collaborazione sarà: altri sviluppatori fanno push su GitHub, tu fai `git pull` regolarmente per aggiornarti; tu fai commit locali e poi `git push` per pubblicarli. Git segnalerà conflitti se due persone modificano le stesse righe negli stessi file - in quel caso occorre risolvere manualmente i conflitti nei file e poi fare commit di merge.

Riassumendo, con GitHub abbiamo visto come **ospitare remotamente** il repository. I comandi chiave introdotti sono: `git remote add` (collegamento remoto), `git push` (invio modifiche) e `git pull` (ricezione modifiche). GitHub offre molte funzionalità aggiuntive (issue tracker, pull request per code review, wiki, ecc.) ma qui ci focalizziamo sull'uso base di Git.

### Git + Forgejo/Gitea (Server Git self-hosted)

Vediamo ora un caso in cui non ci si affida ad una piattaforma di hosting esterna, ma si vuole **ospitare il server Git in proprio**. Strumenti come **Gitea** o **Forgejo** permettono di implementare un servizio tipo “GitHub” sul proprio server. Gitea è un software open-source leggero scritto in Go che fornisce interfaccia web, gestione utenti, issue tracking, wiki, ecc., simile a GitHub ma installabile on-premise. Forgejo è un *fork* comunitario di Gitea nato nel 2022 (quando Gitea ha cambiato modello di governance): in sostanza Forgejo e Gitea al momento offrono funzionalità molto simili, e le istruzioni di installazione/uso sono quasi identiche. Scegliere l'uno o l'altro dipende dalle preferenze di licenza/community, ma a livello tecnico qui li tratteremo in modo intercambiabile.

Scenario: abbiamo un server Ubuntu (può essere una VM, un VPS cloud, o anche la stessa macchina su cui lavoriamo, ma idealmente un'istanza dedicata) su cui installeremo Forgejo/Gitea, e lo useremo come **remoto Git privato**. Dopodiché collegheremo il nostro repository locale a questo server, allo stesso modo di come fatto con GitHub (ma puntando all'host self-hosted). I passi sono:

1. **Installazione di Forgejo/Gitea su Ubuntu Server:** Ci sono diversi modi per installare Gitea/Forgejo. Concentrandoci sulla semplicità per una prova locale, possiamo utilizzare i pacchetti **Snap** ufficiali. Su Ubuntu, Snap è generalmente disponibile di default (per verificarlo: `snap --version`). Se snap è pronto, l'installazione è ridotta a un singolo comando:

   * **Per Gitea:** `sudo snap install gitea` (installa la versione stable corrente di Gitea).
   * **Per Forgejo:** `sudo snap install forgejo`.

   Questo comando scarica e installa l'applicazione. Il package Snap configura Gitea/Forgejo come servizio di sistema attivo automaticamente. **Di default il servizio ascolta sulla porta 3000 TCP** su tutte le interfacce. Assicurarsi che il firewall permetta connessioni su tale porta (su Ubuntu potete usare `sudo ufw allow 3000/tcp` se UFW è attivo). In alternativa ai Snap, si potrebbe installare Gitea/Forgejo manualmente (scaricando il binario dal sito ufficiale, creando un utente dedicato *git*, configurando un servizio systemd, ecc.) oppure usare un container Docker. Per una guida studentesca, tuttavia, lo snap semplifica molto la procedura.

   > **Nota:** Lo Snap di Gitea/Forgejo potrebbe utilizzare una configurazione predefinita (generalmente con database SQLite integrato) e non richiedere un passo di installazione via web. In caso di installazione manuale, il primo avvio di Gitea aprirebbe una pagina di setup (wizard) dove scegliere database e parametri. Con Snap, se il servizio è già attivo, si può comunque accedere direttamente all'interfaccia web. Nel dubbio, procediamo come se dovessimo eseguire la configurazione iniziale via browser (non nuoce ricontrollare).

2. **Configurazione iniziale e utente admin:** Aprire un browser web e collegarsi all'indirizzo del server Forgejo/Gitea. Se il server è la stessa macchina locale, l'URL sarà `http://localhost:3000` (porta 3000). Altrimenti, usare l'IP o hostname del server (es: `http://192.168.1.100:3000`). Dovrebbe comparire la pagina iniziale di Gitea/Forgejo.

   * Se appare un **installer wizard** (schermata di configurazione): compilare i campi richiesti. Di solito bisogna scegliere il tipo di database (per semplicità selezionare **SQLite3** che non richiede setup esterno), e lasciare la maggior parte degli altri valori di default. Verificare che “SSH Server Domain” sia l'IP/host corretto del server e “SSH Port” 22 (se prevedete di usare SSH - vedi più avanti), oppure regolare se il server già usa la porta 22 per altro. Controllare anche che “Gitea HTTP Listen Port” sia 3000 e “Gitea Base URL” corrisponda all'URL con cui accederete (es. http\://<vostroIP>:3000). Una volta pronti, cliccare **Install**. L'installazione verrà completata in pochi istanti e dovrebbe reindirizzare alla pagina di login.
   * Se invece appare direttamente la schermata di **login/registrazione** (possibile con Snap se l'installer è stato auto-configurato): cliccare su “Register” (o “Registrati”).

   A questo punto bisogna creare il primo utente. **Registrare un account** compilando username, email e password. Questo sarà l'utente amministratore di sistema (sia Gitea che Forgejo assegnano automaticamente i privilegi admin al primo utente registrato). Ad esempio, creare un utente `admin` con una password sicura. Dopo la registrazione, effettuare il **login** con le credenziali appena create.

   Ora abbiamo accesso al pannello web di Forgejo/Gitea come amministratori. Possiamo navigare l'interfaccia: creare organizzazioni, gestire utenti (eventualmente invitare o creare account per collaboratori), visualizzare statistiche, ecc. Per i nostri scopi, il prossimo step è creare un repository Git tramite l'interfaccia web.

3. **Creare un repository sul server self-hosted:** Dalla dashboard di Forgejo/Gitea, cliccare su “New Repository” (nuovo repository). Inserire il **nome** del repository (ad esempio *progetto-demo* come prima), una descrizione se desiderata, e selezionare se crearlo sotto il proprio utente (o eventualmente sotto un'organizzazione, se ne avete creata una). Potete scegliere se inizializzarlo con un README stavolta - per coerenza, potete anche qui lasciarlo vuoto, dato che importeremo i dati dal repository locale. Scegliere se il repo sarà **Privato** o **Pubblico**: un repository pubblico su un server self-hosted significa che chiunque conosce l'URL può vederlo (utile se volete condividerlo in sola lettura pubblicamente); privato significa che solo voi e chi autorizzerete potrà accederlo (richiede login). Se state solo testando in un ambiente chiuso, la differenza è poca; scegliete in base alle esigenze.

   Confermate la creazione. Dovreste essere reindirizzati alla pagina del repository appena creato su Forgejo/Gitea. Similarmente a GitHub, verranno mostrati i comandi Git per collegare un repository esistente. Troverete gli URL **HTTPS** e **SSH** del repository (di solito indicati con dei pulsanti o campi di testo in alto). Saranno qualcosa tipo:

   * HTTPS: `http://<ip-server>:3000/<utente>/progetto-demo.git`

   * SSH: `git@<ip-server>:<utente>/progetto-demo.git`

   > **Gestione accessi:** In questo scenario siete admin e proprietari del repo, quindi avete pieno accesso. Se voleste aggiungere altri collaboratori, potete creare altri account utente (come admin, via “Site Administration” potete aggiungere utenti, o abilitare la self-registration se disabilitata) e poi, all'interno delle impostazioni del repository (Settings -> Collaborators o simile), concedere accesso in scrittura ad altri utenti. In Forgejo/Gitea potete anche creare *Organizzazioni* con team, simile al concetto di *organization* su GitHub, per gestire i permessi su più repo. Per semplicità, tralasciamo i dettagli, ma è importante sapere che il controllo accessi può essere raffinato: se il repo è privato, solo gli utenti autorizzati lo vedranno; se è pubblico, chiunque può clonarlo in lettura (ma solo voi potrete pushare, a meno di autorizzare altri).

4. **Collegare il repository locale al server Forgejo/Gitea e fare push delle modifiche:** Ora torniamo di nuovo al terminale sul nostro sistema locale, nella directory del repository Git (`progetto-demo`). Dobbiamo aggiungere un altro *remote* che punti al server self-hosted. Possiamo anche riutilizzare `origin` (sovrascrivendo quello di GitHub se non ci serve più) oppure aggiungerne uno nuovo con altro nome. Per evitare confusione, supponiamo che non stiamo più usando l'origine GitHub, quindi possiamo cambiarne l'URL oppure rimuoverlo. Ad esempio, per rimpiazzare origin con il nuovo server self-hosted:

   ```bash
   git remote set-url origin http://<ip-server>:3000/<utente>/progetto-demo.git
   ```

   In alternativa, se volete mantenere separati (ipotizzando di avere due remoti, uno GitHub e uno self-hosted), potete aggiungere il secondo con un nome diverso, ad es.:

   ```bash
   git remote add forgejo http://<ip-server>:3000/<utente>/progetto-demo.git
   ```

   (In tal caso usereste `forgejo` al posto di `origin` nei comandi di push/pull verso il server self-hosted.)

   Qui per chiarezza continueremo ad assumere un solo remote chiamato `origin` che ora punta al vostro server Forgejo/Gitea. Dopo aver fatto `set-url` come sopra, verificate con `git remote -v` che origin ora riporti l'URL del vostro server.

   Adesso eseguiamo il push al server:

   ```bash
   git push -u origin main

   # nel caso non funzionasse

   git push --set-upstream origin master
   ```

   Similmente a prima, questo caricherà i commit sul repository remoto. Essendo un server nuovo, potrebbe chiedere l'autenticazione. **Con URL HTTP/HTTPS, Forgejo/Gitea chiederà username e password** dell'account utente che sta eseguendo l'operazione (cioè il vostro utente admin creato prima). Inseriteli quando richiesto (tip: potete abilitare la cache delle credenziali in Git per non reinserirle ad ogni push, usando ad es. `git config --global credential.helper cache`). In alternativa, **usando l'URL SSH**, l'autenticazione avviene con chiave: dovreste prima aver aggiunto la *chiave pubblica SSH* del vostro client nelle impostazioni utente di Forgejo/Gitea (di solito sotto “Settings” -> “SSH Keys” nell'interfaccia web, simile a come si fa su GitHub). Una volta aggiunta, potete usare `git remote set-url origin git@<ip-server>:<utente>/progetto-demo.git` e poi il push non richiederà password, autenticandosi via chiave. Scegliete il metodo preferito; per molti versi, *gestire le chiavi SSH è il metodo consigliato* anche in ambienti self-hosted, per comodità e sicurezza.

   Dopo il push, se tutto è andato bene, vedrete output simile a quello già visto con GitHub (conteggio oggetti, new branch main -> main, ecc.). Andate sull'interfaccia web di Forgejo/Gitea, ricaricate la pagina del repository: dovreste ora vedere i file e i commit caricati. **Congratulazioni, il vostro server Git self-hosted è operativo!**

   Da qui in poi, l'utilizzo è analogo a GitHub: potete fare modifiche locali, poi `git push` per aggiornarle sul server; oppure modifiche via interfaccia web o da altri cloni e usare `git pull` per riceverle. Forgejo/Gitea permette anche operazioni come *fork* interni, pull request tra branch, ecc., sebbene in un contesto piccolo potreste semplicemente dare accesso push agli altri collaboratori sullo stesso repo.

   *Nota:* se avete configurato l'accesso SSH, potete anche clonare direttamente dal server con comandi come `git clone git@<ip-server>:<utente>/progetto-demo.git` da altre macchine (dopo aver messo la chiave pubblica appropriata in quell'account). Se usate HTTPS e il repo è privato, il clone via `http://...` chiederà credenziali utente. Per test interni su LAN potete anche lasciare tutto su HTTP non cifrato, ma **in produzione è fortemente raccomandato abilitare HTTPS** (vedi note più avanti su come fare, tipicamente con un proxy e certificati).

## Testing e Verifiche

A questo punto abbiamo coperto tre configurazioni: repository locale, repository ospitato su GitHub, e repository su server Forgejo/Gitea. È importante **verificare il corretto funzionamento** di ciascuna configurazione. Ecco alcuni test e controlli consigliati:

* **Repository locale:** dopo alcuni commit, eseguire `git log --oneline` per assicurarsi che la cronologia contenga i messaggi attesi. Provare a modificare un file, eseguire `git status` (dovrebbe segnalare il file modificato in *modified*), poi fare commit e ricontrollare `git status` (che deve tornare “clean”). Si può anche provare a recuperare una versione precedente con `git checkout <SHAcommit>` (operazione di *detached HEAD*) per vedere come Git ripristina i file allo stato di quel commit, e poi tornare alla punta con `git checkout main`. Se tutto questo funziona, il repository locale è sano.

* **Repository su GitHub:** verificare via browser che i file e commit appaiano nella pagina del repository su GitHub. Ogni commit ha un hash e un messaggio: controllate che corrispondano a quelli fatti in locale. Un semplice test è modificare un file tramite GitHub (come descritto prima), committare via web, quindi eseguire `git pull` in locale: il contenuto locale deve aggiornarsi in base alla modifica fatta sul sito. Inoltre, provate a **clonare** il repository da GitHub in una nuova directory (`git clone https://github.com/tuoUsername/progetto-demo.git`): questo simula un nuovo collaboratore o un nuovo computer. Dopo il clone, eseguite `git log` nella repo clonata per vedere se tutta la cronologia è presente. Fate magari un nuovo commit in questa seconda copia e provate a fare `git push`: GitHub dovrebbe accettarlo (se avete i permessi e avete configurato il PAT/SSH anche lì) e poi quel commit dovrebbe comparire anche nel primo ambiente con un `git pull`. Se incontrate errori di push (es. *non-fast-forward*), vuol dire che la cronologia del remoto ha cose che mancano localmente - seguite le istruzioni dell'errore (tipicamente: fare un pull/merge prima di riprovare il push).

* **Repository su Forgejo/Gitea:** molto simile a GitHub per i test. Dal lato web, controllare che nella repository compaiano tutti i file e i commit inviati. Navigate nell'interfaccia di Forgejo/Gitea: dovrebbe esserci una sezione “Commits” o log, dove potete vedere la lista dei commit con autore e messaggio, confermando che quelli fatti in locale sono registrati. Anche qui, provate a **clonare** il repository dal server self-hosted su un altro sistema (o un'altra cartella) usando l'URL HTTP o SSH fornito, e assicuratevi che il clone funzioni e porti giù tutti i dati. Se il server è in LAN, usate l'IP; se è su internet e avete un DNS, usate il nome di dominio (preferibilmente con HTTPS se configurato). Dopo il clone di prova, fate un commit in questa copia e provate il push: dovrebbe funzionare (se le credenziali/chiavi sono a posto). Verificate poi sul server (via web) e sull'altro clone (via pull) che il commit sia propagato. Inoltre, sperimentate l'interfaccia web: caricate magari un file tramite l'UI (c'è una funzione "Upload file" in Gitea), o aprite un issue se avete abilitato il tracker, tanto per esplorare.

* **Interfaccia web di Forgejo/Gitea:** se avete effettuato l'accesso come admin, controllate anche la sezione di amministrazione del sito (ad es. potete verificare le impostazioni globali, come opzioni di registrazione utenti, e i log di sistema, se presenti). Questo vi fa capire che avete il controllo completo della piattaforma. Se possibile, provate a creare un secondo utente non admin e simulate il suo accesso al repo (ad es. aggiungendolo come collaboratore o rendendo il repo pubblico, poi facendolo clonare e proporre modifiche via pull request). Ciò verifica che i permessi sono corretti.

* **Performance e stabilità:** in ambiente locale con pochi utenti, difficilmente avrete problemi di performance. Tuttavia, vale la pena notare quante risorse consuma il server Forgejo/Gitea. Su Ubuntu, potete usare `sudo systemctl status gitea` (se installato via snap potrebbe essere differente il service name, forse `snap.gitea.gitea` o simili) per vedere se il servizio è *active (running)*. I log applicativi di Gitea/Forgejo di solito risiedono in `/var/snap/gitea/common/log` (per snap) o `/var/lib/gitea/log` (installazione manuale). Verificate che non vi siano errori evidenti lì durante le operazioni di push/pull.

Se tutti questi test passano, significa che le configurazioni funzionano correttamente: Git locale funziona, e la sincronizzazione con remoti sia pubblici (GitHub) che privati (Forgejo/Gitea) è riuscita.

## Considerazioni Finali

Implementare un DVCS e i relativi flussi di lavoro può presentare alcune sfide iniziali. Di seguito elenchiamo **difficoltà comuni**, possibili **miglioramenti** dell'infrastruttura e **suggerimenti utili** per un progetto sui DVCS:

* **Autenticazione e Chiavi SSH:** l'errore forse più frequente quando si configurano remoti è *“Permission denied (publickey)”* durante i push/pull via SSH. Questo accade se non si è configurata una chiave SSH valida per l'account remoto. La chiave pubblica va **registrata nelle impostazioni dell'account** (sia su GitHub che su Gitea/Forgejo) altrimenti il server rifiuterà la connessione. Assicuratevi di generare una chiave (`ssh-keygen`) e aggiungere la public key su GitHub (Settings -> SSH and GPG keys) o su Forgejo/Gitea (User Settings -> SSH Keys). In alternativa, se usate HTTPS con GitHub, ricordate di creare un **Personal Access Token (PAT)** dal sito GitHub e usarlo al posto della password al primo push (specie se il vostro account ha 2FA attivo, il PAT è obbligatorio). Per Forgejo/Gitea in HTTPS, l'autenticazione base con username/password funziona se non avete altre restrizioni, ma valutate l'uso di token se il servizio li supporta. Una volta sistemata l'autenticazione, potete anche configurare Git perché memorizzi temporaneamente le credenziali (es. `git config credential.helper cache` o l'utilizzo del tool `gh` di GitHub CLI per gestire login OAuth).

* **Errori di push/pull e conflitti:** se qualcun altro ha spinto modifiche sul remoto e il vostro repository locale non è aggiornato, un tentativo di `git push` può essere rifiutato con errore *non-fast-forward*. Questo avviso (*“failed to push some refs… non-fast-forward”*) indica che dovete prima allineare il vostro branch con le nuove commit a monte. In pratica, la soluzione è eseguire `git pull` (o `git fetch` + `git merge`) per integrare le modifiche remote nel vostro lavoro, risolvendo eventuali conflitti, e solo poi riprovare il push. I **conflitti di merge** avvengono quando le stesse parti di codice sono state modificate in parallelo: Git vi segnalerà i file in conflitto (inserendo marcatori `<<<<<<` nei file stessi) e sta a voi editarli manualmente per unire le modifiche in modo coerente, poi fare `git add` dei file risolti e infine `git commit` (Git creerà un commit di merge). Per evitare conflitti grossi, **comunicate e sincronizzatevi spesso** col team: fare pull frequenti e magari adottare convenzioni (es. ciascuno lavora su file o funzionalità diverse, o usare branch dedicati per feature e poi pull request).

* **Configurazione HTTPS su server self-hosted:** di default Gitea/Forgejo espone HTTP sulla porta 3000. In un ambiente di produzione sarebbe inaccettabile trasmettere credenziali e dati in chiaro. Il miglioramento standard è mettere un **reverse proxy con SSL** davanti al servizio. Ad esempio, installare **Nginx** o Apache sul server, configurarlo per ascoltare sulla porta 443 con un certificato TLS (magari ottenuto gratuitamente via **Let's Encrypt**), e fare proxy-pass delle richieste verso l'applicazione Gitea in ascolto su 3000. In questo modo gli utenti accederanno via `https://git.tuo-dominio.it` in modo sicuro. La configurazione di Nginx includerà anche il host name corretto, così potrete impostare `Gitea Base URL` uguale all'URL pubblico. Questa procedura è ben documentata nelle guide ufficiali di Gitea/Forgejo. In alternativa, Gitea supporta anche l'ascolto diretto su HTTPS se gli si fornisce un certificato, ma l'uso di un proxy è più flessibile (consente anche di usare lo standard 443 senza dare privilegi di root al processo Gitea, e di servire altri servizi sullo stesso dominio). Per un progetto studentesco, potete menzionare l'aspetto HTTPS anche se magari non implementato completamente, l'importante è mostrarne la consapevolezza.

* **Backup dei dati:** avere repository distribuiti riduce il rischio di perdita totale (ogni clone è un backup). Tuttavia, **non trascurate il backup** del server centrale e dei metadati. Se usate GitHub, il problema non si pone (ci pensa GitHub stesso, anche se può essere utile fare mirror dei repository importanti). Su Gitea/Forgejo invece, oltre ai repository Git (che comunque sono clonabili altrove), esistono dati come: utenti e password, impostazioni, issue e wiki (se usati) che risiedono nel database. Se usate SQLite, fate copie periodiche del file `.db`; se usate MariaDB/Postgres, predisponete un dump programmato. Inoltre, il contenuto dei repo Git risiede tipicamente sotto `/var/lib/gitea/git-repositories`. Potreste impostare un cron job di **backup** che faccia tar.gz di `/var/lib/gitea` e del file di db, mettendoli al sicuro. Questo è un possibile miglioramento infrastrutturale se il progetto dovesse evolvere.

* **Gestione di file di grandi dimensioni:** per default Git non gestisce bene file binari molto pesanti o generati di frequente (es. video, dataset, build artifact). Se il vostro progetto li include, potreste incontrare problemi di repository enorme o conflitti su binari. Una best practice è **non versionare file binari di grandi dimensioni** a meno che sia necessario. Per quelli indispensabili, Git offre l'estensione **Git LFS (Large File Storage)** che conserva i file su storage separato e tiene nel repo solo riferimenti. Questo può essere integrato sia su GitHub (che supporta LFS nativamente) sia su Gitea/Forgejo (va attivato, ma Gitea supporta Git LFS nelle ultime versioni). LFS mitiga uno dei limiti dei DVCS con file grossi. In un contesto didattico forse non avrete file enormi, ma è bene citarlo.

* **Branching e workflow avanzati:** un miglioramento sul semplice flusso main+push è introdurre branch per organizzare lo sviluppo. Ad esempio, adottare la strategia **Git Flow** o una sua variante: usare branch separati per funzionalità (feature branch), un branch `develop` per l'integrazione e `main` (o `master`) per le release stabili. Su GitHub questo si integra con le **Pull Request**: invece di pushare direttamente su `main`, si lavora su un branch, poi si apre una PR sul repo remoto per far revisionare e fondere il codice (molto utile in team per assicurare code review e test prima di merge). Anche Forgejo/Gitea supporta le pull request (chiamate *Merge Requests* in alcune interfacce). Implementare queste pratiche sarebbe un plus nel progetto, mostrando non solo la configurazione tecnica ma anche l'uso corretto di un workflow collaborativo robusto.

* **Politiche di sicurezza e gestione utenti:** se il vostro server self-hosted sarà usato seriamente, configurate adeguatamente le **policy**: ad esempio abilitate l'HTTPS (come detto), eventualmente impostate registrazione utenti solo su invito se non volete intrusioni, e utilizzate chiavi SSH per tutti. Tenete aggiornato il software (GitHub ovviamente si aggiorna da sé; per Gitea/Forgejo, seguite le release e aggiornate lo Snap o il binario periodicamente per ottenere patch di sicurezza e nuove funzioni). Valutate integrazioni di autenticazione (Forgejo/Gitea supportano OAuth, LDAP, ecc. se doveste integrare con sistemi aziendali, ma è avanzato). Per progetti didattici, almeno menzionare l'importanza degli aggiornamenti e della sicurezza dimostra consapevolezza.

* **Integrazioni CI/CD:** un possibile step successivo è collegare il VCS a sistemi di integrazione continua. Su GitHub, ad esempio, potete attivare **GitHub Actions** per eseguire build/test automatici ad ogni push. Su un server self-hosted, potete integrare con tool come Jenkins, GitLab CI (se usate GitLab invece di Gitea), oppure sfruttare plugin di Gitea (ci sono integrazioni con Drone CI, Woodpecker CI, ecc.). Questo esula dalla configurazione base del DVCS, ma è un suggerimento di miglioramento se il progetto cresce - citandolo, fate capire che avete visione di come usare le commit per attivare pipeline automatizzate di verifica o deployment.

* **Utilizzare .gitignore:** un consiglio pratico utilissimo è configurare un file **.gitignore** appropriato per il progetto. Questo file elenca i pattern di file che Git deve ignorare (non includere nei commit). Ad esempio, file temporanei, di configurazione locale, compilati, cache, etc. In un progetto Python si ignoreranno **pycache**, in Java i .class e le cartelle target, in Node i node\_modules, ecc. GitHub, Gitea e altri offrono template di .gitignore per vari linguaggi. Assicuratevi di aggiungere un .gitignore sin dall'inizio per evitare di committare file indesiderati. È più facile evitare di aggiungere file sensibili (es. file con password in chiaro) con un buon .gitignore piuttosto che rimuoverli a posteriori (il *purge* dalla cronologia di Git è possibile ma non banale).

* **Comunicazione e documentazione:** infine, ricordate che un sistema di versione è efficace se tutti lo usano in maniera consistente. Documentate nel README del progetto le convenzioni adottate (es: branch naming, convensione dei commit message, processo per fare merge). Insegnate al team ad usare i comandi base e magari fornite qualche alias Git utile. Eseguite periodicamente un **git pull --prune** per pulire eventuali branch remoti eliminati. E soprattutto, **commit e push frequenti**: lavorare a lungo senza salvare i cambi su Git porta a grandi commit poco leggibili e a rischio di conflitti; è meglio fare commit atomici e descrittivi man mano che si completano piccole parti di lavoro.

Con questa guida avete una panoramica completa: dalla teoria dei DVCS, alla pratica con Git in locale, all'uso di un servizio hosted come GitHub, fino all'installazione e configurazione di un server Git self-hosted con Forgejo/Gitea. Seguendo questi passi, uno studente dovrebbe essere in grado di **realizzare un progetto sui DVCS** comprendendo sia i concetti che le competenze tecniche necessarie. Buon versionamento!

**Fonti Utili:**

* Pro Git book - *“Getting Started”*, per approfondire l'uso di Git e le best practice.
* Documentazione GitHub: *“Adding a local repository to GitHub”* (istruzioni ufficiali per collegare un repo esistente).
* Documentazione Gitea/Forgejo: guide di installazione su Ubuntu e configurazione avanzata (es. integrazione con Nginx per HTTPS).
* Atlassian Blog - *“Centralized vs Distributed VCS”*, per una discussione dettagliata sui vantaggi dei DVCS.
* Wikipedia - *“Distributed version control”*, per definizioni e confronto sintetico tra DVCS e CVCS.