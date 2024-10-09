# Gestion des évènements et dates des sites via JSON

**Sommaire**

---

## 1. Modification du fichier JSON sur GitHub

### Accès et édition du fichier `content.json` sur GitHub

1. **Se connecter à GitHub :**
    - Accédez à [GitHub.com](https://github.com/) et connectez vous à votre compte.
    - Naviguez jusqu'au dépôt du projet contenant le fichier `content.json`. Par exemple : [https://github.com/BrestOpenCampus/website_agendas](https://github.com/BrestOpenCampus/website_agendas).
2. **Localiser le fichier `content.json` :**
    - Depuis la page d'accueil du dépôt, naviguez dans la structure des dossiers jusqu'à trouver le fichier `content.json` (souvent dans le dossier `content` ou similaire).
    
    ![image.png](image.png)
    
3. **Modifier le fichier `content.json` :**
    - Cliquez sur le fichier `content.json` pour l'ouvrir.
    - En haut à droite du fichier, cliquez sur l'icône en forme de crayon (Edit) pour commencer la modification.
    
    ![image.png](image%201.png)
    
4. **Modification des sections :**
    - Pour **ajouter un nouvel événement dans les agendas** (un nouveau bloc dans la section `blocks`), copiez un bloc existant et ajustez les champs comme suit :
        - `leftBlockText`: Texte principal, comme les horaires.
        - `admissionTitle`: Le titre de l'événement.
        - `subtext`: Un texte secondaire ou complémentaire pour l'événement.
        - `location`: La localisation de l’évènement.
            - Si il s’agit d’un événement avec une seul localisation définie, comme un salon par exemple, alors il faut utilisé la nomenclature suivante :
            
            ```json
            "location": "Vannes"
            ```
            
            - Si il y a plusieurs localisation possible, comme pour une JPO, alors il faut écrire la localisation pour chaque sites de la façon suivante :
            
            ```json
            "location": {
            	"OC": "Brest, Angers et Caen",
              "SUPIMMO" : "Brest et Caen",
              "HTBS": "Brest, Angers et Caen",
              "LIT": "Brest et Angers",
              "LDS": "Brest et Angers",
              "SSB": "Brest"
            },
            ```
            
        - `buttonText`: Le texte qui sera affiché sur le bouton.
        - `buttonLinks`: Ajoutez ou modifiez l'URL vers lequel le bouton est sensé redirigé pour chaque site sur lequel l’événement sera affiché. Ex: si l’événement doit être affiché sur 3 sites, il doit y avoir 3 liens.
            
            ```json
            "buttonLinks": {
            	"OC": "https://reseau-opencampus.com/demande-dossier-candidature-goc/",
            	"LIT": "https://learnit-school.com/dossier-de-candidature/",
            	"LDS": "https://ladigitalschool.com/fr/candidature/"
            },
            ```
            
        - `site`: Liste des sites (`OC`, `LIT`, `HTBS`, etc.) où cet événement doit s’afficher.
            
            ```json
            "site": ["OC", "LIT", "HTBS"]
            ```
            
    - Pour **ajouter une nouvelle date de test d’admission** `tdaDates`, copiez une entrée existante dans la section `tdaDates` :
        
        ```json
        { "date": "October 02, 2025", "label": "Mercredi 02/10" }
        ```
        
        Modifiez la date et le label en fonction de la nouvelle date.
        
        - La “date” doit être au format, `"Mois jj, aaaa"`
        - Le label représente se qui sera affiché sur la page web
    - Pour **ajouter une nouvelle date de JPO** `jpoDates`, ajoutez une nouvelle date dans le bon groupe (`OC` ou `PBA`) et pour le bon campus (`Brest`, `Angers`, etc.):
        - Exemple pour ajouter 2 nouvelles dates pour OC Brest :
        
        ```json
        "jpoGroup": {
                    "OC": {
                        "Brest": [
                            "October 19, 2024 09:30:00",
                            "November 16, 2024 09:30:00"
                        ]
                    },
                    "PBA": { }
         }
        ```
        
5. **Enregistrer vos modifications :**
    - Une fois les modifications effectuées, faites défiler la page jusqu'à la section "Commit changes".
    - Ajoutez un message de description de vos changements dans le champ "Commit message".
    - Cliquez sur "Commit changes" pour enregistrer les modifications dans le dépôt.
        
        ![image.png](image%202.png)
        

---

## 2. Explication des script JavaScript dans WordPress (Elementor)

### Script JavaScript personnalisés

Les scripts de wordpress sont utilisés pour récupérer les informations des événements, des dates d'admission, et des JPO (Journée Portes Ouvertes) depuis le fichier JSON hébergé sur GitHub. Ces scripts sont intégrés dans WordPress via Elementor, dans la section **Code personnalisé** du site.
Normalement ils ne seront pas a modifier, cette partie de la doc est faite pour décrire le fonctionnement général en cas de problème.

### Fonctionnement d’un script :

1. **Définition du site actuel :**
    - Le script commence par définir une constante `currentSite` qui représente le site actuel où le script est exécuté. Cette constante permet de filtrer les données du fichier JSON pour ne sélectionner que celles correspondant au site en cours. Le site peut être défini comme suit :
        
        ```jsx
        const currentSite = "SSB";  // Modifiez cette valeur selon le site
        ```
        
        <aside>
        ⚠️
        
        Ce paramètre est crucial car il permet de filtrer les événements en fonction du site paramétré dans le fichier JSON. Si cette valeur n’est pas bonne, les informations qui seront affichées sur le site seront fausse.
        
        </aside>
        
    
2. **Récupération du fichier JSON :**
    - Le fichier JSON, qui contient toutes les informations des événements, est récupéré dynamiquement à partir d'un serveur hébergé via Cloudflare. La récupération se fait avec un mécanisme de "cache-busting" grâce à un paramètre `v` dans l'URL, qui ajoute un timestamp unique afin d'éviter que le navigateur ne serve une version mise en cache de ce fichier :
        
        ```jsx
        let cacheBuster = new Date().getTime();
        let jsonUrl = `https://website-agendas.pages.dev/content.json?v=${cacheBuster}`;
        ```
        
    - Le fichier est ensuite récupéré via la fonction `fetch()` qui effectue une requête HTTP asynchrone et attend la réponse au format JSON :
        
        ```jsx
        fetch(jsonUrl)
            .then(response => response.json())
            .then(data => {
                // Traitement des données JSON
            })
            .catch(error => console.error('Erreur lors de la récupération du fichier JSON:', error));
        ```
        
3. **Filtrage des événements en fonction du site :**
    - Après avoir récupéré les données JSON, le script filtre les événements en fonction du site défini par `currentSite`. Chaque événement dans le fichier JSON contient un champ `site` qui correspond à une liste des sites auxquels cet événement s'applique. Seuls les événements contenant le site actuel dans leur liste seront sélectionnés :
        
        ```jsx
        const filteredBlocks = data.blocks.filter(block => block.site.includes(currentSite));
        ```
        
    - Si aucun bloc n'est trouvé pour le site actuel, un message informatif est affiché à l'utilisateur :
        
        ```jsx
        if (filteredBlocks.length === 0) {
            container.innerHTML = `<p>Aucun bloc disponible pour le site ${currentSite}</p>`;
        }
        ```
        
4. **Création et affichage des blocs :**
    - Pour chaque bloc filtré, le script appelle une fonction `createBlock()` qui génère le HTML correspondant à l'événement et l'insère dynamiquement dans la page. Cette fonction crée un élément HTML `<div>` pour chaque événement et y ajoute le titre, la description, la localisation et un bouton d'action avec un lien d'inscription.
    La forme la plus courante de la fonction `createBlock()` pour les sites OC est la suivante :
        
        ```jsx
        function createBlock(blockData) {
            const section = document.createElement('div');
            section.classList.add('section');
        
            // Création du contenu du bloc
            section.innerHTML = `
                <div class="event-content">
                    <h2 class="event-title">${blockData.admissionTitle}</h2>
                    <p class="event-subtext">${blockData.leftBlockText}</p>
                    <p class="event-location"><span class="emoji">📍</span> ${getLocation(blockData)}</p>
                    <div class="event-button-container">
                        <a href="${getButtonLink(blockData)}" class="event-button-link">
                            <button class="event-button">${blockData.buttonText}</button>
                        </a>
                    </div>
                </div>
            `;
            return section;
        }
        ```
        
5. **Gestion des exceptions : Location et ButtonLinks :**
    - **Gestion de la localisation `location` :**
    Le champ `location` dans le fichier JSON peut être un objet (lorsqu'il contient des localisations spécifiques à chaque site) ou une simple chaîne de caractères. Pour s'adapter à ces deux formats, la fonction `getLocation` est utilisée pour vérifier si `location` est un objet ou une chaîne. Si c'est un objet, elle retourne la localisation spécifique au site actuel, sinon, elle renvoie la localisation générale :
        
        ```jsx
        function getLocation(blockData) {
            if (typeof blockData.location === 'object') {
                return blockData.location[currentSite] || 'Lieu non disponible';
            } else {
                return blockData.location || 'Lieu non disponible';
            }
        }
        ```
        
    - **Gestion des liens d'action `buttonLinks` :**
    Le champ `buttonLinks` fonctionne de manière similaire à `location`. S'il est un objet, il contient des liens spécifiques à chaque site. La fonction `getButtonLink` vérifie si le champ est un objet ou une chaîne. Si c'est un objet, elle renvoie le lien spécifique au site actuel, sinon, elle renvoie le lien général :
        
        ```jsx
        function getButtonLink(blockData) {
            if (typeof blockData.buttonLinks === 'object') {
                return blockData.buttonLinks[currentSite] || '#';
            } else {
                return blockData.buttonLinks || '#';
            }
        }
        ```
        
6. **Ajout des blocs HTML au conteneur :**
    - Après avoir créé un bloc pour chaque événement, ces blocs sont ajoutés au conteneur HTML principal qui est identifié par l'ID `admission-sections` :
        
        ```jsx
        const container = document.getElementById('admission-sections');
        filteredBlocks.forEach(block => {
            const newBlock = createBlock(block);
            container.appendChild(newBlock);
        });
        ```
        
    
7. **Gestion des erreurs :**
    - Si une erreur survient lors de la récupération du fichier JSON (par exemple, si le fichier n'est pas accessible ou contient une erreur de syntaxe), le script affiche un message d'erreur dans la console du navigateur pour aider au diagnostic :
        
        ```jsx
        .catch(error => console.error('Erreur lors de la récupération du fichier JSON:', error));
        ```
        

---

## 3. Lien entre GitHub et Cloudflare pour l'accès au fichier JSON

Le fichier `content.json` est hébergé sur GitHub, mais il est mis à disposition via Cloudflare pour améliorer les performances et la distribution.

### Fonctionnement du lien GitHub-Cloudflare :

1. **Fichier JSON sur GitHub :**
    - Le fichier `content.json` est hébergé sur le dépôt GitHub du projet. Les modifications apportées au fichier sur GitHub sont publiques et peuvent être consultées via l'URL GitHub.
2. **Cloudflare pour le cache et la distribution :**
    - Cloudflare est utilisé pour distribuer le fichier JSON via une URL optimisée : 
    [https://website-agendas.pages.dev/content.json](https://website-agendas.pages.dev/content.json).
    - Grâce à Cloudflare, le fichier est mis en cache dans plusieurs serveurs à travers le monde, ce qui réduit les temps de chargement et optimise l'accès aux utilisateurs de différentes régions. Chaque mise à jour du fichier est presque instantanée et est distribué sur tous les sites concernés.
3. **Cache-Busting pour forcer l’actualisation des données :**
    - Pour s'assurer que la version la plus récente du fichier JSON est toujours utilisée, le paramètre `cacheBuster` est ajouté à l'URL du fichier lors de chaque récupération :
        
        ```jsx
        let cacheBuster = new Date().getTime(); let jsonUrl = `https://website-agendas.pages.dev/content.json?v=${cacheBuster}`;
        ```
        
        Ce paramètre ajoute un timestamp unique à l'URL, ce qui permet de contourner le cache et de récupérer la dernière version du fichier.
        

---

## 4. Exemple de fichier JSON et Script

### Fichier JSON du déploiement :

```json
{
    "blocks": [
        {
            "leftBlockText": "Tous les mercredis et vendredis après-midi",
            "admissionTitle": "TESTS D'ADMISSION",
            "subtext": "Consultez toutes nos prochaines dates",
            "location": {
                "OC": "Brest, Angers et Caen",
                "SUPIMMO" : "Brest et Caen",
                "HTBS": "Brest, Angers et Caen",
                "LIT": "Brest et Angers",
                "LDS": "Brest et Angers",
                "SSB": "Brest"
            },
            "buttonText": "S'inscrire",
            "buttonLinks": {
                "OC": "https://reseau-opencampus.com/demande-dossier-candidature-goc/",
                "LIT": "https://learnit-school.com/dossier-de-candidature/",
                "LDS": "https://ladigitalschool.com/fr/candidature/",
                "SUPIMMO": "https://sup-immo.com/demande-de-candidature/",
                "HTBS": "https://htbs.fr/demande-de-candidature/",
                "SSB": "https://schoolofsportbusiness.fr/demande-de-candidature/"
            },
            "site": ["OC", "LIT", "LDS", "SUPIMMO", "HTBS", "SSB"]
        },
        {
            "leftBlockText": "Samedi 19 Octobre",
            "admissionTitle": "Journée portes ouvertes",
            "subtext": "Venez découvrir votre future école",
            "location": {
                "OC": "Brest et Caen",
                "SUPIMMO" : "Brest et Caen",
                "HTBS": "Brest et Caen",
                "LIT": "Brest",
                "LDS": "Brest",
                "SSB": "Brest"
            },
            "buttonText": "S'inscrire",
            "buttonLinks": {
                "OC": "https://reseau-opencampus.com/inscription-journee-portes-ouvertes/",
                "LIT": "https://learnit-school.com/inscription-jpo/",
                "LDS": "https://ladigitalschool.com/journee-portes-ouvertes-la-digital-school/",
                "SUPIMMO": "https://sup-immo.com/inscription-jpo-2/",
                "HTBS": "https://htbs.fr/inscription-journee-portes-ouvertes/",
                "SSB": "https://schoolofsportbusiness.fr/journee-portes-ouvertes-2/"
            },
            "site": ["LDS", "OC", "LIT", "SUPIMMO", "HTBS", "SSB"]
        }
    ],
    "tdaDates": [
        { "date": "October 02, 2024", "label": "Mercredi 02/10" },
        { "date": "October 04, 2024", "label": "Vendredi 04/10" },
        { "date": "October 09, 2024", "label": "Mercredi 09/10" },
        { "date": "October 11, 2024", "label": "Vendredi 11/10" },
        { "date": "October 16, 2024", "label": "Mercredi 16/10" },
        { "date": "October 18, 2024", "label": "Vendredi 18/10" },
        { "date": "October 23, 2024", "label": "Mercredi 23/10" },
        { "date": "October 25, 2024", "label": "Vendredi 25/10" },
        { "date": "October 30, 2024", "label": "Mercredi 30/10" },
        { "date": "November 01, 2024", "label": "Vendredi 01/11" },
        { "date": "November 06, 2024", "label": "Mercredi 06/11" },
        { "date": "November 08, 2024", "label": "Vendredi 08/11" },
        { "date": "November 13, 2024", "label": "Mercredi 13/11" },
        { "date": "November 15, 2024", "label": "Vendredi 15/11" },
        { "date": "November 20, 2024", "label": "Mercredi 20/11" },
        { "date": "November 22, 2024", "label": "Vendredi 22/11" },
        { "date": "November 27, 2024", "label": "Mercredi 27/11" },
        { "date": "November 29, 2024", "label": "Vendredi 29/11" },
        { "date": "December 04, 2024", "label": "Mercredi 04/12" },
        { "date": "December 06, 2024", "label": "Vendredi 06/12" },
        { "date": "December 11, 2024", "label": "Mercredi 11/12" },
        { "date": "December 13, 2024", "label": "Vendredi 13/12" },
        { "date": "December 18, 2024", "label": "Mercredi 18/12" },
        { "date": "December 20, 2024", "label": "Vendredi 20/12" },
        { "date": "December 25, 2024", "label": "Mercredi 25/12" },
        { "date": "December 27, 2024", "label": "Vendredi 27/12" }
    ],
    "jpoDates": {
        "jpoGroup": {
            "OC": {
                "Brest": [
                    "October 19, 2024 09:30:00",
                    "November 16, 2024 09:30:00",
                    "December 7, 2024 09:30:00",
                    "January 11, 2025 09:30:00",
                    "February 1, 2025 09:30:00",
                    "March 1, 2025 09:30:00",
                    "April 5, 2025 09:30:00",
                    "May 17, 2025 09:30:00",
                    "June 7, 2025 09:30:00"
                ],
                "Angers": [
                    "November 9, 2024 09:30:00",
                    "December 14, 2024 09:30:00",
                    "January 11, 2025 09:30:00",
                    "February 1, 2025 09:30:00",
                    "March 15, 2025 09:30:00",
                    "April 5, 2025 09:30:00",
                    "May 17, 2025 09:30:00",
                    "June 7, 2025 09:30:00"
                ],
                "Caen": [
                    "October 19, 2024 09:30:00",
                    "November 16, 2024 09:30:00",
                    "December 14, 2024 09:30:00",
                    "January 18, 2025 09:30:00",
                    "February 1, 2025 09:30:00",
                    "April 5, 2025 09:30:00",
                    "May 17, 2025 09:30:00",
                    "June 7, 2025 09:30:00"
                ]
            },
            "PBA": {
                "Brest": [
                    "October 19, 2024 09:30:00",
                    "October 19, 2024 14:00:00",
                    "November 9, 2024 09:30:00",
                    "November 9, 2024 14:00:00",
                    "December 7, 2024 09:30:00",
                    "December 7, 2024 14:00:00",
                    "January 11, 2025 09:30:00",
                    "January 11, 2025 14:00:00",
                    "February 8, 2025 09:30:00",
                    "February 8, 2025 14:00:00",
                    "March 8, 2025 09:30:00",
                    "March 8, 2025 14:00:00",
                    "April 5, 2025 09:30:00",
                    "April 5, 2025 14:00:00"
                ],
                "Angers": [
                    "November 9, 2024 09:30:00",
                    "November 9, 2024 14:00:00",
                    "December 14, 2024 09:30:00",
                    "December 14, 2024 14:00:00",
                    "January 18, 2025 09:30:00",
                    "January 18, 2025 14:00:00",
                    "February 8, 2025 09:30:00",
                    "February 8, 2025 14:00:00",
                    "March 8, 2025 09:30:00",
                    "March 8, 2025 14:00:00",
                    "April 5, 2025 09:30:00",
                    "April 5, 2025 14:00:00"
                ],
                "Caen": [
                    "October 19, 2024 09:30:00",
                    "October 19, 2024 14:00:00",
                    "November 9, 2024 09:30:00",
                    "November 9, 2024 14:00:00",
                    "December 14, 2024 09:30:00",
                    "December 14, 2024 14:00:00",
                    "January 18, 2025 09:30:00",
                    "January 18, 2025 14:00:00",
                    "February 8, 2025 09:30:00",
                    "February 8, 2025 14:00:00",
                    "March 8, 2025 09:30:00",
                    "March 8, 2025 14:00:00",
                    "April 5, 2025 09:30:00",
                    "April 5, 2025 14:00:00"
                ],
                "Rennes": [
                    "October 19, 2024 09:30:00",
                    "October 19, 2024 14:00:00",
                    "November 23, 2024 09:30:00",
                    "November 23, 2024 14:00:00",
                    "December 14, 2024 09:30:00",
                    "December 14, 2024 14:00:00",
                    "January 18, 2025 09:30:00",
                    "January 18, 2025 14:00:00",
                    "February 8, 2025 09:30:00",
                    "February 8, 2025 14:00:00",
                    "March 8, 2025 09:30:00",
                    "March 8, 2025 14:00:00",
                    "April 5, 2025 09:30:00",
                    "April 5, 2025 14:00:00"
                ]
            }
        }
    }
}
```

### Script, gestion des agendas ***reseau-opencampus.com***

```jsx
// Définir le site actuel (par exemple, "OC", "PBA", ou "LDS")
    const currentSite = "OC";  // Modifiez cette valeur selon le site

    // Créer un cache-buster avec un timestamp pour forcer la récupération de la dernière version du fichier JSON
    let cacheBuster = new Date().getTime();
    let jsonUrl = `https://website-agendas.pages.dev/content.json?v=${cacheBuster}`;

    // Fonction pour créer un bloc HTML dynamiquement
    function createBlock(blockData) {
        const section = document.createElement('div');
        section.classList.add('section');
        
        // Récupérer le lien du bouton pour le site actuel ou '#' par défaut
        let buttonLink;
        if (typeof blockData.buttonLinks === 'object') {
            buttonLink = blockData.buttonLinks[currentSite] || '#';
        } else {
            buttonLink = blockData.buttonLinks || '#';
        }
        
        // Récupérer la location pour le site actuel ou 'Lieu non disponible'
        let location;
        if (typeof blockData.location === 'object') {
            location = blockData.location[currentSite] || 'Lieu non disponible';
        } else {
            location = blockData.location || 'Lieu non disponible';
        }
        
        section.innerHTML = `
            <div class="left-block">
                <p class="block-text">${blockData.leftBlockText}</p>
            </div>
            <div class="block">
                <h2 class="admission-title">${blockData.admissionTitle}</h2>
                <p class="subtext">${blockData.subtext}</p>
                <p class="location"><span class="emoji">📍</span> ${location}</p>
                <div class="button-container">
                    <a href="${buttonLink}" class="button-link">
                        <button class="btn">${blockData.buttonText}</button>
                    </a>
                </div>
            </div>
        `;
        
        return section;
    }

    // Récupérer le fichier JSON avec le cache-buster
    fetch(jsonUrl)
        .then(response => response.json())
        .then(data => {
            const container = document.getElementById('admission-sections');
            
            // Filtrer les blocs par site
            const filteredBlocks = data.blocks.filter(block => block.site.includes(currentSite));
            
            // Créer et ajouter chaque bloc filtré au container
            filteredBlocks.forEach(block => {
                const newBlock = createBlock(block);
                container.appendChild(newBlock);
            });

            // Si aucun bloc n'est trouvé pour le site actuel, afficher un message
            if (filteredBlocks.length === 0) {
                container.innerHTML = `<p>Aucun bloc disponible pour le site ${currentSite}</p>`;
            }
        })
        .catch(error => console.error('Erreur lors de la récupération du fichier JSON:', error));
```
