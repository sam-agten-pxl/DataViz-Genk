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

Run je project en inspecteer het resultaat. Je zou <b>29</b> resultaten moeten hebben.

## Eerste visualisatie

Het meest vervelende stuk is uit de weg. We hebben nu werkbare data. 

We passen eerst de html aan van onze `App.vue`: 

    <template>
        <div id="app">
            <div id="content">
                <svg id="canvas" :width=width height=1000>
                </svg>
            </div>
        </div>
    </template>

We hebben de svg gewikkeld in een eigen `div` met als id `content`. Dat doen we zodat hier later nog wat css op kunnen toepassen. Het meest merkwaardige is dat we de breedte van ons canvas op een eigenaardige manier instellen:

    :width=width

de `:width` is eigenlijk een afkorting van het `v-bind` directive dat we al gezien hebben. Dus eigenlijk staat hier:

    v-bind:width=width

Dat betekent dus dat we een een stukje data van onze component (die toevallig ook `width` heet) gaan binden aan de breedte van ons svg canvas. Die property `width` bestaat nog niet dus we voegen snel wat data toe aan onze component:

    export default {
    name: 'App',
    components: {
    },
    data: function() {
        return {
        width: 1800
        }
    },
    async mounted() {
    ...

Ik stel hier de width in op 1800 pixels. Je kan dat uiteraard ook kleiner maken.

We tekenen ook nog eens een circle op (50,50) met een straal van 50 op onze canvas om wat te oefenen:

    <svg id="canvas" :width=width height=1000>
        <g transform="translate(50, 50)" class="data-circle">
            <circle r=50 />
        </g>
    </svg>

We passen de css voor onze component aan tussen de `style` tags om onze cirkel wat stijl te geven (je kan ook rechtstreeks de fill zetten van de cirkel als je dat wilt). We gebruiken de huisstijl van Genk:

    <style>
    @font-face {
    font-family: "SansaPro";
    src: local("SansaPro"),
        url(./fonts/SansaPro-Normal.otf) format("truetype");
    }

    #app {
    font-family: "SansaPro";
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
    }

    .data-circle {
    fill: #1a7ebe;
    }
    </style>

Vanaf nu gaan we de css van onze component tussen de style tags plaatsen (we gaan dat niet elke keer vermelden).

## V-for
We willen nu deze cirkel herhalen voor elk datapunt. In de vorige labs hebben we dat gedaan met behulp van d3 joins. Dat kan je hier ook doen. Je kan het echter ook oplossen met Vue. Vue heeft een directive genaamd `v-for`. Daarmee kunnen we een html element x aantal keer laten tekenen op basis van een rij van elementen, net zoals we dat kunnen met d3 joins. De data die we ingelezen hebben in onze `mount()` functie wordt voorlopig echter nog niet bijgehouden. We gaan daarom de data van onze component uitbreiden:

    data: function() {
        return {
        width: 1800,
        data: Object
        }
    }

We slaan dan het resultaat van al ons hard werk in `mounted()` op in deze variabele:

    async mounted() {
        const data = await d3.csv('abonnees_genk.csv', this.typeConversion);
        const filteredData = this.filterData(data);
        const preparedData = this.prepareData(filteredData);
        console.log(preparedData);

        this.data = preparedData;
    },

We kunnen nu met `v-for` loopen door deze data en de cirkel hertekenen:

    <g v-for="d in data" :key="d.postcode" transform="translate(50, 50)" class="data-circle">
        <circle r=50 />
    </g>

de `v-for` zal een voor een door de data rij gaan en voor elk element wordt het `g` element opnieuw getekend. Als je nu runt zal je nog steeds maar 1 cirkel zien, maar schijn bedriegt. Inspecteer de html van je pagina en je zal zien dat er in feite 29 cirkels getekend worden. De cirkels staan helaas allemaal op dezelfde coördinaten (50, 50) en hebben dezelfde straal: 50.

in de `v-for` staat het volgende: `d in data`. We kunnen dus elk individueel element in de lus aanspreken met `d`. Zo kunnen we bijvoorbeeld de x coordinaat en y coordinaat instellen van elke cirkel. Voor dat we dat doen, maken we een kleine omweg: wat is de x-coordinaat van de eerste cirkel? Wat is de x-coordinaat van de 8e cirkel? En de y-coordinaat van de derde cirkel? Op die vragen moeten we eerst een antwoord kunnen geven.

## X en Y

Als `i` de index van de cirkel is zouden we een formule kunnen bedenken voor de x-coordinaat van de cirkel. Bijvoorbeeld:

    x = 50 + i * 100

In dat geval verschijnt de eerste cirkel (`i=0`) op coordinaat 50, de tweede cirkel (`i=1`) op 150, etc. Onze laatste, 29e cirkel (`i=28`) zou echter van ons canvas vallen! We zouden graag onze cirkels tonen in rijen. Voor deze visualisatie gaan we 9 datapunten tonen per rij. We voegen die informatie even toe aan de data van onze component zodat we daar mee kunnen rekenen:

    data: function() {
        return {
        width: 1800,
        data: Object,
        rowLength: 9
        }
    }

Probeer zelf eerst gerust eens een formule te bedenken, maar de wiskundige logica achter de formule ligt wat buiten scope van deze oefening. We voegen 2 nieuwe methodes toe aan onze component. Eentje die de X coordinaat kan berekenen en eentje die de Y coordinaat kan berekenen:

    
    calculateXPosition: function(i)
    {
      const w = this.width - 75 * 2;
      const div = w / this.rowLength;
      return 75 + div/2 + (i % this.rowLength) * div;
    },
    calculateYPosition: function(i)
    {
      return 50+ Math.floor(i/ this.rowLength) * 150;
    }

Het is belangrijk om te melden dat X coordinaat een marge neemt van 75 pixels aan beide kanten (vandaar `const w = this.width - 75 * 2`). Het `%` teken is hier de modulo-berekening. Die ziet er misschien wat eng uit maar die berekent simpelweg de rest bij deling. Als je bijvoorbeeld 10 moet delen door 3, dan krijgt iedereen 3 stukken en blijft er 1 stuk over. Met andere woorden: 10/3 = 3 en 10 % 3 = 1.

## V-for. Nog eens.

Tijd om terug te grijpen naar onze `v-for` en de x en y coordinaten in te pluggen:

    <g v-for="(d,index) in data" :key="d.postcode" :transform="`translate(${calculateXPosition(index)}, ${calculateYPosition(index)})`" class="data-circle">
        <circle r=50 />
    </g>

Er zijn hier een paar zaken die misschien wat verwarrend zijn. Eerst en vooral is onze `v-for` aangepast naar: 

    v-for="(d,index) in data"

Dat komt omdat we standaard niet de index krijgen voor elk element, en dat hebben we nodig voor de x en y te berekenen. Door die declaratie uit te breiden steekt `v-for` voor ons de index in een tijdelijke variabele die we hier `index` hebben genoemd. Ten slotte laten we het transform attribuut aanpassen door Vue. Dan moeten we dus `v-bind` voor het attribuut zetten, of de verkorte versie: `:`. De notatie met de backticks (`) en het dollarteken is een manier om code te injecteren in een string. Die notatie heb je normaal gezien al eens gezien in de lessen web.

Run opnieuw je project. Nu heb je normaal gezien een cirkel per datapunt.

## Data injecteren

We hebben nu de x en y-coordinaat van de cirkel ingesteld op basis van de index, maar we willen ook de straal aanpassen van de cirkel op basis van de hoeveelheid abonnees. Denk eraan dat in ons object `data` er momenteel elementen zitten van de vorm `{postcode: ..., amount:..., city:...} `. We krijgen elk element netjes mee in de variabele `d` in onze `v-for`. We zouden de straal als volgt kunnen aanpassen:

    <g v-for="(d,index) in data" :key="d.postcode" :transform="`translate(${calculateXPosition(index)}, ${calculateYPosition(index)})`" class="data-circle">
        <circle :r="d.amount" />
    </g>

Dat is echter niet zo een slim idee, want die `amount` kan serieus oplopen waardoor de straal van onze cirkels uit de hand loopt. We gaan daarom een methode toevoegen die de straal voor ons kan berekenen op basis van zo een datapunt:

    calculateRadius: function(da)
    {
      const extents = d3.extent(this.data, d => d.amount);
      const t = da.amount / extents[1];
      return t * 50;
    },

Deze methode neemt een datapunt en berekent met behulp van d3 de "extents", oftewel uiterste, van onze data. We delen dan `amount` door het maximum dat we gevonden hebben met `d3.extent()`. Dat levert ons een getal op tussen 0 en 1. Dat getal vermenigvuldigen we met 50. De grootste cirkel zal dus 50 pixels aan straal hebben. Al de rest ligt daaronder. De kleinste amount die we theoretisch kunnen hebben is 100 (de andere hebben we er uitgefilterd) en die zou dan een straal hebben van 100/1686 * 50, oftwel een slordige 3 pixels.
We gebruiken nu deze methode in onze `v-for` om de straal te laten berekenen:

    <g v-for="(d,index) in data" :key="d.postcode" :transform="`translate(${calculateXPosition(index)}, ${calculateYPosition(index)})`" class="data-circle">
        <circle :r="calculateRadius(d)" />
    </g>

Denk aan de `:` voor de `r`! Run je project.

## Labels

We willen ook graag een label onder elke cirkel met daarop de postcode of de naam van de gemeente. Dat is nu gemakkelijk toegevoegd. We voegen eerst een svg text elementje toe onder onze cirkel:

    <g v-for="(d,index) in data" :key="d.postcode" :transform="`translate(${calculateXPosition(index)}, ${calculateYPosition(index)})`" class="data-circle">
        <circle :r="calculateRadius(d)" />
        <text y=75 class="label">
            Label
        </text>
    </g>

We hebben het text elementje een css klasse gegeven: `label`, de waarde is `Label` en we hebben het text elementje 75 pixels onder de cirkel gezet. We voegen ook meteen de css toe voor ons label zodat de tekst gecentreerd staat:

    .label {
        fill: #1a7ebe;
        text-anchor: middle;
    }

Run je project. Je hebt nu blauwe tekst onder elke cirkel staan. Nu staat er overal "Label" maar we zouden dit graag vervangen door de naam van de gemeente. Gelukkig staat die in ons `d` object. De naam van de gemeente is `d.city`. We hebben al gezien dat je met Vue code kan injecteren in een html attribuut met behulp van het `v-bind` directive, maar je kan ook code injecteren in de <i>content</i> van een html element. Dat doe je tussen `{{` en `}}`:

    <text y=75 class="label">
        {{d.city}}
    </text>

Run opnieuw je project.

## Reactieve UI

Het volgende dat we voor elkaar willen krijgen is dat de cirkel reageert als je er met je muis over gaat. We gaan een extra cirkel tekenen bovenop onze cirkel die de diepblauwe kleure heeft. Die "circle-overlay" krijgt een straal van 0. Als we met de muis over onze cirkel gaan, gaan we de straal van deze overlay laten toenemen. Dat klinkt misschien wat abstract dus laat ons meteen beginnen met een nieuwe cirkel te tekenen bovenop onze cirkel:

    <g v-for="(d,index) in data" :key="d.postcode" :transform="`translate(${calculateXPosition(index)}, ${calculateYPosition(index)})`" class="data-circle">
        <circle :r="calculateRadius(d)" />
        <circle class="circle-overlay" :r="calculateRadius(d)" />
        <text y=75 class="label">
            {{d.city}}
        </text>
    </g>

De nieuwe cirkel krijgt een css klass `circle-overlay` en dezelfde straal als de oorspronkelijke cirkel. We geven deze cirkel de donkere kleur in onze css:

    .circle-overlay {
        fill: #024d9d;
    }

Run je project. Nu heb je donkerblauwe cirkels in plaats van lichtblauwe cirkels. We willen deze cirkels zo klein mogelijk maken en enkel groot maken als de muis beweegt over een cirkel. Je kan dat op heel veel verschillende manieren doen. Hier gaan we het doen met simpele css. Als je niet weet hoe je kan animeren met css, geen paniek, je kan gewoon onderstaande css code kopiëren. CSS animatie is geen onderdeel van deze cursus.

    .circle-overlay {
        fill: #024d9d;
        transform: scale(0);
        transition: ease-out 300ms;
    }

    .data-circle:hover .circle-overlay {
        transform: scale(1);
    }

Run je project. Als je nu met je muis over de cirkels beweegt animeert de donkerblauwe cirkel.

## Card

Er zit nog een component in je project: `Card.vue`. Laat ons die component eens tekenen op ons scherm. Daarvoor moeten we eerst `Card` importeren in onze `App` component:

    import * as d3 from "d3";
    import Card from './components/Card.vue';

    export default {
        name: 'App',
        components: {
            Card
        },
        ...

Dan kunnen we de Card component tekenen in onze html:

    <div id="content">
      <svg id="canvas" :width=width height=1000>
          <g v-for="(d,index) in data" :key="d.postcode" :transform="`translate(${calculateXPosition(index)}, ${calculateYPosition(index)})`" class="data-circle">
            <circle :r="calculateRadius(d)" />
            <circle class="circle-overlay" :r="calculateRadius(d)" />
            <text y=75 class="label">
              {{d.city}}
            </text>
          </g>
      </svg>

      <Card />
    </div>

We zetten de Card component naast ons svg canvas. Run je project en... we zien helemaal niets extra. De Card component is een extra component die ik voor jullie geschreven heb. De component heeft 5 eigenschappen:

- x: de x-coordinaat van de Card.
- y: de y-coordinaat van de Card.
- show: is de Card zichtbaar of niet? Standaard staat deze op 'false'.
- title: de titel van de Card.
- content: de tekst op de Card.

Misschien kunnen we wel iets zien als we het `show` attribuut op `true` zetten. Card is een Vue-component dus vergeet niet dat je weer het `v-bind` directive moet gebruiken:

    <Card :show="true" />

Run je project. Dat is al beter, scroll helemaal naar beneden en je zou de kaart moeten kunnen zien. Voorlopig staat er enkel de "# Abonnees" op. We proberen ook eens `title` en `content` aan te passen:

    <Card :show="true" :title="'GENK'" :content="'1618'" />

Run je project.

Wat we nu gaan doen is zo een kaart tekenen bij elke cirkel en daar de juiste data in injecteren. Daarna gaan we al die kaarten op onzichtbaar zetten en enkel tonen als de muis over de cirkel beweegt. Mijn eerste instinct was om de kaart mee toe te voegen in onze `v-for`. Helaas is daar een probleem: html staat het niet toe dat je elementen zet onder een `svg` element dat geen vectorelement is. Dus onder een `svg` element kan je enkele zaken zetten zoals `rect`, `circle`, `text`, `g`, etc. Je kan bijvoorbeeld geen `h2` element zetten onder een `svg` element. Verdorie...

Dat betekent dat we wat creatief gaan moeten zijn: we gaan een nieuwe `v-for` lus opzetten:

    <svg id="canvas" :width=width height=1000>
        <g v-for="(d,index) in data" :key="d.postcode" :transform="`translate(${calculateXPosition(index)}, ${calculateYPosition(index)})`" class="data-circle">
        <circle :r="calculateRadius(d)" />
        <circle class="circle-overlay" :r="calculateRadius(d)" />
        <text y=75 class="label">
            {{d.city}}
        </text>
        </g>
    </svg>

    <div v-for="(d, index) in data" :key="d.postcode">
        <Card :show="true" :x="calculateXPosition(index)" :y="calculateYPosition(index)" :title="'GENK'" :content="'1618'"/>
    </div>

We hebben hier een nieuwe `v-for` lus dit in zijn eigen div voor elk datapunt een kaart tekent. Als x en y positie geven we dezelfde positie mee als de x en y positie van de cirkels. Run je project. Je zal nu een kaart hebben liggen bovenop elke cirkel. Als je je afvraagt waarom de kaarten bovenop de cirkels worden getekend en niet onder het canvas, dat komt omdat de css van de `Card` component zo ingesteld is dat de coordinaten uitgelezen worden als absolute coordinaten. De CSS hiervan is nogmaals buiten de scope van dit vak.

We maken nog een kleine aanpassing zodat er niet altijd "GENK" en "1618" getoond wordt:

    <div v-for="(d, index) in data" :key="d.postcode">
        <Card :show="true" :x="calculateXPosition(index)" :y="calculateYPosition(index)" :title="d.city" :content="d.amount"/>
    </div>

Mogelijks krijg je ondertussen ook de volgende waarschuwing in je console:

    vue.runtime.esm.js?2b0e:619 [Vue warn]: Invalid prop: type check failed for prop "content". Expected String with value "131", got Number with value 131.

Vue zegt tegen ons dat we er typefout zit (niet typen, zoals je met je keyboard doet, wel type zoals in datatype). Specifiek schuilt die fout in de property `content` van onze `Card`. We geven daar nu het volgende door:

    :content="d.amount"

`content` is een eigenschap van Card die een `String` als type verwacht. Maar `d.amount` is een getal. Dat legt Vue ook helder uit: `Expected String with value "131", got Number with value 131.`. Lees dus zeker goed de fouten van de console na!
We kunnen deze fout het zwijgen opleggen door `d.amount` eerst om te zetten naar een `String`. Dat kan met de `toString()` methode:

    :content="d.amount.toString()"

Run opnieuw je project en refresh de pagina. De fout zou verdwenen moeten zijn.

## V-on

Nu willen we graag reageren op de muis die over een cirkel beweegt. Dat kan opnieuw met d3 zoals we in de vorige labs gezien hebben. Er is wel een nadeel verbonden aan die werkwijze: d3 kan niet aan de context van je component. Met andere woorden - d3 kan niet uitbreken en aanpassingen maken aan de data van je component, d3 wordt uitgevoerd binnen zijn eigen context. Daarom gaan we opnieuw Vue gebruiken om dit probleem op te lossen.

We gaan de data van onze component opnieuw uitbreiden met een `selectedData` eigenschap:

    data: function() {
        return {
        width: 1800,
        data: Object,
        rowLength: 9,
        selectedData: Object
        }
    }

De waarde van deze eigenschap is initieel leeg. Elke keer als onze muis beweegt over een cirkel steken we het datapunt dat bij die cirkel hoort in `selectedData`. Is `selectedData` leeg? Dan is onze muis momenteel niet over een cirkel. Anders bevat `selectedData` het datapunt van de cirkel waar onze muis over staat. Om dat te laten werken gaan we gebruik maken van Vue zijn ingebouwde event: `v-on`. Door bijvoorbeeld `v-on:mouseover` toe te voegen aan een html element kunnen we code of een functie uitvoeren als de muis over dat element beweegt. We voegen `v-on` toe aan elke cirkel in onze `v-for`:

    <g v-for="(d,index) in data" :key="d.postcode" :transform="`translate(${calculateXPosition(index)}, ${calculateYPosition(index)})`" class="data-circle" v-on:mouseover="selectedData = d">
        <circle :r="calculateRadius(d)" />
        <circle class="circle-overlay" :r="calculateRadius(d)" />
        <text y=75 class="label">
            {{d.city}}
        </text>
    </g>

Elke keer als nu de muis over een cirkel komt voeren we de volgende code uit: `selectedData = d`, en `d` is het datapunt dat bij de cirkel hoort. Verschuift de muis naar een andere cirkel zal die weer de inhoud van `selectedData` overschrijven, enzovoort. 

Nu gaan we de kaarten verbergen. Elke kaart wordt aangemaakt in een `v-for` die ook een variabele `d` heet met daarin een datapunt. Er is dus een datapunt voor elke kaart. Een kaart is enkel zichtbaar als het geselecteerde datapunt hetzelfde is als het datapunt van de kaart. Oftewel als `selectedData` gelijk is aan `d`:

    <div v-for="(d, index) in data" :key="d.postcode">
        <Card :show="selectedData == d" :x="calculateXPosition(index)" :y="calculateYPosition(index)" :title="d.city" :content="d.amount.toString()"/>
    </div>

Run je project. Enkel de kaart van de cirkel waar je muis over beweegt zal nu tonen.

## Hoofding

Dit deel is eigenlijk html en css. We gaan een hoofding toevoegen aan onze applicatie. Voeg de volgende html toe aan je app component:

    <template>
    <div id="app">
        <div class="titlebox">
        <img class="logo" alt="Genk Logo" src="./assets/logo.png" width="150"/>
        <div id="title">
            <h1>RACING GENK</h1>
            <p>Abonnees per gemeente met minstens <b>100</b> abonnees</p>
        </div>
        </div>
        <div id="content">
        <svg id="canvas" :width=width height=1000>
        ...

En de volgende css:

    .titlebox {
    display: flex;
    height: 200px;
    margin-left: 150px;
    }

    #title h1 {
    margin-bottom: 0px;
    }

    #title p {
    margin-top : 0px;
    }

    #title {
    margin-left: 50px;
    padding-top: 50px;
    }

    h1 {
    text-align: left;
    }

Run je project.

## Filterknoppen

We willen ook wat extra interactiviteit toevoegen aan onze visualisatie. De gebruiker kan kiezen of ze liever gemeentes zien of de postcodes. Daarom voegen we met html enkele radio buttons toe onder onze hoofding:

    ...
    <div class="titlebox">
      <img class="logo" alt="Genk Logo" src="../assets/logo.png" width="150"/>
      <div id="title">
        <h1>RACING GENK</h1>
        <p>Abonnees per gemeente met minstens <b>100</b> abonnees</p>
      </div>
    </div>
    <div class="filter-buttons" id="v-model-radiobutton">
      <input type="radio" id="town" value="town" v-model="filter" />
      <label for="town">Gemeente</label>
      <input type="radio" id="postalCode" value="postalCode" v-model="filter" />
      <label for="postalCode">Postcode</label>
    </div>
    <div id="content">
      <svg id="canvas" :width=width height=1000>
    ...

Voeg ook de volgende css toe:

    .filter-buttons {
        margin-top: 50px;
        margin-bottom: 20px;
        margin-right: 150px;
        display:flex;
        justify-content: right;
    }

Run je project. Je zou nu twee radio buttons moeten zien aan de rechterkant. Helaas werken ze nog niet zo goed.

We hebben hier weer wat nieuwe functionaliteit gebruikt van Vue:

    <div class="filter-buttons">
      <input type="radio" id="town" value="town" v-model="filter" />
      <label for="town">Gemeente</label>
      <input type="radio" id="postalCode" value="postalCode" v-model="filter" />
      <label for="postalCode">Postcode</label>
    </div>

Eerst en vooral hebben we aan elke radio button een `v-model` gegeven. Dat is de waarde in de data van onze component waaraan deze radio button gekoppeld is. Dat attribuut moeten we nog toevoegen aan de data van onze component:

    data: function() {
        return {
        width: 1800,
        data: Object,
        rowLength: 9,
        selectedData: Object,
        filter: ''
        }
    },

Tot nu toe hebben we daarvoor altijd `v-bind` gebruikt. Het verschil is dat `v-bind` een variabele uitleest en die toekent aan een attribuut of html element. Dat is strikt eenrichtingsverkeer. `v-model` is tweerichtingsverkeer. We binden de waarde van de button aan de eigenschap `filter`. Als de status van de knop verandert zal ook automatisch de waarde van `filter` worden aangepast. Wat is dan de waarde van filter? We zien dat elke radio button ook een `value` attribuut heeft. De bovenste/linkse knop heeft als `value` `town`. Als op deze knop gedrukt wordt zal `filter` de waarde `town` krijgen. Dat werkt ook omgekeerd: als we de waarde `town` aan `filter` toekennen zal die knop geselecteerd worden.

Laat ons dat een proberen:

    data: function() {
    return {
      width: 1800,
      data: Object,
      rowLength: 9,
      selectedData: Object,
      filter: 'town'
    }
  },


Run je project en je zal zien dat nu "Gemeente" geselecteerd is. Al de rest is html.

## Filterfunctionaliteit

De filterknoppen doen op dit moment nog niets. We gaan onze code op verschillende plaatsen moeten aanpassen om die filter te kunnen toepassen. Op dit moment geven we bijvoorbeeld vaak de naam van de gemeente door op de volgende manier:

    d.city

Nu willen we op die plaatsen soms de postcode doorgeven:

    d.postcode

We willen dus eigenlijk:

    if(filter == 'town')
    {
        d.city
    }
    else
    {
        d.postcode
    }

Dat kunnen we ook korter schrijven op 1 regel als volgt:

    filter == 'town' ? d.city : d.postcode

We passen onze labels aan:

    <text y=75 class="label">
        {{filter == 'town' ? d.city : d.postcode.toString()}}
    </text>

(We gebruiken hier opnieuw `toString()` om waarschuwingen te vermijden). We passen ook de data aan die we een `Card` geven:

    <div v-for="(d, index) in data" :key="d.postcode">
        <Card :show="selectedData == d" :x="calculateXPosition(index)" :y="calculateYPosition(index)" :title="filter == 'town' ? d.city : d.postcode.toString()" :content="d.amount.toString()"/>
    </div>

Run opnieuw je project. Hoera!

## Uitdaging

Je hebt nu een hele simpele, interactieve visualisatie. Je zou de visualisatie nog verder kunnen uitbreiden. Probeer bijvoorbeeld een knop toe te voegen die de data sorteert van groot naar klein.




