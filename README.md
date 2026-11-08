# Lorag pour Claude Code

Plugin qui branche Claude Code sur [Lorag](https://github.com/raucheacho/lorag), la mémoire locale des agents. Une seule installation ajoute :

- **le serveur MCP** `lorag` (`http://localhost:8080/mcp`) : `memory_recall`, `memory_save`, `memory_feedback`, `search`, `get_document`, `ingest`, `get_stats` ;
- **un hook** : quand une commande Bash échoue, Lorag cherche un correctif connu et le place à côté de l'erreur, sans que Claude ait à le demander ;
- **le skill** `/lorag:memory` : quand rappeler, quand enregistrer un correctif, une réponse ou un fait, quand donner un retour, et quand chercher dans tes documents.

## Prérequis

Lorag doit tourner sur la machine :

```sh
brew install raucheacho/tap/lorag
brew services start lorag
ollama pull bge-m3
```

Le hook utilise `loragctl hook claude-code`, présent à partir de Lorag 0.1.4. Avec une version plus ancienne, ou sans `loragctl` dans le `PATH`, le hook ne fait rien : le serveur MCP et le skill marchent quand même.

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

## Ce que fait le hook

Sur un échec de commande (`PostToolUseFailure`, outil `Bash`), `loragctl hook claude-code` envoie l'erreur à `/api/v1/memory/recall`, avec le nom du dossier courant comme projet. Si la mémoire a un correctif, il arrive dans le contexte de Claude, marqué comme une donnée à vérifier et non un ordre.

Il ne fait rien quand Lorag est arrêté, quand la commande échoue sans message (un `grep` qui ne trouve rien), quand elle a été interrompue ou quand aucun correctif ne correspond. Il ne bloque jamais la commande.

## Autre port

Le plugin suppose le port 8080. Pour un autre port, ajoute le serveur à la main et désactive celui du plugin :

```sh
claude mcp add --transport http --scope user lorag http://localhost:9090/mcp
export LORAG_PORT=9090   # pour loragctl et le hook
```

## Confidentialité

Le plugin ne parle qu'à Lorag sur `localhost`. Rien ne quitte la machine : la mémoire, l'index et les embeddings (Ollama) restent locaux.

## Licence

MIT
