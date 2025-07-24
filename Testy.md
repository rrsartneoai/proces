Przechodzimy od projektowania do kluczowego etapu zapewnienia jakości i niezawodności. Stworzenie solidnego pakietu testów automatycznych jest nie tylko dobrą praktyką deweloperską, ale w branży legaltech stanowi fundament budowania zaufania i ograniczania ryzyka prawnego.

Poniżej przedstawiam kompletną strategię testowania, szczegółowe scenariusze do zaimplementowania oraz – co kluczowe – analizę prawną, która nadaje tym testom głębszy, biznesowy sens.

---

### **Wprowadzenie: Dlaczego Testowanie Jest Krytyczne dla Tej Platformy?**

W naszym projekcie testy nie tylko weryfikują, czy kod działa. One **gwarantują, że proces prawny jest spójny, bezpieczny i zgodny z założeniami**. Błędy w systemie mogłyby prowadzić do udostępnienia poufnych danych, wygenerowania nieprawidłowych pism lub zerwania ścieżki audytowej, co niosłoby za sobą poważne konsekwencje. Naszym celem jest stworzenie "cyfrowego notariusza" – systemu, któremu można bezwzględnie ufać.

### **Strategia Testowania: Piramida Jakości**

Zastosujemy sprawdzony model "piramidy testów", aby zapewnić maksymalne pokrycie przy optymalnym koszcie i szybkości.

1.  **Testy Jednostkowe (Unit Tests) - Fundament:** Szybkie testy weryfikujące małe, odizolowane fragmenty logiki (funkcje, komponenty).
2.  **Testy Integracyjne (Integration Tests) - Spoiwo:** Testy sprawdzające współpracę kilku modułów, np. czy API poprawnie komunikuje się z bazą danych.
3.  **Testy End-to-End (E2E) - Weryfikacja Biznesowa:** Pełna symulacja ścieżki użytkownika w przeglądarce, od logowania po finalny rezultat. Są najwolniejsze, ale dają największą pewność co do działania całego systemu.

---

### **Co Należy Zaimplementować: Sekwencja Działań i Przykłady**

#### **Krok 1: Testy Jednostkowe (Unit Tests)**

Te testy piszemy jako pierwsze, równolegle z tworzeniem kodu.

**Narzędzia:** `Jest` lub `Vitest`.

**Co testujemy?**
*   **Funkcja walidacji pliku:**
    *   `czy_plik_jest_czytelny(plik_testowy_rozmyty)` -> zwraca `false`.
    *   `czy_plik_jest_czytelny(plik_testowy_wyrazny)` -> zwraca `true`.
    *   `czy_format_jest_poprawny('.exe')` -> zwraca `false`.
*   **Funkcja budowania promptu dla AI:**
    *   `zbuduj_prompt(tekst_pisma, dyrektywy)` -> sprawdź, czy wynikowy string zawiera zarówno tekst pisma, jak i wszystkie dyrektywy.
*   **Komponenty Frontendowe (w izolacji):**
    *   Czy komponent `<CaseStatusViewer>` poprawnie renderuje status "Analiza w toku", gdy otrzyma taki `prop`?
    *   Czy przycisk w `<FileUploadForm>` jest nieaktywny podczas przesyłania pliku?

#### **Krok 2: Testy Integracyjne (Integration Tests)**

Testujemy współpracę kluczowych modułów backendowych.

**Narzędzia:** `Jest`/`Vitest` z `Supertest` (do testowania API).

**Co testujemy?**
*   **Endpoint `POST /api/cases/upload`:**
    *   **Scenariusz:** Wysyłamy poprawny plik na ten endpoint.
    *   **Weryfikacja:**
        1.  Czy API zwróciło status `200 OK` i `caseId`?
        2.  Czy w bazie danych powstał nowy rekord w tabeli `Sprawy` ze statusem `VALIDATING`?
        3.  Czy została wywołana (zamockowana) funkcja `validation_pipeline` z odpowiednimi argumentami?
*   **Interakcja z Bazą Danych:**
    *   **Scenariusz:** Wywołujemy funkcję `zmien_status_sprawy(caseId, 'OPERATOR_REVIEW')`.
    *   **Weryfikacja:** Sprawdzamy bezpośrednio w testowej bazie danych, czy status dla danego `caseId` został poprawnie zaktualizowany.

#### **Krok 3: Testy End-to-End (E2E) - Pełna Symulacja Procesu**

To jest symulacja Państwa głównego procesu biznesowego.

**Narzędzia:** `Cypress` lub `Playwright`.

**Scenariusz do zaimplementowania: "Pełna Cyfrowa Ścieżka Sprawy"**

**Przygotowanie (Setup):** Przed każdym testem system automatycznie czyści bazę danych i tworzy dwóch użytkowników: `test_klient@example.com` oraz `test_operator@example.com`.

**Część 1: Klient Inicjuje Sprawę**
1.  **GIVEN:** Użytkownik jest na stronie logowania.
2.  **WHEN:** Wpisuje `test_klient@example.com` i hasło, a następnie klika "Zaloguj".
3.  **THEN:** Zostaje przekierowany do swojego Panelu Klienta.
4.  **WHEN:** Klika "Rozpocznij nową sprawę" i przesyła **poprawny, czytelny plik PDF** (`pismo_testowe.pdf`).
5.  **THEN:** W widoku sprawy widzi status zmieniający się na "Weryfikacja formalna", a po chwili na "Analiza w toku".
6.  **AND:** Po pewnym czasie (symulującym pracę AI), status zmienia się na "Oczekuje na weryfikację Operatora".
7.  **FINALLY:** Użytkownik się wylogowuje.

**Część 2: Operator Weryfikuje i Odsyła Dokument**
1.  **GIVEN:** Użytkownik jest na stronie logowania.
2.  **WHEN:** Wpisuje `test_operator@example.com` i hasło, a następnie klika "Zaloguj".
3.  **THEN:** Zostaje przekierowany do swojego Panelu Operatora i na liście zadań widzi nową sprawę zainicjowaną przez klienta.
4.  **WHEN:** Klika w tę sprawę, widzi treść dokumentu i wygenerowany przez AI projekt odpowiedzi. Dokonuje drobnej **edycji ręcznej** w tekście i klika przycisk **"Zatwierdź i wyślij do klienta"**.
5.  **THEN:** Sprawa znika z jego listy aktywnych zadań.
6.  **FINALLY:** Użytkownik się wylogowuje.

**Część 3: Klient Odbiera i Akceptuje Finalny Dokument**
1.  **GIVEN:** Użytkownik jest na stronie logowania.
2.  **WHEN:** Ponownie loguje się jako `test_klient@example.com`.
3.  **THEN:** W panelu sprawy widzi status "Gotowe do wglądu".
4.  **AND:** Widoczny jest przycisk "Pobierz dokument".
5.  **WHEN:** Klika przycisk "Pobierz dokument".
6.  **THEN:** Rozpoczyna się pobieranie pliku, a jego treść zawiera **dokładnie te zmiany, które wprowadził Operator**.
7.  **WHEN:** Klient klika przycisk "Akceptuję dokument".
8.  **THEN:** Status sprawy zmienia się na "Zakończona".

---

### **Ulepszenie: Profesjonalna Analiza Prawna w Kontekście Testowania**

Testy automatyczne muszą odzwierciedlać nie tylko poprawność techniczną, ale i **ryzyka prawne**. Oto, jak rozszerzyć powyższe scenariusze o wymiar prawny:

1.  **Test Integralności Dokumentu (Checksumming):**
    *   **Problem Prawny:** Musimy mieć 100% pewności, że dokument przeanalizowany przez Operatora jest tym samym dokumentem, który wgrał Klient. Jakakolwiek podmiana lub uszkodzenie pliku podważa wiarygodność całego procesu.
    *   **Implementacja w Teście:**
        *   Po wgraniu pliku przez Klienta (krok 1.4), test automatycznie oblicza jego sumę kontrolną (np. SHA-256).
        *   W kroku 2.4, kiedy Operator otwiera sprawę, test weryfikuje, czy suma kontrolna dokumentu widocznego w jego panelu jest **identyczna** z tą zapisaną w kroku 1.4.

2.  **Test Ścieżki Audytowej (Audit Trail):**
    *   **Problem Prawny:** W przypadku sporu musimy być w stanie odtworzyć każdą akcję w systemie. Kto, co i kiedy zrobił?
    *   **Implementacja w Teście:**
        *   Po każdym kluczowym kroku scenariusza E2E (np. wgranie pliku, zatwierdzenie przez operatora, akceptacja klienta), test wykonuje dodatkowe zapytanie do bazy danych, aby sprawdzić, czy w tabeli `historia_zdarzen` (`audit_logs`) powstał odpowiedni wpis z poprawnym `caseId`, `userId` i `timestamp`.

3.  **Test Kontroli Dostępu (Access Control):**
    *   **Problem Prawny:** Ochrona tajemnicy zawodowej i danych osobowych (RODO). Operator A nie może mieć wglądu w sprawy Operatora B, a Klient X nie może przypadkowo zobaczyć dokumentów Klienta Y.
    *   **Implementacja w Teście:**
        *   Należy stworzyć dodatkowy scenariusz, w którym logujemy się jako `inny_operator@example.com` i próbujemy uzyskać dostęp do sprawy stworzonej przez `test_klienta` (np. przez bezpośrednie wklejenie URL).
        *   Test musi potwierdzić, że system zwrócił błąd `403 Forbidden` (Brak dostępu) lub przekierował na stronę główną.

4.  **Test Odpowiedzialności (Accountability & Disclaimers):**
    *   **Problem Prawny:** Jasne określenie, że dokument został wygenerowany przez AI i zweryfikowany przez człowieka, jest kluczowe dla zarządzania odpowiedzialnością prawną.
    *   **Implementacja w Teście:**
        *   W kroku 3.6, po pobraniu finalnego dokumentu, test powinien nie tylko sprawdzić zmiany wprowadzone przez Operatora, ale również **przeanalizować treść pliku** w poszukiwaniu stopki lub klauzuli o treści: *"Niniejszy dokument został wygenerowany przy użyciu technologii AI i zweryfikowany przez Operatora nr [ID Operatora]"*.

Implementując testy w ten sposób, budują Państwo nie tylko sprawną aplikację, ale system o wysokim stopniu wiarygodności i bezpieczeństwa prawnego, co stanowi Państwa kluczową przewagę konkurencyjną.
