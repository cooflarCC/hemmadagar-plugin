# Hemmadagar för Claude

Version 0.1.0

> **Status 2026-10-28: inte i drift.** Repot är privat och servern på
> `hemmadagar.pages.dev/mcp` är inte publicerad ännu. Pluginen fungerar först
> när Hemmadagar har lanserats. Då görs repot publikt och adressen byts till
> den riktiga domänen.

Hemmadagar räknar på svensk föräldrapenning, föräldralön och hur
föräldraledigheten bör fördelas mellan två föräldrar. Det sker med samma
regelmotor som webbappen: 2026 års regler, skatt per kommun,
kollektivavtalens föräldralön och löneavdrag, helgregeln, SGI-skyddet,
semester och förskoleavgift. Ingen inloggning behövs.

## Så fungerar pluginen

Pluginen har två delar:

- **Skillen `hemmadagar`** är arbetsinstruktionen. Den säger vilka uppgifter
  Claude ska fråga efter, vilket verktyg som svarar på frågan och hur svaret
  ska redovisas, med varningar, källor och friskrivning.
- **Anslutningen `hemmadagar`** i `.mcp.json` är Hemmadagars MCP-server. Det
  är den som räknar. Claude räknar aldrig själv på dagbelopp, skatt eller
  föräldralön.

Utan nyckel ger servern gratisnivån. Har familjen köpt familjerapporten ger
köpkoden hela svaret; lägg då till anslutningen med adressen
`https://hemmadagar.pages.dev/mcp?nyckel=<köpkod>` i stället.

## Installera en gång

I Claude på webben eller i appen:

1. Öppna **Anpassa > Plugins**.
2. Välj **Lägg till > Lägg till marknadsplats** och skriv
   `cooflarCC/hemmadagar-plugin`.
3. Slå på **Synka automatiskt** för marknadsplatsen.
4. Lägg till **Hemmadagar** från marknadsplatsen.

I Claude Code:

```bash
claude plugin marketplace add cooflarCC/hemmadagar-plugin
claude plugin install hemmadagar@hemmadagar
```

Så länge repot är privat behöver den som installerar läsrätt till det på
GitHub.

Fråga sedan till exempel: "Vi väntar barn i mars. Jag tjänar 42 000 kr i
månaden med kommunalt avtal och min partner 62 000 kr på ett teknikföretag.
Hur bör vi dela på föräldraledigheten?"

## Källa och uppdateringar

Skillen underhålls i Hierateks repo under
`affarsplaner/foraldrapenning/produkt/skill/hemmadagar/SKILL.md`, där ett
test håller den i takt med serverns verktyg. Filen här är en kopia. Nya
versioner kommer automatiskt när marknadsplatsen synkas. Verktygen på servern
uppdateras direkt för alla.

Hemmadagars svar är vägledande beräkningar. De ersätter inte beslut från
Försäkringskassan eller besked från arbetsgivaren.
