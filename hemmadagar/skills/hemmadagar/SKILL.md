---
name: hemmadagar
description: Räkna på svensk föräldrapenning, föräldralön och föräldraledighet med Hemmadagars MCP-verktyg. Använd när användaren har konkreta uppgifter (månadslön, kommun, kollektivavtal, barnets födelsedatum, förskolestart) och vill veta dagbelopp, hur ledigheten bör fördelas mellan två föräldrar, vad en egen plan ger netto, eller vad en regel som helgregeln eller SGI-skyddet innebär. Använd den inte för allmänna frågor utan siffror att räkna på, för barn födda utanför Sverige eller för tillfällig föräldrapenning vid vård av sjukt barn. Kräver att Hemmadagar-anslutningen (/mcp) är tillagd.
---

# Hemmadagar för AI-agenter

Hemmadagar-servern räknar deterministiskt med samma regelmotor som webbappen:
2026 års regler för föräldrapenning, skatt per kommun, tjugo kollektivavtals-
modeller för föräldralön och löneavdrag, helgregeln, SGI-skyddet, semester,
pension och förskoleavgift. Din uppgift är att samla rätt indata, anropa rätt
verktyg och redovisa resultatet med varningar, källor och friskrivning. Räkna
inte själv på dagbelopp, skatt eller föräldralön; låt verktygen göra det.

Gäller Hemmadagar MCP med regelmotor 2026.2.0, den publika anslutningen: fem
verktyg, ingen inloggning. Utan nyckel ger `optimera_plan` gratisnivån
(referensen och vad det bästa förslaget ger per månad, inga datum) och
`simulera_plan` en hänvisning till webbappen. Med köpkoden från
familjerapporten som nyckel (`Authorization: Bearer cs_…` eller `?nyckel=cs_…`
i adressen) ger båda hela svaret. Koden gäller familjen som köpte rapporten;
be aldrig användaren om någon annans kod.

Ge alltid användaren länken `oppnaIHemmadagar` och rekommendera att öppna
den i webbläsaren. Där visas planen som tidslinje med belopp per månad,
att-göra-lista och ansökningslista, vilket är lättare att följa än text. Med
köpkod öppnas planen upplåst. Filen hålls i takt med verktygen i
`src/mcp/server.ts` och serverns instruktion där; `test/skill.test.ts`
jämför dem.

## När skillen ska användas

- Användaren väntar eller har fått barn och vill veta hur föräldrapenning,
  föräldralön och semester bör läggas upp mellan två föräldrar.
- Användaren vill veta vad en viss månadslön ger per dag eller månad.
- Användaren har en egen plan och vill veta vad den ger netto och om den
  bryter mot någon regel.
- Användaren frågar vad en regel innebär och vill ha paragraf och källa.

Använd inte räkneverktygen för allmänna frågor utan siffror (vad är
föräldrapenning, hur fungerar förskolan), för vård av sjukt barn eller för
personer som inte är försäkrade i Sverige. Svara då själv, eller hänvisa till
guiden på webbplatsen (`/wiki/`).

Saknas verktygen i sessionen är anslutningen inte tillagd. Hänvisa då till
`/ai/` på webbplatsen och gissa inga belopp i Hemmadagars namn.

## Arbetsgång

1. Samla fakta per förälder: månadslön före skatt, kommun (för skattesatsen),
   kollektivavtal (`lista_avtalsmodeller` ger giltiga id:n; `utan` om inget),
   anställningstid vid födseln i månader, semesterdagar och övrig betald
   ledighet att använda (flex, komp) i dagar. Inga personnummer eller namn;
   etiketter som "Förälder 1" räcker.
2. Samla familjens fakta: barnets födelsedatum (eller beräknat), tidigaste
   och senaste förskolestart, antal barn, ensam vårdnad, och för pågående
   ärenden hur många dagar som är kvar per förälder och nivå.
3. Fråga vad som är viktigt: pengar mot längd, jämn fördelning, lägsta netto
   per månad hushållet klarar, vem som börjar, dubbeldagar, låsta perioder.
   Verktyget har rimliga standardvärden; fråga bara efter det som saknas.
4. Anropa verktyget som svarar på frågan (tabellen nedan). För en hel
   familjeplanering är `optimera_plan` startpunkten.
5. Redovisa: förslagens namn och motivering, netto per månad för hushållet,
   skillnaden mot att vara lediga rakt igenom, dagar kvar, varningarna med
   allvarlighet, och länken `oppnaIHemmadagar` som öppnar förslaget i
   webbappen. Avsluta alltid med friskrivningen i svaret.

## Verktyg

| Verktyg | Använd när | Viktigast indata |
|---|---|---|
| `berakna_foraldrapenning` | En månadslön: SGI, dagbelopp på sjukpenningnivå och lägstanivå, ungefärlig månadsersättning | `manadslon`, valfritt `ar`, `omfattning` |
| `optimera_plan` | Två föräldrar vill ha 3 till 5 förslag på uttagsplan med netto, föräldralön och varningar | `familj`, valfritt `prioriteringar` |
| `simulera_plan` | Användaren har en egen plan (segment per förälder) och vill veta vad den ger | `familj`, `segment` |
| `forklara_regel` | Vad en regel innebär, med paragraf, källa och verifieringsnivå; utan id listas alla | valfritt `regel_id` |
| `lista_avtalsmodeller` | Vilka kollektivavtal som finns som modell och vad de ger | inga |

## Så läser du svaren

- `oppnaIHemmadagar` är en länk som bär uppgifterna i adressens del efter `#`;
  den skickas aldrig till servern. Ge den till användaren och säg att den
  öppnar förslaget i webbappen där det kan ändras, exporteras och delas.
- `varningar` har `kod`, `allvarlighet`, `text` och `regel`. Återge dem med
  deras allvarlighet; en varning om helgregeln eller SGI-skyddet betyder att
  planen kan ge lägre ersättning än beräknat.
- `jamfortMedRaktIgenom` visar vad förslaget ger utöver att vara lediga
  hälften var, fem dagar i veckan, till senaste förskolestart. Det är den
  siffra användaren oftast vill ha.
- Källor har verifieringsnivå A (primärkälla läst), B (sekundärkälla) eller
  C (ej belagd). Säg när en uppgift bygger på nivå C.
- Varje svar bär `ansvar`. Upprepa det:
  vägledande beräkning; ersätter inte beslut från Försäkringskassan eller besked från arbetsgivaren.

## Vad verktygen inte gör

- Hämtar inget från Försäkringskassan eller arbetsgivaren; alla uppgifter
  kommer från användaren.
- Skickar ingen ansökan. Ansökningslistan i svaret för användaren in själv.
- Modellerar inte skift eller oregelbunden arbetstid, och räknar slutlig
  årsskatt fördelad per månad, inte preliminärskatt enligt tabell.
