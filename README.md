**GUIDA IN FASE DI SCRITTURA**

# Indice

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:0 orderedList:0 -->

- [Indice](#indice)
- [Introduzione](#introduzione)
- [Generazione sicura di chiavi OpenPGP](#generazione-sicura-di-chiavi-openpgp)
	- [Prerequisiti](#prerequisiti)
	- [Generazione delle chiavi](#generazione-delle-chiavi)
	- [Il certificato di revoca](#il-certificato-di-revoca)
	- [Impostare le preferenze della chiave](#impostare-le-preferenze-della-chiave)
	- [Impostare il file gpg.conf](#impostare-il-file-gpgconf)
	- [Impostare il keyserver](#impostare-il-keyserver)
	- [Esportare la chiave pubblica sul keyserver](#esportare-la-chiave-pubblica-sul-keyserver)
- [Trasferire le chiavi OpenPGP su YubiKey](#trasferire-le-chiavi-openpgp-su-yubikey)
- [Importare la chiave principale per operazioni speciali](#importare-la-chiave-principale-per-operazioni-speciali)
- [Estendere la validità delle nostre chiavi](#estendere-la-validità-delle-nostre-chiavi)
- [Bibliografia](#bibliografia)

<!-- /TOC -->

# Introduzione

Come sappiamo, tutto il sistema della [crittografia a chiave pubblica](https://it.wikipedia.org/wiki/Crittografia_asimmetrica) si basa sull'uso di una chiave che ha una parte pubblica, da divulgare il più possibile, e una parte privata, che va tenuta rigorosamente in un luogo sicuro. Qualora avessimo il dubbio che le chiavi private possano essere in mani altrui, saremmo obbligati a generare un nuovo mazzo di chiavi, con tutto quello che ne comporta (revoca delle chiavi, perdita del lavoro fatto per il [Web of Trust](https://it.wikipedia.org/wiki/Web_of_trust), perdita delle firme apposte sulla nostra chiave).

Con questa guida vedremo come:
1. **generare** in un luogo abbastanza sicuro il nostro mazzo di chiavi personale;
1. **proteggere** le chiavi private in modo da usarle ogni giorno in un modo abbastanza sicuro.

Generalmente un mazzo di chiavi personale è composto in modo predefinito da:
1. una chiave master o principale per la firma e la certificazione;
1. una sottochiave per la cifratura.

Noi, invece, per elevare la sicurezza del nostro portachiavi spingeremo al massimo il concetto delle sottochiavi. La struttura del nostro portachiavi personale, infatti, sarà il seguente:

1. **Chiave master (o principale) per sola certificazione di chiavi altrui** con dimensione 4096 bit e scadenza dopo 3 anni;
	1. **Sottochiave per firma** con dimensione 2048 bit e scadenza dopo 1 anno;
	1. **Sottochiave per cifratura** con dimensione 2048 bit e scadenza dopo 1 anno;
	1. **Sottochiave per autenticazione** con dimensione 2048 bit e scadenza dopo 1 anno.

Avremo quindi una chiave distinta per ogni operazione.

La chiave più importante è la **chiave master**. Questa chiave è talmente importante che, qualora fosse stata compromessa, ci costringerebbe a revocare l'intero mazzo di chiavi e a generarne di nuove. Per questo motivo la terremo in cassaforte, separata dal resto del mazzo di chiavi. Visto che la terremo separata dal resto, la abiliteremo solo per scopi che non sono quotidiani, ma solo periodiche o per occasioni speciali come un [Key signing party](https://it.wikipedia.org/wiki/Key_signing_party). Lo scopo della chiave master sarà quello di certificare chiavi altrui ed, essendo la chiave principale, di effettuare operazioni di manutenzione del nostro mazzo di chiavi (estensione validità, aggiunta di nomi utente, ecc.).

Le **sottochiavi**, invece, saranno utilizzate quotidianamente. Le sottochiavi hanno la caratteristica che possono esistere nel portachiavi di GnuPG anche senza la presenza della chiave principale. Per questo motivo, dopo aver fatto un sicuro backup, elimineremo dal portachiavi la chiave principale, lasciando solo le sottochiavi. Le sottochiavi, per loro natura, potrebbero anche essere compromesse (cosa che non dobbiamo permettere mai e, a tal scopo, vedremo come proteggerle), ma questo non invaliderebbe l'intero mazzo di chiavi: qualora una sottochiave fosse compromessa, potremmo revocarla e crearne un'altra per lo stesso scopo. Le sottochiavi potrebbero anche scadere e, qualora non volessimo rinnovarle, potremmo generarne di nuove. Oppure potremmo generare una sottochiave per un evento particolare (un convegno o altro), che ha quindi una durata limitata (un mese, un anno) e lasciarla scadere dopo la fine dell'evento.

Per proteggere le nostre sottochiavi, le sposteremo su un token USB come la [Yubikey](https://www.yubico.com/). Si tratta di token praticamente indistruttibili, che rendono l'uso della firma digitale, la cifratura e l'autenticazione su sistemi (come SSH) molto comodi (basterà usare un PIN numerico al posto della passphrase). Io uso la [Yubikey 4](https://www.yubico.com/product/yubikey-4-series/#yubikey-4). I token come YubiKey hanno il pregio che non permettono l'esportazione delle chiavi lì conservate; questo significa che posso usare le mie chiavi anche in un ambiente poco sicuro come un Internet Point, un PC condiviso, ecc., senza il timore che le chiavi private possano essere copiate da qualche parte a mia insaputa. Quand'anche fosse stato intercettato il PIN da un keylogger o da un malware, la passphrase è ancora al sicuro, ed eventualmente posso cambiare il PIN. Il fatto che le sottochiavi, una volta trasferite, non possano essere esportate dalla YubiKey, ci forzerà a effettuare il backup del nostro mazzo di chiavi prima del trasferimento, ma questo aspetto lo vedremo al momento opportuno.

Alla fine della guida avremo quindi la seguente situazione:
1. la chiave principale sarà rimossa dal portachiavi sul disco e trasferita in un luogo sicuro;
1. le sottochiavi risiederanno sulla YubiKey.

Nel nostro disco fisso non avremo quindi nessuna chiave privata. Potremmo anche perdere il nostro PC o potrebbero anche rubarcelo, ma il nostro mazzo di chiavi sarà sempre al sicuro. Potremmo anche perdere la nostra YubiKey, ma essa contiene solo le sottochiavi che, come abbiamo visto, possiamo revocare con la chiave principale (che sta in cassaforte) e ricrearne di nuovi. Quando sarà necessario effettuare un'operazione speciale, come ad esempio apporre la nostra firma su chiavi altrui o aggiungere/eliminare una sottochiave o una UserID, importeremo la chiave principale nel nostro portachiavi, effettueremo l'operazione necessaria e infine la rimuoveremo di nuovo.

Un ultimo punto prima di passare alla pratica. Il motivo di avere una scadenza sulle chiavi è un ulteriore sicurezza per noi. Immaginiamo il caso in cui dovessimo perdere il controllo del nostro mazzo di chiavi, compresa la chiave principale, e non avessimo più a disposizione nemmeno il certificato di revoca. In questo caso la scadenza sarà una garanzia per noi che a un certo punto le chiavi scadranno in modo naturale. Per i tempi di scadenza ognuno può scegliere quello che più ritenga opportuno, ma 3 anni per la principale e 1 anno per le sottochiavi dovrebbe essere un buon compromesso tra la seccatura di dover manutenzionare il portachiavi e il tempo di scadenza naturale dopo eventuale compromissione.

# Generazione sicura di chiavi OpenPGP

Con questa guida genereremo la nostra chiave privata che avrà questo schema:

1. **Chiave master (o principale)** [C - Certification/Certificazione]
	1. **Sottochiave per firma** [S - Signing/Firma]
	1. **Sottochiave per cifratura** [E - Encryption/Cifratura]
	1. **Sottochiave per autenticazione** [A - Authentication/Autenticazione]

Per la lunghezza delle chiavi avremo la chiave primaria a 4096 bit mentre le sottochiavi a 2048 bit. Per ulteriori informazioni su questo dibattito della lunghezza delle chiavi si può vedere:
* [Why does GnuPG default to 2048 bit RSA-2048?](https://www.gnupg.org/faq/gnupg-faq.html#default_rsa2048) sul sito di GnuPG;
* [The Big Debate, 2048 vs. 4096, Yubico’s Position](https://www.yubico.com/2015/02/big-debate-2048-4096-yubicos-stand/) sul blog di Yubico.

## Prerequisiti

1. È necessario lavorare su un sistema avviato con una distribuzione Linux in modalità *live*. È preferibile una [Tails](https://tails.boum.org/index.it.html), ma va bene una qualunque. La distribuzione va avviata isolandola da qualsiasi rete.
1. È necessario usare GnuPG versione 2.1 o successiva.

Qualora il comando `gpg` lanci GnuPG versione 1, accertarsi che la versione 2 sia installata e aggiungere questa riga a `~/.bash_aliases`:

~~~
alias gpg='gpg2'
~~~

Questa scorciatoia permetterà l'avvio di GnuPG versione 2 ogni volta che digitiamo `gpg`.

## Generazione delle chiavi

Sul sistema di generazione delle chiavi accertarsi di avere GnuPG versione 2.1 o successiva:

~~~
gpg --version
~~~

Iniziare quindi la procedura con:

~~~
gpg --expert --full-gen-key
~~~

che darà:

~~~
Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
   (9) ECC and ECC
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
Your selection?
~~~

Scegliamo `8`:

~~~
Possible actions for a RSA key: Sign Certify Encrypt Authenticate
Current allowed actions: Sign Certify Encrypt

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection?
~~~

Come si vede le attuali capacità della chiave che stiamo creando sono: `Sign Certify Encrypt`.
Rimuoviamo le capacità di firma premendo `S`:

~~~
Possible actions for a RSA key: Sign Certify Encrypt Authenticate
Current allowed actions: Certify Encrypt

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection?
~~~

Rimuoviamo le capacità di cifratura premendo `E`:

~~~
Possible actions for a RSA key: Sign Certify Encrypt Authenticate
Current allowed actions: Certify

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection?
~~~

Ora la chiave primaria avrà solo la capacità di certificazione (`Certify`) su altre chiavi.

Premiamo `Q`:

~~~
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048)
~~~

Inserire `4096`:

~~~
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0)
~~~

Inseriamo `3y` (3 anni):

~~~
Key expires at mer 31 mar 2021 19:46:05 CEST
Is this correct? (y/N)
~~~

Premiamo `Y`:

~~~
GnuPG needs to construct a user ID to identify your key.

Real name:
Email address:
Comment:
~~~

Inseriamo il nostro nome e cognome e quindi l'indirizzo email.
Come commento lasciamo il campo vuoto.

~~~
You selected this USER-ID:
    "Mario Rossi <mario.rossi@example.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit?
~~~

Accettare con `O`:

~~~
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
~~~

A seguire GnuPG chiederà di impostare la passfrase.

Impostiamo la passphrase.

~~~
gpg: key 4B6E6777 marked as ultimately trusted
gpg: directory '/home/mariorossi/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/home/mariorossi/.gnupg/openpgp-revocs.d/491532759708014725CFE3A79F676B5A4B6E6777.rev'
public and secret key created and signed.

gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: PGP
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: next trustdb check due at 2021-03-31
pub   rsa4096/4B6E6777 2018-04-01 [C] [expires: 2021-03-31]
      Key fingerprint = 4915 3275 9708 0147 25CF  E3A7 9F67 6B5A 4B6E 6777
uid         [ultimate] Mario Rossi <mario.rossi@example.com>
~~~

GnuPG mostra la chiave appena generata ed esce.

Come si vede, esiste solo la chiave principale con la sola capacità di certificazione `[C]`.
Bisogna ora generare le sottochiavi con:

~~~
gpg --expert --edit-key 0x4B6E6777
~~~

~~~
gpg (GnuPG) 2.1.11; Copyright (C) 2016 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

sec  rsa4096/4B6E6777
     created: 2018-04-01  expires: 2021-03-31  usage: C   
     trust: ultimate      validity: ultimate
[ultimate] (1). Mario Rossi <mario.rossi@example.com>

gpg>
~~~

Dare il comando `addkey`:

~~~
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
Your selection?
~~~

Scegliere `8`:

~~~
Possible actions for a RSA key: Sign Encrypt Authenticate
Current allowed actions: Sign Encrypt

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection?
~~~

Togliere la capacità di cifratura premendo `E`:

~~~
Possible actions for a RSA key: Sign Encrypt Authenticate
Current allowed actions: Sign

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection?
~~~

Scegliere `Q`:

~~~
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048)
~~~

Premere `Invio` per accettare `2048`:

~~~
Requested keysize is 2048 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0)
~~~

Inserire `1y` per farla scadere dopo 1 anno:

~~~
Key expires at lun 01 apr 2019 19:57:16 CEST
Is this correct? (y/N)
~~~

Accettare inserendo `Y`:

~~~
Really create? (y/N)
~~~

Accettare inserendo `Y`.

Inserire quindi la passphrase e la sottochiave sarà aggiunta al portachiavi:

~~~
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
~~~

Attendiamo che finisca, quindi apparirà:

~~~
sec  rsa4096/4B6E6777
     created: 2018-04-01  expires: 2021-03-31  usage: C   
     trust: ultimate      validity: ultimate
ssb  rsa2048/9114F367
     created: 2018-04-01  expires: 2019-04-01  usage: S   
[ultimate] (1). Mario Rossi <mario.rossi@example.com>

gpg>
~~~

Aggiungiamo ora con la stessa procedura (partendo da `addkey`) la chiave di cifratura e poi quella di autenticazione.
Alla fine, dopo aver aggiunto la sottochiave di autenticazione, dare il comando:

~~~
save
~~~

per salvare e uscire.

Alla fine avremo questo portachiavi:

~~~
gpg --list-secret-keys
~~~

~~~
/home/mariorossi/.gnupg/pubring.kbx
-----------------------------
sec   rsa4096/4B6E6777 2018-04-01 [C] [expires: 2021-03-31]
uid         [ultimate] Mario Rossi <mario.rossi@example.com>
ssb   rsa2048/9114F367 2018-04-01 [S] [expires: 2019-04-01]
ssb   rsa2048/2F3AC08A 2018-04-01 [E] [expires: 2019-04-01]
ssb   rsa2048/AFBE0F10 2018-04-01 [A] [expires: 2019-04-01]
~~~

Possiamo aggiungere anche altri UserID o una foto con:

~~~
gpg --expert --edit-key 0x4B6E6777
~~~

e poi con `adduid`.

## Il certificato di revoca

Con GnuPG versione 2 la generazione del certificato di revoca è stata fatta automaticamente quando le chiavi sono state generate e si trova nella directory `~/.gnupg/openpgp-revocs.d/`. Il file ha lo stesso nome dell'impronta della chiave, nel nostro caso:

~~~
491532759708014725CFE3A79F676B5A4B6E6777.rev
~~~

Questo file andrà tolto da questa directory e conservato in un luogo sicuro.

## Impostare le preferenze della chiave

Modificare la chiave con:

~~~
gpg --expert --edit-key 0x4B6E6777
~~~

Dal prompt di Gnupg dare:

~~~
setpref S10 S9 S8 H10 H9 H8 Z2 Z3 Z1 Z0
~~~

che significa:

~~~
S10	Cifrario Twofish
S9	Cifrario AES256
S8	Cifrario AES256
H10	Hash SHA512
H9	Hash SHA384
H8	Hash SHA256
Z2	Compressione ZLIB
Z3	Compressione BZIP2
Z1	Compressione ZIP
Z0	Non compresso
~~~

Alla domanda di GnuPG seguente:

~~~
Set preference list to:
     Cipher: TWOFISH, AES256, AES192, 3DES
     Digest: SHA512, SHA384, SHA256, SHA1
     Compression: ZLIB, BZIP2, ZIP, Uncompressed
     Features: MDC, Keyserver no-modify
Really update the preferences? (y/N)
~~~

premere `Y` e quindi:

~~~
save
~~~

## Impostare il file gpg.conf

Aprire il file `gpg.conf` dentro la directory `~/.gnupg`, eliminare tutto il contenuto e incollarvi questo testo, ricordandosi di **aggiornare le due ID di chiave** nelle righe `default-key` e `encrypt-to` con l'ID della vostra chiave:

~~~
# Per lo skeleton di questo file vedi:
# /usr/share/gnupg2/gpg-conf.skel

#-----------------------------
# default key
#-----------------------------

# The default key to sign with. If this option is not used, the default key is
# the first key found in the secret keyring

default-key 0x9F676B5A4B6E6777
encrypt-to  0x9F676B5A4B6E6777

#-----------------------------
# behavior
#-----------------------------

# Disable inclusion of the version string in ASCII armored output
no-emit-version

# Disable comment string in clear text signatures and ASCII armored messages
no-comments

# Display long key IDs
keyid-format 0xlong

# List all keys (or the specified ones) along with their fingerprints
with-fingerprint

# List all keys (or the specified ones) along with their keygrips
with-keygrip

# Display the calculated validity of user IDs during key listings
list-options show-uid-validity
verify-options show-uid-validity

# Try to use the GnuPG-Agent. With this option, GnuPG first tries to connect to
# the agent before it asks for a passphrase.
use-agent

#-----------------------------
# keyserver
#-----------------------------

# This is the server that --recv-keys, --send-keys, and --search-keys will
# communicate with to receive keys from, send keys to, and search for keys on
# QUESTA OPZIONE È GESTITA IN DIRMNGR.CONF
# keyserver hkps://hkps.pool.sks-keyservers.net

# Set the proxy to use for HTTP and HKP keyservers - default to the standard
# local Tor socks proxy
# It is encouraged to use Tor for improved anonymity. Preferrably use either a
# dedicated SOCKSPort for GnuPG and/or enable IsolateDestPort and
# IsolateDestAddr
#keyserver-options http-proxy=socks5-hostname://127.0.0.1:9050

# When using --refresh-keys, if the key in question has a preferred keyserver
# URL, then disable use of that preferred keyserver to refresh the key from
keyserver-options no-honor-keyserver-url

# When searching for a key with --search-keys, include keys that are marked on
# the keyserver as revoked
keyserver-options include-revoked

#-----------------------------
# algorithm and ciphers
#-----------------------------

# list of personal digest preferences. When multiple digests are supported by
# all recipients, choose the strongest one
personal-cipher-preferences TWOFISH AES256 AES192 3DES

# list of personal digest preferences. When multiple ciphers are supported by
# all recipients, choose the strongest one
personal-digest-preferences SHA512 SHA384 SHA256 SHA224

# message digest algorithm used when signing a key
cert-digest-algo SHA512

# list of personal compression preferences.
personal-compress-preferences ZLIB BZIP2 ZIP Uncompressed

# This preference list is used for new keys and becomes the default for
# "setpref" in the edit menu
default-preference-list SHA512 SHA384 SHA256 SHA224 TWOFISH AES256 AES192 3DES ZLIB BZIP2 ZIP Uncompressed
~~~

## Impostare il keyserver

Modificare (o creare) il file `~/.gnupg/dirmngr.conf`:

~~~
keyserver hkps://hkps.pool.sks-keyservers.net
~~~

## Esportare la chiave pubblica sul keyserver

~~~
gpg --send-keys 0x9F676B5A4B6E6777
~~~

# Trasferire le chiavi OpenPGP su YubiKey

In questa parte vedremo come:
1. separare la chiave principale dal resto del portachiavi;
1. trasferire le sottochiavi nella Yubikey.

TODO.

# Importare la chiave principale per operazioni speciali

TODO.

# Estendere la validità delle nostre chiavi

TODO.

# Bibliografia

* [Creazione di chiavi sicure (ossìa, come generare con GnuPG una coppia di chiavi per poterla conservare in modo "abbastanza" sicuro)](http://tjl73.altervista.org/secure_keygen/).
* [PGP and SSH keys on a Yubikey NEO](https://www.esev.com/blog/post/2015-01-pgp-ssh-key-on-yubikey-neo/).
* [Creating the perfect GPG keypair](https://alexcabal.com/creating-the-perfect-gpg-keypair/).
* [Using GPG and SSH keys (GnuPG 2.1) with a Smartcard (Yubikey 4)](https://suva.sh/posts/gpg-ssh-smartcard-yubikey-keybase/).
* [Guide to using YubiKey as a SmartCard for GPG and SSH](https://github.com/drduh/YubiKey-Guide).
* [OpenPGP Best Practices](https://riseup.net/en/security/message-security/openpgp/best-practices).
