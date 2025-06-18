# OpenSSL-TD

**Exercice 1 – «la clé des champs"**
1. Quelle est la taille par défaut d’une clé RSA ? Comment créer une clé 4096 bits ?

Taille par défaut: 2048 bits.

Clé 4096 bits: openssl genrsa -out server.key 4096

2. Extraire la clé publique de server.key et l’enregistrer dans server.pub.
   openssl rsa -in server.key -pubout -out server.pub

**Exercice 2 – «le cadenas et la clé"**
1. Générer une nouvelle paire de clés RSA protégée avec l’algorithme DES3.

openssl genrsa -des3 -out server_des3.key 2048
2. Extraire la clé publique de cette clé privée. Quelle différence remarque-t-on ?

openssl rsa -in server_des3.key -pubout -out server_des3.pub

Différence: le fichier server_des3.key est chiffré et ne peut être lu sans mot de passe, tandis que la clé publique server_des3.pub reste en clair, identique à celle extraite d’une clé non protégée.

**Exercice 3 – «Je signe en bas?"**
1. Calculer le condensat du fichier pour sha256, md5 et sha1. Déterminer la taille en bits des empreintes MD5 et SHA-1.

openssl dgst -sha256 monfichier.txt    # Empreinte SHA-256 (256 bits)

openssl dgst -md5 monfichier.txt       # Empreinte MD5 (128 bits)

openssl dgst -sha1 monfichier.txt      # Empreinte SHA-1 (160 bits)

2. Signer le fichier avec la clé privée de l’exercice 1 et vérifier la signature
- Signature

openssl dgst -sha256 -sign server.key -out monfichier.sign monfichier.txt

- Vérification

openssl dgst -sha256 -verify server.pub -signature monfichier.sign monfichier.txt

On obtient bien => Verified OK

3. Usurpation et intégrité

- Usurpation: tenter la vérification avec la seconde clé publique:

openssl dgst -sha256 -verify server_des3.pub -signature monfichier.sign monfichier.txt

- Altération: modifier monfichier.txt (p.ex. ajouter un caractère), puis :

openssl dgst -sha256 -verify server.pub -signature monfichier.sign monfichier.txt

**Exercice 4 – JWT**
1. On va générer deux jetons ayant le même payload:

2. Encodage Base64URL de l’en-tête et du payload:

header=$(echo -n '{"alg":"HS256","typ":"JWT"}' | openssl enc -base64 -e -A | tr '+/' '-_' | tr -d '=')

payload=$(echo -n '{"sub":"1234567890","name":"Tyrion Lannister","iat":1704388115}' | openssl enc -base64 -e -A | tr '+/' '-_' | tr -d '=')

3. Signature:

secret="Ma clé secrète"

signature=$(printf "%s.%s" "$header" "$payload" | openssl dgst -sha256 -hmac "$secret" -binary | openssl enc -base64 -e -A | tr '+/' '-_' | tr -d '=')

4. Jeton complet:

echo "$header.$payload.$signature"

**2. JWT RSA + SHA256**

1. Encodage Base64URL (mêmes commandes pour header/payload, en remplaçant algorie par RS256):

header=$(echo -n '{"alg":"RS256","typ":"JWT"}' | openssl enc -base64 -e -A | tr '+/' '-_' | tr -d '=')

2. Signature avec la clé privée:

payload=$(echo -n '{"sub":"1234567890","name":"Tyrion Lannister","iat":1704388115}' | openssl enc -base64 -e -A | tr '+/' '-_' | tr -d '=')

signature=$(printf "%s.%s" "$header" "$payload" | openssl dgst -sha256 -sign server.key | openssl enc -base64 -e -A | tr '+/' '-_' | tr -d '=')

3. Jeton complet :
   echo "$header.$payload.$signature"

