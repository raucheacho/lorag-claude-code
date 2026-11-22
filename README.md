# Lorag pour Claude Code

Plugin qui branche Claude Code sur [Lorag](https://github.com/raucheacho/lorag), la mémoire locale des agents. Une seule installation ajoute :

- **le serveur MCP** `lorag` (`http://localhost:7411/mcp`) : `memory_recall`, `memory_save`, `memory_feedback`, `search`, `get_document`, `ingest`, `get_stats` ;
- **trois hooks** : quand une commande Bash échoue, Lorag cherche un correctif connu et le place à côté de l'erreur, sans que Claude ait à le demander ; quand tu décris un problème, il cherche une fiche qui y répond déjà ; en fin de tour, si un rappel n'a rien trouvé et que rien n'a été enregistré, Claude est invité une fois à enregistrer la solution ;
- **le skill** `/lorag:memory` : quand rappeler, quand enregistrer un correctif, une réponse ou un fait, quand donner un retour, et quand chercher dans tes documents.

## Prérequis

Lorag doit tourner sur la machine :

```sh
brew install raucheacho/tap/lorag
brew services start lorag
ollama pull bge-m3
```

Les hooks utilisent `loragctl hook claude-code` : le rappel sur erreur à partir de Lorag 0.1.4, le rappel d'enregistrement à partir de Lorag 0.1.6, le rappel sur ta demande et l'avertissement « Lorag injoignable » à partir de Lorag 0.1.9. Avec une version plus ancienne, ou sans `loragctl` dans le `PATH`, le hook concerné ne fait rien : le serveur MCP et le skill marchent quand même.

## Installation

Dans Claude Code :

```
/plugin marketplace add raucheacho/lorag-claude-code
/plugin install lorag@lorag
```

Ou depuis un terminal :

```sh
claude plugin marketplace add raucheacho/lorag-claude-code
claude plugin install lorag@lorag
```

## Ce que font les hooks

### Correctif connu sur une erreur

Sur un échec de commande (`PostToolUseFailure`, outil `Bash`), `loragctl hook claude-code` envoie l'erreur à `/api/v1/memory/recall`, avec le nom du dossier courant comme projet. Si la mémoire a un correctif, il arrive dans le contexte de Claude, marqué comme une donnée à vérifier et non un ordre.

Il ne fait rien quand la commande échoue sans message (un `grep` qui ne trouve rien), quand elle a été interrompue ou quand aucun correctif ne correspond avec assurance : une fiche trouvée par signature, ou au-dessus du seuil de similarité de la mémoire. Un rappel automatique arrive sans avoir été demandé, et une fiche voisine qui ne partage que quelques mots coûte plus qu'elle n'apporte. Il ne bloque jamais la commande.

### Une fiche pour ta demande

Beaucoup de problèmes ne passent jamais par une commande qui échoue : « l'app quitte au lancement sur iOS ». Quand tu envoies un message (`UserPromptSubmit`), le hook cherche une fiche qui y répond, avec le même seuil, et la place dans le contexte de Claude. Les messages courts (« ok », « merge ») et les commandes (`/…`) ne déclenchent rien.

### Lorag injoignable

Quand Lorag ne répond pas, ou qu'un autre programme répond à sa place sur son port, le hook le dit une fois par session : à Claude, qui te le signale, et sur stderr. Sans ça, un conflit de port faisait passer la mémoire pour vide sans que personne ne le voie.

### Enregistrer ce qui a été résolu

Claude rappelle la mémoire quand un problème arrive, mais oublie souvent d'enregistrer la solution une fois trouvée. En fin de tour (`Stop`), `loragctl hook claude-code` lit le transcript : si un `memory_recall` de ce tour n'a rien trouvé et qu'aucun `memory_save` n'a suivi, il demande une fois à Claude d'enregistrer la solution s'il en a une vérifiée, sinon de terminer sans rien dire.

Il ne demande jamais deux fois de suite, et ne fait rien dans tous les autres tours.

## Autre port

Le plugin suppose le port 7411. Pour un autre port, ajoute le serveur à la main et désactive celui du plugin :

```sh
claude mcp add --transport http --scope user lorag http://localhost:7412/mcp
export LORAG_PORT=7412   # pour loragctl et le hook
```

## Confidentialité

Le plugin ne parle qu'à Lorag sur `localhost`. Rien ne quitte la machine : la mémoire, l'index et les embeddings (Ollama) restent locaux.

## Licence

MIT
