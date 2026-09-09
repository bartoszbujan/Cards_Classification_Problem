# Rozpoznawanie kart do gry — system Computer Vision / AI

Projekt inżynierski realizujący temat **„Zastosowanie narzędzi sztucznej inteligencji do
klasyfikacji kart do gry"** (Polsko-Japońska Akademia Technik Komputerowych).

System wykrywa karty do gry na obrazie z pliku lub z kamery, wycina je, prostuje
perspektywicznie i klasyfikuje do jednej z 52 klas, podając skalibrowaną miarę pewności.
Rozpoznania niepewne są oznaczane, a nie zgadywane.

> **Status: Faza 0 — plan zaakceptowany do realizacji.**
> Kod źródłowy jeszcze nie istnieje. Poniższa sekcja „Uruchomienie" opisuje stan docelowy
> i będzie uzupełniana wraz z postępem prac.

---

## Architektura

Pipeline dwuetapowy: detektor odpowiada wyłącznie na pytanie *gdzie jest karta*,
klasyfikator na pytanie *jaka to karta*.

```
obraz / kamera
   → preprocessing
   → detekcja kart (YOLO, 1 klasa | detektor konturowy)
   → wycięcie
   → korekcja perspektywy (opcjonalna)
   → klasyfikacja (52 klasy)
   → kalibracja pewności i decyzja accept / reject
   → agregacja wyników
   → GUI / CLI / JSON
```

Detektor, klasyfikator i korekcja perspektywy są wymienne z poziomu pliku konfiguracyjnego.
Dzięki temu przeprowadzenie eksperymentu porównawczego nie wymaga zmian w kodzie.

Uzasadnienie wyboru tej architektury (wraz z porównaniem dwóch odrzuconych alternatyw)
znajduje się w [PROJECT_PLAN.md, sekcja 3](docs/PROJECT_PLAN.md#3-analiza-i-wybór-architektury).

---

## Stos technologiczny

| Obszar | Wybór |
|---|---|
| Język | Python 3.12 |
| Uczenie głębokie | PyTorch |
| Detekcja | Ultralytics YOLO (wariant `n`) — licencja AGPL-3.0 |
| Klasyczne CV | OpenCV |
| Augmentacja | Albumentations |
| Metryki | scikit-learn |
| Wykresy | Matplotlib |
| GUI | PySide6 (Qt, LGPL) |
| Testy | pytest |

Każdy wybór jest uzasadniony porównaniem z alternatywami w
[PROJECT_PLAN.md, sekcja 4](docs/PROJECT_PLAN.md#4-technologie).

---

## Struktura repozytorium

```
├── README.md            ten plik
├── AGENTS.md            zasady współpracy przy projekcie
├── configs/             wszystkie parametry systemu (YAML)
├── data/                dane — poza gitem, oprócz splits/ i metadata/
├── docs/                dokumentacja i tekst pracy inżynierskiej
├── experiments/         wyniki przebiegów: config + metryki + wykresy
├── models/              wagi modeli (poza gitem)
├── scripts/             punkty wejścia CLI
├── src/cardvision/      kod źródłowy (pakiet instalowalny)
└── tests/               testy jednostkowe i integracyjne
```

Pełny opis wraz z uzasadnieniem: [PROJECT_PLAN.md, sekcja 13](docs/PROJECT_PLAN.md#13-struktura-repozytorium).

---

## Uruchomienie

> Sekcja docelowa — komendy zaczną działać po Fazie 6.

```bash
# instalacja
py -3.12 -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
pip install -e .

# weryfikacja środowiska
python -c "import torch; print(torch.__version__, torch.cuda.is_available())"

# rozpoznanie kart na zdjęciu
python scripts/predict.py --input obraz.jpg --config configs/pipeline/default.yaml

# aplikacja GUI
python scripts/run_app.py

# uruchomienie eksperymentu
python scripts/run_experiment.py --config configs/experiments/e01_architectures.yaml

# testy
pytest
```

---

## Dokumentacja

| Plik | Zawartość |
|---|---|
| [PROJECT_PLAN.md](docs/PROJECT_PLAN.md) | pełny plan projektu: architektura, dane, eksperymenty, 15 faz z zadaniami |
| `docs/DATASET.md` | raport z audytu danych — powstaje w Fazie 2 |
| `docs/EXPERIMENTS.md` | rejestr przeprowadzonych eksperymentów i wniosków |
| `docs/JOURNAL.md` | dziennik prac — co zrobiono, co nie działa, następny krok |
| `docs/THESIS_OUTLINE.md` | mapowanie wyników projektu na rozdziały pracy |
| `docs/Zastosowanie narzędzi...docx` | tekst pracy inżynierskiej |

---

## Część eksperymentalna

Projekt jest zbudowany wokół dwunastu eksperymentów porównawczych, a nie wokół jednego
wytrenowanego modelu. Każdy ma postawioną hipotezę, jedną zmienną niezależną i wspólny
zbiór testowy; eksperymenty o spodziewanych małych różnicach powtarzane są z trzema
ziarnami losowymi.

| | Eksperyment |
|---|---|
| E1 | porównanie architektur klasyfikatora |
| E2 | transfer learning kontra uczenie od zera |
| E3 | 52 klasy kontra figura + kolor kontra multi-head |
| E4 | wpływ augmentacji (w tym weryfikacja szkodliwości odbicia lustrzanego) |
| E5 | wpływ korekcji perspektywy |
| E6 | architektura jednoetapowa kontra dwuetapowa |
| E7 | detektor klasyczny kontra uczony |
| E8 | wpływ rozdzielczości wejściowej |
| E9 | kalibracja pewności i wyznaczenie progu |
| E10 | odporność: oświetlenie, kąt, liczba kart i przesłonięcia |
| E11 | domain gap — dane publiczne kontra własne |
| E12 | dane syntetyczne kontra rzeczywiste |

Szczegóły: [PROJECT_PLAN.md, sekcja 11](docs/PROJECT_PLAN.md#11-eksperymenty).

---

## Zasady projektu

1. **Żadnych wymyślonych wyników.** Każda liczba w pracy ma pokrycie w zapisanym katalogu
   eksperymentu. Wartości nieznane są oznaczane jako `[DO ZMIERZENIA]`.
2. **Zbiór testowy używany raz**, na końcu — nigdy do strojenia parametrów.
3. **Podziały danych rozłączne na poziomie sesji nagraniowej**, nie pojedynczej klatki —
   inaczej wyniki są zawyżone przez przeciek informacji.
4. **Zero magicznych liczb w kodzie** — wszystkie parametry pochodzą z plików konfiguracyjnych.
5. **Wyniki negatywne są pełnoprawnymi wynikami** i trafiają do pracy tak samo jak pozytywne.

---

## Autor

Bartosz Bujanowicz — praca inżynierska, PJATK.
