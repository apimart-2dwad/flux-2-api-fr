# API FLUX.2 : identifiants de modèle, tarif par image, exemples

> **0.024 $ par image (1MP)**, facturé à l'usage. Recharge à partir de 1 $ et une seule URL compatible OpenAI : `https://api.apimart.ai/v1`.

**[Page modèle FLUX.2](https://go.apimart.ai/k-bdf494)** · **[Tarifs en direct](https://go.apimart.ai/k-215563)** · **[Obtenir une clé API](https://go.apimart.ai/k-8f584f)**

Facturation au mégapixel : 2,4 cents l'image en 1MP ; pro, max ou flex se changent sans toucher au code.

## Pourquoi appeler FLUX.2 via APIMart

- **Une clé pour tout le catalogue.** La même URL de base et le même en-tête d'authentification donnent accès à FLUX.2 et à plus de 300 modèles image, vidéo et langage : seul le champ `model` change.
- **1 $ minimum, à l'usage.** Ni abonnement ni forfait prépayé, et aucun crédit gratuit à épuiser d'abord : le tarif du tableau est le tarif réel.
- **Le montant est renvoyé dans la réponse.** Chaque appel retourne `cost` / `credits_cost`, donc la dépense se lit appel par appel.
- **Pensé pour l'asynchrone.** Soumission, récupération du `task_id`, puis interrogation de `GET /v1/tasks/{id}` : les lots et les reprises restent de la logique de file classique.

## Identifiant de modèle et endpoint

| Champ | Valeur |
| --- | --- |
| `model` | `flux-2-pro` |
| endpoint | `POST https://api.apimart.ai/v1/images/generations` |
| task | GET /v1/tasks/{id} |

## Tarifs relevés

<!-- pricing:model:start -->
| Sortie | Tarif |
| --- | --- |
| `1MP` | $0.024 |
| `2MP` | $0.036 |
| `3MP` | $0.048 |
<!-- pricing:model:end -->

## Paramètres de requête

| Champ | Valeur |
| --- | --- |
| `model` | `flux-2-pro` |
| `resolution` | `1MP / 2MP / 3MP` |
| `size` | `1:1 / 16:9 / 21:9` |
| `n` | `1-4` |

## Démarrer en 60 secondes

```bash
export APIMART_API_KEY="<token>"
curl --request POST \
  --url https://api.apimart.ai/v1/images/generations \
  --header "Authorization: Bearer $APIMART_API_KEY" \
  --header 'Content-Type: application/json' \
  --data '{"model":"flux-2-pro", "prompt":"a cozy reading nook by a rainy window, warm lamp light", "n":1}'
```

```python
import os, time, requests

BASE = "https://api.apimart.ai/v1"
HEADERS = {"Authorization": f"Bearer {os.environ['APIMART_API_KEY']}", "Content-Type": "application/json"}

r = requests.post(f"{BASE}/images/generations", headers=HEADERS, timeout=60, json={
    "model": "flux-2-pro",
    "prompt": "a cozy reading nook by a rainy window, warm lamp light",
    "n": 1,
})
r.raise_for_status()
task_id = (r.json().get("data") or {}).get("id")
while True:
    t = requests.get(f"{BASE}/tasks/{task_id}", headers=HEADERS, timeout=60).json().get("data", {})
    if t.get("status") in ("completed", "failed"):
        print(t.get("status"), t.get("cost"))
        break
    time.sleep(5)
```

## Coût en volume

| Volume | Coût |
| --- | --- |
| 1,000 | $24 |
| 10,000 | $240 |

Calcul linéaire au tarif indiqué, sans remise de volume. Vérifiez les tarifs en direct avant de budgéter.（snapshot 2026-09-21）

## Dépannage du premier appel

| Symptôme | Cause | Correctif |
| --- | --- | --- |
| `401` / invalid api key | clé absente, tronquée, ou retour à la ligne collé dans l'en-tête | Recopiez-la depuis la console ; l'en-tête est `Authorization: Bearer $APIMART_API_KEY` |
| solde insuffisant / erreur credit | le compte n'a pas de solde | Rechargez à partir de 1 $ dans la console — il n'y a pas de quota gratuit |
| `429` | trop de requêtes simultanées sur une clé | Attendez puis réessayez avec le même `Idempotency-Key` |
| `400` / modèle introuvable | identifiant ou paramètre incorrect | Reprenez la valeur exacte de `model` dans le tableau ci-dessus ; les champs diffèrent selon le palier |
| tâche en `failed` | prompt filtré ou URL d'image de référence expirée | Relancez avec un **nouveau** `Idempotency-Key` et réhébergez l'image de référence |

## Questions fréquentes

**Quelle est l'unité de facturation ?**

L'image, la seconde de vidéo ou le million de tokens selon le modèle. Le montant figure dans la réponse de la tâche, donc il se vérifie ligne par ligne.

**Peut-on consulter le détail ?**

La page de facturation de la console liste chaque appel et la consommation associée.

**Depuis quel langage appeler l'API ?**

N'importe quel client HTTP. Comme l'API est compatible OpenAI, Python fonctionne en changeant simplement la base_url du SDK openai.

**Les URL de résultat expirent-elles ?**

Oui. Téléchargez le fichier dans votre propre stockage dès que la tâche est terminée.

## Divulgation

Ce dépôt décrit l'usage d'APIMart, un service de relais tiers, sans lien avec les fournisseurs de modèles. Les tarifs et paramètres correspondent au relevé indiqué ; la facturation réelle fait foi sur la console.

## Structure du dépôt

```
README.md            本文件
data/model.json      模型 ID、价格快照、参数
examples/curl.sh     curl 示例
examples/python.py   Python（提交 + 轮询）
LICENSE, .gitignore
```

## License

MIT
