# Shui – API-specifikation

## 1. Generell information

Shui är ett REST API för en digital anslagstavla där användare kan publicera, läsa och hantera inlägg.

API:et har två funktionsnivåer:

- **G (Godkänt):** Grundläggande funktionalitet.
- **VG (Väl godkänt):** Utökad funktionalitet utöver G.

### Allmänna regler

- Alla request- och response bodies använder JSON.
- Alla inlägg och kommentarer har unika ID:n som genereras av servern.
- Registrerade användarnamn är unika och kan inte ändras.
- Datum anges i ISO 8601-format.
- Skyddade endpoints kräver `Authorization: Bearer <token>`.
- Användaridentiteten hämtas från en verifierad JWT, aldrig från request body.
- Listor returneras som JSON-arrayer. Om inga resultat finns returneras `[]`.
- Inlägg returneras med det senaste först.
- **VG:** Gäster ska kunna publicera inlägg utan JWT och tilldelas ett slumpmässigt användarnamn.
- **VG:** Inloggade användare ska kunna kommentera inlägg.

### Felhantering

Samtliga felmeddelanden följer formatet:

```json
{
  "error": "Beskrivning av felet"
}
```

| Statuskod | Betydelse |
|---|---|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 409 | Conflict |
| 500 | Internal Server Error |

---

## 2. Auth

### 2.1 Registrera användare (G)

**Metod:** `POST`

**Endpoint:** `/auth/register`

**RequestBody:**

```json
{
  "username": "jesper",
  "password": "Password123!"
}
```

**ResponseBody – 201 Created:**

```json
{
  "message": "User registered successfully",
  "username": "jesper"
}
```

**Felkoder:**

- `400` – Ogiltiga eller saknade uppgifter.
- `409` – Användarnamnet är redan registrerat.
- `500` – Internt serverfel.

**Beskrivning:**

Registrerar en ny användare med ett unikt användarnamn och lösenord. Lösenordet lagras som en hash och returneras aldrig till klienten.

### 2.2 Logga in (G)

**Metod:** `POST`

**Endpoint:** `/auth/login`

**RequestBody:**

```json
{
  "username": "jesper",
  "password": "Password123!"
}
```

**ResponseBody – 200 OK:**

```json
{
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "username": "jesper"
}
```

**Felkoder:**

- `400` – Ogiltiga eller saknade uppgifter.
- `401` – Felaktigt användarnamn eller lösenord.
- `500` – Internt serverfel.

**Beskrivning:**

Verifierar användarens inloggningsuppgifter. Vid lyckad inloggning returneras en signerad JWT med begränsad giltighetstid. Token innehåller användarens `username`.

---

## 3. Posts

### Datamodell

```json
{
  "id": "post-123",
  "username": "jesper",
  "isGuest": false,
  "title": "Mitt första inlägg",
  "content": "Hej på er allihop!",
  "createdAt": "2026-11-20T10:00:00Z",
  "updatedAt": "2026-11-20T10:00:00Z"
}
```

**VG:** Fältet `isGuest` används för att skilja gästinlägg från registrerade användares inlägg. För G räcker det att samtliga inlägg tillhör inloggade användare.

### 3.1 Hämta alla inlägg (G)

**Metod:** `GET`

**Endpoint:** `/posts`

**RequestBody:** Ingen.

**ResponseBody – 200 OK:**

```json
[
  {
    "id": "post-123",
    "username": "jesper",
    "isGuest": false,
    "title": "Mitt första inlägg",
    "content": "Hej på er allihop!",
    "createdAt": "2026-11-20T10:00:00Z",
    "updatedAt": "2026-11-20T10:00:00Z"
  }
]
```

**Felkoder:**

- `400` – Ogiltiga queryparametrar.
- `500` – Internt serverfel.

**Beskrivning:**

Hämtar samtliga inlägg, sorterade med det senaste först.

Det går även att filtrera på användarnamn genom queryparametern `username`.

Exempel: `GET /posts?username=jesper`

Om inga inlägg matchar returneras `[]`.

**VG:** Filtreringen ska även omfatta gästinlägg och deras automatiskt genererade användarnamn.

### 3.2 Hämta ett inlägg (G)

**Metod:** `GET`

**Endpoint:** `/posts/{postId}`

**RequestBody:** Ingen.

**ResponseBody – 200 OK:**

```json
{
  "id": "post-123",
  "username": "jesper",
  "isGuest": false,
  "title": "Mitt första inlägg",
  "content": "Hej på er allihop!",
  "createdAt": "2026-11-20T10:00:00Z",
  "updatedAt": "2026-11-20T10:00:00Z"
}
```

**Felkoder:**

- `404` – Inlägget finns inte.
- `500` – Internt serverfel.

**Beskrivning:**

Hämtar ett specifikt inlägg utifrån dess ID.

**VG:** Endpointen ska även kunna hämta gästinlägg.

### 3.3 Skapa ett inlägg (G + VG)

**Metod:** `POST`

**Endpoint:** `/posts`

**RequestBody:**

```json
{
  "title": "Mitt första inlägg",
  "content": "Hej på er allihop!"
}
```

**ResponseBody – 201 Created:**

```json
{
  "message": "Post created successfully",
  "post": {
    "id": "post-123",
    "username": "jesper",
    "isGuest": false,
    "title": "Mitt första inlägg",
    "content": "Hej på er allihop!",
    "createdAt": "2026-11-20T10:00:00Z",
    "updatedAt": "2026-11-20T10:00:00Z"
  }
}
```

**Felkoder:**

- `400` – Ogiltig eller saknad data.
- `401` – Saknad eller ogiltig JWT för G, eller ogiltig JWT som skickats med för VG.
- `500` – Internt serverfel.

**Beskrivning – G:**

Skapar ett nytt inlägg.

- En giltig JWT krävs.
- `username` hämtas från den verifierade JWT:n.
- Inlägget kopplas till den inloggade användaren.
- Anrop utan JWT ska returnera `401 Unauthorized`.

**Beskrivning – VG:**

Utöver funktionaliteten för G ska även gäster kunna skapa inlägg utan JWT.

- Om en giltig JWT skickas med skapas ett vanligt inlägg.
- Om ingen JWT skickas med skapas ett gästinlägg.
- Gästinlägg tilldelas automatiskt ett slumpmässigt användarnamn, exempelvis `unicorn123`.
- Gästinlägg ska ha `isGuest: true`.
- Vanliga inlägg ska ha `isGuest: false`.
- Om en ogiltig JWT skickas med returneras `401 Unauthorized`.

**ResponseBody – 201 Created (VG, gästinlägg):**

```json
{
  "message": "Post created successfully",
  "post": {
    "id": "post-456",
    "username": "unicorn123",
    "isGuest": true,
    "title": "Ett gästinlägg",
    "content": "Hej från en gäst!",
    "createdAt": "2026-11-20T11:00:00Z",
    "updatedAt": "2026-11-20T11:00:00Z"
  }
}
```

### 3.4 Uppdatera ett inlägg (G)

**Metod:** `PUT`

**Endpoint:** `/posts/{postId}`

**RequestBody:**

```json
{
  "title": "Uppdaterad rubrik",
  "content": "Detta är mitt uppdaterade inlägg."
}
```

**ResponseBody – 200 OK:**

```json
{
  "message": "Post updated successfully",
  "post": {
    "id": "post-123",
    "username": "jesper",
    "isGuest": false,
    "title": "Uppdaterad rubrik",
    "content": "Detta är mitt uppdaterade inlägg.",
    "createdAt": "2026-11-20T10:00:00Z",
    "updatedAt": "2026-11-20T12:00:00Z"
  }
}
```

**Felkoder:**

- `400` – Ogiltig eller saknad data.
- `401` – Saknad eller ogiltig JWT.
- `403` – Användaren saknar behörighet att uppdatera inlägget.
- `404` – Inlägget finns inte.
- `500` – Internt serverfel.

**Beskrivning – G:**

Uppdaterar titel och innehåll på ett befintligt inlägg.

- En giltig JWT krävs.
- Endast inläggets ägare får uppdatera det.
- Ägarskapet avgörs genom att jämföra inläggets `username` med användarnamnet i JWT:n.
- `id`, `username` och `createdAt` får inte ändras.
- `updatedAt` uppdateras automatiskt.

**VG:** Gästinlägg får inte uppdateras, även om en registrerad användare har samma användarnamn. Vid auktorisering ska därför även `isGuest: false` kontrolleras.

### 3.5 Radera ett inlägg (G)

**Metod:** `DELETE`

**Endpoint:** `/posts/{postId}`

**RequestBody:** Ingen.

**ResponseBody – 204 No Content:** Ingen.

**Felkoder:**

- `401` – Saknad eller ogiltig JWT.
- `403` – Användaren saknar behörighet att radera inlägget.
- `404` – Inlägget finns inte.
- `500` – Internt serverfel.

**Beskrivning – G:**

Raderar ett befintligt inlägg.

- En giltig JWT krävs.
- Endast inläggets ägare får radera det.
- Ägarskapet avgörs genom att jämföra inläggets `username` med användarnamnet i JWT:n.

**Beskrivning – VG:**

- Gästinlägg får inte raderas, även om användarnamnet matchar en registrerad användares.
- Vid auktorisering ska även `isGuest: false` kontrolleras.
- Om inlägget har kommentarer ska även dessa raderas.

---

## 4. Comments (VG)

**Hela denna resurs ingår endast i VG-funktionaliteten.**

### Datamodell

```json
{
  "id": "comment-123",
  "postId": "post-123",
  "username": "jonte",
  "content": "Vilket bra inlägg!",
  "createdAt": "2026-11-20T13:00:00Z"
}
```

### 4.1 Hämta kommentarer (VG)

**Metod:** `GET`

**Endpoint:** `/posts/{postId}/comments`

**RequestBody:** Ingen.

**ResponseBody – 200 OK:**

```json
[
  {
    "id": "comment-123",
    "postId": "post-123",
    "username": "jonte",
    "content": "Vilket bra inlägg!",
    "createdAt": "2026-11-20T13:00:00Z"
  }
]
```

**Felkoder:**

- `404` – Inlägget finns inte.
- `500` – Internt serverfel.

**Beskrivning:**

Hämtar samtliga kommentarer till ett specifikt inlägg.

- Endpointen är publik.
- Kommentarerna returneras med den äldsta först.
- Om inlägget finns men saknar kommentarer returneras `[]`.
- Kommentarer ska kunna hämtas från både vanliga inlägg och gästinlägg.

### 4.2 Skapa en kommentar (VG)

**Metod:** `POST`

**Endpoint:** `/posts/{postId}/comments`

**RequestBody:**

```json
{
  "content": "Vilket bra inlägg!"
}
```

**ResponseBody – 201 Created:**

```json
{
  "message": "Comment created successfully",
  "comment": {
    "id": "comment-123",
    "postId": "post-123",
    "username": "jonte",
    "content": "Vilket bra inlägg!",
    "createdAt": "2026-11-20T13:00:00Z"
  }
}
```

**Felkoder:**

- `400` – Ogiltig eller saknad data.
- `401` – Saknad eller ogiltig JWT.
- `404` – Inlägget finns inte.
- `500` – Internt serverfel.

**Beskrivning:**

Skapar en kommentar till ett befintligt inlägg.

- En giltig JWT krävs.
- `username` hämtas från den verifierade JWT:n.
- Kommentaren kopplas till inlägget genom `postId`.
- Det ska gå att kommentera både vanliga inlägg och gästinlägg.
- Kommentarens innehåll ska valideras.

---

## 5. Sammanställning av endpoints

| Metod | Endpoint | Nivå | Åtkomst |
|---|---|---|---|
| POST | `/auth/register` | G | Publik |
| POST | `/auth/login` | G | Publik |
| GET | `/posts` | G | Publik |
| GET | `/posts/{postId}` | G | Publik |
| POST | `/posts` | G/VG | G: Inloggad, VG: Även gäst |
| PUT | `/posts/{postId}` | G | Inloggad ägare |
| DELETE | `/posts/{postId}` | G | Inloggad ägare |
| GET | `/posts/{postId}/comments` | VG | Publik |
| POST | `/posts/{postId}/comments` | VG | Inloggad |

**Totalt: 7 endpoints för G och 2 ytterligare endpoints för VG.**

VG omfattar dessutom gästpublicering och tillhörande regler för ägarskap på befintliga endpoints.
