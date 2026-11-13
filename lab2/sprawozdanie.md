# Sprawozdanie z laboratorium 2 — plan rozpoznawania spamu

[Repozytorium laboratoria-ai](https://github.com/patrycjaklein-ui/laboratoria-ai)

**Aplikacja:** Codex desktop  
**Model:** GPT-5.6 Luna  
**Poziom rozumowania:** Wysoki

## 1. Cel zadania

Celem zadania było zaplanowanie programu, który będzie rozpoznawał, czy wiadomość e-mail jest spamem, czy prawidłową wiadomością. W danych prawidłowa poczta ma etykietę `ham`.

Na tym etapie nie tworzyliśmy programu, nie trenowaliśmy modelu i nie zmienialiśmy plików z danymi. Najpierw chcieliśmy zrozumieć dane, wybrać prostą metodę reprezentacji tekstu i zaplanować późniejsze uczenie oraz ocenę modelu.

## 2. Dane

Dane znajdują się w plikach:

- `data/train.csv` — wiadomości używane później do uczenia;
- `data/validation.csv` — osobne wiadomości używane później do oceny.

Każdy wiersz odpowiada jednej wiadomości. Pliki mają kodowanie UTF-8, a treść może zawierać przecinki i znaki nowej linii, dlatego trzeba je odczytywać jako prawdziwe pliki CSV.

Każdy wiersz ma trzy kolumny:

- `message_id` — identyfikator wiadomości. Służy do odnajdywania wiadomości, ale nie powinien być podawany modelowi jako cecha;
- `text` — temat i pełna tekstowa treść wiadomości;
- `label` — poprawna kategoria: `spam` albo `ham`.

### Liczebność danych

| Zbiór | Wszystkie wiadomości | `ham` | `spam` |
|---|---:|---:|---:|
| `train.csv` | 180 | 120 | 60 |
| `validation.csv` | 60 | 40 | 20 |

W obu zbiorach dwie trzecie wiadomości to `ham`, a jedna trzecia to spam. Oznacza to, że model zawsze zwracający `ham` uzyskałby około 66,7% accuracy, ale nie wykryłby żadnego spamu.

### Kontrola plików

Odczytaliśmy oba pliki i sprawdziliśmy ich podstawową jakość. Nie znaleźliśmy:

- pustych identyfikatorów;
- pustych tekstów;
- pustych etykiet;
- etykiet innych niż `spam` i `ham`;
- powtórzonych `message_id` w obrębie zbiorów;
- powtórzonych treści w obrębie zbiorów;
- identycznych treści między `train.csv` i `validation.csv`.

Treści porównano po ujednoliceniu wielkości liter i białych znaków. Nie oznacza to jednak, że wykluczyliśmy wiadomości tylko podobne, należące do tego samego wątku albo mające wspólny fragment.

Przykłady wiadomości `ham` dotyczą technicznych tematów, między innymi Spambayes, CVS-a i Red Hata. Wiadomości `spam` zawierają między innymi oferty tanich połączeń międzynarodowych oraz propozycje współpracy. W tekstach mogą występować adresy URL, cytowane odpowiedzi, adresy e-mail i elementy charakterystyczne dla dawnych list mailingowych.

Dane pochodzą z korpusu Apache SpamAssassin. Jest to mały, historyczny i dydaktyczny podzbiór. Proporcje spamu w tych plikach zostały ustalone na potrzeby zajęć i nie opisują udziału spamu w prawdziwej skrzynce. Dobry wynik na tym zbiorze nie gwarantuje dobrego działania na współczesnej poczcie.

## 3. Wybrana reprezentacja tekstu: TF-IDF

Komputer nie może bezpośrednio wykonać obliczeń na całych zdaniach. Dlatego tekst trzeba najpierw zamienić na liczby. Wybraliśmy reprezentację TF-IDF dla pojedynczych słów i bigramów.

TF-IDF nadaje słowom wagi zależnie od tego:

- jak często występują w danej wiadomości;
- jak charakterystyczne są w porównaniu z całym zbiorem.

Częste, mało informacyjne słowa dostają mniejszą wagę. Bardziej charakterystyczne słowa i frazy mogą dostać większą wagę.

**Bigram** to para kolejnych słów. Dla wiadomości:

> International calls for only 33 cents per minute with no subscription

przykładowymi cechami mogą być:

- `international`;
- `calls`;
- `33 cents`;
- `per minute`;
- `no subscription`.

TF-IDF sam nie rozpoznaje spamu. On tylko tworzy liczbowy opis tekstu. Dopiero model korzysta z tych liczb i uczy się, które cechy są częstsze w spamie, a które w `ham`.

Wybraliśmy TF-IDF, ponieważ:

- jest prosty do wyjaśnienia;
- dobrze pasuje do krótkich i średnich tekstów;
- działa przy małej liczbie przykładów;
- nie wymaga dużego modelu językowego ani skomplikowanych obliczeń;
- pozwala korzystać z charakterystycznych słów i fraz.

Na początku nie chcemy automatycznie usuwać wszystkich URL-i, liczb ani znaków specjalnych, ponieważ takie elementy mogą być sygnałami spamu.

## 4. Wybrany model: regresja logistyczna

Regresja logistyczna jest modelem, który uczy się, jak poszczególne cechy wpływają na decyzję `spam` lub `ham`.

Może na przykład nauczyć się, że:

- `33 cents` i `no subscription` częściej występują w spamie;
- `CVS` i `[Spambayes]` częściej występują w zwykłych wiadomościach.

Model przypisuje cechom wagi, a potem łączy ich wpływ dla całej wiadomości. Wynik można traktować w przybliżeniu jako stopień przekonania, że wiadomość jest spamem. Następnie wynik porównujemy z progiem, na przykład 0,50.

Regresja logistyczna pasuje do naszego zadania, ponieważ:

- dobrze współpracuje z rzadką macierzą cech TF-IDF;
- jest szybka i stosunkowo prosta;
- może działać dobrze na małym zbiorze tekstowym;
- łatwo analizować jej błędy;
- można zmieniać próg decyzji zależnie od tego, jak bardzo chcemy chronić zwykłe wiadomości.

### Rozważana alternatywa: liniowy SVM

Liniowy SVM również otrzymuje liczby z TF-IDF. Próbuje znaleźć granicę, która oddziela wiadomości spam od wiadomości `ham`.

SVM często dobrze działa przy dużej liczbie cech tekstowych i może okazać się skuteczniejszy od regresji logistycznej. Jego wynik jest jednak przede wszystkim wynikiem decyzji, a nie prawdopodobieństwem. Trudniej wyjaśnić go początkującej osobie i trudniej intuicyjnie zmieniać próg.

Dlatego regresję logistyczną wybraliśmy jako główne, prostsze podejście, a liniowy SVM pozostawiliśmy jako sensowną alternatywę do porównania. Nie wiemy jeszcze, który model uzyska lepsze wyniki, ponieważ żadnego z nich jeszcze nie trenowaliśmy.

## 5. Plan późniejszej realizacji

### Krok 1: przygotowanie danych

Z `train.csv` pobierzemy teksty z kolumny `text` oraz etykiety z kolumny `label`. `message_id` zostawimy tylko do identyfikowania wiadomości w wynikach.

Nie zmienimy dostarczonego podziału na 180 wiadomości treningowych i 60 walidacyjnych.

### Krok 2: dopasowanie TF-IDF

TF-IDF zostanie dopasowany tylko na danych treningowych. Pozna słownictwo, bigramy i statystyki potrzebne do zamiany tekstu na liczby.

Na przykład wiadomość z ofertą:

> International calls for only 33 cents per minute with no subscription

zostanie zamieniona na wartości opisujące między innymi `international`, `33 cents` i `no subscription`.

TF-IDF nie będzie dopasowywany na `validation.csv`. Podczas późniejszej oceny użyjemy tego samego słownika i tych samych wag, które powstały na danych treningowych.

### Krok 3: uczenie modeli

Wytrenujemy dwa modele na tych samych cechach:

1. regresję logistyczną;
2. liniowy SVM.

Dzięki temu porównanie będzie uczciwe. Różnica w wyniku będzie wynikała głównie z modelu, a nie z innego przygotowania tekstu.

### Krok 4: porównanie i wybór progu

Na `train.csv` zastosujemy stratified cross-validation, czyli kilka podziałów danych zachowujących podobne proporcje `spam` i `ham`. W każdym podziale TF-IDF będzie dopasowywany tylko na części używanej do uczenia.

Dla regresji logistycznej sprawdzimy różne progi. Niższy próg może wykryć więcej spamu, ale zwykle zwiększa liczbę fałszywych alarmów. Wyższy próg lepiej chroni zwykłe wiadomości, ale może przepuścić więcej spamu.

Ponieważ szczególnie zależy nam na tym, żeby prawidłowe wiadomości nie trafiały do spamu, będziemy zwracać uwagę na liczbę fałszywych alarmów, a nie tylko na ogólną accuracy.

Jeśli regresja logistyczna i SVM będą miały podobne wyniki, pozostaniemy przy regresji logistycznej ze względu na prostotę i łatwiejszą kontrolę progu. Jeśli SVM będzie wyraźnie lepszy przy podobnej liczbie fałszywych alarmów, wybierzemy SVM.

### Krok 5: finalne dopasowanie i zapis

Po ustaleniu modelu i progu:

1. dopasujemy TF-IDF na wszystkich 180 wiadomościach z `train.csv`;
2. dopasujemy wybrany model;
3. połączymy TF-IDF, model i próg w jeden pipeline;
4. zapiszemy pipeline do osobnego pliku.

Zapisany pipeline powinien zawierać zarówno reprezentację TF-IDF, jak i model. Dzięki temu później nie użyjemy przez pomyłkę innego słownika lub innych wag.

### Krok 6: przewidywanie nowych wiadomości

Dla nowej wiadomości proces będzie wyglądał tak:

```text
tekst wiadomości
→ zapisany TF-IDF
→ liczby
→ zapisany model
→ wynik i próg
→ spam albo ham
```

Na przykład tekst zawierający ofertę cenową i frazę `no subscription` może otrzymać wysoki wynik spamu. Jest to przykład poglądowy, a nie wynik już wytrenowanego modelu.

### Krok 7: ocena na `validation.csv`

Dopiero po wybraniu rozwiązania użyjemy 60 wiadomości z `validation.csv` do końcowej oceny. Nie będziemy na nich dopasowywać TF-IDF, uczyć modelu ani wielokrotnie dobierać progu.

Wyniki zapiszemy osobno, razem z `message_id`, prawdziwą etykietą, przewidywaną etykietą i wynikiem modelu. Dostarczonych plików nie będziemy nadpisywać.

## 6. Jak ocenimy model?

Sama accuracy nie wystarczy. Przy naszych danych prosty model zwracający zawsze `ham` uzyskałby około 66,7%, a jednocześnie nie wykryłby żadnego spamu.

Sprawdzimy:

- **precision** — spośród wiadomości oznaczonych jako spam, ile rzeczywiście było spamem;
- **recall** — spośród wszystkich prawdziwych spamów, ile model wykrył;
- **F1** — wspólną miarę precision i recall;
- **macierz pomyłek** — ile było poprawnie rozpoznanych wiadomości, przeoczonych spamów i fałszywych alarmów.

Szczególnie ważne są fałszywe alarmy, czyli sytuacje, w których zwykła wiadomość `ham` zostaje oznaczona jako spam. Taki błąd może oznaczać utratę ważnej korespondencji. Z drugiej strony zbyt ostrożny model może przepuszczać spam. Próg trzeba więc dobrać do tego kompromisu.

## 7. Najważniejsze wątpliwości i ograniczenia

### Czy model nie zapamięta tylko 180 wiadomości?

Sprawdzimy to przez porównanie wyników treningowych i walidacyjnych oraz przez cross-validation. Jeżeli model będzie prawie idealny na treningu, ale dużo słabszy na nowych wiadomościach, będzie to oznaczało przeuczenie.

Dobry wynik na `validation.csv` będzie dowodem, że model radzi sobie z częścią nowych przykładów, ale nie będzie gwarancją skuteczności na każdej współczesnej skrzynce.

### Co jeśli pojawi się spam z nowymi słowami?

Zwykły TF-IDF słów nie ma osobnej cechy dla słowa, którego nie było w `train.csv`. Model może jednak wykorzystać inne znane słowa i frazy.

Jeśli nowa wiadomość będzie prawie całkowicie zbudowana z nieznanych słów, może zostać pomylona z `ham`. W takiej sytuacji można rozważyć TF-IDF znaków, który lepiej radzi sobie z literówkami, nowymi nazwami, URL-ami i celowym zniekształcaniem słów.

### Co jeśli jest za dużo fałszywych alarmów?

Najpierw można podnieść próg decyzji albo przenosić podejrzane wiadomości do folderu kwarantanny zamiast automatycznie je usuwać. Można też zmienić parametry uczenia lub reprezentację tekstu.

### Co jeśli jest za dużo przeoczonego spamu?

Można obniżyć próg albo sprawdzić reprezentację znakową. Połączenie cech słownych i znakowych może pomóc w przypadku literówek, nietypowych zapisów i nowych słów.

### Co jeśli SVM będzie lepszy?

Jeżeli SVM uzyska wyraźnie lepsze wyniki przy podobnej liczbie fałszywych alarmów, będzie rozsądną zmianą modelu. Jeżeli oba modele będą popełniały podobne błędy, problemem może być reprezentacja tekstu albo ograniczenia danych, a nie sam klasyfikator.

## 8. Wnioski

Sprawozdanie przygotowałam w aplikacji Codex desktop, korzystając z modelu GPT-5.6 Luna i poziomu rozumowania „Wysoki”. Wspólnie przeanalizowaliśmy dane, wyjaśniliśmy podstawowe pojęcia i przygotowaliśmy plan rozwiązania, ale nie pisaliśmy jeszcze programu, nie trenowaliśmy modelu i nie zmienialiśmy plików z danymi.

W laboratorium 2 korzystałam z GPT-5.6 Luna z poziomem rozumowania „Wysoki”, zamiast ponownie wybierać nowszy model GPT-6, którego używałam wcześniej. Potraktowałam to jako okazję do sprawdzenia, czy wcześniejszy model również poradzi sobie z analizą danych, wyjaśnianiem pojęć i przygotowaniem planu rozwiązania. Chciałam zobaczyć, czy do takiego zadania rzeczywiście potrzebny jest najnowszy model. GPT-5.6 pomógł mi przeanalizować dane, porównać metody i zrozumieć ich ograniczenia, więc do przygotowania planu okazał się wystarczający.

Na początku nie rozumiałam dokładnie, czym różni się TF-IDF od modelu. Wydawało mi się, że TF-IDF już częściowo rozpoznaje spam. Po wyjaśnieniu zrozumiałam, że TF-IDF jest tylko sposobem zamiany tekstu na liczby. Tworzy cechy opisujące słowa i frazy, ale sam nie decyduje, czy wiadomość jest spamem. Dopiero regresja logistyczna uczy się na podstawie etykiet `spam` i `ham`, które cechy mogą wskazywać na daną kategorię.

Przy naszych 180 wiadomościach treningowych lepiej zacząć od prostszego rozwiązania, które nie wymaga dużych obliczeń i którego działanie łatwiej wyjaśnić. TF-IDF słów i bigramów pasuje do tekstowego charakteru danych, a regresja logistyczna dobrze współpracuje z taką reprezentacją. Pozwala też zmieniać próg decyzji. Jest to dla nas ważne, ponieważ bardziej zależy nam na tym, żeby zwykłe wiadomości nie były przypadkowo oznaczane jako spam, niż na bezwarunkowym wykrywaniu każdej podejrzanej wiadomości.

Rozważaliśmy również liniowy SVM. SVM może dobrze działać z TF-IDF, ale nie wybraliśmy go od razu, ponieważ regresja logistyczna jest dla mnie łatwiejsza do zrozumienia, a jej wynik i próg są bardziej intuicyjne. Nie oznacza to, że SVM jest gorszy. Nie trenowaliśmy jeszcze żadnego z tych modeli, więc nie znamy ich rzeczywistego porównania. SVM zostawiliśmy jako alternatywę do późniejszego sprawdzenia na tych samych danych i cechach.

W późniejszej realizacji `train.csv` posłuży do dopasowania TF-IDF i uczenia modeli, a `validation.csv` pozostanie osobnym zbiorem do końcowego sprawdzenia. Modele porównamy za pomocą precision, recall, F1, macierzy pomyłek, liczby fałszywych alarmów i stabilności wyników. Nie będziemy opierać wyboru wyłącznie na accuracy, ponieważ model zawsze zwracający `ham` osiągnąłby około 66,7% accuracy, ale nie wykryłby żadnego spamu.

Jeżeli wynik na `train.csv` będzie dużo lepszy niż na `validation.csv`, może to wskazywać na przeuczenie, czyli zbyt mocne dopasowanie do wiadomości treningowych. Taka różnica nie daje jednak pewności, że wystąpiło przeuczenie, ponieważ może wynikać również z małego rozmiaru walidacji, różnic między zbiorami, przypadkowego podziału albo trudniejszych wiadomości w walidacji. Dlatego potrzebne będą także cross-validation i analiza konkretnych błędów.
