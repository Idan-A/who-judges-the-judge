# SPIEGAZIONE – solo per te, NON da pubblicare (è già nel .gitignore)

## L'idea in una frase
Molte aziende fanno valutare le risposte dei loro modelli AI da un altro LLM (il "giudice") invece che
da persone. Tu controlli se di quel giudice ci si può fidare, confrontandolo con esperti umani. È il
lavoro del team AI Quality di Planck: misurare se l'AI fa il lavoro giusto.

## I dati
- **MT-Bench** (LMSYS, 2023): 80 domande in 8 categorie (scrittura, roleplay, ragionamento, matematica,
  coding, estrazione di informazioni, materie scientifiche, materie umanistiche). Sei modelli hanno
  risposto.
- Per molte coppie di risposte (A vs B) ci sono:
  - i voti di **65 esperti umani** (soprattutto dottorandi del settore): 3.355 voti;
  - i voti di **GPT-4 usato come giudice** sulle stesse risposte: 2.400 voti.
- Ogni voto dice: vince A, vince B, oppure pareggio.
- Licenza CC BY 4.0: si possono usare citando la fonte (è nel README).

## Cosa fa il codice, passo per passo
Tutto è in un solo notebook, `who_judges_the_judge.ipynb`, diviso in sezioni numerate.
1. Sezione 1 scarica i 3 file in una versione fissa, così chi rifà l'analisi trova gli stessi dati.
2. Sezione 2 carica i voti e li "normalizza":
   - mette sempre i due modelli in ordine alfabetico (e scambia il voto di conseguenza). **Perché:** un
     esperto può aver visto (GPT-4 vs Claude) e un altro (Claude vs GPT-4); senza riordinare, la stessa
     opinione sembrerebbe un disaccordo;
   - unisce "tie" e "tie (bothbad)" in un solo "pareggio".
3. Sezioni 3-7 fanno 4 analisi (l'ultima cella ti fa leggere un disaccordo alla volta: cambia `i`):
   - **Accordo generale.** Prende tutte le coppie di voti sullo stesso item: esperto–esperto (la
     **baseline**) e GPT-4–esperto. Calcola la percentuale di accordo con e senza pareggi e il kappa di
     Cohen (accordo corretto per il caso: 0 = come tirare a sorte, 1 = accordo perfetto).
   - **Accordo per categoria.** Stessa cosa, divisa per tipo di compito.
   - **Lunghezza.** Quando una risposta è almeno il 20% più lunga, quante volte viene scelta la più
     lunga? Confronta GPT-4 ed esperti **sugli stessi item**, altrimenti il confronto non è giusto.
   - **Disaccordi netti.** Item dove almeno 2 esperti sono tutti d'accordo e GPT-4 sceglie l'altra
     risposta: li salva in `results/disagreements.csv` da leggere a mano.

## I risultati (numeri veri, prodotti dallo script il 16.11.2026)
- Accordo senza pareggi: esperto–esperto 0,827; GPT-4–esperto 0,856. Kappa 0,47 e 0,48.
  → In media il giudice va d'accordo con gli esperti quanto gli esperti tra loro. È il risultato del
  paper originale: lo **riproduci**, non lo scopri tu. Dillo così, è più credibile.
- Per categoria: da 0,74 (writing) a 0,95 (math). Nel **coding** GPT-4 è sotto la baseline (0,86 vs
  0,92); nei compiti aperti (writing, roleplay) gli esperti stessi sono meno d'accordo tra loro.
- Lunghezza: GPT-4 sceglie la risposta più lunga un po' più spesso degli esperti (73% vs 67%, e 82% vs
  77% quando è più del doppio).
- Disaccordi netti: 22 su 485 (4,5%), soprattutto writing (7) e roleplay (4). In questi casi GPT-4 ha
  scelto la più lunga solo nel 32%: gli errori netti non dipendono dalla lunghezza.

## Cosa ti chiedo prima di pubblicare (30-45 minuti, rende il progetto tuo)
Apri `results/disagreements.csv`, leggi almeno 10 dei 22 casi e scrivi nella colonna `my_note` perché
secondo te il giudice ha sbagliato (es. "ha premiato il tono e ignorato un errore di calcolo", "non ha
seguito il vincolo della domanda"). Poi aggiungi al README una sezione breve "What I found reading the
disagreements" con 2-3 tipi di errore ricorrenti. Scrivila tu: al colloquio ti chiederanno proprio questo.

## Scelte fatte e alternative
- **Niente giudice fatto girare da me** (era nella prima versione, l'ho tolto): serviva torch e un
  modello, più pesante e meno credibile per un progetto junior. I voti di GPT-4 sono già nel dataset.
- **Accordo con e senza pareggi:** il pareggio è ambiguo (una persona dice "uguali", l'altra sceglie per
  poco). Riportare entrambi evita di scegliere la versione che fa sembrare migliore il risultato.
- **Baseline umana:** senza, "86% di accordo" non vuol dire niente.
- **Caratteri come misura di lunghezza:** semplice e trasparente; i token sarebbero più precisi.
- Alternativa scartata: intervalli di confidenza con bootstrap. Utili, ma rendevano il progetto più
  pesante; nel README dico chiaramente che le differenze di pochi punti sono incerte.

## Domande da colloquio (con risposte)
1. **Perché un'azienda usa un LLM come giudice?** Valutare a mano migliaia di output è lento e caro; un
   giudice automatico permette di testare ogni versione del modello. Ma va validato contro persone.
2. **Cos'è la baseline e perché serve?** È quanto gli esperti sono d'accordo tra loro (0,83). Se gli umani
   stessi non concordano, non posso pretendere che il giudice "concordi con gli umani" al 100%.
3. **Cos'è il kappa di Cohen?** Accordo corretto per quello che ci si aspetterebbe per caso. 0,47-0,48 è
   un accordo moderato, uguale per giudice ed esperti.
4. **Perché hai riordinato i modelli in ordine alfabetico?** Perché (A vs B) e (B vs A) sono lo stesso
   confronto: senza riordinare, opinioni identiche sembrerebbero disaccordi.
5. **Cosa vuol dire che nel coding il giudice è sotto la baseline?** Dove c'è una risposta giusta da
   verificare (codice che funziona o no) gli esperti concordano molto e il giudice sbaglia di più. Lì
   affiancherei controlli automatici (eseguire il codice) o revisione umana. Con circa 200 coppie è
   un'indicazione, non una prova.
6. **Cos'è il verbosity bias e l'hai trovato?** La tendenza a premiare le risposte lunghe. GPT-4 sceglie
   la più lunga 5-6 punti più spesso degli esperti sugli stessi item: c'è, ma è piccolo. E nei disaccordi
   netti non è la causa (32%).
7. **Come applicheresti questo a Planck?** Prima di usare un giudice per gli agenti assicurativi: un
   piccolo gold set valutato da persone, accordo misurato per tipo di compito (estrazione dati da
   documenti, confronto polizze, risposte libere), controllo dei bias e lettura a mano dei disaccordi.
8. **Limiti?** GPT-4 e modelli del 2023; categorie con poche centinaia di coppie; non ho fatto girare un
   giudice io; la lunghezza in caratteri è un'approssimazione.
9. **Cosa faresti dopo?** Ripetere con un giudice recente sullo stesso dataset; controllare il position
   bias (scambiare A e B e vedere se il voto cambia); costruire una piccola tassonomia degli errori dai
   disaccordi.
10. **Hai inventato qualcosa rispetto al paper?** No: riproduco il risultato generale e aggiungo il
    confronto per categoria con la baseline, il controllo della lunghezza sugli stessi item e la lista
    dei disaccordi da analizzare a mano.

## Come pubblicarlo
1. Prova prima su Colab: colab.research.google.com → *File → Carica notebook* → scegli
   `who_judges_the_judge.ipynb` → *Runtime → Esegui tutto*. Devi vedere gli stessi numeri di sopra.
2. Crea un repository pubblico `who-judges-the-judge` sul tuo GitHub.
3. Carica (*Add file → Upload files*): `who_judges_the_judge.ipynb`, `README.md`, `requirements.txt`,
   `.gitignore`, le cartelle `results/` e `figures/`. **Non** caricare `SPIEGAZIONE.md` né `data/`.
   (Se carichi il notebook dopo averlo eseguito su Colab, su GitHub si vedono anche i grafici: meglio.)
3. Mandami il link: lo aggiungo ai Fatti del prompt e lo usiamo nel CV e nel messaggio per Planck.
