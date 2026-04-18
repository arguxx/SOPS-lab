# Generate Key (age)

```bash
age-keygen -o key.txt
```

Cek isi:

```bash
cat key.txt
```

Output:

```
# public key:
age1xxxxxxx

# private key:
AGE-SECRET-KEY-1...
```

- Public key → untuk encrypt
- Private key → untuk decrypt (JANGAN DISHARE)

---

# Encrypt decrypt Manual age

```bash
echo "DB_PASSWORD=supersecret" | age -r age1xxxxxxx
echo "DB_PASSWORD=supersecret" | age -r age1xxxxxxx > db_encyrpted.env

# decrypt
age -d -i key.txt db_encyrpted.env
age -d -i key.txt -o secret.txt db_encyrpted.env

```

# gabungin dengan SOPS

Buat file:

```bash
cat <<EOF > .env
DB_USER=admin
DB_PASSWORD=supersecret
API_KEY=12345
EOF
```

# Encrypt .env dengan SOPS

```yaml
sops --encrypt \
  --age age1xxxxxxx \
  .env
```

Hasil:

```
DB_USER=ENC[...]
DB_PASSWORD=ENC[...]
API_KEY=ENC[...]
```

---

# Decrypt .env

```bash
sops --encrypt \
  --age age1xxxxxxx \
  .env > .env.ec
  
SOPS_AGE_KEY=AGE-SECRET-KEY-xxxx sops decrypt  .env.ec

SOPS_AGE_KEY=AGE-SECRET-KEY-xxxx sops decrypt --input-type dotenv  .enc.ec

SOPS_AGE_KEY=AGE-SECRET-KEY-xxxx sops decrypt --input-type dotenv  --output-type dotenv .enc.ec 

SOPS_AGE_KEY=AGE-SECRET-KEY-xxxx sops -i decrypt .encyrpted.env

SOPS_AGE_KEY=AGE-SECRET-KEY-xxxx sops -d .encyrpted.env

export SOPS_AGE_KEY=AGE-SECRET-KEY-xxxx

sops -d .encyrpted.env 
```

```bash
sops --decrypt .env.enc
```

---

# Encrypt with Regex

```bash
sops --encrypt \
  --age age1xxxxxxx \
  --encrypted-regex 'PASSWORD|KEY' \
  .env > .env.enc
```

Hasil:

```
DB_USER=admin
DB_PASSWORD=ENC[...]
API_KEY=ENC[...]
```

---

# work with fluxCD

buat secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name:  secret-demo
stringData:
  .env: |-
    DATABASE_USER="user_database"
    DATABASE_HOST="host.database.com"
```

```yaml
sops -e --age agexxx \
clusters/my-cluster/coba-nginx/secret.yaml > \
clusters/my-cluster/coba-nginx/secret.enc.yaml
```

commit push flux error

```yaml
# kind gak akan kebaca, pake regex dulu
sops -e --age agexxx \
--encrypted-regex '^(data|stringData)' \
clusters/my-cluster/coba-nginx/secret.yaml > \
clusters/my-cluster/coba-nginx/secret.enc.yaml

apply secret priv key

masukin di kustomization

  decryption:
    provider: sops
    secretRef:
      name: sops-age

reconcile
```

cek file .env ada apa engga

```yaml
k exec -it nginxxxxx -- sh
```


bikin creation files (optional)

```yaml
creation_rules:
  - path_regex: .*.yaml
    encrypted_regex: '^(data|stringData)$'
    age: >
      agexxxx

```