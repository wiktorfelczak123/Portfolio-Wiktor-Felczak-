# Przewodnik po analizie logów błędów

## Wprowadzenie

Ten przewodnik opisuje praktyczne podejście do analizy logów aplikacyjnych, ze szczególnym uwzględnieniem:
- komunikatów JSON,
- kodów odpowiedzi HTTP,
- błędów integracji API.

Celem jest szybka identyfikacja przyczyny problemu i przekazanie zwięzłej, kompletnej informacji zespołowi IT lub developerom.

Dokument powstał na podstawie doświadczeń z pracy w supportcie aplikacji biznesowych. Wszystkie przykłady są fikcyjne lub zanonimizowane.

---

## 1. Struktura typowego logu

Przykładowy log z aplikacji integrującej się przez API:

```json
{
  "timestamp": "2026-09-16T13:51:00Z",
  "level": "ERROR",
  "service": "payment-gateway",
  "message": "Request failed with status 401",
  "details": {
    "endpoint": "/api/v1/transactions",
    "method": "POST",
    "request_id": "a1b2c3d4",
    "response": {
      "status": 401,
      "body": {
        "error": "Unauthorized",
        "code": "AUTH_001",
        "description": "Invalid or expired token"
      }
    }
  }
}
```

Na co zwrócić uwagę:

· timestamp – czas wystąpienia błędu,
· level – waga zdarzenia (ERROR, WARN, INFO),
· service – nazwa usługi, która zgłosiła błąd,
· message – krótki opis,
· details.endpoint – adres, pod który wysłano żądanie,
· details.method – metoda HTTP,
· details.request_id – identyfikator do korelacji z logami po stronie serwera,
· details.response.status – kod HTTP,
· details.response.body.error – nazwa błędu,
· details.response.body.code – wewnętrzny kod błędu,
· details.response.body.description – opis słowny.

---

2. Najczęstsze kody HTTP i ich znaczenie

Kod Znaczenie Typowa przyczyna Co zrobić
200 OK Sukces Brak akcji.
201 Created Zasób utworzony Brak akcji.
400 Bad Request Błędne dane wejściowe Sprawdź format JSON, wymagane pola, typy danych.
401 Unauthorized Brak lub nieprawidłowy token Sprawdź token, jego ważność i uprawnienia.
403 Forbidden Brak dostępu Sprawdź uprawnienia użytkownika lub klucza API.
404 Not Found Zły endpoint lub zasób Zweryfikuj URL i identyfikator zasobu.
409 Conflict Konflikt danych Sprawdź, czy zasób już istnieje lub czy nie ma konfliktu wersji.
422 Unprocessable Entity Błąd walidacji Sprawdź komunikaty walidacyjne w ciele odpowiedzi.
429 Too Many Requests Przekroczony limit zapytań Sprawdź limity API, zastosuj backoff.
500 Internal Server Error Błąd po stronie serwera Przekaż do IT/developera z pełnym logiem.
502 Bad Gateway Problem z bramą Sprawdź, czy usługa nadrzędna działa poprawnie.
503 Service Unavailable Usługa niedostępna Sprawdź, czy nie ma planowanej przerwy.
504 Gateway Timeout Przekroczony czas oczekiwania Sprawdź, czy usługa odpowiada w rozsądnym czasie.

---

3. Analiza komunikatów JSON

Przy analizie logów JSON warto zwrócić uwagę na:

· pole error lub code – identyfikator błędu,
· pole description – opis słowny,
· request_id – do korelacji z logami po stronie serwera,
· timestamp – czas wystąpienia.

Przykład błędu walidacji

```json
{
  "status": 400,
  "error": "ValidationError",
  "details": [
    {
      "field": "email",
      "message": "Invalid email format"
    },
    {
      "field": "phone",
      "message": "Required field missing"
    }
  ]
}
```

Przykład błędu autoryzacji

```json
{
  "status": 401,
  "error": "Unauthorized",
  "code": "AUTH_001",
  "description": "Invalid or expired token",
  "request_id": "a1b2c3d4"
}
```

Przykład błędu serwera

```json
{
  "status": 500,
  "error": "InternalServerError",
  "code": "SRV_500",
  "description": "Unexpected error occurred",
  "request_id": "e5f6g7h8"
}
```

---

4. Błędy integracji API – podejście krok po kroku

1. Sprawdź endpoint – czy metoda i URL są poprawne.
2. Zweryfikuj nagłówki – np. Authorization, Content-Type.
3. Sprawdź token – czy jest ważny i nie wygasł.
4. Przeanalizuj ciało odpowiedzi – kod błędu, opis, request_id.
5. Skoreluj request_id z logami po stronie usługi.
6. Jeśli błąd jest po stronie serwera (5xx) – eskaluj do IT/developera z pełnym logiem.
7. Jeśli błąd jest po stronie klienta (4xx) – zweryfikuj dane wejściowe i uprawnienia.

---

5. Najczęstsze błędy i ich rozwiązania

Kod błędu Opis Prawdopodobna przyczyna Rozwiązanie
AUTH_001 Invalid or expired token Token wygasł lub jest błędny Odśwież token, sprawdź czas życia.
AUTH_002 Missing token Brak nagłówka Authorization Dodaj nagłówek z tokenem.
VAL_001 Validation error Błędne dane wejściowe Sprawdź pola wymagane i format.
SRV_500 Internal server error Błąd po stronie serwera Przekaż do IT z request_id.
NET_001 Connection timeout Problem z siecią Sprawdź łączność, ponów próbę.
RATE_001 Too many requests Przekroczony limit Zastosuj backoff, sprawdź limity.

---

6. Checklista dla supportu przed eskalacją

□ Sprawdzony endpoint i metoda HTTP.
□ Zweryfikowane nagłówki (Authorization, Content-Type).
□ Sprawdzona ważność tokenu.
□ Przeanalizowane ciało odpowiedzi (JSON).
□ Zapisany request_id i timestamp.
□ Sprawdzone, czy błąd jest powtarzalny.
□ Przygotowany pełny opis problemu dla IT.

---

7. Przykład rozwiązania problemu

Problem: Aplikacja zwraca błąd 401 przy próbie pobrania danych klienta.

Analiza:

· Log wskazuje na AUTH_001 – nieprawidłowy token.
· Token został wygenerowany ponad 24 godziny temu.
· System wymaga odświeżenia tokenu co 12 godzin.

Rozwiązanie:

· Zaktualizowano procedurę odświeżania tokenu.
· Dodano monitoring czasu życia tokenu.
· Problem nie wystąpił ponownie.

---

8. Dobre praktyki

· Zawsze zapisuj request_id – to klucz do korelacji logów.
· Nie eskaluj bez wstępnej analizy – zbierz komplet informacji.
· Dokumentuj powtarzające się błędy – twórz bazę wiedzy.
· Standaryzuj opisy błędów – ułatwia to komunikację z IT.
· Testuj przypadki brzegowe – wiele błędów wynika z nietypowych danych.

---

9. Wnioski

· Analiza logów pozwala szybko zawęzić obszar problemu.
· Kluczowe jest zrozumienie kodów HTTP i struktur JSON.
· Dobrze opisany błąd (z request_id, kodem i opisem) skraca czas reakcji IT.
· Standaryzacja i dokumentacja błędów przekładają się na realne oszczędności czasu.

---

10. Słownik pojęć

· API – interfejs programowania aplikacji.
· JSON – lekki format wymiany danych.
· HTTP – protokół przesyłania danych w sieci.
· request_id – unikalny identyfikator żądania.
· endpoint – adres URL, pod który wysyłane jest żądanie.
· token – klucz dostępu, często z ograniczonym czasem życia.
· backoff – strategia ponawiania prób z rosnącym opóźnieniem.

---

Dokument przygotowany na podstawie doświadczeń zawodowych. Wszystkie dane są fikcyjne lub zanonimizowane.