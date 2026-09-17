# Przewodnik po analizie logów błędów

Ten dokument opisuje podejście do analizy logów aplikacyjnych, ze szczególnym uwzględnieniem:
- komunikatów JSON,
- kodów odpowiedzi HTTP,
- błędów integracji API.

Celem jest szybka identyfikacja przyczyny problemu i przekazanie zwięzłej informacji zespołowi IT lub developerom.

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