# Examination – Shui

[Inspelad genomgån](https://funet.sharepoint.com/:v:/s/MobilapplikationsutvecklareYhdistans-Klass25/IQAPwtcyBuiQTLIgIahCFwl3AVFKwlgtwdeSCvs-wn6vLBM?e=X1IgDt)

## Uppgiftsbeskrivning

I denna examination ska du utveckla **Shui**, ett REST API för en digital anslagstavla där användare kan registrera sig, logga in och publicera inlägg.

Applikationen ska byggas som en serverless backend med AWS Lambda, API Gateway och DynamoDB. API:et ska driftsättas och vara tillgängligt via en publik URL.

Samtliga endpoints, request- och responseformat samt felkoder finns beskrivna i [API-dokumentationen](./api-spec.md).

## Krav för Godkänt (G)

- Implementera samtliga endpoints markerade med **G** i API-dokumentationen.
- Registrering och inloggning med bcrypt och JWT.
- Endast inloggade användare får skapa inlägg.
- Endast inläggets ägare får redigera och radera det.
- Alla ska kunna läsa inlägg och filtrera dem efter användarnamn.
- Validera inkommande data med Zod.
- Hantera fel och returnera korrekta HTTP-statuskoder.
- Lagra användare och inlägg i DynamoDB.
- Driftsätt API:et på AWS.

## Krav för Väl godkänt (VG)

Samtliga krav för G ska vara uppfyllda. Dessutom ska du:

- Implementera all funktionalitet markerad med **VG** i API-dokumentationen.
- Tillåta gäster att publicera inlägg utan inloggning. Gäster ska tilldelas automatiskt genererade användarnamn.
- Låta inloggade användare kommentera inlägg och alla besökare läsa kommentarer.
- Använda en tydlig och genomtänkt kodstruktur där:
  - Autentisering, auktorisering och validering hanteras genom återanvändbara middlewares.
  - Databasoperationer är separerade från övrig logik.
  - Affärslogik är separerad från HTTP-hanteringen.
  - Handlers hålls kortfattade och fokuserar på HTTP-anrop och svar.

## Tekniker

- Node.js och JavaScript
- AWS Lambda och API Gateway
- DynamoDB
- Serverless Framework
- Middy
- bcrypt och JWT
- Zod
- Git och GitHub

Ingen frontend ska utvecklas.

## Inlämning

<text weight="medium">Deadline: Söndag 29 november 2026 kl. 23:59</text>

Inlämningen sker via **Moodle** och ska innehålla:

1. Länk till ditt GitHub-repository.
2. URL till ditt driftsatta API.

Ditt repository ska innehålla en README med instruktioner för projektet, information om hur API:et körs och en länk till det driftsatta API:et.

**Observera:** API:et ska vara driftsatt och fungerande vid inlämning. Det är inte tillräckligt att enbart lämna in källkoden.
