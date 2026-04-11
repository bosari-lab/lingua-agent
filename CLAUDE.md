# Claude Code: AI Language Learning Agent

## 1. Stack Techniczny
- **Frontend**: React + Tailwind CSS
- **Backend**: FastAPI (Python)
- **AI**: OpenAI SDK / LangChain
- **Baza danych**: PostgreSQL (Supabase)
- **Komunikacja**: REST API

## 2. Strategia Oszczędzania Tokenów (Efficiency)
- **Plan Mode**: Zawsze opisz logikę funkcji/komponentu w punktach przed pisaniem kodu. Czekaj na "OK".
- **Partial Updates**: Jeśli zmieniamy jedną funkcję, nie generuj całego pliku. Pokaż tylko zmieniony fragment.
- **No Comments**: Nie dodawaj oczywistych komentarzy (np. // funkcja do logowania). Kod ma być czysty i samowyjaśniający się.
- **Depedencies**: Zanim dodasz nową bibliotekę, zapytaj czy jest niezbędna.

## 3. Standardy Kodowania (Coding Rules)
- **Modułowość**: Rozdzielaj logikę AI od API i Frontendu.
- **Typowanie**: W Pythonie używaj Type Hinting, w JS używaj czystego kodu z dokumentacją JSDoc.
- **Obsługa błędów**: Każdy call do LLM musi mieć obsługę wyjątków (try/except) i limit timeoutu.
- **Prompt Engineering**: Prompty dla Agenta trzymaj w oddzielnych plikach/stałych, nie hardkoduj ich wewnątrz funkcji.

## 4. Zadania Agenta Językowego
- **Persona**: Agent ma być cierpliwym nauczycielem, korygującym błędy w czasie rzeczywistym.
- **Spójność**: Pilnuj, aby system oceniania gramatyki był logiczny i zgodny z poziomami CEFR (A1-C2).
- **Logika**: Przy generowaniu ćwiczeń, najpierw pobierz kontekst użytkownika z bazy.

## 5. Komunikacja
- **Język**: Feedback dla mnie po polsku. Kod i dokumentacja w kodzie po angielsku.
- **Błędy**: Jeśli widzisz błąd w mojej architekturze – wytykaj go bez litości, zanim zaczniesz pisać.
- **Exit Signal**: Gdy skończysz zadanie (naprawisz kod lub przeanalizujesz punkt regulaminu), zakończ odpowiedź tagiem <DONE>. Nie dodawaj po nim żadnych wyjaśnień ani pytań "czy mogę jeszcze w czymś pomóc".

## 6. Zasady Inżynierskie
- **Minimalist Changes**: Rób najmniejsze możliwe zmiany, aby osiągnąć cel. Nie zmieniaj kodu, o który nie prosiłem.
- **Fight Entropy**: Kod musi być czysty i lepszy niż go zastałeś. Unikaj "hacków".
- **JSDoc/ABOUTME**: Każdy nowy plik zaczynaj od komentarza `// ABOUTME: [cel pliku]`. To pomoże ci (i mi) szybko zrozumieć architekturę przy następnej sesji bez skanowania całego pliku.