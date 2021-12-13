# Data Visualisatie: Genk Abonnees
Tijd dat we eens wat dieper graven met de kennis die we opgedaan hebben om een grotere visualisatie te maken. 

## Verkenning Project
Normaal gezien heb je het project kunnen downloaden via github (idealiter heb je dat via git clone gedaan!). Het startersproject is geen magie. Het is een basis Vue-project dat gemaakt is met `vue create`. Daarna is de hello world component verwijderd. Er zijn nog enkele aanpassingen gebeurd:

- In de public folder is het logo van Genk toegevoegd.
- In de public folder is het .csv bestandje ook al toegevoegd (`abonnees_genk.csv`).
- In de `src/assets/` folder staat ook het logo van Genk zodat we dat kunnen gebruiken in onze html.
- In de components folder is een Vue component voorgemaakt: `Card.vue`. Die gaan we gebruiken in onze visualisatie. Die component moet je niet zelf kunnen maken. Je mag uiteraard gerust eens kijken naar de broncode van deze component!
- In `App.vue` onder de `style` tags is het standaard font ingesteld op SansaPro, de huisstijl van Genk. Dat font is meegeleverd in de map `src/fonts`. In de `style` tags importeren we dat font en stellen we dat in als de de facto standaard voor het project.

Voordat we beginnen, kijk eerst eens of je het project aan de praat krijgt:

    npm run serve

Je zou in het groot het logo van Genk moeten zien.

## Inlezen van de data



