# Routine — Code de la routine d'analyse

> Ce dossier contient les modules Python qui font le vrai travail d'analyse.

## Modules à écrire

Voir `../CONTEXT.md` section "CE QUI EST À CONSTRUIRE" pour la liste complète et l'ordre de priorité.

## Convention de nommage

- Chaque module a une responsabilité unique
- Imports relatifs depuis le package `routine.` quand exécuté depuis racine
- Logs structurés avec niveau (DEBUG, INFO, WARNING, ERROR)
- Exceptions custom dans `routine/exceptions.py`

## Pattern de chaque module

```python
"""
routine/<module_name>.py — <description>
"""
import logging
from typing import ...

logger = logging.getLogger(__name__)

def main_function(input: ...) -> ...:
    """Docstring claire."""
    logger.info(f"Starting <module_name> with ...")
    try:
        # logic
        result = ...
        logger.info(f"<module_name> completed: ...")
        return result
    except Exception as e:
        logger.error(f"<module_name> failed: {e}")
        raise


if __name__ == "__main__":
    # CLI standalone pour tester ce module
    import argparse
    parser = argparse.ArgumentParser()
    parser.add_argument("--target-date", required=True)
    args = parser.parse_args()
    
    result = main_function(args.target_date)
    print(json.dumps(result, indent=2))
```

## Test isolé de chaque module

```bash
python -m routine.download_aw_db --target-date 2026-04-27
python -m routine.parse_aw --db /tmp/aw_test.db --target-date 2026-04-27
# etc.
```

## Orchestration finale

```bash
python -m routine.main --target-date 2026-04-27 --debug --dry-run
```
