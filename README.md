# Bitnami's Sealed Secrets Controller

## How Sealed Secrets Work
![Sealed Secrets Flow](https://github.com/Ahmed-wa7eed/Bitnami-s-sealed-secrets-controller/blob/main/how%20Sealed%20Secrets%20work.jpg?raw=true)

🔐 Inside the Kubernetes Cluster:
1. ✅ The Sealed Secrets Controller runs in the `kube-system` namespace.
2. It has a key pair (public and private):
   - 🟢 The public key is shared so developers can use it.
   - 🔴 The private key stays secure inside the controller and is used to decrypt.

🛠️ Outside the cluster (Developer's side):
1. 📝 You create a normal Secret YAML (like passwords or tokens).
2. 🛠️ You use the `kubeseal` CLI tool and the public key.
3. 🔐 `kubeseal` reads your Secret YAML and converts it into a SealedSecret YAML (which is encrypted).
4. 🧷 You can store this SealedSecret YAML in Git safely.

🔁 When applying the SealedSecret to the cluster:
1. 📦 You apply the SealedSecret YAML to the cluster using `kubectl apply`.
2. The Sealed Secrets Controller sees the new SealedSecret.
3. It uses the private key to decrypt it and creates a regular Kubernetes Secret in the specified namespace (like `my-namespace`).
4. Your app can now use the Secret as usual.

## Problem of Key-Rotation:
⚠️ The controller creates new keys every 30 days to keep things secure. Secrets that were sealed with old keys will still work, but they won't be updated automatically. To use the new key, you have to re-seal the secret manually — **the goal is to make this automatic to save time and effort**.

## Plan for the Automatic Re-Sealing Feature in `kubeseal`

Add a subcommand called `kubeseal reseal`, developed using **Go language**, and will add some flags to it.

| Flag               | Description                                           |
|--------------------|-------------------------------------------------------|
| `--namespace`       | Re-seals SealedSecrets in a specific namespace.       |
| `--all-namespaces`  | Re-seals SealedSecrets in all namespaces.             |
| `--output-report`   | Saves a report showing what happened (successes, errors). |
| `--dry-run`         | Shows what would happen without making any changes.   |

Bring all public keys using this command:

`
kubeseal --fetch-cert --controller-name=sealed-secrets --controller-namespace=kube-system
`


after we Bring all public keys, now we will **decryption all sealed secrets**

## Decrypt all SealedSecrets
Since kubeseal doesn't support direct decryption (because the private keys are stored inside the controller) , we will createa job to do this task
After decryption, **re-encryption is done using the new public key**.

## re-encryption using new public key

`
kubeseal --cert my-latest-cert.pem -o yaml < decrypted-secret.yaml > resealed-secret.yaml
`
## update resources
to update sealed secret resource you use command `kubectl apply -f resealed-secret.yaml`

**Note the file name not default this file you create it**
## Bonus
to see logging and mechanism we use this flag `--output-report`

to handle large number we can use this flag ` --concurrency=N`

to ensure security of private key we must ensure:
- private key don't leave cluster controll
- decryption must done inside the cluster
- use RBAC

## References 
https://github.com/bitnami-labs/sealed-secrets

https://youtu.be/ShGHCpUMdOg

https://www.middlewareinventory.com/blog/kubernetes-cronjob-best-practices/







