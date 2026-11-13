Repozytorium: [https://github.com/patrycjaklein-ui/laboratoria-ai](https://github.com/patrycjaklein-ui/laboratoria-ai)

**Codex (aplikacja desktopowa) · GPT-6.1 Sol · Lekki**

# Sprawozdanie z laboratorium 1

## 1. Zlecony wynik, zakres zmian i sposób sprawdzania

Zleciłam przygotowanie raportu JSON na podstawie pliku `data/exercise.csv`. Zanim uruchomiłam program, chciałam zrozumieć README, kod i dane. Poprosiłam Codexa o wyjaśnienia krok po kroku, prostym językiem.

Zgody udzielałam na konkretne czynności osobno. Na początku poprosiłam o pokazanie poleceń sprawdzających folder roboczy i wersję Pythona przed ich wykonaniem. Codex pokazał je, ale od razu uruchomił, nie czekając na moje potwierdzenie. Po zwróceniu mu uwagi ustaliłam wyraźną zasadę oczekiwania na zgodę. Później osobno zatwierdzałam sprawdzenie istnienia raportu, uruchomienie programu, odczyt raportu i wykonanie testu. W kolejnym etapie osobno zatwierdziłam zmianę jednej etykiety w danych, a później ponowne uruchomienie programu i nadpisanie raportu. Nie była to zgoda na dowolne zmiany w projekcie. Nie zezwalałam na modyfikowanie kodu ani testów.

Prosiłam o pokazywanie dokładnych komend, wyników terminala i kodów zakończenia. Zauważyłam, że Codex czasami wykonywał polecenia bez poczekania na moje zatwierdzenie. Ustaliłam więc zasadę: najpierw pokazuje komendę i wyjaśnia jej działanie, a dopiero po mojej zgodzie ją wykonuje.

## 2. Powstanie raportu i znaczenie wyników

Danymi wejściowymi był plik `data/exercise.csv`. Zawiera on wiadomości pochodzące z publicznego zbioru Apache SpamAssassin. Jeden rekord CSV odpowiada jednej wiadomości i zawiera jej identyfikator `message_id`, temat i treść w polu `text` oraz gotową etykietę `ham` albo `spam`.

Program uruchomiłam za pośrednictwem Codexa, korzystającego z terminala PowerShell na moim komputerze z Windowsem. Dostępny był Python 3.12.6. Pracowałam w folderze:

```text
C:\Users\patry\Downloads\lab-1-project\project
```

Użyłam polecenia:

```powershell
py -3 describe_data.py --data data/exercise.csv --out results/exercise-summary.json
```

Program zakończył się kodem `0` i zapisał raport. Początkowo wynik wynosił **8 wiadomości: 5 `ham` i 3 `spam`**. Odczytałam JSON i sprawdziłam, czy `input_file` wskazuje właściwy plik wejściowy oraz czy liczby zgadzają się z wynikiem terminala.

`total_messages` oznacza liczbę wszystkich wiadomości, a `label_counts` zawiera liczby istniejących etykiet. Program sam nie ocenia, czy wiadomość jest spamem. Raport zapisuje również sumę kontrolną danych wejściowych (`input_sha256`) oraz informacje o użytym Pythonie i folderze roboczym (`runtime`).

Po zmianie jednej etykiety i ponownym uruchomieniu programu terminal zwrócił:

```text
Report saved: C:\Users\patry\Downloads\lab-1-project\project\results\exercise-summary.json
Messages: 8; labels: {'ham': 4, 'spam': 4}
```

Kod zakończenia wynosił **`0`**. Końcowy raport zawiera więc **8 wiadomości: 4 `ham` i 4 `spam`**.

**Końcowy raport:** [results/exercise-summary.json](results/exercise-summary.json).

## 3. Zrozumiany fragment kodu i własne przewidywanie

Chciałam zrozumieć, dlaczego wiadomość zajmująca kilka linii tekstu jest liczona jako jedna. Pomogło mi wyjaśnienie tych fragmentów `describe_data.py`:

```python
rows = csv.DictReader(io.StringIO(raw.decode("utf-8"), newline=""))
```

```python
for number, row in enumerate(rows, start=1):
```

```python
counts[row["label"]] += 1
```

Rozumiem to tak: `csv.DictReader` odczytuje całe rekordy CSV i rozpoznaje wieloliniowy tekst umieszczony w cudzysłowach. Pętla przechodzi po wiadomościach, a ostatnia instrukcja zwiększa odpowiedni licznik o jeden, zgodnie z wartością pola `label`. Puste linie i słowa występujące w treści nie zwiększają liczby wiadomości ani nie decydują o jej kategorii.

Dla przygotowanego przykładu z pięcioma wiadomościami **sama przewidziałam wynik: 5 wiadomości, 3 `ham` i 2 `spam`**, zanim otrzymałam rozwiązanie. Uzasadniłam, że puste linie są częścią treści jednego maila. Dopiero potem wspólnie z Codexem sprawdziłyśmy moje przewidywanie przez analizę kodu. **Nie uruchamiałyśmy tego przykładu.** Moja odpowiedź była poprawna.

## 4. Uruchomienie testu i zakres sprawdzenia

Po zapoznaniu się z opisem testu zatwierdziłam polecenie:

```powershell
py -3 -m unittest discover -s tests -v
```

Test zakończył się poprawnie. Terminal pokazał `Ran 1 test` oraz `OK`, a kod zakończenia wynosił `0`.

Test korzystał z osobnego pliku `tests/fixtures/messages.csv`, zawierającego trzy sztuczne wiadomości, również z wieloliniową treścią. Oczekiwany wynik był ustalony z góry: **3 wiadomości, 2 `ham` i 1 `spam`**. Test sprawdzał poprawne zakończenie programu, zapis czytelnego raportu JSON, wskazanie właściwego wejścia i oczekiwane liczby.

Przed testem i po nim porównałam sumę kontrolną zapisanego raportu `results/exercise-summary.json`. Była identyczna, co potwierdziło, że jego zawartość się nie zmieniła.

Zrozumiałam, że pozytywny wynik potwierdza działanie programu dla tego małego zestawu testowego. Nie gwarantuje poprawności dla wszystkich możliwych danych ani poprawności źródłowych etykiet. Test został wykonany przed zmianą etykiety opisanej w kolejnym punkcie.

## 5. Zmiana etykiety i porównanie wyników

Najpierw wybrałam wiadomość `exercise-001`. Następnie zatwierdziłam zmianę tylko jej etykiety z `ham` na `spam`, bez uruchamiania programu i bez aktualizowania raportu. Była to celowa zmiana na potrzeby ćwiczenia, a nie ponowna ocena poprawności źródłowej etykiety.

Po tej zmianie **sama przewidziałam wynik: 8 wiadomości, 4 `ham` i 4 `spam`**. Wyjaśniłam, że jedna wiadomość zmieniła kategorię, ale liczba wszystkich wiadomości pozostała taka sama.

Raport nie aktualizuje się automatycznie po zmianie CSV. Dlatego dopiero po przedstawieniu mojego przewidywania zgodziłam się na ponowne wykonanie programu i nadpisanie wcześniejszego raportu. Program zakończył się kodem `0`, a zapisany wynik potwierdził moje przewidywanie:

| Wynik | Przed zmianą | Po zmianie |
|---|---:|---:|
| Wszystkie wiadomości | 8 | 8 |
| `ham` | 5 | 4 |
| `spam` | 3 | 4 |

Zmieniło się również pole `input_sha256`, ponieważ zawartość pliku wejściowego była już inna. Ścieżka `input_file` nadal wskazywała ten sam plik `data/exercise.csv`. Po tej zmianie nie uruchamiałam ponownie testów.

## Wnioski

Podczas laboratorium zrozumiałam, że Codex może pomóc mi przeanalizować kod i uruchomić program, nawet jeśli wcześniej nie znałam dobrze projektu. Nie powinnam jednak bez sprawdzania przyjmować wszystkich jego odpowiedzi za poprawne. Porównywałam wyniki z terminala, sprawdzałam plik JSON i sama przewidywałam, co powinno się zmienić.

Zauważyłam też, że Codex nie zawsze przestrzegał moich instrukcji. Zdarzyło mu się uruchomić polecenia bez wcześniejszego zatwierdzenia, mimo że prosiłam o pokazanie ich przed wykonaniem. Po zwróceniu uwagi poprawiał swoje zachowanie, ale później ponownie wykonał polecenie odczytu bez osobnego potwierdzenia. Nauczyłam się, że przy pracy z agentem AI ważne jest nie tylko poprawne wykonanie zadania, ale też kontrolowanie jego działań i jasne określanie, na co wyrażam zgodę.

Wybrałam model GPT-6.1 Sol z poziomem rozumowania „Lekki”, ponieważ uznałam go za wystarczający do analizy prostego kodu i sprawdzenia wyników w tym ćwiczeniu.
