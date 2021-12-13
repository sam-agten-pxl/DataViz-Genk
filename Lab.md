# Data Visualisatie: Genk Abonnees
Tijd dat we eens wat dieper graven met de kennis die we opgedaan hebben om een grotere visualisatie te maken. 

## Verkenning Project
Normaal gezien heb je het project kunnen downloaden via github (idealiter heb je dat via git clone gedaan!). Het startersproject is geen magie. Het is een basis Vue-project dat gemaakt is met `vue create`. Daarna is de hello world component verwijderd. Er zijn nog enkele aanpassingen gebeurd:

- In de public folder is het logo van Genk toegevoegd.
- In de public folder is het .csv bestandje ook al toegevoegd (`abonnees_genk.csv`).
- In de `src/assets/` folder staat ook het logo van Genk zodat we dat kunnen gebruiken in onze html.
- In de components folder is een Vue component voorgemaakt: `Card.vue`. Die gaan we gebruiken in onze visualisatie. Die component moet je niet zelf kunnen maken. Je mag uiteraard gerust eens kijken naar de broncode van deze component!
- In `App.vue` onder de `style` tags is het standaard font ingesteld op SansaPro, de huisstijl van Genk. Dat font is meegeleverd in de map `src/fonts`. In de `style` tags importeren we dat font en stellen we dat in als de de facto standaard voor het project.
- D3 is al geinstalleerd voor het project.

Voordat we beginnen, kijk eerst eens of je het project aan de praat krijgt:

    npm run serve

Je zou in het groot het logo van Genk moeten zien.

## Inlezen van de data
We gaan het het aantal abonnees per postcode visualiseren. We willen per postcode weten wat de naam is van de gemeente en hoeveel abonnees er in die postcode zitten. We zijn enkel geïnteresseerd in gemeentes met minstens 100 abonnees.

We gaan de data stapsgewijs inlezen. Als algemene tip: voeg zo weinig mogelijk code toe voor je test. Gebruik `console.log()` om de tussentijdse resultaten te inspecteren.

Open `App.vue`. Voeg een `mounted()` functie toe aan de App component. We markeren deze functie als `async` zoals we gezien hebben in het lab rond data processing. Vervolgens lezen we de data in:

    async mounted() {
        const data = await d3.csv('abonnees_genk.csv');
        console.log(data);
    }

Vergeet niet dat je ook D3 moet importeren in je component:

    import * as d3 from "d3";

## Filteren op kolom

We gaan eerste filteren op kolom. We willen enkel de postcode en de naam van de gemeente behouden. Op dit moment is de postcode ook een stukje tekst. We gaan de postcode transformeren naar een getal (hoewel we in dit project niet gaan 'rekenen' met postcodes, dus in principe kan de postcode ook tekst blijven). Daarvoor voegen we onder de `methods` van onze App component een nieuwe methode toe:

    methods: {
        typeConversion: function(d) {
            return {
                postcode: +d.Postcode,
                city: d.City
            }
        }
    }

Deze methode krijgt elke rij binnen van het .csv bestandje en voor elke rij maakt die een nieuw objectje aan dat bestaat uit `postcode` en `city`. Voor `postcode` kopieren we de waarde uit het veld `Postcode` (met hoofdletter! Want zo heet de kolom in het csv-bestand) als een getal. De conversie gebeurt impliciet dankzij het `+`-teken. In het veld `city` kopiëren we gewoon de waarde uit de `City` kolom.

We geven deze methode ook mee aan de methode die de data inleest. D3 zal dan automatisch onze methode oproepen voor elke rij:

    const data = await d3.csv('abonnees_genk.csv', this.typeConversion);

Run opnieuw je project en inspecteer de data die uitgelezen wordt. Heb je 2 veldjes per rij? Staan daar de waardes in die je zou verwachten?
    
## Filteren op rij

We filteren ook enkele rijen er uit. Het zou bijvoorbeeld kunnen dat er foutieve data is ingevoerd of dat de postcode ongeldig is. We beschouwen een postcode geldig als die bestaat uit 4 cijfers. Het veld van de postcode mag ook niet leeg zijn. We voegen opnieuw een methode toe aan onze component onder `methods` (we gaan vanaf hier niet elke keer opnieuw vermelden dat je een methode kan toevoegen aan een component onder de `methods` property):

    filterData: function(d) {
      const filtered = d.filter(r=> {
        return r.postcode && r.postcode > 999 && r.postcode < 10000;
      });
      return filtered;
    }

Deze methode krijgt ook een parameter `d` binnen, maar hier is `d` de gehele rij aan objecten die we ingelezen hebben. Die rij van objecten gaan we filteren: we behouden enkel postcodes die bestaan (`r.postcode`) en die uit 4 cijfers bestaan (`r.postcode > 999 && r.postcode < 10000`). Hier helpt het natuurlijk dat we de postcode kunnen behandelen als een cijfer. Als je postcode hebt gelaten als tekst dan ga je het aantal karakters moeten bekijken.

We gaan deze functie ook oproepen nadat we onze csv hebben ingelezen:

    const data = await d3.csv('abonnees_genk.csv', this.typeConversion);
    const filteredData = this.filterData(data);
    console.log(filteredData);

Run opnieuw je project en inspecteer je resultaat. We blijven over met <b>12 145</b> datapunten.


## Dataverwerking

Nu hebben we een rij van objecten. Elk object bevat een postcode en de naam van een gemeente. Het object `{postcode: 3600, city: "Genk"}` zal er dus meerdere keren tussenzitten. We gaan nu `d3.rollup` gebruiken om onze data te groeperen. Op die manier kunnen we achterhalen hoeveel abonnees er zijn per gemeente.

    prepareData : function(data)
    {
      const dataMap = d3.rollup(
          data,
          r => d3.count(r, x => x.postcode),
          d => d.postcode
      )

      const dataArray = Array.from(dataMap, d => ({ postcode: d[0], amount: d[1], city: data.find(e => e.postcode == d[0]).city.toUpperCase() }));
      const filteredDataArray = dataArray.filter(r => {
        return r.amount > 99;
      })
      return filteredDataArray;
    }

Er gebeurt hier redelijk wat. Dit werd al behandeld in het lab van data processing maar hier geven we nog eens een kort overzicht:

    const dataMap = d3.rollup(
          data,
          r => d3.count(r, x => x.postcode),
          d => d.postcode
      )

De rollup functie gaat de volledige lijst van 12 145 datapunten nemen (`data`) en gaat die groeperen op basis van postcode (`d => d,postcode`). Per groepje berekent rollup een resultaat. In ons geval geven we de instructie om per groepje `count` uit te voeren, oftewel het aantal elementen tellen per groepje (`r => d3.count(r, x => x.postcode`)). De `x => x.postcode` is nodig omdat je aan d3 moet vertellen welke element er geteld mogen worden (we bijvoorbeeld enkel de elementen in een groepje kunnen tellen waarvan de waarde boven een bepaald getal ligt). Hier zeggen we tegen d3 dat elk element in het groepje geteld mag worden als de postcode niet leeg is. In ons geval is dat dus elk element want de elementen zonder postcode hebben we er al uitgefilterd. Print het resultaat (`dataMap`) van deze rollup gerust eens naar je console. Je krijgt een datastructuur die vol zit met dit soort zaken:

    {3620 => 277}

Rollup maakt een `map` structuur. Elk element in een map bestaat uit een sleutel (`3620`) en een waarde (`277`). De waarde hebben we laten berekenen als de `count` van een groepje, dus met andere woorden: postcode 3620 komt 277 voor in onze data. 

Nu moeten we deze map terug vertalen naar een rij/array. Dat doen we hier: 

    const dataArray = Array.from(dataMap, d => ({ postcode: d[0], amount: d[1], city: data.find(e => e.postcode == d[0]).city.toUpperCase() }));

We nemen de `dataMap` en voor elk element maken we een nieuw object dat bestaat uit 3 velden:
- postcode
- amount
- city

Denk eraan dat datamap enkel bestaat uit element zoals `{3620 => 277`}. De naam van de gemeente staat daar dus niet meer in. Voor de postcode nemen we het eerste deel van een element uit `dataMap`: `d[0]`. Amount is het tweede gedeelte van een element: `d[1]`. De gemeente hebben we niet meer. Daarom nemen we het eerste gedeelte van een element uit de dataMap, de postcode, en gaan we zoeken in de oorspronkelijke lijst van datapunten. We lezen uit wat de waarde was van de gemeente voor het eerste element dat we vinden:

    city: data.find(e => e.postcode == d[0]).city.toUpperCase()

We zetten die naam ook in hoofdletters. 

Ten slotte doen we nog een kleine filtering op onze nieuwe data:
    
    const filteredDataArray = dataArray.filter(r => {
        return r.amount > 99;
      })

We verwijderen elk object uit onze `dataArray` waarvan `amount` kleiner is dan 100. 

Vergeet de methode ook niet aan te roepen:

    const data = await d3.csv('abonnees_genk.csv', this.typeConversion);
    const filteredData = this.filterData(data);
    const preparedData = this.prepareData(filteredData);
    console.log(preparedData);




