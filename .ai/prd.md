# Dokument wymagań produktu (PRD) - InflorAI

## 1. Przegląd produktu
InflorAI to narzędzie webowe, które automatyzuje konwersję niejednolitego tekstu z ofert resztkowych dostawców kwiatów na spójny cennik. Aplikacja umożliwia: wklejenie surowych danych, ustawienie globalnych parametrów cenowych, automatyczną ekstrakcję i przeliczenie pozycji, ręczną weryfikację w edytowalnym polu tekstowym oraz wygenerowanie finalnego cennika w prostym formacie tekstowym (z tabulatorami). Wersja MVP obejmuje prostą autentykację i moduł historii zapytań.

Kontekst biznesowy: przedsiębiorcy w branży florystycznej regularnie otrzymują oferty z Holandii o nieustandaryzowanych formatach. Ręczna praca nad czyszczeniem danych, przeliczaniem cen (waluty, marża, transport, VAT) i przygotowaniem cennika jest czasochłonna i podatna na błędy. InflorAI skraca ten proces oraz zmniejsza ryzyko pomyłek.

Założenia danych i reguł cenowych (MVP):
- Stawki VAT: 8% na wszystkie produkty i usługi.
- Kursy walut predefiniowane w konfiguracji: EUR = 4.30 PLN, USD = 3.80 PLN (możliwość edycji w UI przed przetwarzaniem).
- Marża netto i koszt transportu zależne od długości produktu:
  - Długość < 60 cm: marża 0,40 PLN netto/szt., transport 1,32 PLN netto/szt.
  - Długość > 60 cm: marża 0,60 PLN netto/szt., transport 1,42 PLN netto/szt.
- Grupowanie: jeśli w surowych danych wystąpią nagłówki, ich treść zasila pole GRUPA aż do kolejnego nagłówka.
- Kolumna UWAGI zbiera elementy, których nie udało się jednoznacznie skategoryzować.
- Data dostępności: system oczekuje daty na początku bloku wejściowego; w razie braku użytkownik ustawia datę domyślną dla całego dokumentu.

## 2. Problem użytkownika
Przedsiębiorcy otrzymują oferty resztkowe w formacie niejednolitego tekstu. Ręczne przepisywanie, normalizacja i przeliczanie cen zajmuje dużo czasu i prowadzi do błędów. Potrzebne jest narzędzie, które:
- automatycznie zinterpretuje kluczowe pola (nazwa, ilość, długość, cena, waluta, grupa, data),
- przeliczy ceny według zdefiniowanych zasad (kursy, marże, transport, VAT),
- umożliwi szybkie poprawki w tabeli oraz wyświetli popup warning przed wygenerowaniem cennika, jeżeli pozycje mają braki,
- wygeneruje gotowy cennik do skopiowania i zapisze całą operację w historii.

## 3. Wymagania funkcjonalne
1. Wejście danych
   - Pole do wklejenia surowego tekstu ofertowego.
   - Możliwość ustawienia daty domyślnej, jeżeli w wejściu nie ma daty.
   - Wsparcie nagłówków w danych w celu wyznaczania pola GRUPA.
2. Parametry globalne i konfiguracja
   - Edycja kursów walut (domyślnie EUR 4.30, USD 3.80 PLN).
   - Edycja domyślnych marż netto i kosztów transportu netto zależnych od długości.
   - Ustawienie stawki VAT (domyślnie 8%).
3. Parsowanie i ekstrakcja
   - Mechanizm oparty na promptach z kilkoma przykładami (few-shot) do ekstrakcji pól: grupa, nazwa, długość [cm], ilość [szt], cena szt.[pln], cena transport [pln], data dostępności, uwagi
   - Nierozpoznane elementy trafiają do kolumny UWAGI, wiersz jest oznaczony kolorem żółtym
4. Wykrywanie błędów i weryfikacja
   - Pozycje z brakami (np. brak ceny lub ilości) są oznaczane i trafiają do tabeli edycyjnej i są w niej podświetlone innym kolorem
   - Aplikacja pozwala na generowanie cennika, jeżeli istnieją pozycje problematyczne uzupełnione lub usunięte, natomiast przed każdą próbą generacji wyświetli popup warning i informacją o brakach
5. Edytowalne pole tekstowe
   - Edytowalne pole tekstowe, w którym użytkownik może ręcznie korygować całą zawartość cennika (pola rozdzielone tabulatorami, wiersze rozdzielone enter).
6. Przeliczanie cen
   - Konwersja ceny bazowej na PLN według kursu waluty (jeżeli cena jest w EUR/USD; ceny w PLN pozostają bez konwersji).
   - Wyznaczenie marży netto i kosztu transportu netto na sztukę według progu długości 60 cm.
   - Zastosowanie stawki VAT 8% do sumy składowych netto.
   - Wynikowe ceny cząstkowe na sztukę oraz transport w PLN.
7. Generowanie cennika
   - Format tekstowy z tabulatorami, możliwy do łatwego skopiowania.
   - Wiersze zgodne z danymi po weryfikacji; opcjonalny nagłówek kolumn.
8. Historia zapytań
   - Zapis surowych danych wejściowych, stanu po automatycznej ekstrakcji, stanu po korektach oraz wygenerowanego cennika.
   - Dostęp do historii tylko dla zalogowanego użytkownika. Historia widoczna w chowanym panelu bocznym po lewej stronie.
9. Uwierzytelnianie i dostęp
   - Prosty login/hasło, sesja użytkownika, wylogowanie.
   - Ochrona dostępu do historii i generowania cennika.
10. Obserwowalność i pomiar jakości
   - Logowanie różnic między danymi po automatycznej konwersji a wersją finalną po korektach (delta) dla każdej pozycji.
   - Zestawienia najczęstszych korekt na potrzeby dalszych usprawnień algorytmów.

## 4. Granice produktu
Wersja MVP nie obejmuje:
- Interaktywnych mechanizmów uczenia modelu na nowych formatach wejściowych przez użytkownika.
- Integracji z systemami księgowymi/ERP oraz CRM.
- Automatycznego składania zamówień u dostawców.
- Aplikacji mobilnej.
- Wielojęzyczności poza polskim/angielskim.
- Automatycznego wysyłania cennika do listy klientów e‑mailem.
- Zaawansowanej walidacji typów w tabeli edycyjnej (poza zakresem MVP).

## 5. Historyjki użytkowników
ID: US-001
Tytuł: Wklejenie surowego tekstu oferty
Opis: Jako przedsiębiorca, chcę wkleić surowy tekst z ofertą od dostawcy, aby rozpocząć przetwarzanie.
Kryteria akceptacji:
- Given otwarta aplikacja, When wklejam tekst do pola wejściowego, Then widzę tekst i mogę przejść dalej.
- Given brak treści w polu, When próbuję parsować, Then otrzymuję komunikat o pustym wejściu.

ID: US-002
Tytuł: Ustawienie parametrów globalnych (kursy, marże, transport, VAT)
Opis: Jako przedsiębiorca, chcę ustawić parametry wpływające na cennik przed przetwarzaniem.
Kryteria akceptacji:
- Given ekran parametrów, When zmieniam kursy walut, Then nowe kursy są użyte przy przeliczaniu.
- Given ekran parametrów, When ustawiam marże i koszty transportu, Then wartości te są zastosowane według długości.
- Given ekran parametrów, When ustawiam VAT, Then cena końcowa uwzględnia VAT.

ID: US-003
Tytuł: Data dostępności
Opis: Jako przedsiębiorca, chcę aby data z początku wejścia była użyta, a w razie jej braku ustawię datę domyślną dla dokumentu.
Kryteria akceptacji:
- Given wejście zawiera datę na początku, When przetwarzam, Then data trafia do wszystkich pozycji.
- Given wejście nie zawiera daty, When ustawiam datę domyślną, Then data ta jest przypisana wszystkim pozycjom.

ID: US-004
Tytuł: Automatyczna ekstrakcja pól
Opis: Jako przedsiębiorca, chcę aby system automatycznie wydobył pola: grupa, nazwa, długość, ilość, cena, waluta, data, uwagi.
Kryteria akceptacji:
- Given wklejony tekst, When uruchamiam przetwarzanie, Then otrzymuję tabelę z wyekstrahowanymi polami.
- Given brak dopasowania, When pozycja nie może być zinterpretowana, Then pola niepewne/nieznane trafiają do UWAGI.

ID: US-005
Tytuł: Grupowanie produktów według nagłówków
Opis: Jako przedsiębiorca, chcę aby nagłówki w tekście wyznaczały wartość pola GRUPA dla kolejnych pozycji.
Kryteria akceptacji:
- Given nagłówek między blokami pozycji, When przetwarzam, Then wszystkie pozycje do następnego nagłówka mają GRUPA ustawione na treść nagłówka.

ID: US-006
Tytuł: Oznaczanie błędów parsowania
Opis: Jako przedsiębiorca, chcę widzieć, które wiersze wymagają uzupełnienia (np. brak ceny/ilości).
Kryteria akceptacji:
- Given pozycje z brakami, When parsowanie się kończy, Then widzę je oznaczone w tabeli edycyjnej z powodem.

ID: US-007
Tytuł: Edycja w tabeli
Opis: Jako przedsiębiorca, chcę poprawić wyekstrahowane pola bezpośrednio w tabeli.
Kryteria akceptacji:
- Given tabela wynikowa, When edytuję komórkę, Then wartość jest aktualizowana i widzę przeliczoną cenę.
- Given tabela wynikowa, When dodaję lub usuwam wiersz, Then tabela i wyniki przeliczają się odpowiednio.

ID: US-008
Tytuł: Kolumna UWAGI
Opis: Jako przedsiębiorca, chcę aby wszystkie nieprzypisane elementy trafiły do kolumny UWAGI.
Kryteria akceptacji:
- Given element nie pasuje do znanych pól, When kończy się parsowanie, Then element trafia do UWAGI do manualnej oceny.

ID: US-009
Tytuł: Blokada generowania przy błędach
Opis: Jako przedsiębiorca, chcę aby można było warunkowo wygenerować cennika, dopóki pozycje problematyczne nie zostaną naprawione lub usunięte.
Kryteria akceptacji:
- Given istnieją pozycje z brakami, When klikam Generuj, Then aplikacja wyświetla listę problemów i wyświetla popup. Generowanie wciąż możliwe.
- Given wszystkie błędy naprawione, When klikam Generuj, Then cennik jest generowany.

ID: US-010
Tytuł: Przeliczanie końcowej ceny w PLN
Opis: Jako przedsiębiorca, chcę aby cena końcowa na sztukę była obliczana z uwzględnieniem kursu, marży, transportu i VAT.
Kryteria akceptacji:
- Given cena bazowa w EUR/USD i kursy, When przeliczam, Then cena bazowa jest w PLN.
- Given długość pozycji < 60 cm, When przeliczam, Then stosowana jest marża 0,40 PLN netto/szt. i transport 1,32 PLN netto/szt.
- Given długość pozycji > 60 cm, When przeliczam, Then stosowana jest marża 0,60 PLN netto/szt. i transport 1,42 PLN netto/szt.
- Given wartości netto, When stosuję VAT 8%, Then otrzymuję cenę końcową brutto w PLN.

ID: US-011
Tytuł: Logowanie różnic (delta) między auto a final
Opis: Jako właściciel produktu, chcę rejestrować różnice między wynikiem automatycznym a wersją po korektach, aby mierzyć skuteczność.
Kryteria akceptacji:
- Given modyfikacje w tabeli, When zapisuję lub generuję cennik, Then delta zmian jest rejestrowana per pole i pozycję.

ID: US-012
Tytuł: Generowanie cennika tekstowego z tabulatorami
Opis: Jako przedsiębiorca, chcę otrzymać prosty, spójny cennik tekstowy gotowy do skopiowania.
Kryteria akceptacji:
- Given poprawne dane, When generuję cennik, Then otrzymuję tekst z tabulatorami zgodny z widokiem tabeli.
- Given konfiguracja przewiduje nagłówek, When generuję, Then na pierwszym wierszu jest nagłówek kolumn.

ID: US-013
Tytuł: Kopiowanie/eksport cennika
Opis: Jako przedsiębiorca, chcę łatwo skopiować gotowy cennik do schowka lub zapisać jako plik.
Kryteria akceptacji:
- Given wygenerowany cennik, When klikam Kopiuj, Then zawartość trafia do schowka.
- Given wygenerowany cennik, When klikam Zapisz, Then pobieram plik tekstowy.

ID: US-014
Tytuł: Zapis historii zapytań
Opis: Jako zalogowany użytkownik, chcę aby aplikacja zapisała całe przetwarzanie (wejście, auto, korekty, wynik) do historii.
Kryteria akceptacji:
- Given proces zakończony, When kończę generowanie, Then w historii pojawia się rekord z pełnym stanem i datą/czasem.
- Historia zapisuje: surowe wejście → dane po ekstrakcji (auto-wynik) → edycje użytkownika w polu tekstowym → finalny cennik

ID: US-015
Tytuł: Podgląd i przywracanie z historii
Opis: Jako zalogowany użytkownik, chcę przeglądać poprzednie cenniki i w razie potrzeby je odtworzyć.
Kryteria akceptacji:
- Given panel historii, When otwieram rekord, Then widzę wejście, auto-wynik, korekty i finalny cennik.
- Given rekord historii, When wybieram Przywróć, Then dane wejściowe oraz wyjściowe edytowalne pole tekstowe wczytują się ponownie wraz z zapisanymi wartościami

ID: US-016
Tytuł: Uwierzytelnianie użytkownika (login/hasło)
Opis: Jako użytkownik, chcę logować się do aplikacji, aby chronić dostęp do historii i generowania.
Kryteria akceptacji:
- Given formularz logowania, When podaję poprawne dane, Then uzyskuję dostęp do aplikacji.
- Given formularz logowania, When podaję błędne dane, Then otrzymuję czytelną informację o błędzie.

ID: US-018
Tytuł: Ochrona dostępu do historii
Opis: Jako właściciel danych, chcę aby historia była dostępna wyłącznie po zalogowaniu.
Kryteria akceptacji:
- Given niezalogowany użytkownik, When wchodzi na historię, Then jest przekierowany do logowania.

ID: US-019
Tytuł: Obsługa pustego i nieprawidłowego wejścia
Opis: Jako użytkownik, chcę otrzymywać zrozumiałe komunikaty, gdy wejście jest puste lub nieczytelne.
Kryteria akceptacji:
- Given puste wejście, When klikam Przetwórz, Then widzę komunikat o braku treści.
- Given wejście nieczytelne, When klikam Przetwórz, Then widzę listę problemów i wskazówki.

ID: US-020
Tytuł: Różne formaty liczb i walut
Opis: Jako użytkownik, chcę aby system radził sobie z kropką/przecinkiem dziesiętnym oraz symbolami walut (EUR/€, USD/$, PLN/zł).
Kryteria akceptacji:
- Given wartości 10,50 i 10.50, When przetwarzam, Then obie interpretowane są jako 10.50.
- Given symbol € przy cenie, When przetwarzam, Then waluta rozpoznana jako EUR.

ID: US-021
Tytuł: Ręczne nadpisanie GRUPA i długości
Opis: Jako użytkownik, chcę móc ręcznie zmienić GRUPA oraz długość produktu, aby poprawić klasyfikację i przeliczenia.
Kryteria akceptacji:
- Given tabela, When edytuję GRUPA lub długość, Then zmiana wpływa na przeliczenia marży/transportu i jest widoczna.

ID: US-022
Tytuł: Usuwanie pozycji problematycznych
Opis: Jako użytkownik, chcę móc usunąć pozycje, których nie da się uzupełnić, aby odblokować generowanie.
Kryteria akceptacji:
- Given pozycja weryfikacyjna, When ją usuwam, Then znika z listy błędów, a blokada generowania znika, jeżeli nie ma innych błędów.

Lista kontrolna kompletności historyjek i testowalności:
- Każda historyjka zawiera kryteria akceptacji możliwe do przetestowania.
- Uwzględniono scenariusze podstawowe, alternatywne i skrajne (puste/nieczytelne wejście, różne formaty, brak daty, błędy parsowania, usuwanie pozycji).
- Ujęto wymagania dotyczące uwierzytelniania i ochrony dostępu (US-016, US-017, US-018).

## 6. Metryki sukcesu
1. Automatyczna poprawność ekstrakcji i przeliczeń
   - Współczynnik rekordów wymagających manualnej korekty poniżej 10%.
   - Cel: co najmniej 90% rekordów przetwarzanych automatycznie i poprawnie.
2. Efektywność operacyjna
   - Redukcja całkowitego czasu przygotowania cennika o minimum 50% względem procesu ręcznego (baseline 8 minut na 50 wierszy)
   - Mierzalnik: Czas od wklejenia danych do wygenerowania gotowego cennika
   - Cel: Proces, który wcześniej zajmował np. 2 godziny, teraz trwa maksymalnie 15 minut
3. Telemetria jakości i możliwości ulepszeń
   - Logowanie delty między wynikiem automatycznym i finalnym dla wszystkich pozycji.
   - Raportowanie najczęstszych pól wymagających korekt jako input do roadmapy jakości parsowania.


