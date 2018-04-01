**GUIDA IN FASE DI SCRITTURA**
# Introduzione

Con questa guida vedremo come:
1. generare in un luogo abbastanza sicuro il nostro mazzo di chiavi personale;
1. proteggere le chiavi private in modo da usarle ogni giorno in un modo abbastanza sicuro.

La struttura del nostro portachiavi personale sarà il seguente:

1. **Chiave master (o principale) per sola certificazione di chiavi altrui** con dimensione 4096 bit e scadenza dopo 3 anni;
	1. **Sottochiave per firma** con dimensione 2048 bit e scadenza dopo 1 anno;
	1. **Sottochiave per cifratura** con dimensione 2048 bit e scadenza dopo 1 anno;
	1. **Sottochiave per autenticazione** con dimensione 2048 bit e scadenza dopo 1 anno.

La chiave più importante è la chiave master che, nel nostro caso, limiteremo all'uso esclusivo di certificazione di chiavi altrui e per effettuare operazioni sul nostro portachiavi. Per questo motivo la useremo raramente o comunque poco durante l'anno.

Le sottochiavi, invece, saranno utilizzate quotidianamente. Queste sottochiavi possono esistere nel portachiavi di GnuPG anche senza la presenza della chiave principale. Per questo motivo, dopo aver fatto un sicuro backup, elimineremo dal portachiavi la chiave principale, lasciando solo le sottochiavi.

Per maggiore sicurezza sposteremo le sottochiavi su un token USB come la [Yubikey](https://www.yubico.com/). Si tratta di token praticamente indistruttibili, che rendono l'uso della firma digitale, la cifratura e l'autenticazione su sistemi (come SSH) molto comodi (basterà usare un PIN numerico). Io uso la [Yubikey 4](https://www.yubico.com/product/yubikey-4-series/#yubikey-4).

Alla fine della guida avremo quindi la seguente situazione:
1. la chiave principale sarà rimossa dal portachiavi sul disco e trasferita in un luogo sicuro;
1. le sottochiavi risiederanno sulla YubiKey.

Nel nostro disco fisso non avremo quindi nessuna chiave privata. Quando sarà necessario effettuare un'operazione speciale, come ad esempio apporre la nostra firma su chiavi altrui o aggiungere/eliminare una sottochiave o una UserID, importeremo la chiave principale nel nostro portachiavi, effettueremo l'operazione necessaria e infine la rimuoveremo di nuovo.

# Generazione sicura di chiavi OpenPGP

Con questa guida genereremo la nostra chiave privata che avrà questo schema:

1. **Chiave master (o principale)** [C - Certificazione]
	1. **Sottochiave per firma** [S - Sign/Firma]
	1. **Sottochiave per cifratura** [E - Encrypt/Cifratura]
	1. **Sottochiave per autenticazione** [A - Authentication/Autenticazione]

Per la lunghezza delle chiavi avremo la chiave primaria a 4096 bit mentre le sottochiavi a 2048 bit. Per ulteriori informazioni su questo dibattito vedere la [FAQ dedicata](https://www.gnupg.org/faq/gnupg-faq.html#default_rsa2048) sul sito di GnuPG.

## Prerequisiti

1. È necessario lavorare su un sistema avviato con una distribuzione Linux in modalità live. È preferibile una [Tails](https://tails.boum.org/index.it.html), ma va bene una qualunque. La distribuzione va avviata isolandola da qualsiasi rete.
1. È necessario usare GnuPG versione 2 o successiva.

Qualora il comando `gpg` faccia partire GnuPG versione 1, accertarsi che la versione 2 sia installata e aggiungere questa riga a ''~/.bash_aliases'':

~~~
alias gpg='gpg2'
~~~

## Generazione delle chiavi

Sul sistema di generazione delle chiavi accertarsi di avere GnuPG v2 o successiva:

~~~
gpg --version
~~~

Iniziare quindi la procedura con:

~~~
gpg --expert --full-gen-key
~~~

che darà:

~~~
Per favore scegli che tipo di chiave vuoi:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (firma solo)
   (4) RSA (firma solo)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
   (9) ECC and ECC
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (13) Existing key
~~~

Scegliamo `8`:

~~~
Possible actions for a RSA key: Sign Certify Encrypt Authenticate
Current allowed actions: Sign Certify Encrypt

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished
~~~

Rimuoviamo le capacità di firma con `S`:

~~~
Possible actions for a RSA key: Sign Certify Encrypt Authenticate
Current allowed actions: Certify Encrypt

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished
~~~

Rimuoviamo le capacità di cifratura con `E`:

~~~
Possible actions for a RSA key: Sign Certify Encrypt Authenticate
Current allowed actions: Certify

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished
~~~

Ora la chiave primaria avrà solo la capacità di certificazione su altre chiavi.

Premiamo `Q`:

~~~
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048)
~~~

Inserire `4096`:

~~~
La dimensione richiesta della chiave è 4096 bit
Per favore specifica per quanto tempo la chiave sarà valida.
         0 = la chiave non scadrà
      <n>  = la chiave scadrà dopo n giorni
      <n>w = la chiave scadrà dopo n settimane
      <n>m = la chiave scadrà dopo n mesi
      <n>y = la chiave scadrà dopo n anni
Chiave valida per? (0)
~~~

Inseriamo `3y`:

~~~
Key expires at dom 27 dic 2020 11:54:08 CET
Is this correct? (y/N)
~~~

Inseriamo `y`:

~~~
GnuPG needs to construct a user ID to identify your key.
Nome e Cognome:
Indirizzo di Email:
Commento:
~~~


Inseriamo nome e cognome e indirizzo email.
Come commento lasciamo il campo vuoto.

~~~
Hai selezionato questo User Id:
    "Mario Rossi <mario.rossi@example.com>"

Modifica (N)ome, (C)ommento, (E)mail oppure (O)kay/(Q)uit?
~~~


Accettare con `O`:

~~~
Dobbiamo generare un mucchio di byte casuali. È una buona idea eseguire
qualche altra azione (scrivere sulla tastiera, muovere il mouse, usare i
dischi) durante la generazione dei numeri primi; questo da al generatore di
numeri casuali migliori possibilità di raccogliere abbastanza entropia.
~~~

A seguire GnuPG chiederà di impostare la passfrase.

Impostiamo la passphrase.

~~~
gpg: key 05F0303DA9F107EC marked as ultimately trusted
gpg: revocation certificate stored as '/home/aldo/.gnupg/openpgp-revocs.d/D4E03D8613D611D10D36B05205F0303DA9F107EC.rev'
chiavi pubbliche e segrete create e firmate.

pub   rsa4096 2017-12-27 [C] [expires: 2020-12-27]
      D4E03D8613D611D10D36B05205F0303DA9F107EC
uid                      Mario Rossi <mario.rossi@example.com>
~~~


GnuPG mostra la chiave appena generata e si chiude.

Come si vede, esiste solo la chiave principale con la sola capacità di certificazione `[C]`.
Bisogna ora generare le sottochiavi con:

~~~
gpg --expert --edit-key 0x05F0303DA9F107EC
~~~

~~~
gpg (GnuPG) 2.2.3; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

È disponibile una chiave segreta.

sec  rsa4096/05F0303DA9F107EC
     created: 2017-12-27  expires: 2020-12-27  usage: C
     trust: ultimate      validity: ultimate
[ultimate] (1). Mario Rossi <mario.rossi@example.com>
~~~


Dare `addkey`:

~~~
Per favore scegli che tipo di chiave vuoi:
   (3) DSA (firma solo)
   (4) RSA (firma solo)
   (5) Elgamal (encrypt only)
   (6) RSA (cifra solo)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
Cosa scegli?
~~~


Scegliere `8`:

~~~
Possible actions for a RSA key: Sign Encrypt Authenticate
Current allowed actions: Sign Encrypt

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Cosa scegli?
~~~


Togliere la capacità di cifratura con `E`:

~~~
Possible actions for a RSA key: Sign Encrypt Authenticate
Current allowed actions: Sign

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Cosa scegli?
~~~


Scegliere `Q`:

~~~
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048)
~~~


Premere `Invio` per accettare `2048`:

~~~
La dimensione richiesta della chiave è 2048 bit
Per favore specifica per quanto tempo la chiave sarà valida.
         0 = la chiave non scadrà
      <n>  = la chiave scadrà dopo n giorni
      <n>w = la chiave scadrà dopo n settimane
      <n>m = la chiave scadrà dopo n mesi
      <n>y = la chiave scadrà dopo n anni
Chiave valida per? (0)
~~~


Inserire `1y` per farla scadere dopo 1 anno:

~~~
Key expires at gio 27 dic 2018 12:13:02 CET
Is this correct? (y/N)
~~~


Accettare inserendo `Y`:

~~~
Really create? (y/N)
~~~

Accettare inserendo `Y`.

Inserire quindi la passphrase e la sottochiave sarà aggiunta al portachiavi:

~~~
Dobbiamo generare un mucchio di byte casuali. È una buona idea eseguire
qualche altra azione (scrivere sulla tastiera, muovere il mouse, usare i
dischi) durante la generazione dei numeri primi; questo da al generatore di
numeri casuali migliori possibilità di raccogliere abbastanza entropia.

sec  rsa4096/05F0303DA9F107EC
     created: 2017-12-27  expires: 2020-12-27  usage: C
     trust: ultimate      validity: ultimate
ssb  rsa2048/99A0D3F9AFDC63F8
     created: 2017-12-27  expires: 2018-12-27  usage: S
[ultimate] (1). Mario Rossi <mario.rossi@example.com>
~~~


Aggiungiamo ora con la stessa procedura la chiave di cifratura e poi quella di autenticazione.
Dopo aver aggiunto la sottochiave di autenticazione dare:

~~~
save
~~~

per salvare e uscire.

Alla fine avremo questo portachiavi:

~~~
gpg --list-secret-keys
~~~

~~~
sec   rsa4096 2017-12-27 [C] [expires: 2020-12-27]
      D4E03D8613D611D10D36B05205F0303DA9F107EC
uid           [ultimate] Mario Rossi <mario.rossi@example.com>
ssb   rsa2048 2017-12-27 [S] [expires: 2018-12-27]
ssb   rsa2048 2017-12-27 [E] [expires: 2018-12-27]
ssb   rsa2048 2017-12-27 [A] [expires: 2018-12-27]
~~~


Possiamo aggiungere anche altri UserID o una foto con

~~~
gpg --expert --edit-key 0x05F0303DA9F107EC
~~~

e poi con `adduid`.

## Il certificato di revoca

Con GnuPG versione 2 la generazione del certificato di revoca è stata fatta automaticamente quando le chiavi sono state generate e si trova nella directory `~/.gnupg/openpgp-revocs.d/`. Il file ha lo stesso nome dell'impronta della chiave, nel nostro caso:

~~~
D4E03D8613D611D10D36B05205F0303DA9F107EC.rev
~~~

## Impostare le preferenze della chiave

Modificare la chiave con:

~~~
gpg --expert --edit-key 0x05F0303DA9F107EC
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
~~~

~~~
H10	Hash SHA512
H9	Hash SHA384
H8	Hash SHA256
Z2	Compressione ZLIB
Z3	Compressione BZIP2
Z1	Compressione ZIP
Z0	Non compresso
~~~

e quindi:

~~~
save
~~~

## Impostare il file gpg.conf

~~~
# Per lo skeleton di questo file vedi:
# /usr/share/gnupg2/gpg-conf.skel

#-----------------------------
# default key
#-----------------------------

# The default key to sign with. If this option is not used, the default key is
# the first key found in the secret keyring

default-key 0x05F0303DA9F107EC
encrypt-to  0x05F0303DA9F107EC

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

Modificare (o creare) il file ~/.gnupg/dirmngr.conf

~~~
# Per lo skeleton di questo file vedi:
# /usr/share/gnupg2/dirmngr-conf.skel

keyserver hkps://hkps.pool.sks-keyservers.net
~~~

## Esportare la chiave pubblica sul keyserver

~~~
gpg --send-keys 0x05F0303DA9F107EC
~~~

# Trasferire le chiavi OpenPGP su YubiKey

In questa parte vedremo come:
1. separare la chiave principale dal resto del portachiavi;
1. trasferire le sottochiavi nella Yubikey.

TODO.
