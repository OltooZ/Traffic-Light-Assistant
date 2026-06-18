# Demo projektu - komendy do pokazania nauczycielowi
### Kolejność: od instalacji → testy → trening → wyniki → API

---

## KROK 1 — Instalacja środowiska
> *"Projekt używa uv do zarządzania zależnościami - jedną komendą instaluje cały stack"*

```bash
uv sync
```
**Co zobaczysz:** pobieranie i instalacja bibliotek (torch, stable-baselines3, pettingzoo itd.)

---

## KROK 2 — Testy automatyczne
> *"Projekt ma 26 testów jednostkowych pokrywających środowisko, agenta i API"*

```bash
uv run pytest tests/ -v
```
**Co zobaczysz:** 26 testów — wszystkie PASSED (zielony wynik)

Jeśli chcesz pokazać konkretny plik testów:
```bash
uv run pytest tests/test_env.py -v     # testy środowiska symulacji
uv run pytest tests/test_agent.py -v   # testy agenta i baseline
uv run pytest tests/test_api.py -v     # testy REST API
```

---

## KROK 3 — Trening agenta PPO
> *"Agent uczy się przez 300 000 kroków symulacji metodą PPO (Proximal Policy Optimization)"*

```bash
uv run python scripts/train.py --no-redis
```
**Co zobaczysz:** log treningu z krokami i rosnącą nagrodą, na końcu:
`Training complete. Model saved to models/ppo_traffic_final.zip`

Jeśli chcesz szybszy demo (100k kroków zamiast 300k):
```bash
uv run python scripts/train.py --timesteps 100000 --no-redis
```

---

## KROK 4 — Porównanie agenta z baseline
> *"Sprawdzamy czy wytrenowany agent jest lepszy od prostego, cyklicznego sterowania"*

```bash
uv run python scripts/evaluate.py --compare-baseline
```
**Co zobaczysz:**
```
PPO Agent: mean_reward=-30.69 +/- 1.71
Baseline:  mean_reward=-66.43 +/- 6.52
PPO improvement over baseline: +35.74
```
Nagroda bliżej 0 = mniejsze korki. PPO jest ~54% lepszy od cyklicznego.

---

## KROK 5 — Generowanie wykresów
> *"Projekt generuje wizualizacje porównujące dynamikę kolejek PPO vs baseline"*

```bash
uv run python scripts/generate_plots.py
```
**Co zobaczysz:** `Plots saved to figures/`
Wykresy zapisują się do folderu `figures/` — można je otworzyć i pokazać.

---

## KROK 6 — REST API
> *"Projekt udostępnia REST API przez FastAPI — można sterować agentem przez HTTP"*

```bash
uv run python scripts/run_api.py
```
**Co zobaczysz:** `Uvicorn running on http://0.0.0.0:8000`

Potem w przeglądarce otwórz: **http://localhost:8000/docs**
→ interaktywna dokumentacja wszystkich endpointów (Swagger UI)

Albo w drugim oknie terminala:
```bash
# sprawdź aktualną konfigurację
curl http://localhost:8000/config

# status treningu
curl http://localhost:8000/train/status
```

---

## KROK 7 (opcjonalny) — Optymalizacja hiperparametrów
> *"Optuna automatycznie szuka najlepszych parametrów treningu"*

```bash
uv run python scripts/hyperopt.py --n-trials 5
```
**Co zobaczysz:** 5 prób z różnymi hiperparametrami, wynik najlepszej próby.
*(5 prób zajmuje ~2 minuty, 50 prób = pełna optymalizacja)*

---

## Skrót — wszystko naraz (jeśli mało czasu)
```bash
uv run pytest tests/ -v --tb=no -q
uv run python scripts/evaluate.py --compare-baseline
uv run python scripts/generate_plots.py
```
Trzy komendy: testy ✓, wyniki ✓, wykresy ✓ — zajmuje ~2 minuty.

---

## Struktura projektu (żeby pokazać że jest dobrze zorganizowany)
```bash
# pokaż strukturę katalogów
tree /F /A 2>nul | findstr /V ".venv" | findstr /V "__pycache__"
```

---

## Przydatne fakty do powiedzenia przy każdym kroku

| Krok | Co powiedzieć |
|------|---------------|
| `uv sync` | "uv to nowoczesny menedżer pakietów Pythona, szybszy od pip" |
| `pytest` | "26 testów: środowisko, agent, API - wszystkie przechodzą" |
| `train.py` | "PPO z biblioteki Stable-Baselines3, środowisko wieloagentowe PettingZoo" |
| `evaluate.py` | "20 epizodów testowych, seed=42 - wyniki powtarzalne" |
| `generate_plots.py` | "wykresy kolejek, nagród i skumulowanej nagrody w czasie" |
| `run_api.py` | "FastAPI z automatyczną dokumentacją Swagger pod /docs" |
| wynik -30 vs -66 | "nagroda ujemna = kara za długość kolejek, bliżej 0 = lepiej" |
