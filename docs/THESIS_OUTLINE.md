# THESIS_OUTLINE.md — struktura pracy inżynierskiej

**Temat:** „Zastosowanie narzędzi sztucznej inteligencji do klasyfikacji kart do gry"
**Wersja:** 0.1 (2026-10-06)

> **Status: wersja robocza — do dopracowania.**
> Struktura została wstępnie przyjęta, ale tytuły rozdziałów i podrozdziałów, ich kolejność
> oraz zakres mogą się jeszcze zmienić — w szczególności po uzgodnieniu wymagań promotora
> i po uzyskaniu wyników eksperymentów. Szkielet w dokumencie pracy (`.docx`) nie został
> jeszcze dostosowany do tej wersji.

Struktura rozwija propozycję z `PROJECT_PLAN.md` (sekcja 19.2). Najważniejsze różnice:
dodany podrozdział o kalibracji pewności (2.9), podrozdział o klasycznym przetwarzaniu
obrazu (2.2), podrozdziały w rozdziałach 3–8 powiązane z eksperymentami oraz streszczenie
i spisy.

---

## Struktura

**Streszczenie / Abstract** (+ słowa kluczowe)
Problem, zastosowane podejście, najważniejsze wyniki i wnioski. Pisane na końcu.

### 1. Wstęp
Motywacja, problem, cel i zakres pracy, opis struktury pracy.

### 2. Podstawy teoretyczne
Teoria, na której opiera się projekt — w ujęciu ogólnym, bez odniesienia do kart.

- 2.1 Uczenie maszynowe i sieci neuronowe
- 2.2 Klasyczne przetwarzanie obrazu
- 2.3 Sieci konwolucyjne
- 2.4 Klasyfikacja obrazów
- 2.5 Detekcja obiektów i YOLO
- 2.6 Transfer learning
- 2.7 Augmentacja danych
- 2.8 Miary jakości modeli
- 2.9 Pewność predykcji i kalibracja

### 3. Problem rozpoznawania kart
Na czym polega trudność rozpoznawania kart i jak problem rozwiązywano dotychczas.

- 3.1 Specyfika kart do gry
- 3.2 Przegląd istniejących rozwiązań
- 3.3 Metody klasyczne a uczenie maszynowe

### 4. Projekt systemu
Wymagania wobec systemu i uzasadniony wybór architektury.

- 4.1 Wymagania
- 4.2 Wybór architektury
- 4.3 Przepływ przetwarzania obrazu
- 4.4 Dobór technologii

### 5. Dane
Źródła danych, kontrola ich jakości i podział zapobiegający zawyżeniu wyników.

- 5.1 Źródła danych
- 5.2 Audyt zbiorów publicznych
- 5.3 Dane syntetyczne
- 5.4 Własny zbiór testowy
- 5.5 Podział danych
- 5.6 Augmentacja dla kart do gry

### 6. Implementacja
Budowa poszczególnych modułów systemu.

- 6.1 Detekcja kart
- 6.2 Korekcja perspektywy
- 6.3 Klasyfikatory i ich uczenie
- 6.4 Kalibracja i odrzucanie niepewnych wyników
- 6.5 Aplikacja i tryb kamery
- 6.6 Testy i odtwarzalność

### 7. Badania eksperymentalne
Metodyka badań i wyniki pomiarów.

- 7.1 Metodyka badań
- 7.2 Architektura klasyfikatora (E1, E2, E8)
- 7.3 52 klasy a figura i kolor (E3)
- 7.4 Augmentacja i korekcja perspektywy (E4, E5)
- 7.5 Architektura systemu i detekcja (E6, E7, E12)
- 7.6 Próg pewności (E9)
- 7.7 Skuteczność całego systemu

### 8. Analiza wyników
Interpretacja wyników z rozdziału 7.

- 8.1 Analiza pomyłek
- 8.2 Odporność systemu (E10)
- 8.3 Dane publiczne a rzeczywiste (E11)

### 9. Ograniczenia i kierunki rozwoju
Granice działania systemu wynikające z pomiarów oraz możliwe kierunki dalszych prac.

### 10. Podsumowanie
Stopień realizacji celów, najważniejsze wnioski, wkład własny.

**Bibliografia · Spis rysunków · Spis tabel · Załączniki**
(m.in. pełna macierz pomyłek 52×52, protokół testów praktycznych)

---

## Źródła materiału i możliwość napisania przed implementacją

| Rozdział | Źródło w projekcie | Do napisania przed implementacją |
|---|---|---|
| Streszczenie | całość | nie — na końcu |
| 1. Wstęp | `PROJECT_PLAN.md` sekcje 1–2 | tak — wersja robocza, uzupełniana po wynikach |
| 2. Podstawy teoretyczne | literatura; plan sekcje 6, 8–10 | tak — w całości |
| 3. Problem rozpoznawania kart | literatura; plan sekcje 1.2–1.3 | tak — w całości |
| 4. Projekt systemu | plan sekcje 3, 4, 12, 13 | w większości; potwierdzenie wyboru architektury po E6 |
| 5. Dane | plan sekcje 5–6; `DATASET.md` | częściowo (5.1, 5.3 koncepcja, 5.5, 5.6); 5.2 i 5.4 po Fazach 2 i 7 |
| 6. Implementacja | kod źródłowy; plan sekcje 7, 9, 12, 15 | nie |
| 7. Badania eksperymentalne | `EXPERIMENTS.md`; plan sekcja 11 | tylko 7.1 (metodyka) |
| 8. Analiza wyników | `EXPERIMENTS.md`; plan sekcja 10.4 | nie |
| 9. Ograniczenia i kierunki rozwoju | wyniki E10, E11; plan sekcja 21 | nie |
| 10. Podsumowanie | całość | nie |

---

## Do dopracowania

- [ ] Uzgodnienie struktury i objętości z promotorem.
- [ ] Ustalenie wymaganego stylu cytowań i wzoru strony tytułowej.
- [ ] Ostateczne brzmienie tytułów rozdziałów i podrozdziałów.
- [ ] Decyzja, czy podrozdziały 4.x i 7.x wymagają dalszego podziału.
- [ ] Aktualizacja struktury po zakończeniu eksperymentów (np. jeśli któryś eksperyment
      zostanie pominięty lub połączony z innym).
- [ ] Zsynchronizowanie sekcji 19.2 w `PROJECT_PLAN.md` z tą wersją.
